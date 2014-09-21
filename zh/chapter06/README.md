# SWIG && FFI

在开头有说过直接使用C来编写Ruby扩展的方法可能显得略有些过时了，因为还有其他更加方便的方法可以让Ruby调用C的库。本章就简单的介绍一下另外两个工具——swig和ffi。


## SWIG

SWIG是一个开发工具，能够将C、C++与多种语言进行连接。如： PHP、Python、Perl、Ruby等。

使用SWIG的好处是只需要写一份代码就可以实现在多种语言中调用；而缺点是需要学习SWIG自己的编程语法。

以下是摘自官网的一个例子。

首先下载安装SWIG: http://www.swig.org/download.html

然后新建文件 `example.c`：

```C
/* File : example.c */
#include <time.h>
double My_variable = 3.0;

int fact(int n) {
    if (n <= 1) return 1;
    else return n*fact(n-1);
}

int my_mod(int x, int y) {
    return (x%y);
}

char *get_time()
{
    time_t ltime;
    time(&ltime);
    return ctime(&ltime);
}
```

`example.i`：

```
/* example.i */
%module example
%{
/* Put header files here or function declarations like below */
extern double My_variable;
extern int fact(int n);
extern int my_mod(int x, int y);
extern char *get_time();
%}

extern double My_variable;
extern int fact(int n);
extern int my_mod(int x, int y);
extern char *get_time();
```

程序编写完成之后，如果是要编译成 Python 扩展则执行命令：

```
swig -python example.i
gcc -c example.c example_wrap.c -I/usr/include/python2.7
ld -shared example.o example_wrap.o -o _example.so
```

如果要编译在 Ruby 扩展则执行命令：

```
swig -ruby example.i
gcc -c example.c example_wrap.c -I/usr/include/ruby-2.1.0/ -I/usr/include/ruby-2.1.0/i686-linux
ld -shared example.o example_wrap.o -o example.so
```

最后通过一个程序来测试一下结果：

```Ruby
require './example.so'

puts Example.fact(5)
puts Example.my_mod(7, 3)
puts Example.get_time()
```

## FFI

FFI是一个可以直接加载动态链接库的工具。使用 Ruby-FFI 就可以很方便的调用库的方法。

同样的，以下也是摘自官方的例子。首先安装 ffi：

```
[sudo] gem install ffi
```

或者下载源代码进行安装： https://github.com/ffi/ffi

然后运行一个测试程序：

```Ruby
require 'ffi'

module MyLib
  extend FFI::Library
  ffi_lib 'c'
  attach_function :puts, [ :string ], :int
end

MyLib.puts 'Hello, World using libc!'
```

这个例子是在 Ruby 中直接调用 libc 库的 `puts` 方法。结果应该是可以看到输出字符串： "Hello, World using libc!"
