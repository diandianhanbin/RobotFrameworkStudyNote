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

```
*** Test Cases ***
test
    ${str1}     Catenate        Hello   World
    ${str2}     catenate        SEPARATOR=-     Hello   World
    ${str3}     catenate        SEPARATOR=      Hello   World
    log  ${str1}
    log  ${str2}
    log  ${str3}
```

执行结果


```
${str1}  ==>  Hello World	
${str2}  ==>  Hello-World
${str3}  ==>  HelloWorld
```

## Comment

这是一个非常有意思的方法，说有意思是因为他没有任何执行逻辑，只是在日志中打一个点。

源代码

```
@run_keyword_variant(resolve=0)
    def comment(self, *messages):
        pass
```

官方文档中是这么描述的

>This keyword does nothing with the arguments it receives, but as they
        are visible in the log, this keyword can be used to display simple
        messages. Given arguments are ignored so thoroughly that they can even
        contain non-existing variables

执行的结果是这样的

![](http://7xsgl3.com1.z0.glb.clouddn.com//1495790057.png )

我们可以在需要关注的数据上下都打上这个点，方便快速找到需要查看的日志信息。

## Continue To *

`Continue To *`这一系列的方法的源代码实现方案需要对`RobotFramework`的框架实现。所以这里只列了一些执行方法。

### Continue For Loop

代码实现

```
*** Test Cases ***
test
    :FOR    ${item}    IN RANGE   4
    \   run keyword if  ${item}==2      continue for loop
    \    log  ${item}
```

### Continue For Loop If

代码实现

```
*** Test Cases ***
test
    :FOR    ${item}    IN RANGE   4
    \   continue for loop if  ${item}==2
    \    log  ${item}
```

## Convert To *

Convert To * 这一些列的方法都是最基础的数据转换方法，因此全部归到这里来写

### Convert To Interger

转换为`int`类型

示例代码

```

```

















