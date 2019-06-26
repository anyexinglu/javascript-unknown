Unknown things of javascript, inspired by [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS) and [javascript-questions](https://github.com/anyexinglu/javascript-questions)

BTW, all the following examples are running in non-strict mode.


## react

All the following codes can run within https://codesandbox.io

---

###### 1 What's the output?

```javascript
a = {
  get val() {
    return this.val;
  },
  set val(x) {
    this.val = x;
  }
};
a.val = 1;
console.log(a.val);
```

<details><summary><b>Answer</b></summary>
<p>

```
Uncaught RangeError: Maximum call stack size exceeded
    at Object.set val [as val] (<anonymous>:6:12)
```

`this.val = x` invokes the setter again, which leads to an infinite loop. To avoid this, you need to store the actual value in a separate field of the object (e.g. this.\_val), and then have your getter return that value. Here's an example:

```javascript
a = {
  get val() {
    return this._val;
  },
  set val(x) {
    this._val = x;
  }
};
a.val = 1;
console.log(a.val);
```

[reference](https://stackoverflow.com/questions/43780287/javascript-uncaught-rangeerror-maximum-call-stack-size-exceeded)

</p>
</details>

---


## javascript

---

###### 1 What's the output?

```javascript
a = {
  get val() {
    return this.val;
  },
  set val(x) {
    this.val = x;
  }
};
a.val = 1;
console.log(a.val);
```

<details><summary><b>Answer</b></summary>
<p>

```
Uncaught RangeError: Maximum call stack size exceeded
    at Object.set val [as val] (<anonymous>:6:12)
```

`this.val = x` invokes the setter again, which leads to an infinite loop. To avoid this, you need to store the actual value in a separate field of the object (e.g. this.\_val), and then have your getter return that value. Here's an example:

```javascript
a = {
  get val() {
    return this._val;
  },
  set val(x) {
    this._val = x;
  }
};
a.val = 1;
console.log(a.val);
```

[reference](https://stackoverflow.com/questions/43780287/javascript-uncaught-rangeerror-maximum-call-stack-size-exceeded)

</p>
</details>

---

###### 2 What's the output?

```javascript
function delay(func, ms) {
  clearTimeout(func.timer);
  func.timer = setTimeout(function(args) {
    func(args);
  }, ms);
}
obj = {
  a: 1,
  getA() {
    console.log("getA: ", this.a);
  }
};
delay(obj.getA, 100);
```

<details><summary><b>Answer</b></summary>
<p>

Output: `getA: undefined`. Why?

- `func` is just a reference to `obj.getA`
- `func(args)` will execute in global environment, and `this.a` refers to `window.a`, turns to be `undefined`.

The right way:

```javascript
function delay(func, context, ms) {
  clearTimeout(func.timer);
  func.timer = setTimeout(function(args) {
    func.call(context, args);
  }, ms);
}
obj = {
  a: 1,
  getA() {
    console.log("getA: ", this.a);
  }
};
delay(obj.getA, obj, 100);
```

The `context` should be provided, in order to output `getA: 1`

</p>
</details>

---

###### 3 What's the output?

```javascript
obj = {
  a: 1,
  getA() {
    console.log("getA: ", this.a);
  }
};
setTimeout(obj.getA, 100);
```

<details><summary><b>Answer</b></summary>
<p>

Output: `getA: undefined`. The right way:

```javascript
setTimeout(obj.getA.bind(obj), 100);
```

What know why? Read the next question⬇

</p>
</details>

---

###### 4 What's the output?

```javascript
function doFoo(fn) {
  fn();
}
function getA() {
  console.log("getA: ", this.a);
}
obj = {
  a: 1,
  getA
};
doFoo(obj.getA);
```

<details><summary><b>Answer</b></summary>
<p>

Output: `getA: undefined`.

Another case will also output `getA: undefined`:

```javascript
obj = {
  a: 1,
  getA() {
    console.log("getA: ", this.a);
  }
};
let getA = obj.getA;
getA();
```

Why?

- It's `Implicitly Lost`
- Even though getA appears to be a reference to obj.getA, in fact, it's really just another reference to getA itself. Moreover, the call-site is what matters, and the call-site is getA(), which is a plain, un-decorated call and thus the `default binding` applies.
- 被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把 this 绑定到全局对象或者 undefined 上，取决于是否是严格模式。

[reference](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#implicitly-lost)

</p>
</details>

---

###### 5 What's the output?

```javascript
obj = {
  a: 1,
  getA: () => {
    console.log("getA: ", this.a);
  }
};
setTimeout(obj.getA.bind(obj), 100);
```

<details><summary><b>Answer</b></summary>
<p>

Output: `getA: undefined`.

Arrow functions can never have their own this bound. Instead, they always delegate to the lexical scope (Window).

[reference](https://medium.com/javascript-scene/what-is-this-the-inner-workings-of-javascript-objects-d397bfa0708a)

</p>
</details>

---

###### 6 What's the output?

```javascript
let boss1 = { name: "boss1" };
let boss2 = { name: "boss2" };
let boss1returnThis = function() {
  return this;
}.bind(boss1);
console.log(boss1returnThis.bind(boss2)());
console.log(boss1returnThis.apply(boss2));
console.log(boss1returnThis.call(boss2));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
{
  name: "boss1";
}
{
  name: "boss1";
}
{
  name: "boss1";
}
```

Why?

- For binded this, it cannot be reassigned, even with .bind(), .apply() or .call()

[reference](https://medium.com/javascript-scene/what-is-this-the-inner-workings-of-javascript-objects-d397bfa0708a)

</p>
</details>

---

###### 7 What's the output?

```javascript
let boss1 = { name: "boss1" };
let boss2 = { name: "boss2" };
// Begin pay attention
let boss1returnThis = (() => {
  return this;
}).bind(boss1);
// End pay attention
console.log(boss1returnThis.bind(boss2)());
console.log(boss1returnThis.apply(boss2));
console.log(boss1returnThis.call(boss2));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
Window;
Window;
Window;
```

Why?

- Arrow functions can never have their own this bound. Instead, they always delegate to the lexical scope (Window).
- For arrow functions, this can't be reassigned, even with .bind(), .apply() or .call()

[reference](https://medium.com/javascript-scene/what-is-this-the-inner-workings-of-javascript-objects-d397bfa0708a)

</p>
</details>

---

###### 8 What's the output?

```javascript
function foo() {
  let a = 2;
  this.bar();
}

function bar() {
  console.log(this.a);
}

foo();
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
undefined;
```

Why?

- Every time you feel yourself trying to mix lexical scope look-ups with this, remind yourself: there is no bridge.

[reference](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch1.md#its-scope)

</p>
</details>

---

###### 9 What's the output?

```javascript
var value = 1;
var foo = {
  value: 2,
  bar: function() {
    return this.value;
  }
};
console.log(foo.bar());
console.log(foo.bar());
console.log((foo.bar = foo.bar)());
console.log((false || foo.bar)());
console.log((foo.bar, foo.bar)());
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
2;
2;
1;
1;
1;
```

Why?

- Last 3 console.log do apply GetValue to the result of evaluating Expression.
- GetValue(lref) changes `this` to be global(window).

[reference](https://github.com/mqyqingfeng/Blog/issues/7)

</p>
</details>

---

###### 10 What's the output?

```javascript
// Begin pay attention
let value = 1;
let foo = {
  value: 2,
  bar: function() {
    return this.value;
  }
};
// End pay attention
console.log(foo.bar());
console.log(foo.bar());
console.log((foo.bar = foo.bar)());
console.log((false || foo.bar)());
console.log((foo.bar, foo.bar)());
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
2;
2;
undefined;
undefined;
undefined;
```

Why?

- `let` is not global while `var` is.

So the following code will output `1 undefined 2`

```javascript
let a = 1;
var b = 2;
console.log(a, window.a, window.b);
```

</p>
</details>

---

###### 11 What's the output?

```javascript
x = Symbol("x");
a = [2, 3, 4, 5, 6, 7];
a.b = 1;
a[x] = 0;
for (let key in a) {
  console.log(key);
}
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
0;
1;
2;
3;
4;
5;
b;
```

Why?

- `for ... in` loop will iterates all enumerable, non-Symbol properties.

</p>
</details>

---

###### 12 What's the output?

```javascript
x = Symbol("x");
a = [2, 3, 4, 5, 6, 7];
a.b = 1;
a[x] = 0;
for (let val of a) {
  console.log(val);
}
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
2;
3;
4;
5;
6;
7;
```

Why?

- The for...in statement iterates over the enumerable, non-Symbol properties of an object, in an arbitrary order.
- The for...of statement iterates over values that the iterable object defines to be iterated over.

[reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of#Difference_between_for...of_and_for...in)

</p>
</details>

---

###### 13 What's the output?

```javascript
class A {
  x = 1;
  getX() {
    return this.x;
  }
}
a = new A();
b = Object.assign({}, a);
c = { ...a };
console.log(b, c, "getX" in b, "getX" in c);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
`{x: 1} {x: 1} false false`;
```

Why?

- `Object.assign` & `...` & `...in...` only iterates enumerable, non-Symbol properties of the given object directly, excluding the properties of `x.__proto__`, `getter` and `setter`.

</p>
</details>

---

###### 14 What's the output?

```javascript
obj = { a: 1 };
x = Object.create(obj);
Object.defineProperty(x, "b", {
  value: 2,
  enumerable: false
});
x.c = 3;
for (let k in x) {
  console.log("key: " + k);
}
console.log(Object.getOwnPropertyNames(x));
console.log(Object.keys(x));
console.log(Object.assign({}, x));
JSON.stringify(x);
console.log(x.hasOwnProperty("a"), x.hasOwnProperty("c"));
console.log("a" in x, "c" in x);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
key: c;
key: a;
["b", "c"];
["c"]
{c: 3}
"{"c":3}"
false true
true true
```

Why?

- `x = Object.create(obj)` creates a new object, using the existing object `obj` as the prototype of the newly created object `x`.

Remember the keywords:

- `for...in`: excluding `non-enumerable`, including `__proto__`
- `Object.getOwnPropertyNames` & `hasOwnProperty`: including `non-enumerable`, excluding `__proto__`
- `Object.keys` & `Object.assign` & `JSON.stringify`: excluding `non-enumerable` & `__proto__`
- `... in ...`: including `non-enumerable` & `__proto__`

</p>
</details>

---

###### 15 What's the output?

```javascript
a = { x: 2 };
b = Object.create(a);
console.log(b.hasOwnProperty("x"));
b.x++;
console.log(b.hasOwnProperty("x"));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
false;
true;
```

Why?

- Object.create creates a new object, using the existing object as the prototype of the newly created object.
- `b.x++` will run `b.x = b.x + 1`, which will add own property `x` for `b`.

</p>
</details>

---

###### 16 What's the output?

```javascript
function A(name) {
  this.name = name;
}
A.prototype.myName = function() {
  return this.name;
};

function B(name, label) {
  A.call(this, name);
  this.label = label;
}

function C(name, label) {
  A.call(this, name);
  this.label = label;
}

B.prototype = A.prototype;
C.prototype = new A();
B.prototype.myName = function() {
  return 111;
};

x = new A("xxx");
y = new B("yyy");
z = new C("zzz");
console.log(x.myName(), y.myName(), z.myName());
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
111 111 111
```

Why?

- `B.prototype = A.prototype` is assign the reference of object `A.prototype` to `B.prototype`, so `B.prototype.myName=....` changes `A.prototype.myName`.
- `new A()` returns `{name: undefined}`, `C.prototype = new A()` means `C.prototype = {name: undefined}`.
- Since `z.__proto__`( === `C.prototype`) has no `myName`, so `z.myName` will be `z.__proto__.__proto__.myName`( === `C.prototype.__proto__.myName`)
- Since `C.prototype.__proto__ === A.prototype`, so `C.prototype.__proto__.myName` will be `A.prototype.myName`, which has changed by `B.prototype.myName=....`.

So how to make `A.prototype.myName` unchanged when setting `B.prototype.myName=....`?
Fix `B.prototype = A.prototype` by `B.prototype = Object.create(A.prototype)`

</p>
</details>

---

###### 17 What's the output?

```javascript
class C {
  constructor() {
    this.num = Math.random();
  }
}
c1 = new C();
C.prototype.rand = function() {
  console.log("Random: " + Math.round(this.num * 1000));
};
c1.rand();
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
Random: 890; // a random number between 0~1000
```

Why?

- `class` in js is made with [[Prototype]], so `c1.__proto__` === `C.prototype`

</p>
</details>

---

###### 18 What's the output?

```javascript
function Test(x) {
  function F() {}
  F.prototype = x;
  return new F();
}
o = {
  x: 1,
  getX: function() {
    return 111;
  }
};
p = Test(o);
q = Object.create(o);

console.log(p);
console.log(q);
console.log(p.__proto__ === q.__proto__);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
F {}
{}
true
```

Why?

- `p.__proto__` equals `(new F()).__proto__` equals `F.prototype` equals `o`
- `q = Object.create(o)` makes `q.__proto__` equals `o`
- `Test` is polyfill of `Object.create` for browsers which doesn't support es5. [reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create#Polyfill)
- So, don't mock class by setting `__proto__` / `prototype` / `new`, just use `Object.create`:

```javascript
let Widget = {
  init: function(width, height) {
    this.width = width || 50;
  }
};
let Button = Object.create(Widget);
Button.setup = function(width, height, label) {
  this.init(width, height);
  this.label = label || "Default";
};
```

</p>
</details>

###### 19 What's the output?

```javascript
function Animal(name) {
  this.name = name || "Animal";
  this.sleep = function() {
    console.log(this.name + "is sleeping.");
  };
}
function Cat() {}
Cat.prototype = new Animal();
Cat.prototype.name = "Cat";

function Dog() {}
Dog.prototype = Object.create(Animal.prototype);

cat = new Cat();
dog = new Dog();
Animal.prototype.eat = function(food) {
  console.log(this.name + " is eating " + food);
};
console.log(cat.eat("fish"));
console.log(dog.eat("rice"));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
cat is eating fish
dog is eating rice
```

Why?

- `cat.__proto__.__proto__` equals `(Cat.prototype).__proto__` equals `Animal.prototype`
- `cat.eat('fish')` will call`cat.__proto__.__proto__.eat('fish')`
- `dog.__proto__` equals `Dog.prototype` equals `Animal.prototype`
- `dog.eat("rice")` will call`dog.__proto__.eat('rice')`

It means that properties of `Animal.prototype` will be shared by all instances, including those `inherited` earlier.

</p>
</details>

---

###### 20 What's the output?

```javascript
a = [1, 2, 3, 4];
delete a[1];
console.log(a.length);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
4;
```

Why?

- After `delete a[1]`, a becomes `[1, empty, 3, 4]`

</p>
</details>

---

###### 21 What's the output?

```javascript
console.log(42.toFixed(3));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
Uncaught SyntaxError: Invalid or unexpected token
```

Why?

Within `42.toFixed(3)`, the `.` will be regarded as a part of number, so `(42.)toFixed(3)` throws error.

// Correct:

- (42).toFixed(3); // "42.000"
- num = 42; num.toFixed(3); // "42.000"
- 42..toFixed(3); // "42.000"
  </p>
  </details>

---

###### 22 What's the output?

```javascript
console.log(0.1 + 0.2 === 0.3);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
false;
```

Why?

For lauguage following `IEEE 754` rule such as javascript, `0.1 + 0.2` outputs `0.30000000000000004`.

A safe way to comprare values:

```javascript
function numbersCloseEnoughToEqual(n1, n2) {
  return Math.abs(n1 - n2) < Number.EPSILON; // Number.EPSILON: 2.220446049250313e-16
}
let a = 0.1 + 0.2;
let b = 0.3;
numbersCloseEnoughToEqual(a, b);
```

</p>
</details>

---
