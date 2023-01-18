## 数据库驱动

V标准库内置4个常用数据库的驱动：

### sqlite

标准库：vlib/db/sqlite

使用sqlite模块之前，必须先安装好sqlite。

```shell
sudo dnf -y install `sqlite-devel`
或者
sudo apt install -y `libsqlite3-dev`
```

### pg

标准库：vlib/db/pg

如果之前没有安装过postgresql包，则import pg时会报错：缺失<libpq-fe.h>。

需要先安装：brew install postgresql。

更详细的SQL内容，可以参考：[pg章节](./pg.md)。

### mysql

标准库：vlib/db/mysql



### mssql

标准库：vlib/db/mssql



### 内置sql支持

除了使用数据库驱动库，V编译器也内置了sql代码块语句，具体可以参考：[内置sql支持](sql.md)。

### 社区库

vsql：https://github.com/elliotchance/vsql

