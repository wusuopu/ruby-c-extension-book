# 定义函数

在上一节的例子中简单提到了使用`rb_define_method`来定义一个函数。在这一节来进行详细介绍。

首先，函数有两种类型：

  * 参数个数固定；
  * 参数个数可变。

然后，以下是`rb_define_method`的基本用法：

```C
rb_define_method(ClassObj, "name", c_function, num);
```

执行的结果是为ClassObj这个类定义一个名为`name`的实例方法，对应的C函数为c_function，接收num个参数。

  * 若num的值为正数，则表示需要传入num个参数；
  * 若num的值为负数，则表示该方法的参数个数是可变的；
    * num为-1,则传入的参数是一个C的数组；
    * num为-2,则传入的参数是一个Ruby的数组。


## 固定参数

对于参数个数固定的情况，它对应的`c_function`基本形式如下：

```C
VALUE c_function(VALUE self, VALUE arg1, VALUE arg2)
{
    // ...
}
```

其中`self`为该方法的调用者，其余的均是该方法的参数。

## 可变参数

可变参数又分为两种情况：传入的是C数组和传入的是Ruby数组。

### C数组

对于第一种情况，它对应的`c_function`基本形式如下：

```C
VALUE c_function(int argc, VALUE *argv, VALUE self)
{
    // ...
}
```

这种情况比较复杂。

  * `argc` 为传入参数的个数；
  * `argv` 为传入参数的集合；
  * `self` 为方法调用者。

对于这种情况不要直接对`argv`进行操作，而是使用`rb_scan_args`方法：

```C
rb_scan_args(argc, argv, format, ...);
```

`rb_scan_args`是通过`format`字符串指定的格式对`argv`进行解析，当参数不匹配时会抛出异常。

下面通过与之等效的ruby代码来介绍一下`format`的用法：

```ruby
def foo( arg = nil )
rb_scan_args(argc, argv, "01", &arg);
```

这是定义了一个方法，有0个必须参数和1个可选参数；

```ruby
def foo( &block )
rb_scan_args(argc, argv, "0&", &block);
```

这是定义了一个方法，需要一个block类型的参数；

```ruby
def foo( *args )
rb_scan_args(argc, argv, "0*", &args);
```

这是定义了一个方法，可传入任意个数的参数；

```ruby
def foo( arg, opt = nil, *args, &block )
rb_scan_args(argc, argv, "11*&", &arg, &opt, &args, &block);
```

最后这是对上面的整合。


### Ruby数组

这种情况比较简单，它对应的`c_function`形式如下：

```C
VALUE c_function(VALUE self, VALUE args)
{
    // ...
}
```

这里所有的参数都放在ruby类型的数组`args`中。
