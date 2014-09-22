# RDoc && GEM

前面几章介绍了 ruby 扩展的开发方法，最后再来介绍两个有用的工具——RDoc和GEM。

## RDoc

RDoc是 ruby 的一款文档生成工具，它可以通过代码中特殊格式的注释来生成文档。

RDoc是随着 ruby 安装的，因此不需要再进行额外安装。

接下来准备使用第2章的例子，为其添加文档注释。

首先在 `rb_define_module` 语句之前添加模块注释：

```C
/*
    Document-module:  MyTest

    C扩展的 MyTest 模块.
*/
mTest = rb_define_module("MyTest");
```

然后在 `rb_define_class_under` 语句前添加类注释：

```C
/*
    Document-class:  MyTest

    MyTest 模块下的 MyTest 类.
*/
cTest = rb_define_class_under(mTest, "MyTest", rb_cObject);
```

接着就可以使用 `rdoc` 命令来生成文档了，生成 HTML 格式的文档：

```
$ rdoc --main README.rdoc -o doc/site/api README.rdoc ext/my_test.c
```

生成 ri 格式的文档：

```
$ rdoc --main README.rdoc -o doc/ri -f ri README.rdoc ext/my_test.c
```

为了方便起见，可以为 RDoc 添加一个 `rake` 任务。在 `Rakefile` 中添加如下内容：

```Ruby
require 'rdoc/task'

RDOC_FILES = FileList["README.rdoc", "ext/my_test.c"]

Rake::RDocTask.new do |rd|
    rd.main = "README.rdoc"
    rd.rdoc_dir = "doc/site/api"
    rd.rdoc_files.include(RDOC_FILES)
end

Rake::RDocTask.new(:ri) do |rd|
    rd.main = "README.rdoc"
    rd.rdoc_dir = "doc/ri"
    rd.generator = "ri"
    rd.rdoc_files.include(RDOC_FILES)
end
```

现在就可以分别使用 `rake rdoc` 命令和 `rake ri` 命令来生成 HTML 格式和 ri 格式的文档了；然后分别使用 `rake clobber_rdoc` 命令和 `rake clobber_ri` 命令来删除已生成的文档文件。


以上只是一个简单的介绍，关于在C扩展中使用RDoc，可以参考文档： http://docs.seattlerb.org/rdoc/RDoc/Parser/C.html

## GEM

GEM 是 ruby 的包管理工具，类似于 python 的 pip。我们扩展程序开发完成之后可以通过 GEM 打包并与其他人分享。

如果你的系统中没有安装 GEM 的话，可以通过这个地址下载安装： http://rubygems.org/pages/download

首先在项目根目录下创建一个新文件 `.gemspec` ：

```Ruby
SPEC = Gem::Specification.new do |s|
    s.name = "example"
    s.version = "1.0"
    s.date = '2014-08-21'
    s.summary = "C bindings"
    s.description = "C Bindings"
    s.authors = ["Long Changjin"]
    s.email = ["admin@longchangjin.cn"]
    s.files = ["Rakefile", "COPYING", "NEWS", "README.rdoc", "ext/my_test.c", "ext/app.rb", "ext/extconf.rb"]
    s.homepage = "http://www.xefan.com/"
    s.required_ruby_version = '>= 2.1.0'
    s.extensions = "ext/extconf.rb"
end
```

然后执行命令 `gem build .gemspec` 生成 gem 包。在当前目录下应该会生成一个名为 `example-1.0.gem` 的文件。如果想到与他人分享该 gem 包，可以执行命令 `gem push example-1.0.gem` 将该文件上传。


