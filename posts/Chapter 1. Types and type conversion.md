# Chapter 1. Types, type conversion, operators

In JavaScript, variables don't have types, but values do.

## Types

> A primitive value is a member of one of the following built-in types: Undeﬁned, Null, Boolean, Number, String, Symbol, and BigInt.
> BigInt 用得很少，就没列出来

- Undefined: `undefined`
- Null: `null`
- Boolean: `true`, `false`
- Number
- String
- Symbol
- Object
  - Object: `{}`
  - Array: `[]`
  - Function: `function foo() {}`
  - Set: `new Set()`
  - Map: `new Map()`
  - Error: `new Error('msg')`
  - Date: `new Date()`
  - JSON: `JSON.stringify`
  - Math: `JSON.floor`
  - RegExp: `/^foo$/gmi`, g: global, m: multiline, i: case-insensitive
  - 等等

## `typeof` operator

检查值的类型，结果是字符串。

> 当 operand (运算子)无法 resolve 时，结果是`"undefined"`，如何判断是否能 resolve 请参考《Chapter 2. Reference record》
>
> `typeof`和 spec 中的 Type(x)不同：Type(x)的结果是 Undefined, Null, Boolean, Number, String, or Object 等，不会有 Function 之类的。

| Type of operand                       | Result      | Note                                                                                                                   |
| ------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| Undefined                             | "undefined" |                                                                                                                        |
| Null                                  | "object"    | Oops! 历史遗留错误：在面向对象编程中，想 unset 对象时，往往将其赋值为`null`.                                           |
| Boolean                               | "boolean"   |                                                                                                                        |
| Number                                | "number"    |                                                                                                                        |
| String                                | "string"    |                                                                                                                        |
| Symbol                                | "symbol"    |                                                                                                                        |
| Object (does not implement \[[Call]]) | "object"    | 比如`typeof {}`, `typeof []`. 又比如`typeof JSON`, `typeof Math`, **不能用作函数**.                                    |
| Object (implements \[[Call]])         | "function"  | 比如 `typeof function(){}`. 又比如`typeof String`, `typeof Object`, `typeof Array`, `typeof Function`, **能用作函数**. |

1. 判断是不是 `undefined` ，可以用 `foo === undefined`
1. 判断是不是 `null` ，可以用 `foo === null`
1. 判断是不是 `undefined` 或 `null` ，可以用 `foo == null`
1. 判断是不是一个 Boolean 类型的值，可以用`foo === true` 或 `foo === false`
1. 判断是不是一个 Number 类型的值，可以用 `typeof foo == "number"`
1. 判断是不是一个 String 类型的值，可以用 `typeof foo == "string"`
1. 获取准确类型 ( `typeof` 对 Object 的判断很粗放)，可以用 `Object.prototype.toString.call(foo).slice(8, -1).toLowerCase()`
1. 判断是不是一个数组，可以用 `Array.isArray(foo)`
1. 判断是不是一个函数，可以用 `typeof foo == "function"`

## Type conversion

> 我们经常看到的 coercion 说的就是 type conversion，但 spec 中并没有 coercion 这个术语。

> spec 中算法所用符号的说明：
>
> ?表示紧跟的运算可能会抛出异常。
>
> !表示紧跟的运算不会抛出异常 (我都省略掉了。注意：这里的!不是逻辑非运算符)
>
> [] 表示可选。
>
> NewTarget 用来区分`new Boolean()`和`Boolean()` (这里以 Boolean 类型 举例，后一种方式调用时，NewTarget 是 undefined)

### Boolean

以`Boolean(value)`方式被调用时 (前面没有`new`)

> 相当于`!!value`，注意是 2 个逻辑非运算符，通常用`!!value`而不是`Boolean(value)`, 写起来更简短
>
> 如果参数不存在，结果是`false`

1. Let b be ToBoolean(value).
1. If NewTarget is undefined, return b.
1. ...

ToBoolean(argument) (抽象运算，运算结果是`true`或者`false`)

| Type of argument | Result                                                                       |
| ---------------- | ---------------------------------------------------------------------------- |
| Undefined        | Return `false`.                                                              |
| Null             | Return `false`.                                                              |
| Boolean          | Return argument.                                                             |
| Number           | If argument is `0`, `-0`, or `NaN`, return `false`; otherwise return `true`. |
| String           | If argument is `""` (空字符串), return `false`; otherwise return `true`.     |
| Symbol           | Return `true`.                                                               |
| Object           | Return `true`.                                                               |

### Number

以`Number(value)`方式被调用时 (前面没有`new`)

> 相当于`+value`，不建议使用，影响可读性
>
> 如果参数不存在，结果是`0`

1. If value is not present, let n be +0.
1. Else, let n be ? ToNumber(value).
1. If NewTarget is undefined, return n.
1. ...

ToNumber(argument) (抽象运算，运算结果是 Number 类型值或者 Exception)

| Type of argument | Result                                                                                                                                                                                                                                                                  | Note                        |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| Undefined        | Return `NaN`.                                                                                                                                                                                                                                                           |                             |
| Null             | Return `0`.                                                                                                                                                                                                                                                             | Oops! 如果是`NaN`会更合适。 |
| Boolean          | If argument is `true`, return `1`. If argument is `false`, return `0`.                                                                                                                                                                                                  |                             |
| Number           | Return argument. (十进制数字)                                                                                                                                                                                                                                           |                             |
| String           | 去掉首尾的空白(包括空格，tab，换行等)后，如果是空字符串，return `0`, 如果看起来是数字(可能是二进制 Binary，八进制 Octal，十六进制 heXadecimal)，return 十进制 decimal 数字 (输入是科学记数法的字符串，输出可能是十进制数字，也可能是科学计数法数字)，否则 return `NaN`. | `"-0"`->`-0`.               |
| Symbol           | Throw a **TypeError** exception.                                                                                                                                                                                                                                        |                             |
| Object           | Apply the following steps:<br>1. Let primValue be ? ToPrimitive(argument, number). (**指定了 preferredType**)<br>2. Return ? ToNumber(primValue).                                                                                                                       |                             |

ToPrimitive(input[, preferredType]) (抽象运算)

> 该运算追求得到 primitive value，如果得不到就会抛出 Exception。
> 这很好解释了：如果@@toPrimitive 返回的是 Object 类型，直接抛出 Exception 了 (步骤 1-2-6)。

1. If Type(input) is Object, then
   1. Let exoticToPrim be ? GetMethod(input, @@toPrimitive). (获取`Symbol.toPrimitive`属性的值，如果属性不存在或属性值是`undefined`或`null`，返回`undefined`，见示例 1；如果是函数，返回函数，见示例 2；否则抛出 TypeError，见示例 3)
   1. If exoticToPrim is not undefined, then
      1. If preferredType is not present, let hint be "default".
      1. Else if preferredType is number, let hint be "number". (我将 spec 中的这一步和下一步对调了，便于记忆)
      1. Else,
         1. Assert: preferredType is string.
         1. Let hint be "string".
      1. Let result be ? Call(exoticToPrim, input, « hint »). (调用函数，见示例 4)
      1. If Type(result) is not Object, return result. (如果不是 Object，返回结果，见示例 4)
      1. Throw a TypeError exception. (如果是 Object 类型，抛出 TypeError，见示例 5)
   1. If preferredType is not present, let preferredType be number.
      > 这里可以看出 number 的权重比 string 高
   1. Return ? OrdinaryToPrimitive(input, preferredType).
1. Return input.

> > When ToPrimitive is called with no preferredType, then it generally behaves as if the preferredType were number.
>
> 如果没有 preferredType，那大部分情况下 preferredType 被看成是 number
>
> > However, objects may over-ride this behaviour by deﬁning a @@toPrimitive method.
>
> 对象通过定义@@toPrimitive 方法可重写这一行为
>
> > Date objects over-ride the default ToPrimitive behaviour. Date objects treat no preferredType as if the preferredType were string.
>
> Date 重写了 ToPrimitive 的默认行为。Date 将没有 preferredType 看成是 string。ES spec 中搜索 `Date.prototype [ @@toPrimitive ]` 就能看出来怎么回事。见下面的示例
>
> ```js
> const foo = new Date();
> // ToPrimitive(foo, 没有 preferredType)
> // => "Tue Jun 27 2023 08:49:53 GMT+0800 (China Standard Time)" + 1
> foo + 1;
> ```
>
> ```js
> const foo = new Date();
> // ToPrimitive(foo, preferredType 是 number)
> // => 1687826993053 > 1
> foo > 1;
> ```

OrdinaryToPrimitive(o, preferredType) (抽象运算)

> 该运算追求得到 primitive value，如果得不到就会抛出 Exception。
> 这很好解释了：最后一步直接抛出 Exception 了 (步骤 6)。

1. Assert: Type(o) is Object.
1. Assert: preferredType is either number or string.
1. If preferredType is number, then (我将 spec 中的这一步和下一步对调了，便于记忆)
   1. Let methodNames be « "valueOf", "toString" ».
1. Else,
   1. Let methodNames be « "toString", "valueOf" ».
1. For each element name of methodNames, do
   1. Let method be ? Get(o, name). (示例 8)
   1. If IsCallable(method) is true, then (示例 7 中这里是 false)
      1. Let result be ? Call(method, o). (示例 6.a)
      1. If Type(result) is not Object, return result. (示例 6.b)
1. Throw a TypeError exception. (示例 7)

```js
// 示例 1
// 1. 调用ToNumber(arg), 参数是一个Object类型
// 2. 调用ToPrimitive(arg, number), 虽然定义了Symbol.toPrimitive，但其值是undefined
// 3. 调用OrdinaryToPrimitive(arg, number)，先调用valueOf()，结果是对象本身，忽略；
// 再调用toString()，结果是"[object Object]"，返回该字符串。
// 4. 调用ToNumber("[object Object]")
Number({
  // 或者这个属性不存在
  [Symbol.toPrimitive]: undefined,
}); // NaN
Number({
  [Symbol.toPrimitive]: null,
}); // 结果同上，原因参考上面

// 示例 2
Number({
  [Symbol.toPrimitive]: () => {
    return 123;
  },
}); // 123

// 示例 3
Number({
  [Symbol.toPrimitive]: 123, // Symbol.toPrimitive属性不是一个函数导致TypeError
}); // TypeError: xxx is not a function

// 示例 4
Number({
  [Symbol.toPrimitive]: (hint) => {
    if (hint === "string") {
      return "abc";
    } else {
      // hint is "number" or "default"
      return 123;
    }
  },
}); // 123. 如果是String(...), 结果就是abc

// 示例 5
Number({
  [Symbol.toPrimitive]: () => {
    return {}; // 返回值不是原始类型导致TypeError
  },
}); // TypeError: Cannot convert object to primitive value

// 示例 6
// 123. 原因：OrdinaryToPrimitive先调用valueOf，结果是一个数组，它是Object类型，忽略；
// 再调用toString，结果是" 123 "，返回该字符串，再调用ToNumber，最终结果是123
Number({
  // 6.a
  valueOf() {
    return [];
  },
  // 6.b
  toString() {
    return " 123 ";
  },
}); // 123

// 示例 7
// TypeError. 原因：valueOf和toString都不是函数，会到最后一步
Number({
  valueOf: 123,
  toString: "abc",
}); // TypeError: Cannot convert object to primitive value

// 示例 8
// TypeError. 原因：这个对象没有valueOf和toString属性，会到最后一步
Number(Object.create(null)); // TypeError: Cannot convert object to primitive value
```

### String

以`String(value)`方式被调用时 (前面没有`new`)：

> 注意与`` `${value}` ``, `value + ""`之间的比较，三者异同点请看后面
>
> 如果参数不存在，结果是`""`

1. If value is not present, let s be empty string.
1. Else,
   1. If NewTarget is undefined and Type(value) is Symbol, return SymbolDescriptiveString(value). (如果 Description 是`undefined`，结果是`"Symbol()"`，否则是`"Symbol(" + value.[[Description]] + ")"`)
   1. Let s be ? ToString(value).
1. If NewTarget is undefined, return s.
1. ...

ToString(argument) (抽象运算，运算结果是 String 类型值或者 Exception)

| Type of argument | Result                                                                                                                                            | Note              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| Undefined        | Return `"undefined"`.                                                                                                                             |                   |
| Null             | Return `"null"`.                                                                                                                                  |                   |
| Boolean          | If argument is `true`, return `"true"`. If argument is `false`, return `"false"`.                                                                 |                   |
| Number           | Return NumberToString(argument). (结果是十进制数字字符串，数字非常大时会是科学记数法的字符串)                                                     | Oops! `-0`->`"0"` |
| String           | Return argument.                                                                                                                                  |                   |
| Symbol           | Throw a **TypeError** exception.                                                                                                                  |                   |
| Object           | Apply the following steps:<br>1. Let primValue be ? ToPrimitive(argument, string). (**指定了 preferredType**)<br>2. Return ? ToString(primValue). |                   |

```js
// String(value) vs. value + "" vs. `${value}`？
// 相同：三者都用到ToString这个抽象运算

// String(value)单独处理了Symbol类型的值，不会抛异常，后两者都会抛TypeError
String(Symbol()); // "Symbol()"
`${Symbol()}`; // TypeError
Symbol() + ""; // TypeError

const foo = {
  toString() {
    return "abc";
  },
  valueOf() {
    return 123;
  },
};
// 调用了ToString
String(foo); // "abc"
// 调用了ToString
`${foo}`; // "abc"
// 逻辑：先对左右运算子分别调用ToPrimitive，
// 未指定preferredType (其被视为number)，valueOf被调用，得到值123；
// 然后应用+运算符，由于右运算子是个字符串，所以会进行string-concatenation
// 首先对左运算子应用ToString，得到"123"，"123"+""最终结果是"123"
// 具体请参考下面的二元运算符
foo + ""; // "123"
```

### Logical operators (与 ToBoolean 相关)

> 注意：和其他语言大不相同，因为结果不一定是 Boolean 类型的值

`&&` (logical and, 逻辑与), `||` (logical or, 逻辑或)

`leftOperand && rightOperand`或`leftOperand || rightOperand`

1. Let lref be the result of evaluating leftOperand.
1. Let lval be ? GetValue(lref).
1. Let lbool be ToBoolean(lval).
1. If lbool is false, return lval.
   > 如果是`||`，上面的**false** 改为 **true**.
   > 直接返回左运算子的值，这是 short circuit.
1. Let rref be the result of evaluating rightOperand.
1. Return ? GetValue(rref). (返回右运算子的值)

`!` (logical not, 逻辑非)

`!operand`

1. Let ref be the result of evaluating operand.
1. Let oldValue be ToBoolean(? GetValue(ref)).
1. If oldValue is true, return false.
1. Return true.

### Conditional operator (与 ToBoolean 相关)

`? :`

`ShortCircuitExpression ? AssignmentExpression : AssignmentExpression`

1. Let lref be the result of evaluating ShortCircuitExpression.
1. Let lval be ToBoolean(? GetValue(lref)).
1. If lval is true, then
   1. Let trueRef be the result of evaluating the ﬁrst AssignmentExpression.
   1. Return ? GetValue(trueRef).
1. Else,
   1. Let falseRef be the result of evaluating the second AssignmentExpression.
   1. Return ? GetValue(falseRef).

### Binary arithmetic operators (主要与 ToNumber 有关)

> spec 中并没有 arithmetic operators 这个术语，个人猜测原因是：`+`还能串接字符串

`+` (addition), `-` (subtraction), `*` (multiplication), `/` (division), `%` (remainder)

会调用：ApplyStringOrNumericBinaryOperator(lval, opText, rval) (抽象运算，简化)

1. If opText is +, then
   1. Let lprim be ? ToPrimitive(lval).
   1. Let rprim be ? ToPrimitive(rval).
   1. If Type(lprim) is String or Type(rprim) is String, then (用的是 **or**)
      1. Let lstr be ? ToString(lprim).
      1. Let rstr be ? ToString(rprim).
      1. Return the string-concatenation of lstr and rstr.
   1. Set lval to lprim.
   1. Set rval to rprim.
1. Let lnum be ? ToNumber(lval).
1. Let rnum be ? ToNumber(rval).
1. Return the result of applying the above operation to lnum and rnum.

### Unary arithmetic operators (与 ToNumber 有关)

`++` (increment operator, 分为 postfix increment operator 和 prefix increment operator), `--` (decrement operator，分为 postfix 和 prefix)

以 postfix increment operator 为例：

`LeftHandSideExpression++`

1. Let lhs be the result of evaluating LeftHandSideExpression.
1. Let oldValue be ? ToNumber(? GetValue(lhs)).
1. Let newValue be the result of adding the value 1 to oldValue, using the same rules as for the + operator.
1. Perform ? PutValue(lhs, newValue).
1. Return oldValue.

```js
let i = "abc";

// 不要简单的将i++看成是i = i + 1
i++; // i is NaN
```

### Relational operators (主要与 ToNumber 有关)

`<` (less than), `<=` (less than or equals), `>` (greater than), `>=` (greater than or equals)

会调用：Abstract Relational Comparison (抽象运算，简化)

1. Let lprim be ? ToPrimitive(lval, number). (**指定 preferredType**, 这里可以看出这些运算符是期待 Number 类型的值)
1. Let rprim be ? ToPrimitive(rval, number). (**指定 preferredType**)
1. If Type(lprim) is String and Type(rprim) is String, then (用的是**and**)
   1. Return the result of string-comparision. (字符串比较结果)
1. Else,
   1. Let lnum be ? ToNumber(lprim).
   1. Let rnum be ? ToNumber(rprim).
   1. If lnum or rnum is NaN, return undefined. (调用方会将`undefined`视为`false`)
   1. Return the result of number-comparision. (数字比较结果)

### Equality operators

`==` (double equals), `!=`, `===` (triple equals), `!==`

> `==`和`===`如何选择？
> `==`可以考虑用在：`typeof == "string"` (`typeof`运算结果一定是 String 类型), `foo == null` (用来检查 foo 是`null`或者`undefined`)。其他情况推荐使用`===`

`==`和`!=`会用到 Abstract Equality Comparison (抽象运算，简化)

1. If Type(x) is the same as Type(y), then
   1. Return x === y.
1. If x is undefined and y is null, return true.
1. If x is null and y is undefined, return true.
1. If Type(x) is Boolean, return ToNumber(x) == y.
1. If Type(y) is Boolean, return x == ToNumber(y).
1. If Type(x) is Number and Type(y) is String, return x == ToNumber(y).
1. If Type(x) is String and Type(y) is Number, return ToNumber(x) == y.
1. If Type(x) is either Number, String, or Symbol and Type(y) is Object, return x == ? ToPrimitive(y).
1. If Type(x) is Object and Type(y) is either Number, String, or Symbol, return ? ToPrimitive(x) == y.
1. Return false.

`===`和`!==`会用到 Strict Equality Comparison (抽象运算，简化)

1. If Type(x) is different from Type(y), return false.
1. If Type(x) is Number, then
   1. If x is NaN, return false.
   1. If y is NaN, return false.
   1. If x is +0 and y is -0, return true.
   1. If x is -0 and y is +0, return true.
   1. If x is the same as y, return true. (注意进制不同，比如`0xa`和`10`是一样的)
   1. Return false.
1. Return SameValueNonNumber(lval, rval).

SameValueNonNumber(x, y) (抽象运算)

1. Assert: Type(x) is not Number.
1. Assert: Type(x) is the same as Type(y).
1. If Type(x) is Undefined, return true.
1. If Type(x) is Null, return true.
1. If Type(x) is Boolean, then
   1. If x and y are both true or both false, return true; otherwise, return false.
1. If Type(x) is String, then
   1. If x and y are exactly the same sequence of code units (same length and same code units at corresponding indices), return true; otherwise, return false.
1. If Type(x) is Symbol, then
   1. If x and y are the same Symbol value, return true; otherwise, return false. (注意 Symbol 类型值的唯一性)
1. If x and y are the same Object value, return true. Otherwise, return false.

```js
const foo = null;
const bar = 0;

foo >= bar; // true
foo > bar; // false
foo == bar; // false

// 数学常识：如果左运算子大于或者等于右运算子，那么前者要么大于后者，要么等于后者。
// 注意：左右运算子都是数字。
// 粗看上述代码，会觉得它们违背了上述数学常识，但只要熟悉 >= > == 三个运算符的算法，
// 就不会对上述结果感到惊讶。
// 如果一开始将foo和bar都赋值为数字，就和上述数学常识匹配了。
```

```js
"" == 0; // true，对''进行了ToNumber转换，变成0
0 == " "; // true，对' '进行了ToNumber转换，变成0
"" == " "; // false，都是String类型，会进行 ===

const a = new String("foo");
const b = "foo";
const c = new String("foo");
// true，对a进行了ToPrimitive转换(注意：valueOf不是对象本身，而是"foo"，参考`String.prototype.valueOf`)，
// 变成"foo", 然后进行 ===
a == b;
b == c; // true，对c进行了ToPrimitive转换，变成"foo", 然后进行 ===
a == c; // false，进行 ===

// 数学常识：如果a=b, b=c,那么a=c，这称为等式的传递性。注意：左右运算子都是数字。
// 粗看上述2段代码，会觉得它们违背了上述数学常识，但只要熟悉 == 运算符的算法，
// 就不会对上述结果感到惊讶。
```

SameValue(x, y) (抽象运算，`Object.is(x, y)`会用到它)

1. If Type(x) is different from Type(y), return false.
1. If Type(x) is Number, then
   1. If x is NaN and y is NaN, return true.
   1. If x is +0 and y is -0, return false.
   1. If x is -0 and y is +0, return false.
   1. If x is the same as y, return true.
   1. Return false.
1. Return SameValueNonNumber(x, y).

SameValueZero(x, y) (抽象运算，`String.prototype.includes`, `Array.prototype.includes`, `Map`, `Set`会用到它)

1. If Type(x) is different from Type(y), return false.
1. If Type(x) is Number, then
   1. If x is NaN and y is NaN, return true.
   1. If x is +0 and y is -0, return true.
   1. If x is -0 and y is +0, return true.
   1. If x is the same as y, return true.
   1. Return false.
1. Return SameValueNonNumber(x, y).
