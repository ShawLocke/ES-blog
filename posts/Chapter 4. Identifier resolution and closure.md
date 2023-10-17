# Identifier resolution and Closure

## Identifier resolution

> The rule for identifier resolution is: if an identifier is not found in the own environment, there is an attempt to look it up in the outer environment, in the outer of the outer, and so on - until the whole environment chain is considered.
>
> 标识符解析的规则是：如果标识符在自身环境中找不到，就去外部环境中找，外部环境中找不到，就去外部环境的外部环境中找，直至整个环境链。
> environment chain 对应的是以前的 scope chain (作用域链).

标识符解析会用到 ResolveBinding(name[, env]) (抽象运算)

The abstract operation ResolveBinding takes argument name (a String) and optional argument env (an Environment Record).

> ResolveBinding 的结果是 reference record.

1. If env is not present or if env is undeﬁned, then
   1. Set env to the running execution context's LexicalEnvironment.
2. Assert: env is an Environment Record.
3. If the code matching the syntactic production that is being evaluated is contained in strict mode code, let strict be true; else let strict be false.
4. Return ? GetIdentiﬁerReference(env, name, strict).

接下来用到 GetIdentiﬁerReference(env, name, strict) (抽象运算)

The abstract operation GetIdentiﬁerReference takes arguments env (an Environment Record or null), name (a String), and strict (a Boolean).

1. If env is the value null, then
   > 当 env 是`null`时，就意味着整个 environment chain 都搜索完了，返回一个 unresolvable 的 reference record，见示例 1
   1. Return the Reference Record { \[[Base]]: unresolvable, \[[ReferencedName]]: name, \[[Strict]]: strict }.
2. Let exists be ? env.HasBinding(name).
   > 注意：当 env 是 global environment record 时，HasBinding 先检查其 declarative environment record，如果有，返回；如果没有，再检查其 object environment record，即 the global object，因为它是一个 object，还会进行 prototype chain 查找，见示例 2
3. If exists is true, then
   1. Return the Reference Record { \[[Base]]: env, \[[ReferencedName]]: name, \[[Strict]]: strict }.
4. Else,
   1. Let outer be env.\[[OuterEnv]].
   2. Return ? GetIdentiﬁerReference(outer, name, strict).
      > 递归调用自身，也就是上面说的会一直到 outer environment 进行搜索

```js
// 示例 1
// notFound的reference record:
// { [[Base]]: unresolvable, [[ReferencedName]]: 'notFound', [[Strict]]: false }
notFound; // ReferenceError: notFound is not defined

Object.prototype.foo = 10;
// 示例 2
// foo的reference record:
// { [[Base]]: globalEnvRec, [[ReferencedName]]: 'foo', [[Strict]]: false }
foo; // 10
```

## Closure

> A closure is a function which captures the environment where it is declared. Further this environment may be used for identifier resolution.
>
> 闭包说的就是函数，只要弄清楚函数的 environment record 和\[[OutEnv]]如何确定的，闭包是比较容易理解的。

**创建**函数时会确定它的\[[Environment]](即：将函数声明所在的 environment record 存入它的\[[Environment]]，请参考 spec 中的 OrdinaryFunctionCreate。\[[Environment]]是函数对象的一个 internal slot)。**执行**函数时，新生成的 environment record 的\[[OutEnv]]被设置为\[[Environment]]。这样就形成了 environment chain，进而被用来解析标识符。

```js
function bar() {
  const y = 1;
  console.log(x + y);
}

const x = 20;
function foo(funArg) {
  const x = 10;
  funArg();
}

foo(bar); // 21
```

```js
function qux() {
  const x = 10;

  return function () {
    const y = 1;
    console.log(x + y);
  };
}

const x = 20;
const baz = qux();
baz(); // 11
```
