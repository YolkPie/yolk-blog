---
title: node-schedule 简介
date: 2022-03-31 20:10:00
tags:
- Node.js
cover: http://img.nodejs.cn/logo.svg
---

# 

用Node.js写代码时，可以使用node-schedule来实现定时任务的功能，比如定时获取数据、定时发送报警信息等。node-schedule 是 Node.js 中专门处理定时任务的模块，可以根据配置，按照指定时间执行任务。需要注意的是 node-schedule 是基于时间而不是基于时间间隔的，比如 node-schedule 支持在每小时的0、20、40分钟执行，但是如果你想要从现在开始每二十分钟执行一次的话，建议考虑别的模块。

node-schedule 利用 timer 和 events 管理任务，对所有事件进行排序后，会计算当前时间和最近一个事件发生时间的时间间隔，然后调用setTimeOut 设置回调。总的来说分两种事件，一种是一次性的，一种是周期性的，一次性任务调用完就结束，周期性的会不断地循环调用，当一个周期性事件被调用后，会根据周期生成下一个周期任务，并添加到任务列表中，重新排序。每个任务调用结束，都会计算并准备下一个任务。

## 安装

node 6+ 的版本都可支持。

``` 
    npm install node-schedule
```

## 创建

每个定时任务都是 Job 对象的实例，可以使用 scheduleJob 方法创建。

```
    const schedule = require('node-schedule')
    const job = schedule.scheduleJob('30 * * * * *', () => {
        console.log('每分钟的第30秒执行!')
    })
```

上面代码创建了一个定时任务，每次当秒数为 30 的时候就执行一次。scheduleJob 中第一个参数用于设置规则，支持 cron 表达式、Date 类型、 schedule.RecurrenceRule 等方式创建。下面简单介绍下各种方式的实现：

### cron 表达式

cron 表达式应用在 Unix 和类 Unix 操作系统之中，让脚本、任务定时进行周期性重复的执行。

```
*  *  *  *  *  *
┬ ┬ ┬ ┬ ┬ ┬ ┬
│ │ │ │ │ | └ year [optional]
│ │ │ │ │ └ day of week (0 - 7) (0 or 7 is Sun)
│ │ │ │ └───── month (1 - 12)
│ │ │ └────────── day of month (1 - 31)
│ │ └─────────────── hour (0 - 23)
│ └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, OPTIONAL)
```

cron 表达式共 7 位，最后一位可选，可以不写，至少 6 位（对日期英文缩写、特殊字符大小写不敏感），从左到右各位置分别是：

| 位置| 意义 | 取值              | 支持的符号         |
| ---| ----|  ----             | ----             |
| 1  | 秒 | 0-59               | , - * /           |
| 2  | 分 | 0-59               | , - * /           |
| 3  | 时 | 0-59               | , - * /           |
| 4  | 日 | 1-31               | , - * ? / L W C   |
| 5  | 月 | 1-12 或 JAN - DEC  | , - * /           |
| 6  | 周 | 1-7 或 MON - SAT   | , - * ? / L W C   |
| 7  | 年 | 空或 1970-2099      | , - * /           |

|符号| 名称     | 功能                                
|---| ---     |  ----             
| * | 星号	   | 表示重复对应位置上的周期，比如在第四位上表示每日
| ,	| 逗号	   | 代表一个列表值，表示多个指定时间，如周位上SAT,SUN表示每周六周日
| ?	| 问号	   | 无意义，占位符，只能在日、周位上
| -	| 减号	   | 表示一个范围，如时位上 20-22表示 20、21、22点
| /	| 斜杠	   | a/b 可以表示以 a 为起点步长为 b 的时间序列，如日位上10/10表示10日20日30日
| L	| Last	  | 月份最后一天或星期六，周位上 6L 表示月份的最后一个周五
| W	| Weekday |	后边最近的工作日，3W 3日如是周五，则在6日（周一）执行
| #	| 井号	   | a#b 表示当月第 b 个星期 a，如 6#1 当月第一个星期五
| C	| Calendar|	关联的“日历”的计算结果

注意：
- 周位上给定值后，在日位上要用 ?
- “L” 和 “W” 可在日位中联合使用，LW 表示这个月最后一周的工作日
- “W” 字符指定的最近工作日是不能跨月
- W 字符串只能指定单一日期，而不能指定日期范围
- 日位建议最大值为 28 ，因为 2 月有时候是 28 天

上面是 cron 表达式的规则，需要注意的是 node-schedule 不支持 “W” 和 “L“ 。

```
    const schedule = require('node-schedule')
    // 每周五的下午六点
    const job = schedule.scheduleJob('0 0 18 ? * FRI *', () => {
        console.log('这周终于结束了!')
    })
```

### Date 对象

node-schedule 第一个参数还支持 Date 类型，如下：

```
    const schedule = require('node-schedule')
    const date = new Date(2024, 11, 31, 0, 0, 0)
    // 2024 年 12 月 31 号 0 点
    const job = schedule.scheduleJob(date, () => {
        console.log('2024年的最后一天了!')
    })
```

### schedule.RecurrenceRule

node-schedule 还可以通过 schedule.RecurrenceRule 方法创建规则：

```
    const schedule = require('node-schedule')
    const rule = new schedule.RecurrenceRule()
    rule.year = 2024
    // rule.dayOfWeek = 2
    rule.month = 11
    rule.dayOfMonth = 31
    rule.hour = 0
    rule.minute = 0
    rule.second = 0
    // 2024 年 12 月 31 号 0 点
    const job = schedule.scheduleJob(date, () => {
        console.log('2024年的最后一天了!')
    })
```

- rule 支持数组，比如 rule.dayOfMonth = [1, 31] 表示每月的 1 号 和 31 号。
- rule 支持范围，比如 rule.dayOfMonth = [new schedule.Range(4, 6)] 表示每月的 4 号到 6 号。

|参数        | 取值    | 功能                                
|---        | ---     |  ----             
| year      | 	      | 表示年份
| dayOfWeek	| 0-6	  | 表示周几，从周日开始
| month	    | 0-11	  | 表示月份
| date	    | 1-31    | 表示一个月中的日期
| hour	    | 0-23	  | 表示小时
| minute	| 0-59	  | 表示分钟
| second	| 0-59    |	表示秒
| tz        | 时区	   | a#b 表示当月第 b 个星期 a，如 6#1 当月第一个星期五


## 设置开始和结束时间

scheduleJob 第一个参数传入对象时，还可以设置开始时间和结束时间（rule参数可按照上面的三种方式创建）：

```
    const startTime = new Date(Date.now() + 5000)
    const endTime = new Date(startTime.getTime() + 5000)
    const job = schedule.scheduleJob({ start: startTime, end: endTime, rule: '*/1 * * * * *' }, () => {
        console.log('Time for tea!')
    })

```

## 取消

可以使用 cancel 方法取消单个任务：

```
    const schedule = require('node-schedule')
    const job = ...
    job.cancel()
```

如果要取消所有的定时任务，可以使用 gracefulShutdown 方法, gracefulShutdown 返回的是 Promise：

```
    const schedule = require('node-schedule')
    const job1 = ...
    const job2 = ...
    schedule.gracefulShutdown(() => console.log('任务都取消了！'))
```







