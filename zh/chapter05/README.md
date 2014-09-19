# 结构体封装 && 内存管理

前面介绍过在C扩展中使用 `rb_define_class_under` 定义自定义类。而且如果要定义一些变量的话也可以使用 `rb_iv_set` 定义实例变量，以及使用 `rb_define_variable` 或者 `rb_global_variable` 定义全局变量。通过这种方法可以实现 Ruby 与 C 共享数据。本章介绍另一种共享数据的方法——封装结构体。


## 结构体封装

在 Python 的 C 扩展中，定义一个自定义类就是通过定义一个结构体来模拟实现的。

在 Ruby 中定义了几个有用的宏可以方便的对结构体进行封装。

```C
VALUE Data_Wrap_Struct(VALUE class, void (*mark)(), void (*free)(), void *ptr);
VALUE Data_Make_Struct(VALUE class, c-type, void (*mark)(), void (*free)(), c-type *ptr);
Data_Get_Struct(VALUE obj,c-type,c-type *);
```

`Data_Wrap_Struct` 是将 C 的数据类型 `ptr` 进行封装，并返回一个 Ruby 类型的对象。该对象是对应的 C 类型为 `T_DATA` ，对应的 Ruby 类型为 `class` 。

`Data_Make_Struct` 首先分配内存空间，然后执行 `Data_Wrap_Struct` 操作。

`Data_Get_Struct` 获取原始数据的指针。


## 内存分配

如果需要在扩展程序中申请内存空间来存储内容，可以直接使用 C 的原生方法： `malloc` 、 `realloc` 、 `calloc` 。不过这样的话就需要记得手动释放申请的内存，以免出现内存泄漏。为了方便起见还是推荐使用 Ruby 定义的几个api:

  * ALLOC(type) 分配type类型大小的空间，并返回type类型的指针
  * ALLOC_N(type, num) 分配num个type类型大小的空间，并返回type类型的指针
  * REALLOC_N(var, type, num) 将var指向的空间重新分配为num个type类型大小的空间，并返回type类型的指针

这几个api与原生的C方法功能类似，只是内存的申请和释放都是由Ruby进行管理，而不是操作系统系统。

**注意：**使用以上方法申请的内存得使用 `xfree` 进行释放。


## 示例程序

接下来通过一个例子来进行讲解。首先声明一个结构体类型：

```C
typedef struct {
    int i;
} cMyStruct;
```

然后再定义一个类：

```C
cTest = rb_define_class("MyStruct", rb_cObject);
```

接着在类的 `new` 方法内部执行操作，将该结构体进行封装：

```C
cMyStruct *ptr = ALLOC(cMyStruct);
VALUE tdata = Data_Wrap_Struct(class, 0, t_free, ptr);
```

现在可以在其他地方访问该结构体：

```C
cMyStruct *ptr;
Data_Get_Struct(self, cMyStruct, ptr);
```

完成的例子代码可以从 https://github.com/wusuopu/ruby-c-extension-sample 获取到。
