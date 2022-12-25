## 编码风格

### V语言命名风格

- 关键字，模块，常量，变量，枚举值，函数，方法，内置类型，强制采用小蛇式命名风格(lower snake case)。

- 枚举，结构体，接口，类型别名，联合类型等自定义类型，强制采用大骆驼峰式命名风格(upper camel case)。

V语言以小蛇命名风格为主，使得代码阅读起来非常舒服，不会那么突兀，甚至连常量也是小蛇风格。

### 常用命名风格参考

#### camel case(驼峰式)

特点：名称中间没有空格和标点，除第一个单词外后面的单词首字母均大写。

##### 大驼峰命名

如果第一个单词首字母大写，称之为`upper camel case`（`CamelCase`），例如`"GetUserName"`。

##### 小驼峰命名

如果第一个单词首字母小写，称之为`lower camel case`（`camelCase`），例如`"getUserName"`。

#### snake case(蛇式)

特点：名称中间的标点被替换成下划线（`_`）。

##### 小蛇式命名

如果所有单词都小写，称之为`lower snake case`，例如`"get_user_name"`。

##### 大蛇式命名

如果所有单词都大写，称之为`upper snake case`，例如`"GET_USER_NAME"。`

#### kebab case(烤肉串式)

特点：名称中间的标点被替换成连字符（`-`），所有单词都小写，例如`"get-user-name"`。

参考：https://zhuanlan.zhihu.com/p/35098504。

