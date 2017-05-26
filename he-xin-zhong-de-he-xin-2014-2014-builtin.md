## 背景

这次看的是`Robot Framework`的关键性内建库`BuiltIn`。`BuiltIn`可以说是RF的基准库，包含了大部分的基准操作。是一个非常有必要了解的库。

## Call Method

这个方法说实话有点脱裤子放屁，至少我用起来是这么感觉的。首先，官方文档给的一些方法基本上没参考价值，完全没用，因为拿着那个Demo根本就跑不起来。

简单的使用方法如下

```
test.py

class Test(object):

	def ttt(self):
		print 'hello'


def create_test():
	return Test()

test.txt
*** Settings ***
Library     ./test.py

*** Test Cases ***
test
    ${Test}     create test
    call method     ${Test}     ttt
```

首先你要写一个py文件，里面有一个类，然后还要写一个返回类的方法，这个时候在RF中引用这个py文件，先获取这个返回类的方法，赋值之后如上面代码所示，`${Test}`这个变量就是一个`Object`。这个时候才能使用`Call Method`关键字来调用里面的`ttt`方法。

源代码

```python
    def call_method(self, object, method_name, *args, **kwargs):
        try:
            method = getattr(object, method_name)
        except AttributeError:
            raise RuntimeError("Object '%s' does not have method '%s'."
                               % (object, method_name))
        try:
            return method(*args, **kwargs)
        except:
            raise RuntimeError("Calling method '%s' failed: %s"
                               % (method_name, get_error_message()))
```

源代码的逻辑很简单，就是利用Python的`getattr`来获取这个方法的属性，如果没有就报错，有的话就执行这个方法并且返回。

## Catenate

这个方法等同于`Python`中的`join`方法。接受多个值的传入，其中第一个参数最关键，`SEPARATOR=*`是代码中用来判断分割的关键字，如果第一个参数传的是这个值，那么就会以等号后面的标记作为连接符。如果等号后面没有标记，那么就默认以4个空格作为分隔符。如果留空，那么就不会给分隔符，直接把所有的参数连接起来。

示例代码

```sdfsd

```














