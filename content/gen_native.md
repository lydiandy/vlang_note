## 生成native代码

### 可执行文件格式

目前native支持生成这三个平台的64位的可执行文件格式，没有使用其他的依赖或中间表示IR，根据AST直接生成对应平台的可执行文件格式。

#### linux elf

ELF (Executable and Linkable Format)是一种为可执行文件，目标文件，共享链接库和内核转储(core dumps)准备的标准文件格式。 Linux和很多类Unix操作系统都使用这个格式。

#### macos macho

Mach-O是Mach Object文件格式的缩写，是mac以及iOS上可执行文件的格式。是一种用于可执行文件、目标代码、动态库的文件格式。

#### windows pe

PE文件是Windows操作系统下使用的一种可执行文件，由COFF（UNIX平台下的通用对象文件格式）格式文件发展而来。32位成为PE32，64位称为PE+或PE32+。

