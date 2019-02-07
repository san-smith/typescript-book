# Classes Emit

### What's up with the IIFE

The js generated for the class could have been:

```typescript
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.add = function (point) {
    return new Point(this.x + point.x, this.y + point.y);
};
```

The reason it's wrapped in an Immediately-Invoked Function Expression \(IIFE\) i.e.

```typescript
(function () {

    // BODY

    return Point;
})();
```

has to do with inheritance. It allows TypeScript to capture the base class as a variable `_super` e.g.

```typescript
var Point3D = (function (_super) {
    __extends(Point3D, _super);
    function Point3D(x, y, z) {
        _super.call(this, x, y);
        this.z = z;
    }
    Point3D.prototype.add = function (point) {
        var point2D = _super.prototype.add.call(this, point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    };
    return Point3D;
})(Point);
```

Notice that the IIFE allows TypeScript to easily capture the base class `Point` in a `_super` variable and that is used consistently in the class body.

## `__extends`

You will notice that as soon as you inherit a class TypeScript also generates the following function:

```typescript
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```

Here `d` refers to the derived class and `b` refers to the base class. This function does two things: 1. copies the static members of the base class onto the child class i.e. `for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];` 1. sets up the child class function's prototype to optionally lookup members on the parent's `proto` i.e. effectively `d.prototype.__proto__ = b.prototype`

People rarely have trouble understanding 1, but many people struggle with 2. So an explanation is in order.

### `d.prototype.__proto__ = b.prototype`

After having tutored many people about this I find the following explanation to be simplest. First we will explain how the code from `__extends` is equivalent to the simple `d.prototype.__proto__ = b.prototype`, and then why this line in itself is significant. To understand all this you need to know these things:

1. `__proto__`
2. `prototype`
3. effect of `new` on `this` inside the called function
4. effect of `new` on `prototype` and `__proto__`

All objects in JavaScript contain a `__proto__` member. This member is often not accessible in older browsers \(sometimes documentation refers to this magical property as `[[prototype]]`\). It has one objective: If a property is not found on an object during lookup \(e.g. `obj.property`\) then it is looked up at `obj.__proto__.property`. If it is still not found then `obj.__proto__.__proto__.property` till either: _it is found_ or _the latest_ `.__proto__` _itself is null_. This explains why JavaScript is said to support _prototypal inheritance_ out of the box. This is shown in the following example, which you can run in the chrome console or Node.js:

```typescript
var foo = {}

// setup on foo as well as foo.__proto__
foo.bar = 123;
foo.__proto__.bar = 456;

console.log(foo.bar); // 123
delete foo.bar; // remove from object
console.log(foo.bar); // 456
delete foo.__proto__.bar; // remove from foo.__proto__
console.log(foo.bar); // undefined
```

Cool so you understand `__proto__`. Another useful fact is that all `function`s in JavaScript have a property called `prototype` and that it has a member `constructor` pointing back to the function. This is shown below:

```typescript
function Foo() { }
console.log(Foo.prototype); // {} i.e. it exists and is not undefined
console.log(Foo.prototype.constructor === Foo); // Has a member called `constructor` pointing back to the function
```

Now let's look at _effect of_ `new` _on_ `this` _inside the called function_. Basically `this` inside the called function is going to point to the newly created object that will be returned from the function. It's simple to see if you mutate a property on `this` inside the function:

```typescript
function Foo() {
    this.bar = 123;
}

// call with the new operator
var newFoo = new Foo();
console.log(newFoo.bar); // 123
```

Now the only other thing you need to know is that calling `new` on a function assigns the `prototype` of the function to the `__proto__` of the newly created object that is returned from the function call. Here is the code you can run to completely understand it:

```typescript
function Foo() { }

var foo = new Foo();

console.log(foo.__proto__ === Foo.prototype); // True!
```

That's it. Now look at the following straight out of `__extends`. I've taken the liberty to number these lines:

```typescript
1  function __() { this.constructor = d; }
2   __.prototype = b.prototype;
3   d.prototype = new __();
```

Reading this function in reverse the `d.prototype = new __()` on line 3 effectively means `d.prototype = {__proto__ : __.prototype}` \(because of the effect of `new` on `prototype` and `__proto__`\), combining it with the previous line \(i.e. line 2 `__.prototype = b.prototype;`\) you get `d.prototype = {__proto__ : b.prototype}`.

But wait, we wanted `d.prototype.__proto__` i.e. just the proto changed and maintain the old `d.prototype.constructor`. This is where the significance of the first line \(i.e. `function __() { this.constructor = d; }`\) comes in. Here we will effectively have `d.prototype = {__proto__ : __.prototype, constructor : d}` \(because of the effect of `new` on `this` inside the called function\). So, since we restore `d.prototype.constructor`, the only thing we have truly mutated is the `__proto__` hence `d.prototype.__proto__ = b.prototype`.

### `d.prototype.__proto__ = b.prototype` significance

The significance is that it allows you to add member functions to a child class and inherit others from the base class. This is demonstrated by the following simple example:

```typescript
function Animal() { }
Animal.prototype.walk = function () { console.log('walk') };

function Bird() { }
Bird.prototype.__proto__ = Animal.prototype;
Bird.prototype.fly = function () { console.log('fly') };

var bird = new Bird();
bird.walk();
bird.fly();
```

Basically `bird.fly` will be looked up from `bird.__proto__.fly` \(remember that `new` makes the `bird.__proto__` point to `Bird.prototype`\) and `bird.walk` \(an inherited member\) will be looked up from `bird.__proto__.__proto__.walk` \(as `bird.__proto__ == Bird.prototype` and `bird.__proto__.__proto__` == `Animal.prototype`\).

