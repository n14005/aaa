## Vagrantを使用したCentOS 7環境の起動

# Vagrantで起動できるCentOS 7のイメージを登録

作業用ディレクトリをつくって、その中にboxファイルを持ってきて

    vagrant box add CentOS7 boxファイル --force

をする。

# Vagrantの初期設定

作業用ディレクトリの中で

    vagrant init

をすると、Vagrantfileができる。

　　　　config.vm.box = "base"

と書かれているのを

　　　　config.vm.box = "CentOS7"

とします。ここで指定するものはvagrant box addで指定したものの名前を指定します。

# 仮想マシンの起動

    vagrant up

で仮想マシンが起動します。

# 仮想マシンへ接続

    vagrant ssh

で仮想マシンへsshで接続します。

# ホストオンリーアダプターの設定

Vagrantfileの

    Vagrant.configure(2) do |config|

から一番最後の

    end

の間に

    config.vm.network "private_network", ip:"192.168.56.129"

と書くと仮想マシンのNIC2に192.168.56.129のIPアドレスが振られます。

　　　　config.vm.box = "CentOS7"

の下にでも書くといいと思います。

# Vagrantfileの反映

Vagrantfileで変更した設定を反映させるには

    vagrant reload

すると反映されます。ただし、再起動されますので注意してね。

## 2-2 Wordpressを動かす(2)

nginx+PHP+MarinaDB

## phpをインストールする

    yum -y install php-mysql php php-gd php-mbstring php-fpm

で、phpをインストール。

　　　　php --version

で、phpがインストールされたか、バージョンをチェックする。

# php-fpmを設定する

    sudo vi /etc/php-fpm.d/www.conf

で、
　　　　user = apache
    group = apache
を
    user = nginx
    group = nginx

に直す。

さらに、

　　　　pm = dynamic
を
    ;pm = dynamic
    pm = static
に直す。

# php-fpmデーモン起動
  
    systemctl start php-fpm.service

起動を確認
 
    systemctl list-units |grep php-fpm

次に、php-fpmデーモンがブート時に自動起動するように設定しておきます。

    systemctl enable php-fpm.service


## nginxインストール

　　　　wget http://nginx.org/packages//7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

でリポジトリの追加。

    yum -y install nginx

でインストール。

# nginxの設定

    vi /etc/nginx/nginx.conf

で

    user nginx;
を
    user nginx nginx;

に書き換える。

# ウェブサーバー(nginx) を起動します。

    systemctl start nginx.service

# ウェブサーバー(nginx) のデーモン（サービス）が起動しているか確認します。

    systemctl list-units |grep nginx

# 次に、ウェブサーバー(nginx)デーモンがブート時に自動起動するように設定しておきます。

    systemctl enable nginx.service

# ウェブサーバー(nginx)デーモンの登録状態を確認します。

    systemctl list-unit-files |grep ngin


# NginxでPHPを動かすための設定

    vim /etc/nginx/conf.d/default.conf

で、

    server {
listen 80;
server_name localhost;
charset koi8-r;
access_log  /var/log/nginx/log/host.access.log  main;

location / {
    root   /usr/share/nginx/html;
    index  index.php index.html index.htm;
}

error_page  404              /404.html;

 redirect server error pages to the static page /50x.html

error_page   500 502 503 504  /50x.html;
location = /50x.html {
    root   /usr/share/nginx/html;
}

 proxy the PHP scripts to Apache listening on 127.0.0.1:80

location ~ \.php$ {
    proxy_pass   http://127.0.0.1;
}

 pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

location ~ \.php$ {
    root           /usr/share/nginx/html;
    fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}

 deny access to .htaccess files, if Apache's document root
 concurs with nginx's one

location ~ /\.ht {
    deny  all;
}

に書き換える。



# MariaDB(MySQL)をインストール

# MariaDBインストール

	yum -y install mariadb mariadb-server

バージョン確認

	mysql --version

mariadb のデーモン（サービス）を起動します。

    systemctl start mariadb.service

mariadb のデーモン（サービス）が起動しているか確認します。

	systemctl list-units |grep mariadb

次に、mariadbデーモンがブート時に自動起動するように設定しておきます。

　　　　systemctl enable mariadb.service

##rootのパスワード設定

     mysql_secure_installation

で、rootのパスワード設定。

    mysql -u root -p

で、データベースにログイン

    create database 任意のデータベース名;

で、データベースを作ります

次に、データベースに対して全ての権限を持ったユーザを登録する例です

    grant all on データベース名.* to 任意のユーザー名@localhost identified by '任意のパスワード';

で終わり

# Wodpressを動かす

    /usr/share/nginx/html/

の配下にワードプレスをSection1と同じやり方でダウンロード








## 2-3 Wordpressを動かす(3)

[ここ見て](http://d.hatena.ne.jp/yk5656/20140728/1406987849)

#前準備

 yum -y install wget gcc openssl openssl-devel

# Apache HTTP Server 2.2をダウンロード

ソースをダウンロードして、インストールする。

　　　　cd /usr/local/src/
    
    wget http://ftp.kddilabs.jp/infosystems/apache/httpdhttpd-2.2.27.tar.gz

    tar zxvf ./httpd-2.2.27.tar.gz

    cd ./httpd-2.2.27

    ./configure --prefix=/usr/local/apache --enable-so --enable-ssl --enable-rewrite

    make

    make install

　　　　/usr/local/apache/bin/apachectl start

順番にやる。

Apacheが起動したか確認

　　　　ps aux | grep httpd


## php5.5をダウンロードs

# 前準備
    yum -y install libxml2-devel

# ソースをとってくる

# 展開　インストール
    sudo tar zxvf php-5.5.25.tar.gz

    cd php-5.5.25

    sudo ./configure ./configure --with-apxs2=/usr/local/apache2/bin/apxs \
            --enable-mbstring \            
            --enable-mbregex \ 
            --with-mysql \ 
            --with-mysqli \
            --with-pdo-mysql \
            --with-pear \ 
            --with-zlib \
            --prefix=/usr/local/php5.5 \
            --enable-module=so \
            --with-openssl 

    sudo make

    sudo make install

# 確認 

httpd.conf(/usr/local/apache/conf/httpd.conf)に
    
    LoadModule php5_module modules/libphp5.so

が書いてるか確認。

# 追記

ServerName n14005:80

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>

をhttpd.confに追記

 DirectoryIndexと書いてある行にindex.phpを追記

　php.ini(usr/local/lib/php.ini)に

   mysql.default_socket =

という行があるので、

　　　mysql.default_socket = /var/lib/mysql/mysql.sock

を追記。

## ベンチマークを取る

abコマンドをとる

　　　　sudo apt-get install apache2-utils

ベンチマークを取る

    ab -n 10 -c 10 http://192.168.56.126/wordpress


##　プラグインをとって高速化

ftpを使わないでdirectで取る
wp-config.phpに

    define('FS_METHOD', 'direct');

を追記する。
さらに、

    define('WP_PROXY_HOST', 'http://プロキシURL');
    define('WP_PROXY_PORT', 'ポート番号');

を87行目ぐらいに追記する。



ワードプレスのメニューからプラグインを選び、新規追加を押す

wp-super-cacheで検索してインストール

















