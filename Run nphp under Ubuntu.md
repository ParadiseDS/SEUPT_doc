### Nexusphp

参考 nexusphp/INSTALL

```
$sudo apt-get install apache2
$sudo vi /etc/apache2/sites-enabled/000-default
```

**000-default:** (htdocs改成你自己的目录名)

```xml
# configuration starts
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
<VirtualHost *:80>
        DocumentRoot "/var/www/htdocs"
        <Directory "/var/www/htdocs">
                Options FollowSymLinks
                AllowOverride None
                Order allow,deny
                Allow from all
        </Directory>
        <DirectoryMatch /\.svn/>
                AllowOverride None
                Order allow,deny
                Deny from all
        </DirectoryMatch>
        <Directory "/var/www/htdocs/_db">
                AllowOverride None
                Order allow,deny
                Deny from all
        </Directory>
        <Directory "/var/www/htdocs/config">
                AllowOverride None
                Order allow,deny
                Deny from all
        </Directory>
        <Directory "/var/www/htdocs/_doc">
                Options +Indexes
                Order allow,deny
                Allow from all
        </Directory>
        <Directory "/var/www/htdocs/lang">
                AllowOverride None
                Order allow,deny
                Deny from all
        </Directory>
</VirtualHost>
# configuration ends
```

完成以后，接下来是php和mysql

```bash
$ sudo apt-get install php5 php5-gd php5-memcache php5-mysql
$ sudo vi /etc/php5/apache2/php.ini
```

**php.ini:**（注意以下选项，如果是和下面相同的就不用改了）

```ini
; configuration starts
magic_quotes_gpc = Off
magic_quotes_runtime = Off
magic_quotes_sybase = Off
; Optional. Increase it if memory-limit-reached error occurs when uploading large torrent files.
memory_limit = 128M
; configuration ends
```
```
$ sudo apt-get install mysql-server
$ sudo vi /etc/mysql/my.cnf
```

**my.cnf:**（同上，和下面相同就不用改了）

```xml
# configuration starts
sql-mode=""
; Optional. Increase it if mysql connection-failure occurs under heavy traffic load.
max_connections = 1000
# configuration ends
server
```
 
***
14.04 下面按照默认设置会诡异地出现字符集的问题，中文存进去都是乱码 orz，虽然 php 读取的时候没问题但是直接从数据库输出就会变成乱码。
进到 mysql 命令行下面看：
 
```
$ mysql -u root -p
Enter password:
mysql> show variables like "%character%"; show variables like "%collation%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
 
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
3 rows in set (0.00 sec)
```
就是这一堆 latin 编码，就算数据库是 utf8 依然没什么卵用。
百度到的答案千篇一律，也没什么卵用。。。
去google了一下，keyword : mysql character_set_server
提示要更改 my.cnf
在 [mysqld] 加入如下行：

```
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```
重启 mysql 后就 ok 了。
 
***
图形界面管理数据库比较方便，这里安装的是phpmyadmin （htdocs 改为你自己的目录名）

```bash
$ sudo apt-get install phpmyadmin
$ sudo ln -s /home/paradiseds/Work/PT/htdocs /var/www
$ cd htdocs
$ sudo ln -s /usr/share/phpmyadmin/ /var/www/htdocs/
```

访问 127.0.0.1/phpmyadmin 可用，ok
<br />
接下来是memcached
 
    sudo apt-get install memcached
 
run as daemon
 
    memcached -d -u nobody
 
PEAR and Http_Request2

```bash
sudo apt-get install php-pear
sudo pear config-set preferred_state alpha
sudo pear install HTTP_Request2
```
 
Postfix（邮件系统）
 
    sudo apt-get install postfix
 
重启服务
 
    sudo /etc/init.d/apache2 restart
    sudo /etc/init.d/mysql restart
 
通过phpmyadmin导入_db/dbstructure.sql，注意要先新建一个数据库，然后选中，导入。

***

更改 nphp 目录文件权限（htdocs替换成你自己的目录名称）

    $ sudo chmod -Rf 777 /var/www/htdocs

然后编辑nphp的配置文件使其正常访问数据库，文件是 `config/allconfig.php`，以下各项修改成你的内容

```php
// configuration starts
$BASIC=array(
    'SITENAME' => 'yoursitename',
    'BASEURL' => 'yoursiteurl',
    'announce_url' => 'yoursiteurl/announce.php',
    'mysql_host' => 'yourdbhostname',
    'mysql_user' => 'yourdbusername',
    'mysql_pass' => 'yourdbpassword',
    'mysql_db' => 'yourdbname',
);
// configuration ends
```

这样子打开 127.0.0.1应该就可以访问到了
