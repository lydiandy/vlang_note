## V语言服务(VLS)

V实现了语言服务协议LSP v3.15版本，叫做V Language Server(VLS)。

源代码：https://github.com/vlang/vls

目前vls仅支持vs code的插件调用，其他开发环境还未适配。

### 安装

- 方式一：直接在vs code上安装

  在vs code上安装V语言插件后，可以通过命令面板搜索并执行V:Update VLS命令，此命令的默认安装路径为：~/.vls。

  安装后可以重复执行命令，升级VLS。

  

- 方式二：从源代码安装

  ```shell
  #安装依赖
  git clone https://github.com/nedpals/tree-sitter-v ~/.vmodules/tree_sitter_v
  #下载vls源代码，切换到use-tree-sitter分支
  git clone https://github.com/vlang/vls.git --branch use-tree-sitter vls && cd vls/
  #编译vls。目前V的垃圾回收还不成熟，建议编译时加入可选垃圾回收器boehm-gc来帮助内存回收，毕竟vls需要常驻内存中运行
  #安装完成后可以在cmd/vls目录中看到vls可执行文件
  v -gc boehm -cc gcc cmd/vls
  ```

  

- 方式三：直接下载预编译的二进制文件

  下载地址：https://github.com/vlang/vls/releases

如果不希望vs code插件使用默认的vls，而是想使用方式二和方式三的vls,可以在vs code中配置vls可执行文件的自定义路径：

![](vls.assets/instructions.png)

安装并配置完成后，就可以在vs code中使用V语言插件提供的代码完成，代码大纲，跳转到定义等功能。

### 实现

vls基于tree-sitter实现，tree-sitter相关代码库:

**tree-sitter:**

https://tree-sitter.github.io/tree-sitter/

https://github.com/tree-sitter/tree-sitter

**tree-sitter-v:**

https://github.com/nedpals/tree-sitter-v