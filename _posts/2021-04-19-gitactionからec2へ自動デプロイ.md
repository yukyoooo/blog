---
layout: post
title: "gitからawsへのCI/CDをgit actionを使って構築する"
featured-img: shane-rounce-205187
categories: [CI/CD, github actions]
---


## githubのactionsを使ってmasterブランチにpushもしくはPRのmergeを行ったときに自動でec2にデプロイする


1. AWSのec2にssh接続後、ssh鍵の生成。途中の質問は何も入力せずエンターを押す。

```
ssh-keygen -t rsa -b 4096 -m pem -C "hoge@hoge.co.jp"
```
```
cat id_rsa.pub >> authorized_keys
```
<br>
<br>
2. github/ec2にあげるrepository/Setting/Secretに変数を登録

- HOST_NAME：EC2で立ち上げたインスタンスのIPアドレス
- USER_NAME：ユーザー名（今回はec2-user）
- PRIVATE_KEY：秘密鍵の内容をコピーして貼り付ける
<br>
<br>

3. github/Actions/Set up this workflow

```
name: Laravel
 
on:
  push:
    branches: [master]
  pull_request:
    branches: [master]
 
jobs:
  laravel-tests:
    runs-on: ubuntu-latest
 
    steps:
      - uses: actions/checkout@v2
      - name: Copy .env
        run: php -r "file_exists('.env') || copy('.env.example', '.env');"
      - name: Install Dependencies
        run: composer install -q --no-ansi --no-interaction --no-scripts --no-suggest --no-progress --prefer-dist
      - name: Generate key
        run: php artisan key:generate
      - name: Directory Permissions
        run: chmod -R 777 storage bootstrap/cache
      - name: Create Database
        run: |
          mkdir -p database
          touch database/database.sqlite
      - name: Execute tests (Unit and Feature tests) via PHPUnit
        env:
          DB_CONNECTION: sqlite
          DB_DATABASE: database/database.sqlite
        run: vendor/bin/phpunit
 
  deploy:
    if: github.ref == 'refs/heads/master'
    needs: [laravel-tests]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy
        env:
          PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
          USER_NAME: ${{ secrets.USER_NAME }}
          HOST_NAME: ${{ secrets.HOST_NAME }}
        run: |
          echo "$PRIVATE_KEY" > private_key && chmod 600 private_key
          ssh -o StrictHostKeyChecking=no -i private_key ${USER_NAME}@${HOST_NAME} 'cd /var/www/aws-laravel/ && git pull origin master'
```
<br>
<br>

4. 上記ソースの最後から2行目のcd移動先をEC2のlaravelソースのパスに変更する(var/www/laravel)の場合
```
cd /var/www/aws-laravel/ -> cd /var/www/laravel/ 
```
<br>
<br>

5. githubのmasterブランチにpushしてみる








[参考サイト] (https://noumenon-th.net/programming/2020/05/08/github-actions/)
