1.下载:
https://dev.mysql.com/downloads/mysql/8.4.html 
2.下载并且安装vc2019运行库(Microsoft Visual C++ 2019 Redistributable Package)

https://aka.ms/vs/17/release/vc_redist.x64.exe
下载完成后双击安装
3.配置把MySQL解压到d:\programs并且做一些配置
1>.先把mysql的bin路径添加到环境变量中

<img width="889" height="873" alt="image" src="https://github.com/user-attachments/assets/65658ee8-7086-4c54-b036-d108f8a840d9" />

2>在MySQL的根目录下面新建一个my.ini,配置如下

[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8

[mysqld]
# 设置端口，不设置默认3306
port = 3306
# 设置mysql的安装目录
basedir=D:\\programs\\mysql-8.4.7-winx64
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
# datadir=D:\\programs\\mysql-8.4.7-winx64\\data #8.0以下版本需要配置数据目录
# 允许最大连接数
max_connections=200
# 服务端使用的字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# MySQL8.0默认的身份验证插件为caching_sha2_password，这将导致远程户端无法连接，可使用“mysql_native_password”插件认证解决客户端无法连接的问题，mysql_native_password再MySQL8.0中已过时，但任然可使用
# 从MySQL8.4开始，配置为 mysql_native_password=ON
mysql_native_password=ON
# default_authentication_plugin=mysql_native_password 此配置是在8.4以下的版本中的配置方法，8.4无此项
 
[client]
port=3306
default-character-set=utf8

4.mysql数据库初始化
1>以管理员身份打开一个cmd窗口,然后切换到: D:\programs\mysql-8.4.7-winx64\bin
 
2>输入命令（注意：根据需要修改路径）: mysqld --initialize --console执行完成后，会打印 root 用户的初始默认密码
 
比如,这里是: Lkud5tXx(dwa
然后会在数据库根目录下面生成一个data文件夹
 
5.把MySQL安装为服务,也可以不需要服务名称，默认是MySQL
使用命令: mysqld --install [服务名],如mysqld --install  mysql846 或者 mysqld --install
 
6.启动MySQL服务:使用net start mysql846
 
7.登录MySQL服务器,打开一个cmd窗口,输入命令mysql -uroot -p,会提示输入密码,我们输入上面的密码
 
8.修改MySQL密码
1>修改密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
 
2>刷新MySQL的系统权限相关表
mysql> flush privileges;   --刷新MySQL的系统权限相关表
 
3>退出登录
mysql> exit;
4>重新登录: mysql -uroot -p
 
登录成功,说明密码修改操作正确

