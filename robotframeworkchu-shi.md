## 背景
RF这个自动化框架在刚开始学习自动化测试的时候我就有看过，不过当时被网络上坑爹的教程欺骗了，所有的网络教程写的都是用它的ride来编写脚本，编写全部通过表格来填充，编写效率低，局限性太大，当时也很懒，并没有去看官方文档，所以一直感觉这框架很无聊，不如直接去敲代码。

最近换工作，新工作要求使用RF来写自动化脚本，没办法，硬着头皮去看了一下官方文档，然后自己写了几个demo，发现完全不是以前自己认识的那么回事。

新接触框架，还是建议先看看[官方文档](http://robotframework.org/#documentation)。

## 结构
Robot Framework的结构其实就是通过代码中的关键字，映射到框架中，然后执行对应的方法，可以简单整理为如下的结构

![RF结构](http://7xsgl3.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720170421093445.png)

所以Robot Framework实际上就是执行Python的代码。

## 示例
我们来看一个简单的示例：

* 创建一个字典
* 给字典赋值
* 把字典打印出来
 
用Python的代码来实现非常简单。

```python
dict = {
    "a": "1",
    "b": "2",
    "c": "3"
}
print dict
```

我们来看Robot Framework的代码

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${dict}     create dictionary
    set to dictionary   ${dict}  a=1      b=2     c=3
    log  ${dict}
```

执行结果如下
![RF执行结果](http://7xsgl3.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%8720170421092501.png)


Robot Framework的代码实现起来确实有点麻烦，用Python直接写明显是简单粗暴的。

我们来看Robot Framework的底层实现方式，比如`set to dictionary`这个方法，在Python中代码是这样的：

```python
    def set_to_dictionary(self, dictionary, *key_value_pairs, **items):
        if len(key_value_pairs) % 2 != 0:
            raise ValueError("Adding data to a dictionary failed. There "
                             "should be even number of key-value-pairs.")
        for i in range(0, len(key_value_pairs), 2):
            dictionary[key_value_pairs[i]] = key_value_pairs[i+1]
        dictionary.update(items)
        return dictionary
```

封装后的方法是不区分大小写的，也就是说`set to dictionary`和`Set To Dictionary`和`SET TO DICTIONARY`是等效的，调用的都是同一个方法。

代码可以对应Robot Framework的实现方法，需要传入一个字典，然后是k-v对形式做拆分，变为一个一个单独的值，以列表的形式传入，再用for循环用步长为2进行迭代，按顺序生成一个字典然后返回。

## 关于Ride
这个东西貌似所有教程都在推荐，反正我是不喜欢用，写起来很麻烦，我还是习惯了用Pycharm来写，装一个**IntelliBot**的插件就可以高亮语法，也可以直接执行脚本。用Pycharm来写RF的方法请查看[这里](http://www.jianshu.com/p/5a97d28a596d)。

## 编写方式
官方文档推荐的编写方式就是用txt文本来编写。

>Robot Framework test data is defined in tabular format, using either hypertext markup language (HTML), tab-separated values (TSV), plain text, or reStructuredText (reST) formats. The details of these formats, as well as the main benefits and problems with them, are explained in the subsequent sections. Which format to use depends on the context, but the plain text format is recommended if there are no special needs.

最优一句说的很明白了，如果没有特殊的需求，推荐使用plain text格式来写，也就是txt格式。

不建议用ride的表格形式来填写，效率极其低下，而且我也有遇到过提示出错的情况。

## 中文编程
这点应该是看到最奇怪的点了，刚刚接手项目代码的时候，我一直以为中文是方法的说明，看的我一度在怀疑人生，结果和同事聊了一下发现，RF是支持中文关键字的，比如如下代码是可以正常执行的。

```
*** Settings ***
Library  Collections

*** Keywords ***
打印字典
    [Documentation]  传入字典参数，自动打印
    [Arguments]  ${dict}
    set to dictionary  ${dict}      a=1     b=2     c=3
    log  ${dict}

*** Test Cases ***
TestCase001
    ${dict_test}     create dictionary
    打印字典  ${dict_test}
```

执行结果如下

```
D:\Python27\Scripts\pybot.bat -d results test1.txt
==============================================================================
Test1                                                                         
==============================================================================
TestCase001                                                           | PASS |
------------------------------------------------------------------------------
Test1                                                                 | PASS |
1 critical test, 1 passed, 0 failed
1 test total, 1 passed, 0 failed
==============================================================================
Output:  C:\Users\Administrator\PycharmProjects\RobotFramework\Chapter1\results\output.xml
Log:     C:\Users\Administrator\PycharmProjects\RobotFramework\Chapter1\results\log.html
Report:  C:\Users\Administrator\PycharmProjects\RobotFramework\Chapter1\results\report.html
```

恩，我感觉回到了以前写易语言的时代，只要封装的好，也可以用中文来写Python了。

## 总结
Robot Framework是一个不错的框架，尤其是在封装好一定的库和或者关键字后，可以提供给不会写代码的同事来写自动化脚本。

规范化的东西都有局限性，必然会有对应的学习成本。直接用Python写确实是更灵活方便，但是框架的东西就非常有规范，只要懂了这个规范，所有人都能拿来就上手。