Unknown things of javascript, inspired by [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS) and (javascript-questions)[https://github.com/anyexinglu/javascript-questions]

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

output: `getA: undefined`

`func(args)` is the same to `getA(window)`. Here is the right way:

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
function sayHi() {
  console.log(name);
  console.log(age);
  var name = "Lydia";
  let age = 21;
}

sayHi();
```

<details><summary><b>Answer</b></summary>
<p>

</p>
</details>

---
