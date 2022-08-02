## `Ubuntu 22.04`下`MySQL 8.0`的安装与配置

### `MySQL 8.0`安装

```shell
sudo apt install mysql-server-8.0
```

### `MySQL`初始密码获取

```shell
cd /etc/mysql
sudo vim debian.cnf
```

### `MySQL`的`root`密码修改

```mysql
use mysql
ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码';
```

### 远程连接配置

```
systemctl disable ufw
systemctl stop ufw

use mysql
UPDATE user SET host = '%' WHERE user = 'root';
```

### `MySQL`默认字符集查看与修改

```
SHOW VARIABLES LIKE '%char%';
```

