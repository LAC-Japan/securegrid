# 概要

本資料は、SecureGRIDアライアンスへの加盟に際して、必要となる初期作業をまとめたものです。

ご不明点につきましては、
[こちらのお問い合わせフォーム(URLは決まり次第記載)](http://hogehoge)
よりSecureGRIDアライアンス事務局にお気軽にご連絡ください。


# コンテンツ

以下内容について記載します。
* MISPの構築
* MISPへのデータの登録
* SecureGRIDへのMISP追加


# MISP構築

ここでは、基本的なMISPの構築方法についてご説明します。
尚本資料では、実績のある推奨環境を前提として記載しています。事情によりその他環境での構築を行う場合、MISP本家のreadmeを参考に実施をおねがいします。
参考：　[MISP本家のインストール情報](https://misp.github.io/MISP/){:target="_blank"}


## 必要環境

* OS: Linux(本資料ではUbuntu20.04 LTSを前提とします)
* メモリ：　8GB以上
* アクセス可能なグローバルIP（DNS登録は任意）


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

尚、MISPにはユーザをグループ分けするOrgという概念が存在します。
このOrgを追加する場合は、以下流れで行います。
* １　`Administration`をクリック
* ２　`Add Organisations`をクリック
* ３　画面の入力フォームに情報をセットし登録を実行


# MISPへのデータの登録

[MISPデータ投入ガイド](misp-import.md) を参照してください


# SecureGRIDへのMISP追加

以下作業を実施いただく必要があります。
詳細については事務局よりご案内いたしますので、MISPのご用意ができましたら事務局までご連絡ください。
* SecureGRIDからの通信許可
* SecureGRIDでMISPから情報取得するための認証キー（authkey)の提供
