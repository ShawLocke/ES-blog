# Chapter 2. Reference record

> > The reference record type is used to explain the behaviour of such operators as `typeof`, the assignment operators and other language features. E.g. the left-hand operand of an assignment is expected to produce a reference record.
>
> Reference record 是一个抽象概念，可以将其理解为一个 plain object.
>
> Spec 用 specification type, 比如 reference record. 我们编写的 JS 代码用 language type，比如 Boolean, Number, String 等。

Reference record 用于 2 个地方：

- with an identifier.
  > 如何确定 identifier 的 reference record 的\[[Base]]请参考《Chapter 4 中的 identifier resolution》
- with a property accessor (the dot notation or the bracket notation).
  > 会用到 spec 中的 EvaluatePropertyAccessWithIdentiﬁerKey 或 EvaluatePropertyAccessWithExpressionKey 抽象运算

Reference record 的属性有：

- \[[Base]]:
  - any ECMAScript language value except `undeﬁned` or `null`. (见示例 2 和 3)
  - an environment record. (见示例 1)
  - unresolvable. (见示例 4)
    > 对 unresolvable 的 reference record 应用 GetValue 抽象运算会导致 ReferenceError
- \[[ReferencedName]]: the name of binding.
- \[[Strict]]: `true` if in strict mode code, `false` otherwise.

```js
// 示例 1
// foo的reference record:
// { [[Base]]: globalEnvRec, [[ReferencedName]]: 'foo', [[Strict]]: false }
const foo = 10;

// 示例 2
const bar = { x: 20 };
// 整个bar.x的解析过程如下：
// bar的reference record:
// { [[Base]]: globalEnvRec, [[ReferencedName]]: 'bar', [[Strict]]: false }
// 然后应用 GetValue ，得到一个对象
// bar.x的reference record:
// { [[Base]]: bar指向的对象(上面的对象), [[ReferencedName]]: 'x', [[Strict]]: false }
bar.x;

// 示例 3
const baz = null;
// 无法成功生成baz.toString的Reference record: EvaluatePropertyAccessWithIdentiﬁerKey
// 会调用RequireObjectCoercible, `undefined`或`null`会导致TypeError
baz.toString; // TypeError: Cannot read properties of null

// 示例 4
// notFound的reference record:
// { [[Base]]: unresolvable, [[ReferencedName]]: 'notFound', [[Strict]]: false }
notFound; // ReferenceError: notFound is not defined (应用 GetValue 时导致的ReferenceError)
```
