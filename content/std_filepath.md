## filepath模块

文件路径相关的函数

- ext(path string) string

    返回文件的扩展名，如果没有则返回空字符串

- is_abs(path string) bool

    判断路径是否是绝对路径

- join(base string, dir ...string) string

    把参数中的字符串连接成一个路径

- dir(path string) string

    返回路径中的目录部分，根据分隔符判断，如果没有分隔符则返回当前工作目录

- basedir(path string) string

    返回路径中的目录部分，根据分隔符判断，如果没有分隔符则返回参数路径本身

- filename(path string) string

    返回路径中的文件名部分