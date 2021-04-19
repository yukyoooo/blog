---
layout: post
title: "awsのec2にlaravelを設置する"
featured-img: shane-rounce-205187
categories: [Sample, Guides]
---

## ec2にssh接続後行うコマンド  
<br />

gitのインストール
```linux
sudo yum install git
git config --global user.name "hoge"
git config --global user.email hoge@hoge.co.jp
```
<br/>


phpのインストール
```
sudo amazon-linux-extras install -y php7.4
sudo yum install -y php-xml
sudo yum install -y php php-mbstring
sudo yum install -y php php-intl
```
<br />

composerのインストール
```
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer update
composer install
```
<br />


gitに接続するためのsshキーの作成
```
cd ~/.ssh
ssh-keygen -t rsa -C "[Gitlabで登録したEmail]"
sudo chmod 600 id_rsa
```
<br />

権限設定
```
sudo chown ec2-user:ec2-user /var/www
```
<br />

git cloneでlaravelソースの配置
```
cd /var/www
git clone https://github.com/hoge/hoge-laravel.git
```
<br />

apacheのルードドキュメント設定
```
sudo vi /etc/httpd/conf/httpd.conf	
```
<br />

ドキュメントルートをlaravelのpublicに指定
```
-> DocumentRoot "/var/www/hoge-laravel/public"
```
下記を最後の行に追記(https接続可能にするため)
```
-> <Directory /var/www/hoge-laravel/public>
->  AllowOverride All
-> </Directory>
```
<br />

apacheの再起動
```
sudo service httpd restart
```
<br />

laravelのenvの設定
```
cd hoge-laravel/
cp .env.example .env
php artisan key:generate
```
<br />

laravelへの書き込み権限付与
```
chmod -R 777 storage
chmod -R 777 bootstrap/cache
```
<br />

[ssh参考サイト](https://qiita.com/Hide-Zaemon/items/400b21183197481ecef4)