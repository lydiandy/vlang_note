## V语言服务(VLS)

V实现了语言服务协议LSP v3.15版本，叫做V Language Server(VLS)。

源代码：https://github.com/v-analyzer/v-analyzer

目前vls支持vs code和sublime text的插件调用，idea和其他开发环境还未适配。

### 安装

方式一：直接在vs code上安装

在vs code上搜索v-analyzer插件，下载。

方式二：从源代码安装

```shell
git clone https://github.com/v-analyzer/v-analyzer --下载源代码
cd v-analyzer
v build.vsh release --编译
```

方式三：直接下载预编译的二进制文件

下载地址：https://github.com/v-analyzer/v-analyzer/releases

如果不希望vs code插件使用默认的v-analyzer，而是想使用方式二和方式三的v-analyzer，可以在vs code中配置v-analyzer可执行文件的自定义路径：

![](vls.assets/instructions.png)

安装并配置完成后，就可以在vs code中使用V语言插件提供的代码完成，代码大纲，跳转到定义等功能。

### 实现

v-analyzer基于tree-sitter实现，tree-sitter相关代码库:

tree-sitter: https://github.com/tree-sitter/tree-sitter

tree-sitter-v: https://github.com/v-analyzer/v-analyzer/tree/main/tree_sitter_v