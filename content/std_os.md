## os模块

### 环境变量相关

- getenv(string) string    //获取环境变量

- setenv(name string, value string,overwrite bool) int //设置环境变量

- unsetenv(name string) int //清空环境变量

### 操作系统相关

- os.args 			//os模块常量,运行后只读,返回命令行的参数数组
- executable()     //返回当前可执行文件的全路径和文件名
- user_os() string   //返回当前操作系统名字
- home_dir() string          //返回当前用户的主目录

### 执行命令

- exec(string) ?Result //执行命令,然后等待执行结果,根据返回的exit_code和output进行处理

- system(string) int //执行系统命令  

  ```
  os.system('ln -sf $vexe $link_path')
  os.system('ls')
  ```

### 目录相关

- dir(string) string  //返回路径中的目录部分

- getwd() string //返回当前工作目录

- chdir(string) //改变当前工作目录

- ext(string) string //返回文件的扩展名,如果没有扩展名则返回点号

- basedir(string) //返回路径中的目录部分

- filename(string) //返回目录的文件名部分

- clear() //清屏

- mv(old,new) //移动文件

- mkdir(dir)  //创建目录

- rmdir(dir) //删除目录

- is_dir(path) bool //判断是否是目录

- dir_exists(path) bool //判断目录是否存在

- ls(path) [ ]string //获取该目录的所有文件和文件夹

- walk_ext(path,ext) []string   //返回改目录,及其下级子目录中所有,文件扩展名为ext的所有文件

### 文件相关

- realpath(path) string //返回文件夹或者文件的绝对路径

- open(path) ?File //打开文件,返回文件对象

- create(path) ?File //创建文件,返回文件对象

- rm(path) //删除文件

- File.close //关闭文件

- File.write(string) //写入文件

- File.write_bytes(data,size) //写入字节

- File.writeln //写入一行到文件

- read_file(string) ?string //读取文件,返回文件的内容

- write_file(string,string) 

- file_size(string) int //获取文件的大小

- file_exists(file) bool //检查文件或目录是否存在

  

### 控制台输入相关

- get_row_line() string //获取控制台输入的一行字符串,回车换行结束,通过C.getline实现

- get_line() string ////获取控制台输入的一行字符串,回车换行结束

- get_lines() []string//获取控制台输入的多行字符串,空行时回车换行结束,返回字符串数组

- get_lines_joined() string //获取控制台输入的多行字符串,空行时换行结束,返回连接后的字符串

### 常量

os.path_separator	//不同平台的路径分隔符