=====================================
 かけるさん 〜負荷を掛けるよ2019秋〜
=====================================

.. contents::

アプリケーションのパッケージング
--------------------------------

以下で artifacts バケットにアプリケーションが追加される。

::

   $ make upload


AMI焼き
-------

provisioning/ami/packer.json の部分を環境に合わせて変更する。
その後以下のコマンドを実行するとAMIが焼かれる。

::

   $ cd provisioning/ami
   $ make


シングルノードでの実行
======================

インスタンス立てる
------------------

すでにあるはずの kakeru vpc やセキュリティグループを指定してインスタンスを立てる。

負荷を掛ける
------------

立てたインスタンスに ssh して、以下のようにコマンドを実行すると hakaru に負荷がかかる。
シナリオは 1, 2, 3 のうちのどれかが使える。

(社内からじゃないと繋がらないはず

::

   $ ssh -i ${インスタンスを立てたときの.pem} ubuntu@${対象インスタンスのipv4 dns名}
   # cd /opt/kakeru
   # make deploy # artifactsからアプリケーション配備するやつ
   # make -C app kakeru upload HOST=${ELBエンドポイントのドメイン} SCENARIO=${1,2,3}

実行結果は {チーム名}-kakeru-report のバケットにアップロードされるので、見てみましょう。


マルチノードでの実行
====================

ここから先はオプションなので気になったらサポーターに聞いてみてね。

起動テンプレートを作る
~~~~~~~~~~~~~~~~~~~~~~

上記AMIを指定して起動テンプレートを作る。
起動設定でも可

ASGを作る
~~~~~~~~~

上記で作った起動テンプレートを指定してAuto Scaling Groupを作る。
作る際の名前は ``kakeru`` としておくこと。

ASGからインスタンスを起動
~~~~~~~~~~~~~~~~~~~~~~~~~

ASGのdesiredを ``希望する台数+1`` にしてインスタンスを立てる。
複数立てたうちの一台はコントローラーノードなので、ワーカーも含めてマルチノード処理したい場合は最低3台必要。

負荷を掛ける
~~~~~~~~~~~~

::

   $ ssh -i ${インスタンスを立てたときの.pem} ubuntu@${ASGのいずれかのインスタンス}
   # cd /opt/kakeru
   # make
   # make -C app kakeru/multinode HOST=${ELBエンドポイント} SCENARIO=${1,2,3}


アプリケーションの再デプロイ
============================

再パッケ

::

   $ make clean upload

インスタンスへのデプロイ

::

   $ ssh -i ${インスタンスを立てたときの.pem} ubuntu@${いずれかのインスタンス}
   # cd /opt/kakeru
   # make deploy

マルチノードの場合は kakeru を実行するインスタンスだけでデプロイすればOK