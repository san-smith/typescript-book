# JavaScript

There were \(and will continue to be\) a lot of competitors in _Some syntax_ to _JavaScript_ compilers. TypeScript is different from them in that _Your JavaScript is TypeScript_. Here's a diagram:

![JavaScript is TypeScript](https://raw.githubusercontent.com/basarat/typescript-book/master/images/venn.png)

However, it does mean that _you need to learn JavaScript_ \(the good news is _you **only** need to learn JavaScript_\). TypeScript is just standardizing all the ways you provide _good documentation_ on JavaScript.

* Just giving you a new syntax doesn't help fix bugs \(looking at you CoffeeScript\).
* Creating a new language abstracts you too far from your runtimes, communities \(looking at you Dart\).

TypeScript is just JavaScript with docs.

## Making JavaScript Better

TypeScript will try to protect you from portions of JavaScript that never worked \(so you don't need to remember this stuff\):

```typescript
[] + []; // JavaScript will give you "" (which makes little sense), TypeScript will error

//
// other things that are nonsensical in JavaScript
// - don't give a runtime error (making debugging hard)
// - but TypeScript will give a compile time error (making debugging unnecessary)
//
{} + []; // JS : 0, TS Error
[] + {}; // JS : "[object Object]", TS Error
{} + {}; // JS : NaN or [object Object][object Object] depending upon browser, TS Error
"hello" - 1; // JS : NaN, TS Error

function add(a,b) {
  return
    a + b; // JS : undefined, TS Error 'unreachable code detected'
}
```

Essentially TypeScript is linting JavaScript. Just doing a better job at it than other linters that don't have _type information_.

## You still need to learn JavaScript

That said TypeScript is very pragmatic about the fact that _you do write JavaScript_ so there are some things about JavaScript that you still need to know in order to not be caught off-guard. Let's discuss them next.

> Note: TypeScript is a superset of JavaScript. Just with documentation that can actually be used by compilers / IDEs ;\)

