# 对象类型

在Ruby中一切都是对象，在C语言中要访问Ruby的对象是通过VALUE类型的变量进行引用的。

Ruby中定义了一些内建的类型，由于文档中没有介绍，下面的这些内容都是我直接从源代码的 `ruby.h` 头文件中出来的。列表如下：

  * T_NONE
  * T_NIL
  * T_OBJECT
  * T_CLASS
  * T_ICLASS
  * T_MODULE
  * T_FLOAT
  * T_STRING
  * T_REGEXP
  * T_ARRAY
  * T_HASH
  * T_STRUCT
  * T_BIGNUM
  * T_FILE
  * T_FIXNUM
  * T_TRUE
  * T_FALSE
  * T_DATA
  * T_MATCH
  * T_SYMBOL
  * T_RATIONAL
  * T_COMPLEX
  * T_UNDEF
  * T_NODE
  * T_ZOMBIE
  * T_MASK

同时C的api也提供了一些方法用于检测对象的类型：

  * int TYPE(obj)  返回obj对象所属的内建类型；
  * void Check_Type(VALUE obj, int type)  检查对象obj是否属于type类型，如果不属于则会抛出异常；
  * char* rb_obj_classname(obj)  返回对象obj所属的类的名字；

接下来再介绍一下几种常用数据类型及其相关的api。

## Numbers

在Ruby中有两种数字类型：Fixnum和Bignum。
与数字相关的一些api有：

| 接口 | 描述 |
| -- | -- |
| INT2FIX(i)  | 将int类型转化为Fixnum对象 |
| INT2NUM(i)  | 将int类型转化为Fixnum对象或者Bignum对象 |
| LONG2FIX(i) | 与INT2FIX相似    |
| LONG2NUM(i) | 与INT2NUM相似    |
| FIX2INT(o)  | 将Fixnum对象转化为int类型 |
| FIX2LONG(o) | 将Fixnum对象转化为long类型 |
| NUM2INT(o)  | 将Fixnum对象或者Bignum对象转化为int类型 |
| NUM2LONG(o) | 将Fixnum对象或者Bignum对象转化为long类型 |


## Strings

与C字符串和Ruby字符串相关的一些api有：

| 接口 | 描述 |
| -- | -- |
| VALUE rb_str_new(const char \*ptr, long len)  | 将ptr指针指向的长度为len的字符串转化为Ruby的字符串对象   |
| VALUE rb_str_new_cstr(const char \*ptr)       | 将ptr指针指向的字符串转化为Ruby的字符串对象   |
| char \*rb_string_value_cstr(volatile VALUE\*) | 将Ruby的字符串对象转化为C的字符串   |
| char \*StringValueCStr(VALUE)                 | rb_string_value_cstr对应的宏 |


## Arrays

与数组相关的一些api：

| 接口 | 描述 |
| -- | -- |
| VALUE rb_ary_new(void)                            | 创建新数组   |
| VALUE rb_ary_push(VALUE ary, VALUE item)          | 在数组结尾插入新的值  |
| void rb_ary_store(VALUE ary, long idx, VALUE val) | 在idx的位置插入新的值 |


## Hashes

与散列表相关的一些api：

| 接口 | 描述 |
| -- | -- |
| VALUE rb_hash_new()                                                     | 新建一个散列表                                                                                                                                                                                   |
| VALUE rb_hash_aset(VALUE hash, VALUE key, VALUE val)                    | 设置键值                                                                                                                                                                                                                 |
| VALUE rb_hash_aref(VALUE hash, VALUE key)                               | 获取一个键的值                                                                                                                                                                                   |
| void rb_hash_foreach(VALUE hash, int (\*callback)(ANYARGS), VALUE farg) | 遍历散列表，callback格式为 (\*callback)(VALUE key, VALUE val, VALUE in)。同时它的返回值为 ST_CONTINUE 则表示正常操作；为 ST_STOP 则停止遍历；为 ST_DELETE 则删除当前的键值；为 ST_CHECK 则检查在此操作过程中该散列表是否已被修改，如果被修改则停止遍历。 |

## Blocks && Callbacks

代码块：

| 接口 | 描述 |
| -- | -- |
| int rb_block_given_p(void)                         | 如果传递了代码块则返回1, 否则返回0 |
| VALUE rb_yield(VALUE);                             | Ruby中的yield功能 |
| VALUE rb_yield_values(int n, ...);                 | 同上               |
| VALUE rb_yield_values2(int n, const VALUE \*argv); | 同上               |

Proc(lambda)：

| 接口 | 描述 |
| -- | -- |
| VALUE rb_proc_new( VALUE (\*func)(ANYARGS), VALUE val) | 创建Proc对象 |
| VALUE rb_proc_call(VALUE self, VALUE args)             | 调用Proc对象 |

-----------

以上是列出了一些常用的api接口，这些内容在Ruby的文档中也没有介绍，基本上都是在 `ruby.h` 文件和 `intern.h` 文件中找到的。这里再发下牢骚，感觉还是Python的文档详细啊。


关于这些api的例子可以参考本章的示例。
