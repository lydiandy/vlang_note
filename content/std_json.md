## json模块

### 使用

v标准库的json模块有点特别:

1. 基于C语言的cJSON库实现

2. 没有使用运行时反射,性能会更好

3. 解析JSON功能在编译器内部实现,内置支持JSON

具体如何使用json.encode()和json.decode()函数,[参考内置JSON章节](./json.md)

### 实现原理

下面主要分析如何实现内置JSON的基本原理:

1. cJSON源文件存放在编译器源代码thirdparty/cJSON目录中

2. vlib/json/json_primitives.v对cJSON进行简单的封装

3. 编译器进行语法解析时,如果碰到json.encode和json.decode函数,调用json_primitives包的函数进行编码和解码

4. 解析器解析到json.encode(),会根据要解析的变量及其类型,调用cJSON的创建节点对象,构造节点函数,构造出节点数,然后调用cJSON_PrintUnformatted(),生成对应的json字符串

5. 解析器解析到json.decode(),会根据要解析的字符串和结构模板,调用cJSON_Parse(),生成cJSON对象,然后赋值给v变量

   就是在vlib/json/json_primitives.v对两个函数进行了简单封装:

 ```c
   fn json_parse(s string) &C.cJSON { //解码
   	return C.cJSON_Parse(s.str)
   }
   
   // json_string := json_print(encode_User(user)) 
   fn json_print(json &C.cJSON) string { //编码
   	s := C.cJSON_PrintUnformatted(json)
   	return tos(s, C.strlen(s))
   }
 ```

   

cJSON项目:https://github.com/DaveGamble/cJSON

cJSON使用参考:

**cJSON结构体**

链表结构实现:

```c
//节点类型
#define cJSON_Invalid (0)
#define cJSON_False  (1 << 0)
#define cJSON_True   (1 << 1)
#define cJSON_NULL   (1 << 2)
#define cJSON_Number (1 << 3)
#define cJSON_String (1 << 4)
#define cJSON_Array  (1 << 5)
#define cJSON_Object (1 << 6)
#define cJSON_Raw    (1 << 7) /* raw json */
//节点结构体定义
typedef struct cJSON //表示一个json节点
{
    /* next/prev allow you to walk array/object chains. Alternatively, use GetArraySize/GetArrayItem/GetObjectItem */
    struct cJSON *next; //节点的下一个节点
    struct cJSON *prev; //节点的上一个节点
    /* An array or object item will have a child pointer pointing to a chain of the items in the array/object. */
    struct cJSON *child; //节点的子节点

    /* The type of the item, as above. */
    int type; //就是上面的节点类型

    /* The item's string, if type==cJSON_String  and type == cJSON_Raw */
    char *valuestring; 
    /* writing to valueint is DEPRECATED, use cJSON_SetNumberValue instead */
    int valueint;
    /* The item's number, if type==cJSON_Number */
    double valuedouble;

    /* The item's name string, if this item is the child of, or is in the list of subitems of an object. */
    char *string;
} cJSON;
```

**解码(parse)**

```c
/* Memory Management: the caller is always responsible to free the results from all variants of cJSON_Parse (with cJSON_Delete) and cJSON_Print (with stdlib free, cJSON_Hooks.free_fn, or cJSON_free as appropriate). The exception is cJSON_PrintPreallocated, where the caller has full responsibility of the buffer. */
/* Supply a block of JSON, and this returns a cJSON object you can interrogate. */
//把json字符串解析为节点树对象,即解码
CJSON_PUBLIC(cJSON *) cJSON_Parse(const char *value);
```

**编码(print)**

```c
/* Render a cJSON entity to text for transfer/storage. */
//把节点对象输出成json字符串,有格式化
CJSON_PUBLIC(char *) cJSON_Print(const cJSON *item);
/* Render a cJSON entity to text for transfer/storage without any formatting. */
//把节点变量输出成json字符串,不带格式化
CJSON_PUBLIC(char *) cJSON_PrintUnformatted(const cJSON *item);
```

```c
//各种操作节点的函数
CJSON_PUBLIC(cJSON *) cJSON_CreateRaw(const char *raw);//创建原始字符串节点
CJSON_PUBLIC(cJSON *) cJSON_CreateArray(void); //创建节点数组
CJSON_PUBLIC(cJSON *) cJSON_CreateObject(void); //创建节点对象

/* These calls create a cJSON item of the appropriate type. */
CJSON_PUBLIC(cJSON *) cJSON_CreateNull(void); //创建空节点
CJSON_PUBLIC(cJSON *) cJSON_CreateTrue(void);  //创建true节点
CJSON_PUBLIC(cJSON *) cJSON_CreateFalse(void); //创建false节点
CJSON_PUBLIC(cJSON *) cJSON_CreateBool(cJSON_bool boolean); //创建布尔节点
CJSON_PUBLIC(cJSON *) cJSON_CreateNumber(double num); //创建数值节点
CJSON_PUBLIC(cJSON *) cJSON_CreateString(const char *string); //创建字符串节点

/* Append item to the specified array/object. */
//把创建好的节点添加到节点树上
CJSON_PUBLIC(void) cJSON_AddItemToArray(cJSON *array, cJSON *item);
CJSON_PUBLIC(void) cJSON_AddItemToObject(cJSON *object, const char *string, cJSON *item);

/* These functions check the type of an item */
//判断节点类型
CJSON_PUBLIC(cJSON_bool) cJSON_IsInvalid(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsFalse(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsTrue(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsBool(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsNull(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsNumber(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsString(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsArray(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsObject(const cJSON * const item);
CJSON_PUBLIC(cJSON_bool) cJSON_IsRaw(const cJSON * const item);

/* Delete a cJSON entity and all subentities. */
CJSON_PUBLIC(void) cJSON_Delete(cJSON *c); //删除节点


```

不过,按V作者的说法,后面会移除对cJSON库的依赖,自己实现

