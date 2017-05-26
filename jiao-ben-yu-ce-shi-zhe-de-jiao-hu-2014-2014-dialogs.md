## 背景

继续学习RobotFramework，这次看的是`Dialogs`库。

`Dialogs`库主要提供用户一个交互界面，在脚本的执行过程中暂停脚本，用户通过输入一些特殊的值来实现测试脚本的个性化。

>A test library providing dialogs for interacting with users.
Dialogs is Robot Framework's standard library that provides means for pausing the test execution and getting input from users. The dialogs are slightly different depending on whether tests are run on Python, IronPython or Jython but they provide the same functionality.
Long lines in the provided messages are wrapped automatically since Robot Framework 2.8. If you want to wrap lines manually, you can add newlines using the \n character sequence.

## Execute Manual Step

示例代码

```
test
    execute_manual_step     测试
```

以上代码会弹出如下对话框

![exec.png](http://o9iqz76t2.bkt.clouddn.com/exec.png)

点击PASS则会使这条校验通过，点击FAIL就会使案例失败

## Get Selection From User

示例代码

```
test
    ${value}        Get Selection From User     标题     测试2     测试3
    log     ${value}
```

以上代码会弹出如下对话框

![selections](http://o9iqz76t2.bkt.clouddn.com/selections.png)

第一个参数传入对话框的标题，也就是一些说明，之后的参数就是可选项。点击cancel会报失败，选择一个选项后点击OK，那么这个值就会给到`${value}`。

## Get Value From User

示例代码

```
test
    ${rst}      Get Value From User     测试GetValueFromUser
    log     ${rst}
```

以上代码会弹出如下对话框

![inputDialogs](http://o9iqz76t2.bkt.clouddn.com/inputDialogs.png)

方法的入参为对话框的描述内容，当在对话框输入信息之后，就会把值赋给`${rst}`。

## Pause Execution

示例代码

```
test
    pause execution     暂停测试
```

以上代码会弹出如下对话框

![Pause Execution](http://7xsgl3.com1.z0.glb.clouddn.com//1495456206.png )

传入的参数就是对话框显示的内容，来展示测试过程中某些信息，启动对话框时测试会被暂停，点击OK之后会继续执行剩下的用例

## 源代码解析

整个`Dialogs`用的是`Tkinter`这个自带的图形库。入口文件是`Dialogs.py`。从代码里面我们可以看到，`Dialogs`支持`jython`，`ipython`和普通的`python`。在代码最初的时候是这样引用的：

```python
if JYTHON:
    from .dialogs_jy import MessageDialog, PassFailDialog, InputDialog, SelectionDialog
elif IRONPYTHON:
    from .dialogs_ipy import MessageDialog, PassFailDialog, InputDialog, SelectionDialog
else:
    from .dialogs_py import MessageDialog, PassFailDialog, InputDialog, SelectionDialog
```

对外暴露的方法，也就是上面列的4个方法

```python
__all__ = ['execute_manual_step', 'get_value_from_user',
           'get_selection_from_user', 'pause_execution']
```

我用的是普通的`python`，所以我们看的是`dialogs_py.py`这个文件。

在`dialogs_py.py`这个文件中，主要结构是5个类，一个基础类`_TkDialog`，其他的都是对应的方法类`MessageDialog`，`InputDialog`，`SelectionDialog`和`PassFailDialog`，分别对应对外暴露的4个方法。

4个方法类都是一样的逻辑，基础基础类，然后创建一个示例，获取数据后返回结果。我们主要看基础类的各个方法。

```python
    def __init__(self, message, value=None, **extra):
        self._prevent_execution_with_timeouts()
        self._parent = self._get_parent()
        Toplevel.__init__(self, self._parent)
        self._initialize_dialog()
        self._create_body(message, value, **extra)
        self._create_buttons()
        self._result = None

    def _prevent_execution_with_timeouts(self):
        if 'linux' not in sys.platform \
                and currentThread().getName() != 'MainThread':
            raise RuntimeError('Dialogs library is not supported with '
                               'timeouts on Python on this platform.')

    def _get_parent(self):
        parent = Tk()
        parent.withdraw()
        return parent

    def _initialize_dialog(self):
        self.title('Robot Framework')
        self.grab_set()
        self.protocol("WM_DELETE_WINDOW", self._close)
        self.bind("<Escape>", self._close)
        self.minsize(250, 80)
        self.geometry("+%d+%d" % self._get_center_location())
        self._bring_to_front()
        
    def _create_body(self, message, value, **extra):
        frame = Frame(self)
        Label(frame, text=message, anchor=W, justify=LEFT, wraplength=800).pack(fill=BOTH)
        selector = self._create_selector(frame, value, **extra)
        if selector:
            selector.pack(fill=BOTH)
            selector.focus_set()
        frame.pack(padx=5, pady=5, expand=1, fill=BOTH)
        
    def _create_buttons(self):
        frame = Frame(self)
        self._create_button(frame, self._left_button,
                            self._left_button_clicked)
        self._create_button(frame, self._right_button,
                            self._right_button_clicked)
        frame.pack()
```

上面的代码首先做一个系统和线程的检查。检查通过之后就开始穿件一个`Tkinter`的实例。之后初始化`dialog`的一些元素，比如标题，大小等。然后再创建对话框的body部分，包括消息，选项等等内容。之后再创建按钮。按钮在类中就定义好了，一个左按钮，一个右按钮。然后空定义一个结果，之后其他方法调用获取结果后返回这里。

在初始化`dialog`元素的时候有一个超时处理的方法：

```python
    def grab_set(self, timeout=30):
        maxtime = time.time() + timeout
        while time.time() < maxtime:
            try:
                # Fails at least on Linux if mouse is hold down.
                return Toplevel.grab_set(self)
            except TclError:
                pass
        raise RuntimeError('Failed to open dialog in %s seconds. One possible '
                           'reason is holding down mouse button.' % timeout)
```

同时也在这里声明窗口的位置：

```python
    def _get_center_location(self):
        x = (self.winfo_screenwidth() - self.winfo_reqwidth()) // 2
        y = (self.winfo_screenheight() - self.winfo_reqheight()) // 2
        return x, y
```

再接下去就是一些用到的方法

```python
    def _left_button_clicked(self, event=None):
        if self._validate_value():
            self._result = self._get_value()
            self._close()

    def _validate_value(self):
        return True

    def _get_value(self):
        return None

    def _close(self, event=None):
        # self.destroy() is not enough on Linux
        self._parent.destroy()

    def _right_button_clicked(self, event=None):
        self._result = self._get_right_button_value()
        self._close()

    def _get_right_button_value(self):
        return None

    def show(self):
        self.wait_window(self)
        return self._result
```

比如点击左键，就是获取结果后关闭界面，点击右键，就是关闭界面等等。

## 应用场景

在实际使用的过程中，可以将这个方法定义在业务流的最开始，把业务流关键的内容全部抽象出来，通过一个选择框在最开始的时候做一个开关，就可以实现一个代码执行多个业务流。

同时，也可以做调试用，在代码执行过程中暂停，把关键信息打印出来，便于代码调试