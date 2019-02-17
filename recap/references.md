# Ссылки

За исключением литералов, любой объект в JavaScript \(включая функции, массивы, регулярные выражения и т.д.\) является ссылкой. Это означает следующее.

## Изменения по всем ссылкам

```javascript
var foo = {};
var bar = foo; // bar - ссылка на тот же объект

foo.baz = 123;
console.log(bar.baz); // 123
```

## Равенство проверяется для ссылок

```javascript
var foo = {};
var bar = foo; // bar является ссылкой
var baz = {}; // baz - это новый объект, отличный от `foo`

console.log(foo === bar); // true
console.log(foo === baz); // false
```

