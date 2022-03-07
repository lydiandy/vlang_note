## V开发工具

目前V语言开发的主要环境是vscode，由V语言核心团队负责开发的插件和语言服务，而且一直在更新中。

基本的功能都有：

- 语法着色
- 代码提示
- 代码格式化
- 代码折叠
- 代码大纲视图
- 代码跳转定义

### vls

V语言核心团队实现了语言服务协议LSP v3.15版本，叫V Language Server(VLS)。可以提供给所有开发工具使用，目前支持得比较好的是vscode和sublime。

源代码：[https://github.com/vlang/vls](https://github.com/vlang/vls)

安装vls：参考[V语言服务章节](vls.md)

### vscode

安装插件：[https://github.com/vlang/vscode-vlang](https://github.com/vlang/vscode-vlang)

vls支持：参考[V语言服务章节](vls.md)。

### sublime text

安装插件：[https://github.com/elliotchance/vlang-sublime](https://github.com/elliotchance/vlang-sublime)

vls支持：

- 安装sublime的[LSP插件](https://packagecontrol.io/packages/LSP)

  打开命令行面板，输入Package Control，选择Install Package，输入LSP即可看到LSP插件，安装。

- 设置LSP插件，启用vls

  打开命令行面板，输入Package Control，选择Preferences: LSP Settings，加入以下配置：

  ```json
  "clients": {
     "vls": {
         "enabled": true,
         "command": ["<vls-dir>/vls"],
          "selector": "source.v"
      }
  }
  ```

### ved

编辑器：[https://github.com/vlang/ved](https://github.com/vlang/ved)

这是V语言的作者用V开发的编辑器，目前只支持V语言的开发，有兴趣的也可以尝试一下。

### micro

编辑器：[https://github.com/zyedidia/micro](https://github.com/zyedidia/micro)

micro是基于终端的编辑器，内置了V语言的语法高亮。

### vim

安装插件：[https://github.com/ollykel/v-vim](https://github.com/ollykel/v-vim)
