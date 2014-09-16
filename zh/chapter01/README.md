# 一个简单的例子

## 目录结构

首先介绍一下ruby项目的代码目录结构。通常情况下一个ruby扩展项目的目录结构如下：

```

NEWS
Rakefile
README.rdoc
doc/
ext/
```

`COPYING`为版权信息；`NEWS`包含了发行信息；`Rakefile`定义了rake任务；`README.rdoc`包含了用于生成RDoc文档的头部信息；`doc`目录下为该项目的文档；`ext`目录下为扩展程序的源代码以及`extconf.rb`文件。以上仅为参考，实际中还需根据自己的情况新增或者减少一些文件。

## MKMF

MKMF是ruby扩展构建系统的一部分，它用于生成编译C程序所需的头文件和Makefile文件。通常该脚本的文件名为： `extconf.rb`，这个有点类似于python中的`setup.py`脚本。以下是一个简单的例子：

```ruby
require 'mkmf'

RbConfig::MAKEFILE_CONFIG['CC'] = ENV['CC'] if ENV['CC']

extension_name = 'example'

unless pkg_config('library')
    raise "library not found"
end

have_func('some_function', 'library/lib.h')
have_type('some_type', 'library/lib.h')

create_header
create_makefile(extension_name)
```

第1行导入了mkmf模块；

第5行定义了该扩展模块的名称；

第7到9行调用pkg-config检查所需的库是否存在；

第11行调用`have_func`方法检查在对应的库中`some_function`方法是否已定义，如果存在则会在生成的`extconf.h`文件中定义一个名为`HAVE_`的宏；

第12行调用`have_type`方法检查在对应的库中`some_type`结构体是否已定义，如果存在则会在生成的`extconf.h`文件中定义一个名为`HAVE_TYPE_`的宏；

第14行创建`extconf.h`头文件；

第15行创建`Makefile`文件。

`extconf.rb`脚本编写完成之后，可以执行如下命令使用：

```
$ cd ext
$ ruby extconf.rb
$ make
```


## Rakefile

extconf 脚本创建完了，接下来是创建`Rakefile`。顾名思义Rakefile就是ruby的Makefile，即就是用ruby来编写Makefile的功能。以下是一个简单的例子：

```ruby
require 'rake/clean'

EXT_CONF = 'ext/extconf.rb'
MAKEFILE = 'ext/Makefile'
MODULE = 'ext/example.so'
SRC = Dir.glob('ext/*.c')
SRC << MAKEFILE

CLEAN.include [ 'ext/*.o', 'ext/depend', MODULE ]
CLOBBER.include [ 'config.save', 'ext/mkmf.log', 'ext/extconf.h', MAKEFILE ]

file MAKEFILE => EXT_CONF do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
        unless sh "ruby #{File::basename(EXT_CONF)}"
            $stderr.puts "Failed to run extconf"
            break
        end
    end
end
file MODULE => SRC do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
        unless sh "make"
            $stderr.puts "make failed"
            break
        end
    end
end
desc "Build the native library"
task :build => MODULE
```

第9行和第10行分别设置了要删除的文件的列表。`CLEAN`变量中定义的文件列表会在`rake clean`命令中被删除；`CLOBBER`变量中定义的文件列表会在`rake clobber`命令中被删除。

从第12行到29行定义了`build`任务，用于生成扩展模块。

## 程序示例

接下来通过一个简单的例子来介绍C扩展的编写。创建一个新文件 `my_test.c` 内容如下：

```C
#include "ruby.h"

static VALUE mTest;
static VALUE cTest;

static VALUE t_init(VALUE self)
{
    printf("\nCreate a MyTest instance.\n");

    return self;
}

void Init_my_test()
{
    mTest = rb_define_module("MyTest");
    cTest = rb_define_class_under(mTest, "MyTest", rb_cObject);
    rb_define_method(cTest, "initialize", t_init, 0);
}
```

第1行将ruby的头文件导入进来，以便能使用ruby的api。

在该程序代码中定义一个特殊的函数`Init_my_test`，它是该模块的初始化函数，在模块首次加载时执行。每个C扩展都需要定义一个名为`Init_<name>`的函数，这与Python的C扩展相似。

在`Init_my_test` 函数内部使用`rb_define_module` 函数定义了一个名为`MyTest`的Module，然后再用`rb_define_class_under`函数为该Module定义了一个类`MyTest`。并且对应的初始化函数为`t_init`。

  * rb_define_module： 定义一个Module；
  * rb_define_class_under： 在一个Module内定义一个Class；
  * rb_define_method： 定义一个实例方法，所需的参数依次为类对象、方法名、对应的C函数以及参数个数；

*注意*：所有被Ruby调用的C方法都必须返回一个VALUE类型的变量。

以上这段代码等效于如下ruby代码：

```ruby
module MyTest
  class MyTest
    def initialize
      puts "\nCreate a MyTest instance."
    end
  end
end
```

程序写完之后运行命令： `rake build` 进行编译。

最后再通过一个小程序来进行测试：

```ruby
# app.rb
require "./my_test.so"
require "test/unit"

class TestTest < Test::Unit::TestCase
  def test_test
    t = MyTest::MyTest.new
    assert_equal(Object, MyTest::MyTest.superclass)
    assert_equal(MyTest::MyTest, t.class)
  end
end
```
