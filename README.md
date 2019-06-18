Unknown things of javascript, inspired by [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS) and [javascript-questions](https://github.com/anyexinglu/javascript-questions)

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

Within arrow function, `this` refers to the declare location, which equals `window`.

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
let boss1returnThis = (() => {
  return this;
}).bind(boss1);
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

- Why?
- `bind` / `call` / `apply` cannot change the reference of `this` within an arrow function

</p>
</details>

---
