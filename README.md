# chat_app
this is for studying websocket 



## 環境
| サービス | version |
|:-----------|------------:|
|CentOS 8|8.3.2011|
|Apache|2.4.37|
|PHP|8.0.2|
|composer|2.0.9|
|MySQL|8.0.23|

### 1. メモ
  - docker-compose.yml  
      * privileged: true => コンテナ起動時に、privilegedオプションをつける  
      * command: - /sbin/init => これを実行させることにより、systemctlが使用できるようになる。

### 2. build〜動作確認
* ビルド  
`$ docker-compose up -d`
<pre>
    Name                   Command               State                       Ports                     
-------------------------------------------------------------------------------------------------------
chat_mail     MailHog                          Up      0.0.0.0:1025->1025/tcp, 0.0.0.0:8025->8025/tcp
chat_mysql    docker-entrypoint.sh mysqld      Up      0.0.0.0:13309->3306/tcp, 33060/tcp            
chat_php      docker-php-entrypoint php-fpm    Up      0.0.0.0:55009->9000/tcp                       
chat_redis    docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp                        
chat_server   /sbin/init                       Up      0.0.0.0:80->80/tcp                            
chat_sftp     /entrypoint second:second_ ...   Up      0.0.0.0:2222->22/tcp      
</pre>

* centos8, apacheのコンテナに入る  
`$ docker-compose exec chat_server bash`

* apacheの起動設定  
`# systemctl start httpd`  
`# systemctl enable httpd`

* アクセス確認  
[テスト](http://test.lvh.me/)

### 3. laravel環境構築
*  phpのコンテナに入る  
`$ docker-compose exec chat_php bash`

* Laravelのプロジェクトを作成  
`# composer create-project --prefer-dist laravel/laravel chat_app`

* ドキュメントルートを追加（chat_server/apache/conf/apache.conf）  
=> 既に記載してあります。
```conf:chat_server/apache/conf/apache.conf
# laravelのプロジェクト
<VirtualHost *:80>
    ServerName lvh.me
    ServerAlias sample-project.lvh.me
    VirtualDocumentRoot /var/www/html/chat_app/public

    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://chat_php:9000"
    </FilesMatch>

    <Directory /var/www/html>
        DirectoryIndex index.php
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    DirectoryIndex index.php index.html
</VirtualHost>
```

* centos8, apacheのコンテナに入る    
`$ docker-compose exec chat_server bash`

* apache再起動  
`# systemctl restart httpd`

* .envの設定
<pre>
DB_CONNECTION=mysql
DB_HOST=second_mysql
DB_PORT=3306
DB_DATABASE=chat_db
DB_USERNAME=chat_user
DB_PASSWORD=chat

MAIL_MAILER=smtp
MAIL_HOST=chat_mail
MAIL_PORT=1025
MAIL_USERNAME=chat_user
MAIL_PASSWORD=chat_password
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS='sample@sample.com'
MAIL_FROM_NAME="${APP_NAME}"

REDIS_HOST=chat_redis
REDIS_PASSWORD=null
REDIS_PORT=6379
</pre>

### 4. メール確認
[メール確認](http://sample-project.lvh.me:8025)

### 5. 参考サイト
https://readouble.com/laravel/8.x/ja/broadcasting.html

