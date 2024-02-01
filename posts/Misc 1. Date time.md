# Date time

### ISO 8601

> ISO 8601 涵盖范围非常广，编程使用需要限制范围，具体参考：<https://www.w3.org/TR/NOTE-datetime>

1. 以**字符串**表达
1. 完整格式：`YYYY-MM-DDTHH:mm:ss.SSSTZD` (注意大小写)
   - YYYY: Year, 4 位
   - MM: Month, 2 位, 01 (January) - 12
   - DD: Day, 2 位, 01 - 31
   - T: date 和 time 的分隔符, T 意味着 Time 开始
   - HH: Hour, 2 位, 00 - 23. 注意: ISO 8601 采用 24 小时制，所以大小写都可以 (dayjs 中 大写 HH 表示 24 小时制, 小写 hh 表示 12 小时制，我采用 HH)
   - mm: Minute, 2 位, 00 - 59
   - ss: Second, 2 位, 00 - 59
   - SSS: milliSecond, 3 位, 000 - 999. 注意: ISO 8601 没有规定必须用 S (我采用 dayjs 的大写 S, 与小写 s 进行区分), 也没有规定必须 3 位 (我采用 EcmaScript 的 3 位)
   - TZD (等同于UTC offset): Time Zone Designator, `Z` or `+hh:mm` or `-hh:mm`

常用术语：

1. UTC time: TZD 是 `Z`
   - e.g. `2023-07-01T02:03:04.567Z`
1. local time: TZD 是 `+hh:mm` or `-hh:mm`
   - e.g. `2023-07-01T02:03:04.567+08:00`
1. ambiguous time: 没有指定 TZD
   - e.g. `2023-07-01T02:03:04.567`

### 原生 `Date` object

> `Date` 对象内部存储的是 Unix timestamp (精确到 3 位毫秒, 起始点是 `1970-01-01T00:00:00.000Z`)
> e.g. `new Date('1970-01-01T08:00:00.123+08:00')` 这个对象的 `valueOf()` 结果是 123

1. 只有 local mode, parse and display in local time.
   - ambiguous input (without TZD) is assumed to be local time
     - 直接在 input 后追加相应的 TZD, e.g. `2023-07-01T02:03:04.567` 会被看成 `2023-07-01T02:03:04.567+08:00`
     - 注意: EcmaScript 的一个特殊情况，如果 input 是 date-only forms, input 后追加的是 `Z`, e.g. `2023-07-01` -> `2023-07-01T00:00:00.000Z`, 参考: <https://262.ecma-international.org/14.0/#sec-date.parse>.
     - date-only forms: `YYYY`, `YYYY-MM`, `YYYY-MM-DD`, date-time forms: date-only forms + (`THH:mm`, `THH:mm:ss`, `THH:mm:ss.SSS`) + an optional TZD, 参考：<https://262.ecma-international.org/14.0/#sec-date-time-string-format>, 不满足 date-only forms 或者 date-time forms 会导致 Invalid Date
   - unambiguous input (with TZD) is adjusted to local time

- `string` 值 -> `Date` 对象 (这是上面讲的 parse)
  - `new Date(stringArg)`
- `Date` 对象 -> `string` 值
  - `toString()`, 以 GMT 格式表达，e.g. `Sat Jul 01 2023 02:03:04 GMT+0800 (China Standard Time)`，参考：<https://262.ecma-international.org/14.0/#sec-date.prototype.tostring>
  - `toISOString()`, ISO 指的是 ISO 8601, 结果是 UTC time, 一般是用这个方法

### 第三方库 dayjs

> Dayjs object is a wrapper for the `Date` object

1. in local mode, parse and display in local time
   - ambiguous input (without TZD) is assumed to be local time
     - 直接在 input 后追加相应的 TZD, e.g. `2023-07-01T02:03:04.567` 会被看成 `2023-07-01T02:03:04.567+08:00`
   - unambiguous input (with TZD) is adjusted to local time
1. in UTC mode, parse and display in UTC time
   - ambiguous input is assumed to be UTC time
     - 直接在 input 后追加 `Z`
   - unambiguous input is adjusted to UTC time.
1. local 当前时间: `dayjs()`, convert local to UTC time: `.utc()`
1. UTC 当前时间: `dayjs.utc()`, convert UTC to local time: `.local()`

- `string` 值 -> `Dayjs` 对象 (这是上面讲的 parse)
  - `dayjs(stringArg)`, stringArg 符合 ISO 8601 format 时用
  - `dayjs(stringArg, <customParseFormat>)`, 使用前需要引入 `customParseFormat`, 如果不引入，那就可看成是 `dayjs(stringArg)`。实例请看《实战 - 问题 1》
- `Dayjs` 对象 -> `string` 值
  - `format([template])`, template 可选，默认是 `YYYY-MM-DDTHH:mm:ssZ`, 特别注意：这里的 `Z` 不是去获取 UTC time，说的是要保留时区信息。实例请看《实战 - 问题 2》
  - `toISOString()`, 调用的是原生 `Date` 对象的 `toISOString`方法

### 第三方库 moment

> moment 与 dayjs 类似, moment 与 dayjs 不同如下, 参考: <https://github.com/you-dont-need/You-Dont-Need-Momentjs>)

1. moment 是 mutable, 使用时要特别注意，经常会用到 `.clone()`; dayjs 是 immutable
1. `moment.tz.setDefault` 对 `moment.tz()` 和 `moment()` 生效; `dayjs.tz.setDefault` 对 `dayjs.tz()` 生效，对 `dayjs()` 不生效 (<https://github.com/iamkun/dayjs/issues/1227#issuecomment-900816717>)

### 实战

问题 1: 后端返回一个 time (e.g. `2023-07-01T02:03:04Z`), 要求前端把它看成是 local time, 即：`2023-07-01T02:03:04+08:00`, 如何处理?
思路 1: 要点是如何丢掉时区？因为后端返回的字符串可能是`Z`结尾，也可能是`+02:00`之类的结尾
答案 1: `dayjs(backendString, 'YYYY-MM-DDTHH:mm:ss')`, 特别注意：需要引入 `customParseFormat`, 不然就理解是 `dayjs(backendString)`, 这就会进行时区换算, 结果是 `2023-07-01T10:03:04+08:00`, 这不是想要的结果

问题 2: `dayjs('2023-07-01T02:03:04+01:00').format('YYYY-MM-DD HH:mm')` 和 `dayjs('2023-07-01T02:03:04+01:00').format('YYYY-MM-DD HH:mmZ')` 有什么区别？
思路 2: 要点是`dayjs('2023-07-01T02:03:04+01:00')`到底是什么？format 中的结尾处一个没有 Z，一个有 Z，有什么区别？
答案 2: 具体过程如下：

1. `dayjs('2023-07-01T02:03:04+01:00')`生成对象时用的是 local mode, input 会被 adjust to local time, i.e. `2023-07-01T09:03:04+08:00`
1. 调用对象的 format 方法

   - 1st: `YYYY-MM-DD HH:mm`，从左到右解析分别得到 `2023` `-` `07` `-` `01` ` `(空格) `09` `:` `03` (没有`ss`丢掉秒) (没有`SSS`丢掉毫秒) 没有`Z`丢掉时区，结果就是 `2023-07-01 09:03`
   - 2nd: `YYYY-MM-DD HH:mmZ`，从左到右解析类似，区别：有`Z`，保留时区`+08:00`，结果就是 `2023-07-01 09:03+08:00`
