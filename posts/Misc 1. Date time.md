# Date time

### ISO 8601 format

什么是 ISO 8601?

ISO 8601 是表示日期和时间的一种国际标准，完整格式: `YYYY-MM-DDTHH:mm:ss.SSS[UTC offset]`

为什么需要 ISO 8601?

主要有：
1. 消除日期的不确定性，e.g. `07/01/2023`, 在美国会被理解为 `2023-07-01`，在英国被理解为 `2023-01-07`
1. 解决跨世纪的问题，`1999` -> `2000`, 用 `YY` 表示会存在问题，而用 `YYYY` 就没有问题

参考: [What is ISO 8601 by MIT](http://web.mit.edu/jmorzins/www/iso8601/y2kiso.htm)

> 1. YYYY: Year, 4 位
> 1. MM: Month, 2 位, 01 - 12
> 1. DD: Day, 2 位, 01 - 31
> 1. T 或者 空格: date 和 time 的分隔符, T 意味着 Time 开始，可以用空格替代 T，但建议用 T
> 1. HH: Hour, 2 位, 00 - 23. 注意: ISO 8601 用的是小写 hh, 因为它是 24 小时制 (dayjs 中 大写 HH 表示 24 小时制, 小写 hh 表示 12 小时制，我采用 HH)
> 1. mm: Minute, 2 位, 00 - 59
> 1. ss: Second, 2 位, 00 - 59
> 1. SSS: milliSecond, 3 位, 000 - 999. 注意: ISO 8601 没有规定必须用 S (我采用 dayjs 的大写 S, 与小写 s 进行区分), 也没有规定必须 3 位 (我采用 EcmaScript 的 3 位)
> 1. UTC offset: 准确来说是 Time Zone Designator (TDZ) 时区标识, 可以不存在, 可以是 Z, 可以是 +hh:mm 或者 -hh:mm, 一共有 24 个时区: -11 ... -1 Z +1 ... +12

注意：以下几个 time 都是 `string` 值

- local time: UTC offset 不存在, 说的是**不同** time zone 下的**同一**时间点
  - e.g. `2023-07-01T02:03:04.567`
- UTC time: UTC offset 是 `0`, 用 `Z` 表示
  - e.g. `2023-07-01T02:03:04.567Z`
- UTC offset time
  - e.g. `2023-07-01T02:03:04.567+08:00`

### 原生 `Date` object

原生 `Date` object 内部存储的**永远**是 `UTC` 整数值 (精确到 3 位毫秒), 起始点是 `1970-01-01T00:00:00.000Z`. e.g. `new Date('1970-01-01T08:00:00.123+08:00')` 这个对象的 `valueOf()` 结果是 123

非常重要：虽然**内部存储**的是 UTC 日期时间，但我们**使用**的是转换后的 local 日期时间

- `string` 值 -> `Date` 对象
  - `new Date(stringArg)`, **记得让** stringArg 符合 ISO 8601 format, 请勿使用 date-only forms (比如 "2023-07-01"), 会被看成是 UTC time, 而不是 local time, 而且不同 EcmaScript 实现还不相同
- `Date` 对象 -> `string` 值
  - `toISOString()`, 结果是 UTC time

### 第三方库 dayjs

`Dayjs` 对象是原生 `Date` 对象的 wrapper.

非常重要：虽然**内部存储**的是 UTC 日期时间，但我们**使用**的是转换后的 local 日期时间

- `string` 值 -> `Dayjs` 对象
  - `dayjs(stringArg)`, stringArg 符合 ISO 8601 format 时用
  - `dayjs(stringArg, <customParseFormat>)`, stringArg 不符合 ISO 8601 format 时用，使用前需要引入 `customParseFormat`, 请看《实战 - 问题 1 和 2》
- `Dayjs` 对象 -> `string` 值
  - `toISOString()`, 结果是 UTC time
  - `format([template])`, 结果是 UTC offset time 或 UTC time, 可以不输入 template, 默认 template 是 `YYYY-MM-DDTHH:mm:ssZ`, 特别注意：这里的`Z`并不是去获取 UTC time，它说的是要保留时区信息，具体看《实战 - 问题 3》

### 实战

[在线 playground](https://codesandbox.io/s/dayjs-date-time-x9fyt7)

问题 1: 后端返回一个 UTC time 或 UTC offset time, 要求前端把它看成是 local time，如何处理？e.g. 后端返回的是 `2023-07-01T02:03:04Z`，前端应该将看成是 `2023-07-01T02:03:04`
思路 1: 要点是如何丢掉时区？因为后端返回的字符串可能是`Z`结尾，也可能是`+02:00`之类的结尾
答案 1: `dayjs(backendString, 'YYYY-MM-DDTHH:mm:ss')`, 需要特别注意的是：需要引入 `customParseFormat`, 不然 dayjs 会进行时区换算, 那样的话结果就不对了

问题 2: 后端返回`08:00`, 前端需要在 antd 时间组件中显示出来，如何做？
答案 2: `dayjs('08:00', 'HH:mm')`, 需要特别注意的是：需要引入 `customParseFormat`, 不然无法得到一个有效的 Dayjs 对象

问题 3: `dayjs('2023-07-01T02:03:04+01:00').format('YYYY-MM-DD HH:mm')` 和 `dayjs('2023-07-01T02:03:04+01:00').format('YYYY-MM-DD HH:mmZ')` 有什么区别？
思路 3: 要点是`dayjs('2023-07-01T02:03:04+01:00')`到底是什么？format 中的结尾处一个没有 Z，一个有 Z，有什么区别？
答案 3: 具体过程如下

1. `dayjs('2023-07-01T02:03:04+01:00')`得到一个 Dayjs 对象
1. 它存储的是 `2023-07-01T01:03:04Z` 这个 UTC time 的整数值
1. 需要注意的是，我们实际上操作的是当地时区的日期时间，假如我们在东 8 区，这个 Dayjs 对象就是 `2023-07-01T09:03:04+08:00`
1. 然后调用对象的 format 方法 - 1st `YYYY-MM-DD HH:mm`，从左到右解析分别得到 `2023` `-` `07` `-` `01` ` `(空格) `09` `:` `03` (没有`ss`丢掉秒) (没有`SSS`丢掉毫秒) 没有`Z`丢掉时区，结果就是 `2023-07-01 09:03`
1. 2nd `YYYY-MM-DD HH:mmZ`，从左到右解析分别得到 `2023` `-` `07` `-` `01` ` `(空格) `09` `:` `03` (没有`ss`丢掉秒) (没有`SSS`丢掉毫秒) `+08:00`，结果就是 `2023-07-01 09:03+08:00`
