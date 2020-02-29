## term终端模块

命令行终端控制包,用来在终端上控制光标,输出字符,改变字符格式,颜色等

参考指令:https://www.gnu.org/software/screen/manual/html_node/Control-Sequences.html

基于libc的<sys/ioctl.h>

**基本使用**

- ok_message(s string) string

  返回绿色,执行成功的字符串

- fail_message(s string) string

  返回红色,执行失败的字符串

- h_divider(divider string) string

  根据指定的分隔符,显示一行分隔

- header(text, divider string) string

  根据指定的标题文本和分隔符,返回一行标题分隔

```c
module main

import (
	term
)

fn main() {
	s:=term.ok_message('ok')
	println(s)
	err:=term.fail_message('fail')
	println(err)
	d:=term.h_divider('-')
	println(d)
	h:=term.header('title','=')
	println(h)
}
```

**终端字体颜色,背景色,粗体,斜体,下划线等设置**

- bg_blue(msg string) string

  返回红色背景色的文字

- rgb(r, g, b int, msg string) string

  返回指定rgb颜色值的文字

- bg_rgb(r, g, b int, msg string) string

  返回指定rgb背景色的文字

- underline(msg string) string

  返回带下划线的文字

- italic(msg string) string

  返回斜体的文字

- bold(msg string) string

  返回粗体的文字

其他颜色等设置具体参考vlib/term/color.v

**终端鼠标控制部分,显示,隐藏,跳到指定位置等**

- set_cursor_position(x int, y int)

  设置光标位置

- show_cursor()

  显示光标

- hide_cursor()

  隐藏光标

- move(n int, direction string)

  移动光标

​    其他更多鼠标控制设置,参考vlib/term/control.v