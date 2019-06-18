Unknown things of javascript, inspired by [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS) and [javascript-questions](https://github.com/anyexinglu/javascript-questions)

BTW, all the code runs within `loose` environment

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

Output: `getA: undefined`

`func(args)` will be `getA(window)` and `window.a` is `undefined`.

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
  console.log(this.a);
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

- Why?
- `bind` / `call` / `apply` cannot change the reference of `this` within a `bind`ed function

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
- For arrow functions, this can't be reassigned, even with .call() or .bind()

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

- GetValue(lref) changes `this` to be global(window).

[reference](https://github.com/mqyqingfeng/Blog/issues/7)

</p>
</details>

---

###### 10 What's the output?

```javascript
let value = 1;
let foo = {
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
