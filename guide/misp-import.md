---
title: MISP投入ガイド
---

# 概要

本資料は、MISPへのデータ投入を行うためのプログラムを開発するためのガイドになります。
本資料では、python用のMISP操作ライブラリである
[pymisp](https://github.com/MISP/PyMISP){:target="_blank"}
を使って、MISPにイベントを登録する方法について説明します。

ご不明点につきましては、
[こちらのお問い合わせフォーム(URLは決まり次第記載)](http://hogehoge)
よりSecureGRIDアライアンス事務局にお気軽にご連絡ください。


# コンテンツ

以下内容について記載します。
* MISP投入プログラムの開発方法
* 公開済みMISP投入プログラム


# MISP投入プログラムの開発方法

pythonを使ったMISPへのイベント登録には、
[pymisp](https://github.com/MISP/PyMISP){:target="_blank"}
というモジュールが利用可能です。このpymispを活用することで簡単なコードでMISP登録が実現できます。
以下では、最も一般的なパターンであると思われる、入力データが記載されたファイルを読み込み、それをMISPに登録するというユースケースについて、具体例を含めご紹介します。


## 今回のサンプルプログラム

今回は、当社が先日公開した
[ransomwatch](https://www.lac.co.jp/lacwatch/people/20211117_002792.html){:target="_blank"}
で収集した情報をMISPに格納するという例を元に進めたいと思います。

まず、早速ですが、サンプルデータとプログラムを
[ダウンロード](../sample/misp-import-sample.zip)
してください。
こちらのzipファイルを解凍いただくと、以下二つのファイルが入っています。
* sample.tsv: 今回インポート対象とするTSVファイルです。構成についてはヘッダをご参照ください。尚、社名等一部の項目については、今回はあくまでサンプルのためマスクしています。
* import-sample.py: 今回のデータをインポートするためのpythonプログラム。こちらの内容を元に以下で説明を行っていきます


## 事前準備

まず、pymispをインストールします。
pipでインストール可能です。

`pip install pymisp`


## プログラムの基本構成

では早速、import-sample.pyをみて行きたいと思います。
大枠として、以下流れで処理が行われています。

* 1 ファイルから情報を読み込む
* 2 MISPイベントの単位に合わせて上記１の情報を集約する
* 3 MISPにイベント登録する

コードでいうと以下部分をご覧いただくと、全体的な流れが上記のようになっていることが確認できるかと思います。

    if __name__ == '__main__':
    
        # ファイルを読み込み、MISP登録に必要な情報をリストで取得する
        file_values = parse_file(IMPORT_TARGET_FILE)
    
        # MISP接続
        misp = ExpandedPyMISP(MISP_URL, MISP_AUTHKEY, ssl=False, debug=False)
    
        # ファイルから読み込んだデータを元に１イベント分ずつ処理する
        for data in file_values:
    
            # MISPイベント作成
            event_data = create_misp_event(data)
            # イベントをMISPに登録
            misp_event_url = register_misp(misp, event_data)
            print(f"Event URL: {misp_event_url}")

では、個別の処理の中身について順番に説明していきます。


## ファイルから情報を読み込む

今回のメソッドでは「parse_file」部分で処理対象のファイルを読み込み、MISPイベント作成に必要な情報を取得しています。
コードをみて行きます。

    def parse_file(target_file: pathlib.Path)->list:
        """ ファイルを読み込み、イベント登録に必要な情報を抜き出して辞書のリストで返す """
    
        result = []
    
        # ファイルを読み込んで１行１要素でリストに格納
        file_data = target_file.read_text().split('\n')
    
        # ヘッダ行判定用
        header = True
    
        # １行１イベント相当の情報なので、１行分ずつループ処理して必要な項目を抽出
        for line in file_data:
            # ヘッダはスキップする
            if header:
                header = False
                continue
    
            # 空行はスキップ
            if line.strip() == '':
                continue
    
            # 入力ファイルがTSVなのでタブで分割する
            columns = line.split("\t")
    
            # 今回は全カラムをMISP登録に利用するため、抽出して返す
            result.append({
                'actor': columns[0],
                'victim': columns[1],
                'first_seen': columns[2],
                'published': columns[3],
                'url': columns[4]
            })
    
        return result

基本的にコメントの通りの処理となっています。
この部分についてはMISP特有の処理は特に存在せず、pythonによるファイル読み込みとTSV（タブ区切りファイル）のパースを行っています。


## MISPイベントの単位に合わせて上記１の情報を集約する

ここからがMISPインポートのための実装となります。
ファイルから読み取ったデータを元にMISPイベント登録データを作成していきます。
今回の場合は、ファイルの１行が１イベントに対応するため、１行分ずつデータを渡して、MISPイベントデータを作って行きます。
この部分は、インポート対象ファイルの内容に応じて複数行分のデータを集約して一つのMISPイベントにする等、対象とするデータやMISPへどのように登録したいかで実装を決定します。

コードを見てみます。

    def create_misp_event(target_values: dict)->MISPEvent:
        """MISP登録用のイベントを作成する"""
    
        # データを格納するイベント作成
        misp_event = MISPEvent()
    
        # イベントのタイトル
        misp_event.info = f"[ransomwatch] {target_values['actor']} - victim:{target_values['victim']}({target_values['first_seen']})"
    
        # イベント日付としてfirst seenの日付をセット
        misp_event.date = target_values['first_seen'][:10]
    
        # イベントタグ追加
        misp_event.add_tag('ransomwatch')
        misp_event.add_tag(f"actor:{target_values['actor']}")
    
        # アトリビュート追加
        # 被害組織情報を追加
        misp_event.add_attribute(
            value = target_values['victim'],
            category = "External analysis",
            type = "comment",
            comment = "victim"
        )
    
        # first seen
        misp_event.add_attribute(
            value = target_values['first_seen'],
            category = "Other",
            type = "datetime",
            comment = "First Seen"
        )
    
        # 公開日(published)は存在すれば追加
        if target_values['published']  != "N/A":
            misp_event.add_attribute(
                value = target_values['published'],
                category = "Other",
                type = "datetime",
                comment = "Published Date"
            )
    
        # 情報ページ
        misp_event.add_attribute(
            value = target_values['url'],
            category = "Network activity",
            type = "url",
            comment = "Leak site"
        )
    
        return misp_event

pymispで用意されているMISPEventというクラスをインスタンス化し、ここに必要な情報尾設定していきます。
今回行っている内容としてはコード上のコメントをご確認いただければと思いますが、基本的なMISPイベントを作成するためには最低限以下情報を設定します。

* イベントのタイトル：　どういうイベントなのかわかるような内容をセット
* イベント日付：　このイベントがいつのデータなのかを表す日付。前日分を処理する場合などは前日日付をセットするなども考慮
* イベントのタグ：　イベントがどういったものなのかを完結に表すタグ。同一のタグがついているものを検索するなどの使い方ができるので、データ検索なども考慮して設定する
* アトリビュート：　このイベントに含める個別の登録値。複数件登録可能

今回create_misp_eventメソッドでは、上記情報をMISPEventクラスのインスタンスに設定し、それをreturnしています。


## MISPにイベント登録する

ここまででMISPに登録するイベントデータは作成できています。
後はこれをMISPに登録するのみです。
この部分はpymispを使うと非常に簡単に実現できます。
コードをみてみます。

    def register_misp(misp:ExpandedPyMISP, misp_event:MISPEvent) -> str:
        """MISPイベントデータを受け取り、イベントURLを返す"""
    
        # MISPにイベント登録
        event_data = misp.add_event(misp_event)
    
        # イベントURLを返す
        return f"{MISP_URL}/events/view/{event_data['Event']['id']}"

上記の通り、ExpandedPyMISPクラスインスタンスのadd_eventメソッドに、前述のcreate_misp_eventで作成したデータを渡すのみで処理は完了です。
実運用では、この部分はエラーが出やすい（MISPへ実際にネットワーク接続するため、ネットワーク関連のエラーやMISP側の不調に起因するエラーが出ることがある）ため、try～exceptで例外処理したり、ループ処理でリトライ処理を実装することが多いです。今回は、このあたりは一般的なpythonプログラミングのテクニックの話になるので、サンプルには入れていません。


## まとめ

今回は、入力となるファイルを元に、そのファイルを読み込みMISPにイベントとして登録するプログラムの実現方法についてご説明しました。
この入力ファイルに随時最新の情報を格納するようにしておき、これを１日１回読み込む形でプログラムを実行すれば、毎日自動的にMISPへ最新情報を投入するということが実現可能です。
またこの入力ファイルの中身を変更し、読み込み部分と集約する部分（今回の例でいうとparse_fileとcreate_misp_eventメソッド）を適に変更すれば、任意のデータのMISP登録を実現可能です。今回はCSVファイルを例にしましたが、ここはJSONファイルやアクセスログのフォーマットなどなんでも問題ありません。入力ファイルのフォーマットに合わせて一般的なpythonプログラムで行うようにファイルパースを行うことで、様々な入力データに対応が可能です。

ぜひ今回の例を参考に、ご自身の手元にあるデータをMISPに投入してみてください。不明点や思った通りに動かないなど何かございましたら、お気軽にSecureGRIDアライアンス事務局までご連絡ください。

# (参考)公開済みのMISP投入プログラム
[手動作成CSVのMISP投入](https://github.com/LAC-Japan/MISP-CSVImport){:target="_blank"}
[AnyrunデータのMISP投入](https://github.com/LAC-Japan/anyrun_to_misp){:target="_blank"}
