# Section 2 その他のWebサーバー環境

## 2-1 Vagrantを使用したCentOS 6.5環境の起動

サーバーを構築するたびにOSのインストールからやってられないので事前に用意したCentOS 6.5の環境をVagrantで起動します。

### Vagrantで起動できるCentOS 6.5のイメージを登録

USBストレージにVagrant用CentOS boxを用意してありますので登録して使用します。

    vagrant box add CentOS65 コピーしたboxファイル --force

### Vagrantの初期設定

作業用ディレクトリを作成し、その中で初期設定を行ないます。

   vagrant init

上記コマンドを実行するとVagrantfileというファイルが作成されます。このファイルにVagrantの設定が書かれています。
そのままではデフォルトのOS(存在しない)を起動してしまうので、CentOS6.5を起動するようにします。

viでVagrantfileを開き、

    config.vm.box = "base"
d
と書かれているのを

    config.vm.box = "CentOS65"

とします。ここで指定するものはvagrant box addで指定したものの名前(上記の例だとCentOS7)を指定します。

### 仮想マシンの起動

    vagrant up

### 仮想マシンの停止

    vagrant halt

### 仮想マシンの一時停止

    vagrant suspend

### 仮想マシンの破棄

最初からやり直したい…そんな時に破棄するとCentOSが初期化されます。
また`vagrant up`をすると立ち上がります…

    vagrant destroy

### 仮想マシンへ接続

実際の仮想マシンへはsshで接続します。

    vagrant ssh

### ホストオンリーアダプターの設定

サーバーを設定したあと、動作確認するために接続するためのIPアドレスを設定します。
また、そのためのNICを追加します。

Vagrantfileの

    Vagrant.configure(2) do |config|

から一番最後の

    end

の間に

    config.vm.network "private_network", ip:"192.168.56.129"

と書くと仮想マシンのNIC2に192.168.56.129のIPアドレスが振られます。
`config.vm.box = "CentOS7"` の下にでも書くといいと思います。

※ 当然のことながら、複数台の仮想マシンを立ち上げる時には異なるIPアドレスを割り当てる必要があります。

### Vagrantfileの反映

Vagrantfileで変更した設定を反映させるには

    vagrant reload

すると反映されます。ただし、再起動されますので注意してね。

## 2-2 Wordpressを動かす(2)

1-2ではWordpressをApache + PHP + MySQLで動作させたが、今度はNginx + PHP + MariaDBで動作させます。

Nginxはディストリビューターからrpmが提供されていないため、リポジトリを追加する必要があります。
[公式サイト](http://nginx.org/en/linux_packages.html#stable)からリポジトリ追加用のrpmをダウンロードしてインストールしてください。

その後yumでインストールできるようになります。

NginxでPHPを動かすにはコツが必要ですのでがんばって検索して動かしてください。
ヒントは **Nginx php-fpm** です。

###scpと仮想マシンでファイルのやりとりをする
公式サイトからCentOS7用のrpmファイルをダウンロードする。

		vagrant ssh-config

コマンドを実行すると仮想マシンにSSH接続をするためのOpenSSHの設定が得られる。
上記の内容をファイルに書き出しておく。

		vagrant ssh-config > ssh.config

あとはscpコマンドの-Fオプションに先ほど書き出しておいたファイルを指定することでSSH接続用の設定として利用できる。
		scp -F ssh.config [ダウンロードしたファイル名] vagrant@default:~/

実行したら仮想マシンに接続してファイルの転送ができているか確認する。
		vagrant ssh

確認できたらrpmコマンドを使ってインストール
		rpm -ivh [転送したrpmファイル名]

インストールできたらyumコマンドでNginxをインストールする。
まずはyumのproxyの設定。

		sudo vi /etc/yum.conf

で、
		#Proxy Setting
		proxy=http://172.16.40.1:8888

を追加するだけ(てかSection1.mdに書いてある)
設定をしたら

		sudo yum install nginx

インストールしたらバージョンの確認

		nginx -v

####Nginxのデーモンを起動する

ログ出力先ディレクトリを作成
		mkdir -p /var/log/nginx/www.example.com

ログ出食先ディレクトリの所有者をnginxとする。
		chown nginx: /var/log/nginx/www.example.com
		chmod +r+w /var/log/nginx/www.example.com


Nginxデーモンが起動しているか確認する。
		systemctl list-units | grep nginx
何も出力されなければ起動していない状態だとわかる。

次に、php-fpmデーモンの登録状態を確認。
		systemctl list-unit-files | grep nginx
		nginx.service										disabled
と表示されればオッケー

次に、nginxデーモンがブート時に自動起動するように設定しておく。
		sudo systemctl enable nginx.service

再度、nginxデーモンの登録状態を確認。
		systemctl list-unit-files | grep nginx
		nginx.service										enable
(enableならおっけー)

####Nginxの設定

		sudo vi /etc/nginx/conf.d/wordpress
と言うファイルを作成し以下の文を記入。

server {
  listen 80;
	server_name example.com; # 取得したドメインを指定してください
	root /var/www/wordpress;
	index index.php;

 #wp-config.phpへのアクセスをすべて拒否します。
	location ~* /wp-config.php {
		deny all;
	}

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME /var/www/wordpress$fastcgi_script_name;
		fastcgi_param PATH_INFO $fastcgi_script_name;
		include fastcgi_params;
	}
}

以上
設定のチェック
		sudo nginx -t
を実行し、エラーっぽいのがなければおｋ

/etc/nginx/conf.d/default.conf
/etc/nginx/conf.d/example.conf
/etc/nginx/conf.d/example_ssl.conf
の末尾に.noをつけて読み込まれないようにする。


###phpのインストール
		yum -y install php php-mysql php-gd php-mbstring php-fpm

完了したらバージョンの確認
		php --version

####php-fpmの設定

		sudo vi /etc/php-fpm.d/www.conf

に書いてあることを下記のものに置き換える

使用するポートを設定。
listen = 127.0.0.1:9000

許可するクライアントのIPアドレスを指定
listen.allowed_clients = 127.0.0.1

php-fpmのサービス実行ユーザ、グループを指定します。
user = nginx
group = nginx

php-fpmのサービスのプロセス数を定量とするように設定しておきます。
;pm = dynamic
pm = static

php-fpmの最大子プロセス数を設定します。
;pm.max_children = 50
pm.max_children = 3

php-fpmが受け付ける最大要求数を設定します。
ここで設定した要求数を処理したら子プロセスを再起動します。
pm.max_requests = 500

置き換えが完了したら
		sudo vi /etc/nginx/nginx.conf
の中の
		access_log ~~~~~~~
と書いてある行の下に
		error_log /var/log/nginx/error.log;
と追加する。

####php-fpmデーモンを起動

先に追加したグループ、ユーザが存在しない場合は追加する。
		cat /etc/passwd | grep nginx

を実行して何も出力されない場合はユーザやグループが存在しない。

ユーザ、グループが存在していない場合は# グループを追加します。
ユーザ、グループが存在する場合はやらなくてよし。

		groupadd nginx
		getent group nginx
nginx:x:1001
 
ユーザを追加します。
		useradd -s /usr/sbin/nologin -g nginx -d /var/www -c nginx nginx

####php-fpmデーモンの起動

nginxの時とほぼ一緒なので説明は省略。

		systemctl list-units | grep php-fpm

		systemctl list-unit-files | grep php-fpm
		php-fpm.service										disabled

		sudo systemctl start php-fpm.service

		systemctl list-units | grep php-fpm

		systemctl list-unit-files | grep php-fpm
		php-fpm.service										disabled

		sudo systemctl enable php-fpm.service

		systemctl list-unit-files | grep php-fpm
		php-fpm.service										enable


###mariadb(mysql)をインストールする。
		sudo yum -y install mariadb mariadb-server

PHPの時とほぼ一緒なので説明は省略。

		systemctl list-units |grep mariadb
		systemctl list-unit-files |grep mariadb

		systemctl start mariadb.service

		systemctl list-units |grep mariadb
		systemctl list-unit-files |grep mariadb

		systemctl enable mariadb.service

		systemctl list-unit-files |grep mariadb

####Wordpressで使用するデータベースを作成
SQLファイルを作成してデータベース作成とユーザの登録を行う(mysqlコマンドを使って一気に流すパターン)

		vi wordpress.sql

下記の文を書き写す。

		set password for root@localhost=password('[password]');
		insert into user set user="[ユーザ名]", password=password('[password]'),host="localhost";
		create database [データベース名];
		grant all on *.* to [ユーザ名]@localhost;
		FLUSH PRIVIEGES;

書いたら次のコマンドを実行
		mysql -uroot -Dmysql < wordpress.sql

何もエラーが起きなければログインできるか確認。
		mysql -uroot -p

ログインできれば一旦mysql終了
		MariaDB[none]> exit

続けて新規データベースへ新規ユーザでログインできるか確認
		mysql -u[ユーザ名] -p

ログインできれば一旦mysql終了
		MariaDB[none]> exit

###ブラウザで開く

		ip addr

して、IPアドレスを確認したらブラウザで
**http://[IPアドレス]/**
を開く、開けなかったら何かしらのエラーログを確認。
大体Section1.mdかSection2.mdを見返せば解決する。

なんかWordpressのインストールしようとするとバグかなんかでindex.php読もうとするからそれ消したらいいと思うよ。
あとはGUIで適当に。


## 2-3 Wordpressを動かす(3)

1-2や2-2では提供されているrpmファイルを使用してLAMP/Nginx + PHPの環境を構築しましたが、
提供されていないバージョンを使用して環境を構築する必要がある時もあります。

そういう場合はソースコードをダウンロードしてきて自分でコンパイルして動作させます。

Apache HTTP Server 2.2とPHP5.5の環境を構築し、Wordpressを動かしてください。
(MySQL/MariaDBはrpm版を使ってもOKです)

その時は別のVagrantfile(作業ディレクトリ)を作ってやってくださいね。

###別の仮想サーバを立てる方法
簡単。別のディレクトリにVagrantfileをコピーしてvagrantupするだけ

###yumの設定
何回も書いたけど一応。

		sudo vi /etc/yum.conf

の中にプロキシの設定を書く。

#Proxy Setting
proxy=http://172.16.40.1:8888

をどこかに書くだけ

###wgetをget

		sudo yum install wget

wgetのプロキシ設定もする
		vi .wgetrc

#Proxy Setting
http_proxy=172.16.40.1:8888
https_proxy=172.16.40.1:8888

と記入する。


###Apache HTTP Server 2.2のソースからビルドしてインストール


ビルドに必要なパッケージを入れる

		sudo yum install -y gcc make
		sudo yum install prel zlib-devel

次にApacheのインストール

		wget http://ftp.jaist.ac.jp/pub/apache//httpd/httpd-2.2.29.tar.gz
		tar zxvf httpd-2.2.29.tar.gz
		./configure --enable-rewrite=shared --enable-speling=shared
		make
		sudo make install

そのまま起動してもエラーが出る。出ても起動できるけど気持ち悪いから直す。
		sudo vi /etc/hosts
の中を全部コメントアウトして
		127.0.0.1               dacelo localhost.localdomain localhost

と書く。

次にhttpd.confファイルの設定

		sudo vi /usr/local/apache2/conf/httpd.conf
の中を書き換える
		ServerName dacelo:80
		DirectoryIndex index.html index.php

phpを読み込めるように
<FilesMatch \.php$>
	 SetHandler application/x-httpd-php
</FilesMatch>

を最後の行に追加

###MySQLをインストール

MySQLのパッケージをインストール

		sudo yum -y install http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

		sudo yum -y install mysql mysql-devel mysql-server mysql-utilities

インストールできたら設定、以下のものを追記していく
		sudo vi /etc/my.cnfi.d/server.cnf

		[mysqld]
		character-set-server = utf8

		sudo vi /etc/my.cnf.d/mysql-clients.cnf

		[mysql]
		default-character-set = utf8
		show-warning

		sudo systemctl start mysql.service

SQLファイルを作成してデータベース作成とユーザの登録を行う(mysqlコマンドを使って一気に流すパターン)

		vi wordpress.sql

下記の文を書き写す。

		set password for root@localhost=password('[password]');
		insert into user set user="[ユーザ名]", password=password('[password]'),host="localhost";
		create database [データベース名];
		grant all on *.* to [ユーザ名]@localhost;
		FLUSH PRIVIEGES;

書いたら次のコマンドを実行
		mysql -uroot -Dmysql < wordpress.sql

何もエラーが起きなければログインできるか確認。
		mysql -uroot -p

ログインできれば一旦mysql終了
		MariaDB[none]> exit

続けて新規データベースへ新規ユーザでログインできるか確認
		mysql -u[ユーザ名] -p

ログインできれば一旦mysql終了
		MariaDB[none]> exit



###PHP5.5のソースからビルドしてインストール

		wget http://jp1.php.net/distributions/php-5.5.25.tar.gz
		tar xzvf php-5.5.25.tar.gz /usr/local/src/
		cd /usr/local/src/php-5.5.25

必要になるであろうパッケージをインストール
		sudo yum install libxml2 libxml2-devel

ビルドファイル作成
configureコマンドを叩いてエラーが出ないか確認

		./configure \
		--with-apxs2=/usr/local/apache2/bin/apxs \
		--with-mysql=mysqlnd \
		--with-mysqli=mysqlnd \
		--with-pdo-mysql=mysqlnd \
		--with-openssl \
		--with-zlib \
		--enable-mbstring

なんもエラーが出なければ
		make && sudo make install


###WordPressをインストール
公式サイトからwget
		wget http://wordpress.org/latest.tar.gz
		tar zxvf latest.tar.gz
		sudo mv wordpress /usr/local/apache2/htdpcs

		sudo cp /home/vagrant/php-5.5.25/php.ini-development /usr/local/lib/php.ini
		sudo vi /usr/local/lib/php.ini

次の場所にPATHを書き足す

		mysql.default_socket=/var/lib/mysql/mysql.sock
		mysqli.default_socket=/var/lib/mysql/mysql.sock

あとはブラウザで開いてポチポチ


## 2-4 ベンチマークを取る

サーバーの性能測定のためにベンチマークを取ることがあります。

[別ページ](misc/Benchmark.md)にまとめてありますのでそちらを参照してください。

とりあえずWordPressを開く
違うipアドレスからなにか取ってこようとしてエラーが出ていたらwp_optionの設定をいじらなければならない。

mysqlに接続して
		select * from wp_options where option_name = 'siteurl';
		select * from wp_options where option_name = 'home';

を実行し違うipアドレスが設定されているか確認

		update wp_options set option_value = 'http://[ipaddress]/wordpress/' where option_name = 'siteurl';
		update wp_options set option_value = 'http://[ipaddress]/wordpress/' where option_name = 'home';

これで設定を上書きできる。


次にApacheBenchをホスト側(Ubuntu)にインストールする。

		exit
		sudo aptitude install apache2-utils

インストールできたらabコマンドを叩いて保存する

		ab http://[ipaddress]/wordpress > wordpress_ab.text

次に Chrome ウェブストアにアクセスしてPageSpeed Insightsを追加する。
追加できたらWordPressの画面でベンチマークを取る

###WordPress高速化
まずはhtdocs以下のファイルの権限をdaemonに変更する

		sudo chown -R daemon /usr/local/apache2/htdocs/

####圧縮を有効にする

mod_filterとmod_deflateを追加する

		sudo /usr/local/apache2/bin/apxs -cia mod_filter.c
		sudo /usr/local/apache2/bin/apxs -cia mod_deflate.c

httpd.confのLoadModuleなんちゃらの行に下記のコードを追加する。

		LoadModule deflate_module          modules/mod_deflate.so

httpd.confに下記のコードを追加する。

		<Location />
		# Insert filter
		SetOutputFilter DEFLATE
		SetEnvIfNoCase Request_URI \
		\.(?:gif|jpe?g|png)$ no-gzip dont-vary
		</Location>


####プラグインの導入
まずプラグインを導入するためにphp.iniのupload_max_filesizeの値を変更

		upload_max_filesize=8M


**入れたプラグインと設定

・WP Super Cache

画像サイズの変更
・EWWW Image Optimizer

画像の読み込みを後回しにする
・Unveil Lazy Load


## 2-5 セキュリティチェック

サーバーを構築したとしても、セキュリティがガバガバではいろんな意味で駄目です。
Webアプリケーションの脆弱性を突かれたり、設定したサーバーに脆弱性があったりした場合、
情報漏洩とか乗っ取りとか踏み台とかされるとアレです。

定期的にセキュリティチェックを行なう必要があります。

(やり方は[別ページ](misc/SecurityScan.md))

セキュリティチェックを行ない、不具合があるようでしたら修正を行なって再度セキュリティチェックを行ないます。
(可能な限り不具合がなくなるまでチェック&fixを行ないます。)
