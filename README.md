Unknown things of javascript, inspired by [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS) and [javascript-questions](https://github.com/anyexinglu/javascript-questions)

BTW, all the following examples are running in non-strict mode.

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
- `func(args)` will execute `obj.getA()` in global environment, and `this.a` refers to `window.a`, turns to be `undefined`.

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
a = [2, 3, 4, 5, 6, 7];
a.b = "b";
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

- `for ... in` loop will iterates all enumerable, non-Symbol properties, includes `b`.

</p>
</details>

---

###### 12 What's the output?

```javascript
a = [2, 3, 4, 5, 6, 7];
a.b = "b";
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

- The for...in statement iterates over the enumerable properties of an object, in an arbitrary order.
- The for...of statement iterates over values that the iterable object defines to be iterated over.

[reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of#Difference_between_for...of_and_for...in)

</p>
</details>

---
