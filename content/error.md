## 错误处理



函数定义时:

返回类型前面加?,表示这个函数可能返回一个错误

在函数代码中根据逻辑丢出错误:return error('错误内容') 

error函数是内置的

函数调用时:

使用or代码块来处理错误,其中err是内置的错误参数名

err参数也是内置的

or代码块必须以:panic/return/continue/break结尾

