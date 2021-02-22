## time模块

基于C标准库<time.h>进行封装

**time结构体**

```v
pub struct Time {
pub:
	year   int
	month  int
	day    int
	hour   int
	minute int
	second int
	unix int
}
```

**time方法**

```v
//返回月份的字符串简称
pub fn (t Time) smonth() string 
//计算unix时间
pub fn (t &Time) calc_unix() int
//增加秒
pub fn (t Time) add_seconds(seconds int)
//增加天
pub fn (t Time) add_days(days int) Time
//某个时间距离当前时间有多久
pub fn (t Time) relative() string
...
```

**公共函数**

```v
//返回当前时间
pub fn now() Time
//返回时间
pub fn new_time(t Time) Time
//进程休眠,等待参数指定的时间长度:
//秒:  n*time.second
//毫秒: n*time.millisecond
//微秒: n*time.microsecond
//纳秒: n*time.nanosecond

//nanosecond  = Duration(1)
//microsecond = Duration(1000 * nanosecond)
//millisecond = Duration(1000 * microsecond)
//second      = Duration(1000 * millisecond)
//minute      = Duration(60 * second)
//hour        = Duration(60 * minute)
pub fn wait(duration Duration)
//判断是否闰年
pub fn is_leap_year(year int) bool
...
```

