# Generics

## Generics

The key motivation for generics is to provide meaningful type constraints between members. The members can be:

* Class instance members
* Class methods
* function arguments
* function return value

## Motivation and samples

Consider the simple `Queue` \(first in, first out\) data structure implementation. A simple one in TypeScript / JavaScript looks like:

```typescript
class Queue {
  private data = [];
  push = (item) => this.data.push(item);
  pop = () => this.data.shift();
}
```

One issue with this implementation is that it allows people to add _anything_ to the queue and when they pop it - it can be _anything_. This is shown below, where someone can push a `string` onto the queue while the usage actually assumes that only `numbers` were pushed in:

```typescript
class Queue {
  private data = [];
  push = (item) => this.data.push(item);
  pop = () => this.data.shift();
}

const queue = new Queue();
queue.push(0);
queue.push("1"); // Oops a mistake

// a developer walks into a bar
console.log(queue.pop().toPrecision(1));
console.log(queue.pop().toPrecision(1)); // RUNTIME ERROR
```

One solution \(and in fact the only one in languages that don't support generics\) is to go ahead and create _special_ classes just for these constraints. E.g. a quick and dirty number queue:

```typescript
class QueueNumber {
  private data = [];
  push = (item: number) => this.data.push(item);
  pop = (): number => this.data.shift();
}

const queue = new QueueNumber();
queue.push(0);
queue.push("1"); // ERROR : cannot push a string. Only numbers allowed

// ^ if that error is fixed the rest would be fine too
```

Of course this can quickly become painful e.g. if you want a string queue you have to go through all that effort again. What you really want is a way to say that whatever the type is of the stuff getting _pushed_ it should be the same for whatever gets _popped_. This is done easily with a _generic_ parameter \(in this case, at the class level\):

```typescript
/** A class definition with a generic parameter */
class Queue<T> {
  private data = [];
  push = (item: T) => this.data.push(item);
  pop = (): T => this.data.shift();
}

/** Again sample usage */
const queue = new Queue<number>();
queue.push(0);
queue.push("1"); // ERROR : cannot push a string. Only numbers allowed

// ^ if that error is fixed the rest would be fine too
```

Another example that we have already seen is that of a _reverse_ function, here the constraint is between what gets passed into the function and what the function returns:

```typescript
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}

var sample = [1, 2, 3];
var reversed = reverse(sample);
console.log(reversed); // 3,2,1

// Safety!
reversed[0] = '1';     // Error!
reversed = ['1', '2']; // Error!

reversed[0] = 1;       // Okay
reversed = [1, 2];     // Okay
```

In this section you have seen examples of generics being defined _at class level_ and at _function level_. One minor addition worth mentioning is that you can have generics created just for a member function. As a toy example consider the following where we move the `reverse` function into a `Utility` class:

```typescript
class Utility {
  reverse<T>(items: T[]): T[] {
      var toreturn = [];
      for (let i = items.length - 1; i >= 0; i--) {
          toreturn.push(items[i]);
      }
      return toreturn;
  }
}
```

> TIP: You can call the generic parameter whatever you want. It is conventional to use `T`, `U`, `V` when you have simple generics. If you have more than one generic argument try to use meaningful names e.g. `TKey` and `TValue` \(conventional to prefix with `T` as generics are also called _templates_ in other languages e.g. C++\).

## Useless Generic

I've seen people use generics just for the heck of it. The question to ask is _what constraint are you trying to describe_. If you can't answer it easily you might have a useless generic. E.g. the following function

```typescript
declare function foo<T>(arg: T): void;
```

Here the generic `T` is completely useless as it is only used in a _single_ argument position. It might as well be:

```typescript
declare function foo(arg: any): void;
```

### Design Pattern: Convenience generic

Consider the function:

```typescript
declare function parse<T>(name: string): T;
```

In this case you can see that the type `T` is only used in one place. So there is no constraint _between_ members. This is equivalent to a type assertion in terms of type safety:

```typescript
declare function parse(name: string): any;

const something = parse('something') as TypeOfSomething;
```

Generics used _only once_ are no better than an assertion in terms of type safety. That said they do provide _convenience_ to your API.

A more obvious example is a function that loads a json response. It returns a promise of _whatever type you pass in_:

```typescript
const getJSON = <T>(config: {
    url: string,
    headers?: { [key: string]: string },
  }): Promise<T> => {
    const fetchConfig = ({
      method: 'GET',
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      ...(config.headers || {})
    });
    return fetch(config.url, fetchConfig)
      .then<T>(response => response.json());
  }
```

Note that you still have to explicitly annotate what you want, but the `getJSON<T>` signature `(config) => Promise<T>` saves you a few key strokes \(you don't need to annotate the return type of `loadUsers` as it can be inferred\):

```typescript
type LoadUsersResponse = {
  users: {
    name: string;
    email: string;
  }[];  // array of user objects
}
function loadUsers() {
  return getJSON<LoadUsersResponse>({ url: 'https://example.com/users' });
}
```

Also `Promise<T>` as a return value is definitely better than alternatives like `Promise<any>`.

