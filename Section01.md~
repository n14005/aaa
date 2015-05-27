## 3 CentOSのダウンロード

[CentOS公式サイト](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso)に行って、

一覧のActual Country から

http://ftp.nara.wide.ad.jp/pub/Linux/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso

をダウンロード。

## 4　VirtualboxへCentOSをインストール

- ネットワークアダプター2を設定、割り当てをホストオンリーアダプタにする。

- VirtualBoxの最初の画面に戻り、左上の新規を押す。

- 色々指示がでるので、仮想マシンのメモリサイズ1GB、ストレージ容量は8GB、それ以外はすべてデフォルトで大丈夫。

- 上の設定が終わったら起動したら色々指示が出るので流れにそってインストールに入る、インストール中にroot以外の作業用ユーザ（管理者）を作成する。

## 5 ネットワークアダプター1/2へのIPアドレスの設定

- CentOSの中に、/etc/sysconfig/network-scriptにifcfg-enp0s?というファイルがあるのでそれを開く。

    ONBOOT=no	

という、行があるので

    ONBOOT=yes

に書き換えて、保存する。

## 6 SSH接続確認


- Ubuntuのターミナルから、

    ssh CentOSのIPアドレス

または、

　　　　ssh ユーザー名@CentOSのIPアドレス

で、CentOSに入れたら成功。



##　7 インストール後の設定

- CentOSのyum,wgetのプロキシ設定。

    sudo vi /etc/yum.conf

、でproxy=http://172.16.40.1:8888/を適当な場所に記述する。

wgetが入ってないと思うので、

    yum install wget

をした後に、

    sudo vi /etc/wgetrc

で、

#http_proxy=http://IPアドレス：ポート番号/
#https_proxy=http://IPアドレス：ポート番号/
#ftp_proxy=http://IPアドレス：ポート番号/

この三行を、

https_proxy = http://172.16.40.1:8888/
http_proxy = http://172.16.40.1:8888/
ftp_proxy = http://172.16.40.1:8888/

に直す。終わり。

## 8 アップデート確認

    yum update

で、プロキシが通っていたらアップデートできます。



## 9 Worpressを動かす

- LAMP環境を作る

　　　　yum -y install php-mysql php php-gd php-mbstring
　　　　yum -y install httpd
    yum -y install mariadb mariadb-server

をインストール。

###　Apacheの設定

　　　　systemctl start httpd
	↑　httpd起動

　　　　systemctl enable httpd　
	↑　httpd自動起動設定


### MySQLの設定

mariadb のデーモン（サービス）を起動します。
    systemctl start mariadb.service

mariadbのデーモン（サービス）が起動しているか確認。
　　　　systemctl list-units |grep mariadb

active runnningになっていたらOK。

次に、mariadbデーモンがブート時に自動起動するように設定しておきます。
    systemctl enable mariadb.service
 
mariadbデーモンの登録状態を確認します。
    systemctl list-unit-files |grep mariadb

enableになっていたらOK。

- Marinadbの設定

　　　　mysql -u root -p
	↑ルートでMarinadbへログイン

　　　　create databese データベース名
	↑任意の名前のデータベースを作成

    grant all privileges on データベース名.* to ユーザー名@localhost identified by 'パスワード';
	↑データベース名と、任意のユーザー名・パスワードを入力

### Wordpressのインストール

[ここ](http://ufuso.jp/wp/?p=15315)確認しながらやって

　　　　yum -y install php-mysql
	↑何かわからないけどインストール

ここからWordpress

    wget http://ja.wordpress.org/wordpress-3.9.1-ja.tar.gz
	↑Wordpressインストール

　　　　tar zxvf wordpress-3.9.1-ja.tar.gz
	↑Wordpress解凍

　　　　mv wordpress/ /var/www/html/データベース名　
	↑WordPress解凍先ディレクトリを/var/www/html/データベース名　ディレクトリ下へ移動（データベース名はmariadbの奴）

    chmod 777 /var/www/html/データベース名　
	↑WordPressディレクトリを一時的に書込可にする

　　　　chown -R 任意のユーザー名:apache /var/www/html/wpress/
	↑　WordPressディレクトリとその中の全ファイルの所有者を任意のユーザー名、グループをApache実行ユーザへ変更（「apache:apache」も可）

　　　　mkdir /var/www/html/データベース名/wp-content/uploads　
	↑　uploadsフォルダの作成

　　　　mkdir /var/www/html/データベース名/wp-content/upgrade　
	↑　upgradeフォルダの作成

　　　　chmod -R 777 /var/www/html/wpress/wp-content　
	↑wp-contentフォルダとその中の全ファイルに読み書き権限の設定

　
###SELinuxの設定

[ここ](http://ufuso.jp/wp/?p=15315)確認しながらやって

　　　　setsebool -P httpd_can_network_connect_db 1

　　　　setsebool -P httpd_dbus_avahi 1

　　　　setsebool -P httpd_tty_comm 1

　　　　setsebool -P httpd_unified 1

　　　　yum provides *bin/semanage

　　　　yum -y install policycoreutils-python

    semanage fcontext -a -t httpd_sys_content_t "/var/www/html/データベース名(/.*)?"

    restorecon -R -v /var/www/html/データベース名

    semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/データベース名/wp-content(/.*)?"

    restorecon -R -v /var/www/html/データベース名/wp-content

　　　　setsebool -P allow_ftpd_full_access 1

　　　　systemctl restart httpd

###ファイアーウォール止める

    systemctl stop firewalld
	↑　止めます

### Wordpress初期設定

http://CentOSのIPアドレス/データベース名/

で、Wordpressにログイン。

後は流れにそってやる。


参考サイト
http://ufuso.jp/wp/?p=15122

http://server-setting.info/blog/lamp-wordpress-centos7.html




	
