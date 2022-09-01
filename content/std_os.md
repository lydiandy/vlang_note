## os模块

### 环境变量相关

- getenv(key string) string    

    获取指定的环境变量,如果没有指定的环境变量,则返回空字符串

- environ() map[string] string

    获取所有的环境变量

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
    
- config_dir() ?
  
    返回对应操作系统的用户配置目录，window返回%AppData%，macos/ios返回~/Library/Application Support，其他操作系统优先返回XDG_CONFIG_HOME环境变量设置的值，如果没有设置XDG_CONFIG_HOME，则返回~/.config。如果返回的值为空，则抛出错误。

### 执行命令

- os.execute(string) Result 

    运行命令,运行完成后返回Result, Result.exit_code为运行结果码,Result.output为运行结果的字符串内容,运行结果并不输出到终端中

- os.system(string) int

  运行命令,将命令运行结果输出到终端,然后返回结果码,跟exit()的结果码一样
  
  ```
  os.system('ln -sf $vexe $link_path')
  os.system('ls')
  ```

### 进程相关

- os.new_process() &Process

  创建一个新的进程对象,此时并未实际创建进程,而是执行p.run()方法以后,才会实际创建,创建后才会生成对应的进程ID,可以通过p.pid访问到

### 目录相关

- file_name(path string) string

    返回路径中的文件名部分

- dir(path string) string

    返回路径中的目录部分，根据分隔符判断，如果没有分隔符则返回点号(当前目录)

- base(path string) string

    返回路径中的最后一个元素，根据分隔符判断，如果没有分隔符则返回path本身

- is_abs_path(path string) bool

    判断路径是否是绝对路径

- join_path(base string, dir ...string) string

    把参数中的字符串连接成一个路径

- getwd() string

    返回当前工作目录

- chdir(string)

    改变当前工作目录

- clear() 

    清屏

- mv(old, new string)

    移动文件

- cp(old, new string) ?bool

    复制文件

- mkdir(path string, params MkdirParams) ?bool

    创建目录

- rmdir(dir)

    删除目录

- is_dir(path) bool

    判断路径是否为目录，如果是目录返回true，如果不是目录返回false

- ls(path string) [ ]string

    获取该目录的所有文件和文件夹

- is_writable_folder(folder string) ?bool

    判断目录是否可写

- walk_ext(path string,ext string) []string

    返回改目录,及其下级子目录中所有,文件扩展名为ext的所有文件

### 文件相关

- exists(path string) bool

    判断文件或者目录是否存在

- is_file(path string) bool

    判断路径是否为文件，如果是文件返回true，如果不是返回false

- is_executable(path string) bool

    判断文件是否是可执行文件

- is_writable(path string) bool

    判断文件是否可写

- is_readable(path string) bool

    判断文件是否可读

- file_ext(path string) string

    返回文件的扩展名，带点号，如果没有扩展名返回空

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

- read_bytes(path string) ?[]u8

  读取文件,返回字节数组

- write_file(path string,text string) 

  创建文件,并写入text内容

- file_size(path string) int 

  获取文件的大小

- File.write(buf []u8) ?int

    写入文件

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

### 子模块os.cmdline

目前一共有6个公共函数,用于获取选项之后/之前的参数

- cmdline.options(args []string,param string) []string

  获取数组args中,param之后的参数值,如果param在数组中多次出现,之后的参数值都取出来,按顺序保存在返回的参数值数组中

- cmdline.option(args []string,param string,default string) string

  获取数组args中,param之后的参数值,如果param没有出现在数组args中,则返回default 默认值

- cmdline.options_before(args []string,what []string) []string

  获取数组args中,如果元素有包含在数组what中,返回元素之前的参数值

- cmdline.options_after(args []string,what []string) []string

  获取数组args中,如果元素有包含在数组what中,返回元素之后的参数值

- cmdline.only_non_options(args []string) []string

  获取数组args中,所有不是以-开头的元素

- cmdline.only_options(args []string) []string

  获取数组arg中,所有以-开头的元素

