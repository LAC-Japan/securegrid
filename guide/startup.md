---
title: SecureGRIDスタートアップガイド
---

[TOP](/securegrid/)

# 概要

本資料は、SecureGRIDアライアンスへの加盟に際して、必要となる初期作業をまとめたものです。

ご不明点につきましては、
[こちらのお問い合わせフォーム](https://krs.bz/lac/m/securegrid?_ga=2.137599600.1173151998.1640651755-16565361.1638256099&_fsi=8H1ssyw9){:target="_blank"}
よりSecureGRIDアライアンス事務局にお気軽にご連絡ください。


# コンテンツ

以下内容について記載します。
* [MISPの構築](#mispの構築)
* [MISPへのデータの登録](#mispへのデータの登録)
* [SecureGRIDへのMISP追加](#securegridへのmisp追加)


# MISPの構築

ここでは、基本的なMISPの構築方法についてご説明します。
尚本資料では、実績のある推奨環境を前提として記載しています。事情によりその他環境での構築を行う場合、MISP本家のreadmeを参考に実施をおねがいします。
参考：　[MISP本家のインストール情報](https://misp.github.io/MISP/){:target="_blank"}


## 必要環境

* OS: Linux(本資料ではUbuntu20.04 LTSを前提とします)
* メモリ：　8GB以上
* プロセッサ：　２コア以上
* ディスクサイズ：　３０GB以上（投入予定データに依存）
* アクセス可能なグローバルIP（DNS登録は任意）

### 参考：　データ件数と使用領域について

MISPのデータベースはMariaDBです。そのため、データ容量の検討は一般的なMariaDBと変わりません。
MISPの場合、データ構造は決まっていますが、ここにどのようなデータを入れるかによって使用領域は大きく変動してきます（IPアドレスばかりなのかテキストによるコメントなどを入れるのかなど）。
そのためあくまで一例ですが、我々の構築してきたMISPでの実測値をご参考まで記載します。以下をご覧いただくと分かるように、件数とデータサイズは必ずしも比例していません。
尚、以下で記載している件数は、MISPの一番細かいデータ単位であるアトリビュートの件数です。
MISP構築環境の検討にお役立てください。

* 200,000件：　７００MB
* 5,500,000件：　７８GB
* 230,000,000件：　４００GB
* 600,000,000件：　３７７GB


## インストール手順

* インストール対象の環境にログインします
* 以下コマンドを実行し、インストールシェルを開始します
	* `wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh`
	* `bash /tmp/INSTALL.sh`
	* `bash /tmp/INSTALL.sh -c -M`
* 画面の指示に従って入力を進めます
* 問題なく完了した場合、以下でアクセス可能になっていますので確認してください(curlやwget・ブラウザ等）
https://localhost
* 何等かエラーが発生した場合、内容を確認し可能であれば該当のエラーを解消した上で、インストールシェルを再実行してください
* エラー解消が困難な場合、SecureGRIDアライアンス事務局までお問い合わせください

参考：　完全な情報については [MISP本家](https://misp.github.io/MISP/INSTALL.ubuntu2004/){:target="_balnk"} の情報を参照してください


## ユーザ作成

MISPを利用するにあたってはユーザごとにアカウントを登録する必要があります。
インストール完了時に表示された初期ユーザにてMISPにログインし、以下流れで必要なユーザを登録してください。
* １　`Administration`をクリック
* ２　`Add User`をクリック
* ３　画面の入力フォームに情報をセットし登録を実行

ユーザ情報の入力画面の各項目について、よく利用する項目について説明します。
詳細については
[こちらのマニュアル](https://www.circl.lu/doc/misp/administration)
等をご参照ください。

* Email: ユーザのメールアドレスでログインに利用。MISPの機能でメール通知を使う場合このアドレスに通知される。通知を利用しない場合は実在しない値でも問題なし
* Organisation: ユーザの所属グループ。MISP上のデータの公開範囲をこのOrganisationとイベント登録時に設定するdistributionの設定により制限することが可能。Organisationの作成については後述
* Role: ユーザの権限設定。各権限の詳細ついては
[マニュアルを参照](https://www.circl.lu/doc/misp/administration)
* Authkey(編集不可): APIで利用するキー文字列

上述の通り、MISPにはユーザをグループ分けするOrganisationという概念が存在します。
このOrganisationを追加する場合は、以下流れで行います。
* １　`Administration`をクリック
* ２　`Add Organisations`をクリック
* ３　画面の入力フォームに情報をセットし登録を実行


# MISPへのデータの登録

[MISPデータ投入ガイド](misp-import.md) を参照してください


# SecureGRIDへのMISP追加

以下作業を実施いただく必要があります。
* SecureGRIDでMISPから情報取得するための認証キー（authkey)の提供
* SecureGRIDからの通信許可


## SecureGRIDでMISPから情報取得するための認証キー（authkey)の提供

以下条件で、SecureGRIDポータルからデータを検索するためのユーザをMISPに作成してください。
ユーザの作成方法については、
[前述の手順](#ユーザ作成)
をご参照ください。

* Email: securegrid@任意のドメイン。このメールアドレスにSecureGRIDアライアンスからメールをお送りすることはないため、実在しないアドレスで問題ありません
* Organisation: securegrid。MISPのイベントは原則securegridではないデータの所有者を表すorganisationに設定してください。その上で、SecureGRIDアライアンスにデータ公開するイベント・アトリビュートについてはdistributionを「this community」に設定してください。逆にsecuregridへは公開したくないデータについてはdistributionを[organisation only」としていただくことで今回作成いただくユーザからの検索ではヒットしなくなります
* Role: readonly。SecureGRIDポータルではデータの検索のみ利用するため、読み取りのみ許可いただければ問題ありません

上記条件で作成したユーザのAuthkeyをSecureGRIDアライアンス事務局までお知らせください。


## SecureGRIDからの通信許可

構築いただいたMISPへのインターネットからのアクセスについて制限されている場合、SecureGRIDポータル動作サーバからのアクセスを許可いただく必要があります。
アクセス許可いただく必要のあるIPアドレス・ポート番号については事務局より別途お知らせいたします。
