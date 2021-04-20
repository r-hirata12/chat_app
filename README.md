# chat_app
this is for studying websocket 



## 環境
DockerのデフォルトOSである、DebianではなくCentOSで作成してみたかった。  
そのうち、CentOS Streamで再度作成したい。  
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
second_mail     MailHog                          Up      0.0.0.0:1025->1025/tcp, 0.0.0.0:8025->8025/tcp
second_mysql    docker-entrypoint.sh mysqld      Up      0.0.0.0:13309->3306/tcp, 33060/tcp            
second_php      docker-php-entrypoint php-fpm    Up      0.0.0.0:55009->9000/tcp                       
second_redis    docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp                        
second_server   /sbin/init                       Up      0.0.0.0:80->80/tcp                            
second_sftp     /entrypoint second:second_ ...   Up      0.0.0.0:2222->22/tcp      
</pre>

* centos8, apacheのコンテナに入る  
`$ docker-compose exec second_server bash`

* apacheの起動設定  
`# systemctl start httpd`  
`# systemctl enable httpd`

* アクセス確認  
[テスト](http://test.lvh.me/)

### 3. laravel環境構築
*  phpのコンテナに入る  
`$ docker-compose exec second_php bash`

* Laravelのプロジェクトを作成  
`# composer create-project --prefer-dist laravel/laravel sample-project`

* ドキュメントルートを追加（second_server/apache/conf/apache.conf）  
=> 既に記載してあります。
```conf:second_server/apache/conf/apache.conf
# laravelのプロジェクト
<VirtualHost *:80>
    ServerName lvh.me
    ServerAlias sample-project.lvh.me
    VirtualDocumentRoot /var/www/html/sample-project/public

    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://second_php:9000"
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
`$ docker-compose exec second_server bash`

* apache再起動  
`# systemctl restart httpd`

* .envの設定
<pre>
DB_CONNECTION=mysql
DB_HOST=second_mysql
DB_PORT=3306
DB_DATABASE=second_db
DB_USERNAME=second_user
DB_PASSWORD=second

MAIL_MAILER=smtp
MAIL_HOST=second_mail
MAIL_PORT=1025
MAIL_USERNAME=second_user
MAIL_PASSWORD=second_password
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS='sample@sample.com'
MAIL_FROM_NAME="${APP_NAME}"

REDIS_HOST=second_redis
REDIS_PASSWORD=null
REDIS_PORT=6379
</pre>

### 4. メール確認
[メール確認](http://sample-project.lvh.me:8025)

