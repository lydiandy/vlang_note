## gx

gx模块比较简单，主要维护了颜色，图像和字体配置这三个结构体，目前主要给gg模块使用。

gx模块估计很快会被合并到gg模块中。

### Color 颜色

#### 结构体

```v
pub struct Color {
pub mut:
	r byte
	g byte
	b byte
	a byte = 255
}
```

#### 构建函数

- gx.rgb(r, g, b) Color  //通过r，g，b创建颜色，a默认是255
- gx.rgba(r,g,b,a) Color //通过r，g，b，a创建颜色
- gx.hex(c int) Color //通过整数值创建颜色

#### 方法


- c.eq(b Color) bool //判断2个颜色是否相等


- c.str() string //颜色的字符串输出

#### 常量

标准颜色：

gx.red

gx.green

gx.blue

...

#### 运算符重载

颜色支持了+，-，*，/的运算符重载

### Image 图像

#### 结构体

```v
pub struct Image {
mut:
	obj    voidptr //图像数据的首字节指针
pub:
	id     int //图像id
	width  int //图像宽度
	height int //图像高度
}
```

#### 方法

i.is_empty() bool //判断图像对象是否为空

### FontCfg 字体配置

#### 结构体

```v
pub struct TextCfg {
pub:
	color     Color
	size      int
	align     int
	max_width int
	family    string
	bold      bool
	mono      bool
}
```



