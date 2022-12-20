## 数据库驱动

V标准库目前内置了以下这4个数据库的驱动：

### sqlite

标准库：vlib/sqlite

使用sqlite模块之前，必须先安装好sqlite。

```shell
sudo dnf -y install `sqlite-devel`
或者
sudo apt install -y `libsqlite3-dev`
```

### pg

标准库：vlib/pg

如果之前没有安装过postgresql包，则import pg时会报错：缺失<libpq-fe.h>。

需要先安装：brew install postgresql。

更详细的SQL内容，可以参考：[pg章节](./pg.md)。

### mysql

标准库：vlib/mysql



### mssql

标准库：vlib/mssql



### 内置sql支持

除了使用数据库驱动库，V编译器也内置了sql代码块语句，具体可以参考：[内置sql支持](sql.md)。
