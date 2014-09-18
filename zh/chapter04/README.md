# 异常

当程序执行出错时通常的做法是抛出一个异常，这个异常既可以是内建的异常类型也可以是自定义的异常类型。

## 内建异常类

内建的异常类型如下：

  * rb_eException;
  * rb_eStandardError;
  * rb_eSystemExit;
  * rb_eInterrupt;
  * rb_eSignal;
  * rb_eFatal;
  * rb_eArgError;
  * rb_eEOFError;
  * rb_eIndexError;
  * rb_eStopIteration;
  * rb_eKeyError;
  * rb_eRangeError;
  * rb_eIOError;
  * rb_eRuntimeError;
  * rb_eSecurityError;
  * rb_eSystemCallError;
  * rb_eThreadError;
  * rb_eTypeError;
  * rb_eZeroDivError;
  * rb_eNotImpError;
  * rb_eNoMemError;
  * rb_eNoMethodError;
  * rb_eFloatDomainError;
  * rb_eLocalJumpError;
  * rb_eSysStackError;
  * rb_eRegexpError;
  * rb_eEncodingError;
  * rb_eEncCompatError;
  * rb_eScriptError;
  * rb_eNameError;
  * rb_eSyntaxError;
  * rb_eLoadError;
  * rb_eMathDomainError;


## 抛出异常

通常自定义异常类型是 `rb_eException` 或者 `rb_eStandardError` 的子类。先使用  `rb_define_class` 定义一个异常类型，然后再抛出一个异常。

抛出异常的方法：

  * void rb_raise(error_class, error_string, ...) 这是比较常用的一个方法；
  * void rb_sys_fail(error_string)  根据errno抛出一个异常；

其中 `rb_raise(error_class, error_string, ...)` 的作用与 ruby 中的 `raise Error, string` 相同。


## 异常处理

在 Ruby 中可以使用 `rescue` 捕获一个异常，在C代码中与之对应的函数为：

  * rb_rescue(cb, cb_args, rescue_cb, rescue_args)
  * rb_ensure(cb, cb_args, ensure_cb, ensure_args)

`rb_rescue` 方法对应 ruby 的 `rescue`，它的四个参数依次是， `cb`： 要执行的操作，格式声明为 `VALUE cb(VALUE args)`； `cb_args`： 传递给 `cb` 函数的参数； `rescue_cb`： 出现异常时处理异常的回调函数，格式声明为 `VALUE rescue_cb(VALUE args, VALUE err)`； `rescue_args`： 传递给 `rescue_cb` 函数的参数。


转化为 ruby 的语法如下：

```ruby
begin
  cb(cb_args)
rescue
  rescue_cb(rescue_args)
end
```

`rb_ensure` 方法对应 ruby 的 `ensure`，与 `rb_rescue` 类似。 `ensure_cb` 格式声明为 `VALUE ensure_cb(VALUE args)`

转化为 ruby 的语法如下：

```ruby
begin
  cb(cb_args)
ensure
  ensure_cb(ensure_args)
end
```

## Throw/Catch

  * rb_catch(const char\*, VALUE(*)(ANYARGS), VALUE)
  * rb_throw(const char*, VALUE)

`rb_catch` 方法的三个参数分别为： 要catch的代码块名称； catch 的回调函数； 回调参数。
`rb_throw` 方法的两个参数分别为： 返回的catch代码块名称； 返回值。


关于异常的用法可以参考本章的示例。

