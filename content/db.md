## 数据库

### sqlite

使用sqlite模块之前，必须先安装好sqlite。

```shell
sudo dnf -y install `sqlite-devel`
或者
sudo apt install -y `libsqlite3-dev`
```

### mysql



### pg

如果之前没有安装过postgresql包，则import pg时会报错：缺失<libpq-fe.h>。

需要先安装：brew install postgresql。

更详细的SQL内容，可以参考[pg章节](./pg.md)。



### 内置sql支持

具体参考:[内置sql支持](sql.md)。

