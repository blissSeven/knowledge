# MySQL
## 忘记密码
```bash
sudo cat /etc/mysql/debian.cnf

use mysql;
 update mysql.user set authentication_string=password('123456') where user='root' ;

CREATE USER 'lppllppl'@'%' IDENTIFIED BY '12345678'
grant all privileges on *.* to 'lppllppl'@'%'

flush privileges;
 service mysql restart;
```