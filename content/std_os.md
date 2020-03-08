## os模块

### 环境变量相关

- getenv(key string) string    

    获取环境变量,如果没有指定的环境变量,则返回空字符串

- setenv(name string, value string,overwrite bool) int 

    设置环境变量,如果overwrite为true则覆盖原来的环境变量,如果为false,若环境变量已存在,则不覆盖,若环境变量未存在,则新增

- unsetenv(name string) int 

    清空指定名字的环境变量

### 操作系统相关

- os.args 			

    os模块常量,运行后只读,返回命令行的参数数组

- os.getpid()

    获取当前进程ID

- executable() string

    返回当前可执行文件的全路径和文件名

- user_os() string   

    返回当前操作系统名字

- home_dir() string

    返回当前用户的主目录

### 执行命令

- exec(string) ?Result 

    执行命令,然后等待执行结果,根据返回的exit_code和output进行处理

- system(string) int

  执行系统命令  
  
  ```
  os.system('ln -sf $vexe $link_path')
  os.system('ls')
  ```

### 目录相关

- ext(path string) string

    返回文件的扩展名，如果没有则返回空字符串

- filename(path string) string

    返回路径中的文件名部分

- dir(path string) string

    返回路径中的目录部分，根据分隔符判断，如果没有分隔符则返回当前工作目录

- basedir(path string) string

    返回路径中的目录部分，根据分隔符判断，如果没有分隔符则返回参数路径本身

- is_abs(path string) bool

    判断路径是否是绝对路径

- join(base string, dir ...string) string

    把参数中的字符串连接成一个路径

- getwd() string

    返回当前工作目录

- chdir(string)

    改变当前工作目录

- clear() 

    清屏

- mv(old,new string)

    移动文件

- cp(old,new string) ?bool

    复制文件

- mkdir(dir) ?bool

    创建目录

- rmdir(dir)

    删除目录

- is_dir(path) bool

    判断是否是目录

- dir_exists(path) bool

    判断目录是否存在

- ls(path string) [ ]string

    获取该目录的所有文件和文件夹

- walk_ext(path string,ext string) []string

    返回改目录,及其下级子目录中所有,文件扩展名为ext的所有文件

### 文件相关

- realpath(path string) string

    返回文件夹或者文件的绝对路径

- open(path string) ?File 

    打开文件,返回文件对象

- create(path string) ?File

    创建文件,返回文件对象

- rm(path string)

    删除文件

- read_file(path string) ?string

  读取文件,返回文件的内容

- read_bytes(path string) ?[]byte

  读取文件,返回字节数组

- write_file(path string,text string) 

  创建文件,并写入text内容

- file_size(path string) int 

  获取文件的大小

- File.write(s string)

    写入文件

- File.write_bytes(data voidptr,size int)

    写入字节

- File.writeln(s string)

    写入一行到文件
- File.close()

  关闭文件


### 控制台输入相关

- get_row_line() string

    获取控制台输入的一行字符串,回车换行结束,通过C.getline实现

- get_line() string

    获取控制台输入的一行字符串,回车换行结束

- get_lines() []string

    获取控制台输入的多行字符串,空行时回车换行结束,返回字符串数组

- get_lines_joined() string

    获取控制台输入的多行字符串,空行时换行结束,返回连接后的字符串

### 控制台输出相关

- flush_stdout()

  控制台清屏

常量

- os.path_separator

    不同平台的路径分隔符