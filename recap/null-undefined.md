# Null vs. Undefined

> [Free youtube video on the subject](https://www.youtube.com/watch?v=kaUfBNzuUAI)

JavaScript \(и, соответственно, TypeScript\) имеет два базовых типа: `null` и `undefined`. Они предназначены для обозначения разных вещей:

* Что-то не было проинициализировано: `undefined`.
* Что-то в данный момент недоступно: `null`.

## Проверки

Фактически, вы будете иметь дело с каждым из этих типов. Просто проверьте каждый из них с помощью оператора  `==`.

```typescript
/// Представьте, что вы делаете `foo.bar == undefined`, где bar может быть одним из:
console.log(undefined == undefined); // true
console.log(null == undefined); // true

// Вам не нужно беспокоиться о "ложных" (falsy) значениях, которые проходят эту проверку
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```

Рекомендуем использовать `== null` для проверки `undefined` или `null`. Как правило, вы не хотите проводить различие между ними.

```typescript
function foo(arg: string | null | undefined) {
  if (arg != null) {
    // arg должен быть строкой, поскольку `!=` исключает как null, так и undefined. 
  }
}
```

Единственное исключение - неопределенные значения глобального уровня \(root level undefined values\), которые мы обсудим далее.

## Проверка для глобального уровня

Помните, как я сказал, что вы должны использовать `== null`? Конечно вы помните \(ведь я только что сказал это ^\). Не используйте это для вещей глобального уровня. В строгом режиме, если вы используете `foo` и `foo` не определено, то вы получите **исключение** `ReferenceError` и весь стек вызовов раскрутится.

> Вы должны использовать строгий режим... и, фактически, компилятор TS вставит его для вас, если вы используете модули... подробнее об этом позже в книге, так что вам не нужно явно об этом говорить :\)

Таким образом, чтобы проверить, определена ли переменная на _глобальном_ уровне, вы обычно используете `typeof` :

```typescript
if (typeof someglobal !== 'undefined') {
  // someglobal теперь можно безопасно использовать
  console.log(someglobal);
}
```

## Limit explicit use of `undefined`

Because TypeScript gives you the opportunity to _document_ your structures separately from values instead of stuff like:

```typescript
function foo(){
  // if Something
  return {a:1,b:2};
  // else
  return {a:1,b:undefined};
}
```

you should use a type annotation:

```typescript
function foo():{a:number,b?:number}{
  // if Something
  return {a:1,b:2};
  // else
  return {a:1};
}
```

## Node style callbacks

Node style callback functions \(e.g. `(err,somethingElse)=>{ /* something */ }`\) are generally called with `err` set to `null` if there isn't an error. You generally just use a truthy check for this anyways:

```typescript
fs.readFile('someFile', 'utf8', (err,data) => {
  if (err) {
    // do something
  } else {
    // no error
  }
});
```

When creating your own APIs it's _okay_ to use `null` in this case for consistency. In all sincerity for your own APIs you should look at promises, in that case you actually don't need to bother with absent error values \(you handle them with `.then` vs. `.catch`\).

## Don't use `undefined` as a means of denoting _validity_

For example an awful function like this:

```typescript
function toInt(str:string) {
  return str ? parseInt(str) : undefined;
}
```

can be much better written like this:

```typescript
function toInt(str: string): { valid: boolean, int?: number } {
  const int = parseInt(str);
  if (isNaN(int)) {
    return { valid: false };
  }
  else {
    return { valid: true, int };
  }
}
```

## JSON and serialization

The JSON standard has support for encoding `null` but not `undefined`. When JSON-encoding an object with an attribute that is `null`, the attribute will be included with its null value, whereas an attribute with an `undefined` value will be excluded entirely.

```typescript
JSON.stringify({willStay: null, willBeGone: undefined}); // {"willStay":null}
```

As a result, JSON-based databases may support `null` values but not `undefined` values. Since attributes set to `null` are encoded, you can transmit the intent to clear an attribute by settings its value to `null` before encoding and transmitting the object to a remote store.

Setting attribute values to undefined can save of storage and transmission costs, as the attribute names will not be encoded. However, this can complicate the semantics of clearing values vs. absent values.

## Final thoughts

TypeScript team doesn't use `null` : [TypeScript coding guidelines](https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines#null-and-undefined) and it hasn't caused any problems. Douglas Crockford thinks [`null` is a bad idea](https://www.youtube.com/watch?v=PSGEjv3Tqo0&feature=youtu.be&t=9m21s) and we should all just use `undefined`.

However, NodeJS style code bases uses `null` for Error arguments as standard as it denotes `Something is currently unavailable`. I personally don't care to distinguish between the two as most projects use libraries with differing opinions and just rule out both with `== null`.

