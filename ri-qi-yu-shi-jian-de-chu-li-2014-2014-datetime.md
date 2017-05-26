## 背景
继续学习RobotFramework框架，这次看的是DateTime库。

[DateTime](http://robotframework.org/robotframework/latest/libraries/DateTime.html)库是RobotFramework操作时间的一个库，可以进行很多时间方面的操作。

>DateTime is a Robot Framework standard library that supports creating and converting date and time values (e.g. Get Current Date, Convert Time), as well as doing simple calculations with them (e.g. Subtract Time From Date, Add Time To Time). It supports dates and times in various formats, and can also be used by other libraries programmatically.

## Get Current Date

示例代码

```
*** Settings ***
Library     DateTime

*** Test Cases ***
test1
    ${tm}       get_current_date
    log     ${tm
```

执行结果

```
11:08:49.158	INFO	2017-05-01 11:08:49.157
```

源代码

```python
def get_current_date(time_zone='local', increment=0,
                     result_format='timestamp', exclude_millis=False):
    if time_zone.upper() == 'LOCAL':
        dt = datetime.now()
    elif time_zone.upper() == 'UTC':
        dt = datetime.utcnow()
    else:
        raise ValueError("Unsupported timezone '%s'." % time_zone)
    date = Date(dt) + Time(increment)
    return date.convert(result_format, millis=is_falsy(exclude_millis))
```

说明

这个方法可以不传入参数，默认使用的是当地时间，如果第一个参数传入`utc`不区分大小写，则会使用`utc`时间。如果需要一个时间偏移量，则可以在第二个参数中传入时间偏移，传入的参数可以是一个数字或者str的数字，单位为秒，比如如下代码，执行的结果就会相差1分钟

```
test1
    ${tmdl}       get_current_date    local       60
    ${tm}       get_current_date
    log     ${tm}
    log     ${tmdl}
    
Result

KEYWORD BuiltIn . Log ${tm}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170501 11:22:13.600 / 20170501 11:22:13.600 / 00:00:00.000
11:22:13.600	INFO	2017-05-01 11:22:13.600	
00:00:00.001KEYWORD BuiltIn . Log ${tmdl}
Documentation:	
Logs the given message with the given level.
Start / End / Elapsed:	20170501 11:22:13.600 / 20170501 11:22:13.601 / 00:00:00.001
11:22:13.601	INFO	2017-05-01 11:23:13.599
```

当然，也可以传入一个`timedelta`,会通过如下代码解析成秒数进行处理。

```python
class Time(object):

    def __init__(self, time):
        self.seconds = float(self._convert_time_to_seconds(time))

    def _convert_time_to_seconds(self, time):
        if isinstance(time, timedelta):
            # timedelta.total_seconds() is new in Python 2.7
            return (time.days * 24 * 60 * 60 +
                    time.seconds +
                    time.microseconds / 1e6)
        return timestr_to_secs(time, round_to=None)
```

第三个参数为格式化参数，从代码中可以看到，默认传入的是`timestamp`，而这里的解析源代码如下

```python
def convert(self, format, millis=True):
        dt = self.datetime
        if not millis:
            secs = 1 if dt.microsecond >= 5e5 else 0
            dt = dt.replace(microsecond=0) + timedelta(seconds=secs)
        if '%' in format:
            return self._convert_to_custom_timestamp(dt, format)
        format = format.lower()
        if format == 'timestamp':
            return self._convert_to_timestamp(dt, millis)
        if format == 'datetime':
            return dt
        if format == 'epoch':
            return self._convert_to_epoch(dt)
        raise ValueError("Unknown format '%s'." % format)
```

所以，可以传入`timestamp`、`datetime`、`epoch`或者是我们正常格式化时间的`%Y%m%d`

第四个参数传入的是布尔值，是否需要毫秒级的数据，默认是False，也就是有毫秒的，如果不需要的话，传入`${true}`即可

## Convert Date

示例代码

```
*** Settings ***
Library     DateTime

*** Test Cases ***
test1
    ${tm}       convert_date        20170501
    log     ${tm}
```

执行结果


```
11:58:27.558	INFO	2017-05-01 00:00:00.000
```

源代码

```python
def convert_date(date, result_format='timestamp', exclude_millis=False,
                 date_format=None):
    return Date(date, date_format).convert(result_format,
                                           millis=is_falsy(exclude_millis))
                                           
def convert(self, format, millis=True):
        dt = self.datetime
        if not millis:
            secs = 1 if dt.microsecond >= 5e5 else 0
            dt = dt.replace(microsecond=0) + timedelta(seconds=secs)
        if '%' in format:
            return self._convert_to_custom_timestamp(dt, format)
        format = format.lower()
        if format == 'timestamp':
            return self._convert_to_timestamp(dt, millis)
        if format == 'datetime':
            return dt
        if format == 'epoch':
            return self._convert_to_epoch(dt)
        raise ValueError("Unknown format '%s'." % format)
```

说明

该方法就是一个时间格式转换的方法，在上一个方法`Get Current Date`中获取的结果如果需要其他格式，就可以用这个方法进行转换，参数传入与上一个方法一样。

## Convert Time

示例代码

```
*** Settings ***
Library     DateTime

*** Test Cases ***
test1
    ${tm}       convert_time    10      timedelta
    log     ${tm}
```

执行结果

```
12:06:08.990	INFO	0:00:10
```

源代码

```python
def convert_time(time, result_format='number', exclude_millis=False):
    return Time(time).convert(result_format, millis=is_falsy(exclude_millis))
    
class Time(object):

    def __init__(self, time):
        self.seconds = float(self._convert_time_to_seconds(time))
    
    def convert(self, format, millis=True):
        try:
            result_converter = getattr(self, '_convert_to_%s' % format.lower())
        except AttributeError:
            raise ValueError("Unknown format '%s'." % format)
        seconds = self.seconds if millis else float(roundup(self.seconds))
        return result_converter(seconds, millis)

    def _convert_to_number(self, seconds, millis=True):
        return seconds

    def _convert_to_verbose(self, seconds, millis=True):
        return secs_to_timestr(seconds)

    def _convert_to_compact(self, seconds, millis=True):
        return secs_to_timestr(seconds, compact=True)

    def _convert_to_timer(self, seconds, millis=True):
        return elapsed_time_to_string(seconds * 1000, include_millis=millis)

    def _convert_to_timedelta(self, seconds, millis=True):
        return timedelta(seconds=seconds)
```

说明

第一个参数是必传的，会在初始化`Time`类的时候转换成秒。源代码中的`convert`用了一个反射的方法，所以默认调用的是`_convert_to_number`会直接把传入的第一个参数转换成秒，然后返回。我写的示例代码填的是`timedelta`，会转换成时间格式。

## Subtract Date From Date

示例代码

```
test
    ${time}     Subtract Date From Date         2014-05-28 12:05:52     2014-05-28 12:05:10
    log     ${time}
```

执行结果

```
	INFO	42.0
```

源代码

```python
def subtract_date_from_date(date1, date2, result_format='number',
                            exclude_millis=False, date1_format=None,
                            date2_format=None):
    time = Date(date1, date1_format) - Date(date2, date2_format)
    return time.convert(result_format, millis=is_falsy(exclude_millis))
    
class Date(object):

    def __init__(self, date, input_format=None):
        self.seconds = self._convert_date_to_seconds(date, input_format)
        
    def _convert_date_to_seconds(self, date, input_format):
        if is_string(date):
            return self._string_to_epoch(date, input_format)
        elif isinstance(date, datetime):
            return self._mktime_with_millis(date)
        elif is_number(date):
            return float(date)
        raise ValueError("Unsupported input '%s'." % date)
```

说明

方法是获取两个日期之间的差值，最终结果单位是秒，从示例代码中可以看得出来。`Date`类会将传入的`date1`和`date2`转换成默认的格式，然后做一次减法。最后得到的日期结果，通过`time.convert`方法转成秒

## Add Time To Date

示例代码

```
test
    ${time}     add time to date    2014-05-28 12:05:03.111     7 days
    log     ${time}
```

执行结果

```
INFO	2014-06-04 12:05:03.111
```

示例代码

```python
def add_time_to_date(date, time, result_format='timestamp',
                     exclude_millis=False, date_format=None):
    date = Date(date, date_format) + Time(time)
    return date.convert(result_format, millis=is_falsy(exclude_millis))
```

说明

传入的参数`7 days`表示7天，这里的时间类型用的是`timedelta`类，支持以下的单位`days, seconds, minutes, hours`，如果不传，默认是`seconds`

## Subtract Time From Date

示例代码

```
test
    ${time}     subtract time from date    2014-05-28 12:05:03.111     7 days
    log     ${time}
```

执行结果

```
INFO	2014-05-21 12:05:03.111
```

源代码

```python
def subtract_time_from_date(date, time, result_format='timestamp',
                            exclude_millis=False, date_format=None):
    date = Date(date, date_format) - Time(time)
    return date.convert(result_format, millis=is_falsy(exclude_millis))
```

## Add Time To Time

示例代码

```
test
    ${time}     add time to time    1 days     7 days
    log     ${time}
```

执行结果

```
INFO	691200.0
```

源代码

```python
def add_time_to_time(time1, time2, result_format='number',
                     exclude_millis=False):
    time = Time(time1) + Time(time2)
    return time.convert(result_format, millis=is_falsy(exclude_millis))
```

## Subtract Time From Time

示例代码

```
test
    ${time}     subtract time from time    1 days     7 days
    log     ${time}
```

执行结果

```
INFO	-518400.0
```

源代码

```python
def subtract_time_from_time(time1, time2, result_format='number',
                            exclude_millis=False):
    time = Time(time1) - Time(time2)
    return time.convert(result_format, millis=is_falsy(exclude_millis))
```


## 总结

`DateTime`库主要的操作方式就只有`Date`和`Time`这两个类，只要搞懂这两个类的处理逻辑，就能理解`DateTime`这个库的操作方式