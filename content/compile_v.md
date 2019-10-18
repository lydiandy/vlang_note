## 编译V源码

1. **编译准备:**


   目前V语言的编译需要依赖C编译器:gcc或clang

   在命令行执行一下gcc -v 看看是否已经安装了编译器

2. **下载源码:** 

   git clone https://github.com/vlang/v


3. **编译:**

   cd v
   make

   编译成功后,会在当前目录生成v可执行文件

4. **创建应用替身(可选,mac/linux上可用):**

   执行:v symlink

   在linux和mac上会创建:/usr/local/bin/v的应用替身(快捷方式),这样v编译器就可以在任何地方使用

