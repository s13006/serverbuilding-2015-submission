# Section 1 基本のサーバー構築

## 1-1 CentOS 7のインストール

### VirtualBoxへのインストール

####CentOS用の仮想マシン初期設定

環境変数設定→ ネットワーク→ ホストオンリーネットワーク、適当に名前をつけて追加する。
ネットワークアダプター2を設定、割り当てをホストオンリーネットワークに設定。



virtualboxを起動して左上の新規と書かれているところをクリック

パーティションの設定は特に指定しない。

名前、OSのタイプ、バージョンを選択
**ここではCentOS 7、Linux、Red Hat(64bit)と設定する。**

続いてメモリーサイズの選択、今回は1024MBと設定する。

ハードドライブはデフォルトのままで作成ボタンをクリック。


次の画面でファイルサイズの設定を行う今回は8GBを(ry
ハードドライブのファイルタイプはデフォルト。

ネットワークアダプター2を設定します。割り当てを「ホストオンリーアダプター」にします。
ネットワークアダプター1はデフォルト(NAT)で問題ありません)



CentOSの公式サイトよりCentOS 7 Minimal ISO(x86_64)のISOファイルをダウンロードし、 インストールする。

    sudo aptitude install dconf-tools

起動する。

    dconf-editor

起動したらsystem→ proxyを選択、ignore-hostsに['localhost', '127.0.0.0/8', '::1', '172.16.40.0/22', '192.168.0.0/16']と設定。
こうすることでプロキシを通らずにバーチャルボックスと通信することが可能になる。


####ネットワークアダプター1/2へのIPアドレスの設定とssh接続の確認

/etc/sysconfig/network-scriptにifcfg-enp0s?というファイルがあるので、
そのファイルを編集してネットワーク接続ができるように設定します。

DHCPでIPアドレスを取得でるので、[RedHat Enterprise Linux 7のマニュアル(英語)]を読んだらなんとなくいける。
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html-single/Networking_Guide/index.html#sec-Configuring_a_Network_Interface_Using_ifcg_Files



再びvirtualboxを起動し、左上の起動と書かれているボタンをクリック。
[起動ハードディスクを選択]ダイアログが表示されるのでプルダウンメニュー右の画像をクリック、インストールしたいISOイメージを選択して起動。

言語を選択、日本語にするとキーボード配列も日本語になるのでUS配列に変更する必要がある。

[ネットワークとホスト名]を選択し、画面右上にあるスイッチをオンにする。

インストールの開始を選択すると次の画面に移動するので、rootのパスワードとユーザーの作成をする。この時作られたユーザーを管理者に設定。

インストールが完了したらCentOSを再起動。


####SSH接続の確認

Ubuntuからsshで仮想マシンに接続できることを確認してください。
(ついでなので、公開鍵認証でログインできるようにしておくといいと思いますよ。必須ではないけど。)

virtualbox上で

		vi /etc/sysconfig/network-script/ifcfg-enp0s3

		vi /etc/sysconfig/network-script/ifcfg-enp0s8

を編集。

		BOOTPROTO="dhcp"

		ONBOOT="yes"

に変更

		ping [ipaddress]

を叩いてネットワークに接続できるか確認。

		ip addr

を叩けばipaddressがふられているか確認できる。

ホストマシンで

		ssh [ユーザー名]@[ipaddress]

を叩けばssh接続ができる。
 


####インストール後の設定

yumやwgetを使用する時のproxyの設定を行なってください。

アップデート

プロキシの設定後、アップデートができるようになっているのでアップデートを行なってください。

yum,wgetのプロキシ設定

yumの場合

		sudo vi /etc/yum.conf

wgetの場合

		vi ~/.wgetrc

に

		#Proxy Setting
		proxy=http://172.16.40.1:8888

		#Proxy Setting
		http_proxy=http://172.16.40.1:8888
		https_proxy=https://172.16.40.1:8888
を追加。




## 1-2 Wordpressを動かす(1)

Wordpressを動作させるためには下記のソフトウェアが必要になります。 [※1](#LAMP)

* Apache HTTP Server
* MySQL
* PHP

###Apacheをインストール

		yum -y install httpd

####Apache起動確認

		sudo systemctl start httpd

起動できたら一旦終了

		sudo systemctl stop httpd

####firewallを無効化

**Apacheを起動するだけではfirewallで弾かれて接続できないので無効にしなければならない**

		sudo systemctl stop firewalld

**それとSELinuxも無効にする**
(なんかOSを壊したりできないようにする機能らしい)

		sudo vi /etc/selinux/config

に書いてある

		SELINUX=enforcing

を

		SELINUX=disabled

に変更し

		setenforce 0

を実行。


###MySQLインストール

####MySQL公式リポジトリファイルをインストール

		sudo yum -y install http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

####MySQLをインストール
		sudo yum -y install mysql
		sudo yum -y install mysql-devel
		sudo yum -y install mysql-server
		sudo yum -y install mysql-utilities

####MySQLデーモンの起動確認・rootのパスワード設定

		sudo service mysqld start

		mysqladmin -u root@localhost password '[password]'

		sudo service mysqld stop

####ユーザの作成

mysqlに接続

		mysql -u root -p

		CREATE USER [username]@[hostname] IDENTIFIED BY '[password]'

**Query OK, 0 rows affected (0.00 sec)**
と表示されたらユーザ作成が完了。

####ユーザ権限を設定する。

		GRANT ["与える権限"] ON ["データベース名"].["テーブル名"] TO ["ユーザー名"]

ここでは

		GRANT ALL ON *.* TO ['ユーザー名'];

実行して下のコマンドを叩いて権限を更新する。

		FLUSH privileges;


####データベースの追加

MySQLに追加したユーザで接続しデータベースを作成する

		mysql> CREATE DATABASE ['databasename']


###PHPインストール

		sudo yum -y install php php-mbstring php-mysql

####インストールできたか確認

		rpm -qa | grep php

####PHP設定

		sudo vi /etc/php.ini

//692行目
default_charset="UTF-8"

//878行目
date.timezone="Asia/Tokyo"

に変更

###WordPressのインストール

####パッケージをダウンロード・解凍

		wget http://wordpress.org/latest.tar.gz

		tar -xzvf latest.tar.gz


####wp-config.phpの編集

//** MySQL設定 -この情報はホスティング先から入手してください。**//
以下に書いてあるdefine(~~~~)を書き換える。

例 define('DB_NAME','db_wordpress');


ここまで出来たらApacheを起動してwordpressディレクトリをサーバにアップ(?)する。

		sudo mv wordpress /var/www/html

コマンドを実行したらブラウザを開いて

		http://['ipaddress']/wordpress

にアクセス

あとはブラウザに表示されてるやつを適当に記入していく。



Wordpressのインストールは[公式サイトに手順が掲載されています](http://wpdocs.sourceforge.jp/WordPress_%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)のでそちらを参考にすると確実かと思います。

<a name="LAMP">※1</a>: Linux・Apache・MySQL・PHPの頭文字を取ってLAMPといいます。
