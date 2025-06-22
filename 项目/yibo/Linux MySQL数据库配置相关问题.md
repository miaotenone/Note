在Linux配置MySQL，需要安装三个相关的包。
```
sudo apt-get install mysql-server
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```
配置文件的设置
![[mysql配置文件路径.png]]
![[mysql配置文件详细.png]]
bind-address 设置为 0.0.0.0，原因就是解除限制ip地址

登录的命令是
```
mysql -u root -p 
```

给权限的命令
```
CREATE USER 'root'@'ip地址' IDENTIFIED BY '数据库的密码';//不同的ip就要创建一次，创建完才能给权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'ip地址';
```
在实际项目当中MySQL的连接
```
ret = mysql_real_connect(&m_db,
	args.at("host"), args.at("user"),
	args.at("password"), args.at("db"),
	atoi(args.at("port")),
	NULL, 0);
unsigned int xx = mysql_errno(&m_db);
```
xx返回值为1130，这个返回值代表 "Host 'xxx.xxx.xxx.xxx' is not allowed to connect to this MySQL server"。错误可能是由于以下几个原因之一引起的：
1. MySQL 用户账户没有从你的 IP 地址进行连接的权限。
2. MySQL 配置文件（如 my.cnf 或 my.ini）中的 bind-address 设置不允许远程连接。
3. MySQL 的防火墙设置阻止了连接。
该项目的错误原因在于上述的权限设置，不同的ip就需要重新设置。

xx返回值为1049，Unknown database 'xxx'，表示MySQL服务器上没有找到指定的数据库。创建对应的数据库就可。
```
-- 登录 MySQL
mysql -u root -p

-- 显示所有数据库
SHOW DATABASES;

-- 创建新数据库（替换 your_db_name）
CREATE DATABASE your_db_name;

-- 授予用户权限
GRANT ALL PRIVILEGES ON your_db_name.* TO 'root'@'192.168.204.1';
FLUSH PRIVILEGES;
```