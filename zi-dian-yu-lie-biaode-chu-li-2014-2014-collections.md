## 背景
继续学习RobotFramework，这次看的是Collections的源码。

Collections库是RobotFramework用来处理列表和字典的库，[官方文档](http://robotframework.org/robotframework/latest/libraries/Collections.html)是这么介绍的

>A test library providing keywords for handling lists and dictionaries.

## Append To List
示例代码

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${l1}       create list     a       b
      append to list  ${l1}     1       2
      log   ${l1}
```

执行结果：

```
KEYWORD BuiltIn . Log ${l1}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170422 19:11:52.624 / 20170422 19:11:52.624 / 00:00:00.000
19:11:52.624	INFO	[u'a', u'b', u'1', u'2']
```

源代码

```python
    def append_to_list(self, list_, *values):
        for value in values:
            list_.append(value)
```

## Combine List

示例代码

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${l1}       create list     a       b
    ${l2}       create list     1       2
    ${l}        combine lists  ${l1}        ${l2}
    log  ${l}
```

执行结果

```
KEYWORD ${l} = Collections . Combine Lists ${l1}, ${l2}
Documentation:	
Combines the given lists together and returns the result.
Start / End / Elapsed:	20170422 19:15:52.086 / 20170422 19:15:52.087 / 00:00:00.001
19:15:52.087	INFO	${l} = [u'a', u'b', u'1', u'2']
```

源代码

```python
    def combine_lists(self, *lists):
        ret = []
        for item in lists:
            ret.extend(item)
        return ret
```

说明

源代码中的extend方法和append方法是不一样的，append方法是追加一个元素，extend方法是追加一个列表的内容，如果用append方法，则结果会变成如下情况：

```python
l1 = [1,2,3]
l2 = [4,5,6]
l1.append(l2)  # ==>[1,2,3,[4,5,6]]
l1.extend(l2)  # ==>[1,2,3,,4,5,6]
```

所以合并列表的时候使用`extend`而不是`append`

## Count Values In List

计算某一个值在列表中重复的次数

示例

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${l1}       create list  1      2       3       4       3
    ${count}      count values in list          ${l1}     3
    log  ${count}
```

执行结果

```
KEYWORD ${int} = Collections . Count Values In List ${l1}, 3
Documentation:	
Returns the number of occurrences of the given value in list.
Start / End / Elapsed:	20170422 19:30:33.807 / 20170422 19:30:33.807 / 00:00:00.000
19:30:33.807	INFO	${int} = 2
```

源代码

```python
    def count_values_in_list(self, list_, value, start=0, end=None):
        return self.get_slice_from_list(list_, start, end).count(value)
        
    def get_slice_from_list(self, list_, start=0, end=None):
        start = self._index_to_int(start, True)
        if end is not None:
            end = self._index_to_int(end)
        return list_[start:end]
        
    def _index_to_int(self, index, empty_to_zero=False):
        if empty_to_zero and not index:
            return 0
        try:
            return int(index)
        except ValueError:
            raise ValueError("Cannot convert index '%s' to an integer." % index)
```

说明

这里方法是默认从第一位开始计算，如果有需要，可以输入一个区间，比如指定起始位置和结束位置，比如如下代码

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${l1}       create list  1      2       3       4       3       3
    ${count}      count values in list          ${l1}     3     3       5
    log  ${count}
```

执行结果就是1

## Dictionaries Should Be Equal

示例代码

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${dict1}     create dictionary      a=1     b=2
     ${dict2}   create dictionary       a=1     b=3
    dictionaries should be equal  ${dict1}      ${dict2}

```

执行结果

```
KEYWORD Collections . Dictionaries Should Be Equal ${dict1}, ${dict2}
Documentation:	
Fails if the given dictionaries are not equal.
Start / End / Elapsed:	20170422 19:49:36.865 / 20170422 19:49:36.866 / 00:00:00.001
19:49:36.866	FAIL	Following keys have different values:
Key b: 2 != 3
```

源代码

```python
    def dictionaries_should_be_equal(self, dict1, dict2, msg=None, values=True):
        keys = self._keys_should_be_equal(dict1, dict2, msg, values)
        self._key_values_should_be_equal(keys, dict1, dict2, msg, values)
        
    def _keys_should_be_equal(self, dict1, dict2, msg, values):
        keys1 = self.get_dictionary_keys(dict1)
        keys2 = self.get_dictionary_keys(dict2)
        miss1 = [unic(k) for k in keys2 if k not in dict1]
        miss2 = [unic(k) for k in keys1 if k not in dict2]
        error = []
        if miss1:
            error += ['Following keys missing from first dictionary: %s'
                      % ', '.join(miss1)]
        if miss2:
            error += ['Following keys missing from second dictionary: %s'
                      % ', '.join(miss2)]
        _verify_condition(not error, '\n'.join(error), msg, values)
        return keys1
        
    def _key_values_should_be_equal(self, keys, dict1, dict2, msg, values):
        diffs = list(self._yield_dict_diffs(keys, dict1, dict2))
        default = 'Following keys have different values:\n' + '\n'.join(diffs)
        _verify_condition(not diffs, default, msg, values)
        
    def _yield_dict_diffs(self, keys, dict1, dict2):
        for key in keys:
            try:
                assert_equal(dict1[key], dict2[key], msg='Key %s' % (key,))
            except AssertionError as err:
                yield unic(err)
```

说明

本来Python中比较两个字典是否相等只要用`==`就行了，这里为了能够精确的报错，遍历了所有的key和value来做比较。

## Dictionary Should Contain Item

示例代码
```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${dict1}     create dictionary      a=1     b=2
    dictionary should contain item  ${dict1}        a       1
```

结果
```
KEYWORD Collections . Dictionary Should Contain Item ${dict1}, a, 1
Documentation:	
An item of key``/``value must be found in a `dictionary`.
Start / End / Elapsed:	20170422 19:58:37.086 / 20170422 19:58:37.087 / 00:00:00.001
```

源代码
```python
    def dictionary_should_contain_item(self, dictionary, key, value, msg=None):
        self.dictionary_should_contain_key(dictionary, key, msg)
        actual, expected = unic(dictionary[key]), unic(value)
        default = "Value of dictionary key '%s' does not match: %s != %s" % (key, actual, expected)
        _verify_condition(actual == expected, default, msg)
        
    def dictionary_should_contain_key(self, dictionary, key, msg=None):
        default = "Dictionary does not contain key '%s'." % key
        _verify_condition(key in dictionary, default, msg)
```

说明

判断一个k-v是否在一个字典里，同样是通过遍历原有字典的方式来处理，只不过这里传值必须是按照示例代码中那样传递，不能传入a=1或者一个字典。

## Dictionary Should Contain Sub Dictionary

示例

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${dict1}     create dictionary      a=1     b=2
    ${dict2}        create dictionary  a=1
    Dictionary Should Contain Sub Dictionary    ${dict1}        ${dict2}
```

结果

```
KEYWORD Collections . Dictionary Should Contain Sub Dictionary ${dict1}, ${dict2}
Documentation:	
Fails unless all items in dict2 are found from dict1.
Start / End / Elapsed:	20170422 20:09:41.864 / 20170422 20:09:41.865 / 00:00:00.001
```

源代码

```python
    def dictionary_should_contain_sub_dictionary(self, dict1, dict2, msg=None,
                                                 values=True):
        keys = self.get_dictionary_keys(dict2)
        diffs = [unic(k) for k in keys if k not in dict1]
        default = "Following keys missing from first dictionary: %s" \
                  % ', '.join(diffs)
        _verify_condition(not diffs, default, msg, values)
        self._key_values_should_be_equal(keys, dict1, dict2, msg, values)
```

说明

上一个方法只能传入单一的k-v，使用的局限性比较大，这个方法就可以支持传入一个字典

## Get Dictionary Items

示例

```
*** Settings ***
Library  Collections

*** Test Cases ***
TestCase001
    ${dict1}     create dictionary      a=1     b=2
    ${dict2}        create dictionary  a=1
    ${dict3}        Get Dictionary Items    ${dict1}
```

执行结果

```
KEYWORD ${dict3} = Collections . Get Dictionary Items ${dict1}
Documentation:	
Returns items of the given dictionary.
Start / End / Elapsed:	20170424 00:39:49.766 / 20170424 00:39:49.766 / 00:00:00.000
00:39:49.766	INFO	${dict3} = [u'a', u'1', u'b', u'2']
```

源代码

```python
    def get_dictionary_items(self, dictionary):
        ret = []
        for key in self.get_dictionary_keys(dictionary):
            ret.extend((key, dictionary[key]))
        return ret
```


说明

这个方法就是把字典里面的成员以列表的形式返回出来，具体应用场景还不是很确定

## Get Dictionary Values

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${dict}     create dictionary   a=1     b=2
    ${rst}      get from dictionary     ${dict}     a
    log   ${rst}
```

执行结果

```
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 19:45:59.199 / 20170428 19:45:59.200 / 00:00:00.001
19:45:59.200	INFO	1
```

源代码

```python
def get_from_dictionary(self, dictionary, key):
        try:
            return dictionary[key]
        except KeyError:
            raise RuntimeError("Dictionary does not contain key '%s'." % key)
```

说明

源代码其实非常简单，传入一个字典和一个key，然后就返回这个key对应的值

## Get From List

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst}      create list  1      2       3
    ${new_lst}  get from list  ${lst}       0
    log     ${new_lst}
```

执行结果

```
KEYWORD BuiltIn . Log ${new_lst}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 19:50:42.533 / 20170428 19:50:42.533 / 00:00:00.000
19:50:42.533	INFO	1
```

源代码

```python
def get_from_list(self, list_, index):
        try:
            return list_[self._index_to_int(index)]
        except IndexError:
            self._index_error(list_, index)
```

说明

执行方法就是获取列表和索引，然后返回列表的索引，值得一提的是，不论你传入的值是str类型还是int类型，都会被`python`代码转成int类型，所以源代码中`${new_lst}  get from list  ${lst}       0`这样写,执行结果也是一样的。附上转换的源代码。

```python
def _index_to_int(self, index, empty_to_zero=False):
        if empty_to_zero and not index:
            return 0
        try:
            return int(index)
        except ValueError:
            raise ValueError("Cannot convert index '%s' to an integer." % index)
```

## Get Index From List
示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst}      create list  1      2       3       4       5       4
    ${rst}      get index from list  ${lst}       3
    log     ${rst}
```

执行结果

```
KEYWORD BuiltIn . Log ${rst}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 19:59:05.532 / 20170428 19:59:05.532 / 00:00:00.000
19:59:05.532	INFO	2
```

源代码

```python
def get_index_from_list(self, list_, value, start=0, end=None):
        if start == '':
            start = 0
        list_ = self.get_slice_from_list(list_, start, end)
        try:
            return int(start) + list_.index(value)
        except ValueError:
            return -1
```

说明
源代码中的`self.get_slice_from_list`方法是一个切片方法，在`Get Index From List`中还可以传入两个参数，`start`表示列表的开始索引，`end`表示列表结束的索引，如果没有给，默认就是整个列表来取索引结果。

## Get Match Count
示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${count}        get match count     aaabbcc      a
    log     ${count}
```

执行结果

```
KEYWORD BuiltIn . Log ${count}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 20:08:08.699 / 20170428 20:08:08.700 / 00:00:00.001
20:08:08.700	INFO	3
```

源代码

```python
def get_match_count(self, list, pattern, case_insensitive=False,
                        whitespace_insensitive=False):
        return len(self.get_matches(list, pattern, case_insensitive,
                                    whitespace_insensitive))
                                    
def _get_matches_in_iterable(iterable, pattern, case_insensitive=False,
                             whitespace_insensitive=False):
    if not is_string(pattern):
        raise TypeError("Pattern must be string, got '%s'." % type_name(pattern))
    regexp = False
    if pattern.startswith('regexp='):
        pattern = pattern[7:]
        regexp = True
    elif pattern.startswith('glob='):
        pattern = pattern[5:]
    matcher = Matcher(pattern,
                      caseless=is_truthy(case_insensitive),
                      spaceless=is_truthy(whitespace_insensitive),
                      regexp=regexp)
    return [string for string in iterable
            if is_string(string) and matcher.match(string)]
```

说明

这个方法是返回一个字符在字符串中重复的次数。

源代码中的第二个方法才是真正的方法，`Get Match Count`方法是非常强大的，可以做普通匹配，也可以做正则匹配，首先，传入的必须是字符串，否则报错，然后判定传入的条件是否带有正则，如果传入的条件是`regexp=`开头的，表示要使用正则匹配，否则就是普通查找。

第三个参数表示是否忽略大小写，第四个参数表示是否忽略空格。

## Get Matchs
此方法与上面一个方法调用的是一样的底层方法，请参考上面的内容。

## Get Slice From List

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst}      create list  1  2   3   4   5   6   7   8
    ${slice_list}   get slice from list  ${lst}     3   6
    log  ${slice_list}
```

执行结果

```
KEYWORD BuiltIn . Log ${slice_list}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 20:20:00.780 / 20170428 20:20:00.780 / 00:00:00.000
20:20:00.780	INFO	[u'4', u'5', u'6']
```

源代码

```python
def get_slice_from_list(self, list_, start=0, end=None):
        start = self._index_to_int(start, True)
        if end is not None:
            end = self._index_to_int(end)
        return list_[start:end]
```

说明

源代码还是比较容易看懂的，如果开始和结束不传值，那么就是返回原来的列表，否则就返回切片的列表。

## Insert into list

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst}      create list  1  2   3   4   5   6   7   8
    insert into list    ${lst}      0       a
    log  ${lst}
```

执行结果

```
KEYWORD BuiltIn . Log ${lst}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 20:27:02.736 / 20170428 20:27:02.737 / 00:00:00.001
20:27:02.737	INFO	[u'a', u'1', u'2', u'3', u'4', u'5', u'6', u'7', u'8']
```

源代码

```python
def insert_into_list(self, list_, index, value):
        list_.insert(self._index_to_int(index), value)
```

说明

调用的就是`Python`中列表的`insert`方法。如果传入的索引大于列表的长度，那么值会默认插入到列表的最后一位，如果传入的是负数，比如传入-1，那么就会在列表的倒数第二位加入该元素。

根据实现的`Python`代码可以知道，传入的索引是int或者str类型都可以，实现的时候会强制转为int类型。

关于插入索引大于列表长度的问题，可以参考以下代码

```python
Python 2.7.10 (default, Feb  6 2017, 23:53:20)
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.34)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> a = [1,2,3,4,5]
>>> a
[1, 2, 3, 4, 5]
>>> a.insert(9, 'a')
>>> a
[1, 2, 3, 4, 5, 'a']
```

## Keep In Dictionary

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${dict}     create dictionary  a=1  b=2     c=3     d=4
    keep in dictionary  ${dict}     a   b
    log  ${dict}
```

执行结果

```
KEYWORD BuiltIn . Log ${dict}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 20:47:47.528 / 20170428 20:47:47.529 / 00:00:00.001
20:47:47.529	INFO	{u'a': u'1', u'b': u'2'}
```

源代码

```python
def keep_in_dictionary(self, dictionary, *keys):
        remove_keys = [k for k in dictionary if k not in keys]
        self.remove_from_dictionary(dictionary, *remove_keys)
        
def remove_from_dictionary(self, dictionary, *keys):
        for key in keys:
            if key in dictionary:
                value = dictionary.pop(key)
                logger.info("Removed item with key '%s' and value '%s'." % (key, value))
            else:
                logger.info("Key '%s' not found." % key)   
```

说明

该方法是用来保留字典里指定的`key`，源代码也比较好理解，不过此处的注释有问题，

>Keeps the given keys in the dictionary and removes all other.
If the given key cannot be found from the dictionary, it is ignored.

**第二句是说明如果传入的`key`如果不在这个字典里，那么这个操作就会被忽略。但是实际上我们可以看到代码执行时并不是这样的，第一个方法会把字典中与传入的`keys`不匹配的`key`全部拿出来，调用第二个方法来全部从字典里移除，也就是说传入的key如果在字典里不存在，那么原字典就会变为一个空字典。**

实际上的执行结果就是这样

```
KEYWORD ${dict} = BuiltIn . Create Dictionary a=1, b=2, c=3, d=4
00:00:00.001KEYWORD Collections . Keep In Dictionary ${dict}, e
Documentation:	
Keeps the given keys in the dictionary and removes all other.
Start / End / Elapsed:	20170428 20:52:04.051 / 20170428 20:52:04.052 / 00:00:00.001
20:52:04.052	INFO	Removed item with key 'a' and value '1'.	
20:52:04.052	INFO	Removed item with key 'b' and value '2'.	
20:52:04.052	INFO	Removed item with key 'c' and value '3'.	
20:52:04.052	INFO	Removed item with key 'd' and value '4'.	
00:00:00.000KEYWORD BuiltIn . Log ${dict}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 20:52:04.052 / 20170428 20:52:04.052 / 00:00:00.000
20:52:04.052	INFO	{}
```

## List Should Contain Sub List

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst1}     create list  1      2       3       4
    ${lst2}     create list  2      3
    list should contain sub list  ${lst1}       ${lst2}
```

执行结果

```
KEYWORD Collections . List Should Contain Sub List ${lst1}, ${lst2}
Documentation:	
Fails if not all of the elements in list2 are found in list1.
```

源代码

```python
def list_should_contain_sub_list(self, list1, list2, msg=None, values=True):
        diffs = ', '.join(unic(item) for item in list2 if item not in list1)
        default = 'Following values were not found from first list: ' + diffs
        _verify_condition(not diffs, default, msg, values)
```

## List Should Not Contain Duplicates

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst1}     create list  1      2       3       4       2
    list should not contain duplicates      ${lst1}     2
```

执行结果

```
KEYWORD Collections . List Should Not Contain Duplicates ${lst1}, 2
Documentation:	
Fails if any element in the list is found from it more than once.
Start / End / Elapsed:	20170428 21:08:59.719 / 20170428 21:08:59.720 / 00:00:00.001
21:08:59.719	INFO	'2' found 2 times.	
21:08:59.720	FAIL	2
```

源代码

```python
def list_should_not_contain_duplicates(self, list_, msg=None):
        if not isinstance(list_, list):
            list_ = list(list_)
        dupes = []
        for item in list_:
            if item not in dupes:
                count = list_.count(item)
                if count > 1:
                    logger.info("'%s' found %d times." % (item, count))
                    dupes.append(item)
        if dupes:
            raise AssertionError(msg or
                                 '%s found multiple times.' % seq2str(dupes))
```

说明

该方法用于断言某个元素在列表中只会出现一次，如果出现多次则报错。

## Pop From Dictionary

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${dict}     create dictionary  a=1      b=2     c=3
    ${newdict}      pop from dictionary     ${dict}     a
    log   ${newdict}
    log     ${dict}
```

执行结果

```
KEYWORD BuiltIn . Log ${dict}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 21:21:05.523 / 20170428 21:21:05.523 / 00:00:00.000
21:21:05.523	INFO	{u'b': u'2', u'c': u'3'}
```

源代码

```python
def pop_from_dictionary(self, dictionary, key, default=NOT_SET):
        if default is NOT_SET:
            self.dictionary_should_contain_key(dictionary, key)
            return dictionary.pop(key)
        return dictionary.pop(key, default)
```

说明

这里的方法有一个可选参数`default`，是用来处理不存在的键值，如果pop的键值在字典里不存在，那么框架是会报错处理的，但是如果给了一个default，那么传入的键值不存在时，会返回`default`的值。

## Remove Duplicates

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst}      create list  1      2       3       4       1       2
    ${rst}      remove duplicates       ${lst}
```

执行结果

```
KEYWORD ${rst} = Collections . Remove Duplicates ${lst}
Documentation:	
Returns a list without duplicates based on the given list.
Start / End / Elapsed:	20170428 21:52:22.522 / 20170428 21:52:22.522 / 00:00:00.000
21:52:22.522	INFO	2 duplicates removed.	
21:52:22.522	INFO	${rst} = [u'1', u'2', u'3', u'4']
```

源代码

```python
def remove_duplicates(self, list_):
        ret = []
        for item in list_:
            if item not in ret:
                ret.append(item)
        removed = len(list_) - len(ret)
        logger.info('%d duplicate%s removed.' % (removed, plural_or_not(removed)))
        return ret
```

说明

该方法是一个除重方法，可以吧列表中重复的元素移除。

## Sort List

示例代码

```
*** Settings ***
Library     Collections

*** Test Cases ***
test1
    ${lst}      create list  1      2       3       4       1       2       and     dib
    sort list   ${lst}
    log  ${lst}
```

执行结果

```
KEYWORD BuiltIn . Log ${lst}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170428 22:13:57.288 / 20170428 22:13:57.288 / 00:00:00.000
22:13:57.288	INFO	[u'1', u'1', u'2', u'2', u'3', u'4', u'and', u'dib']
```

源代码

```python
def sort_list(self, list_):
        list_.sort()
```

说明

该方法调用了列表的`sort`方法，需要注意的是，如果原始列表还需要使用的话，最好先保留一份，否则原始数据就会被破坏。

>Note that the given list is changed and nothing is returned. Use`Copy List` first, if you need to keep also the original order.

## 总结
`Collections`库是一个非常简单的库，不过里面包含了大部分的字典和列表的操作，本文仅仅覆盖了一些关键字，还有一些重复的，或者一眼就能看出来怎么用的关键字我就没有写了。通过这些示例代码，基本上能够了解Robot Framework框架的使用方法，也能够对`Robot Framework`框架的编写方式和编写思想有了一定的认识。

