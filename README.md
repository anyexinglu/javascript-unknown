# javascript-unknown
Unknown things of javascript, inspired by https://github.com/getify/You-Dont-Know-JS and https://github.com/anyexinglu/javascript-questions



---
###### 1.1 What's the output?

```javascript
a = {
  get val() {
    return this.val
  },
  set val(x) {
    this.val = x;
  }
}
a.val = 1
console.log(a.val)
```
<details><summary><b>Answer</b></summary>
<p>

```
Uncaught RangeError: Maximum call stack size exceeded
    at Object.set val [as val] (<anonymous>:6:12)
```

`this.val = x` invokes the setter again, which leads to an infinite loop. To avoid this, you need to store the actual value in a separate field of the object (e.g. this._val), and then have your getter return that value. Here's an example:
```javascript
a = {
  get val() {
    return this._val
  },
  set val(x) {
    this._val = x;
  }
}
a.val = 1
console.log(a.val)
```
[reference](https://stackoverflow.com/questions/43780287/javascript-uncaught-rangeerror-maximum-call-stack-size-exceeded)
</p>
</details>

---




### 1. 对象属性

---

###### 1.1 What's the output?

```javascript
function throttle(method, context) {
  clearTimeout(method.tid);
  method.tid = setTimeout(function() {
    method.call(context)
  }, 100)
}

sayHi();
```
<details><summary><b>Answer</b></summary>
<p>

#### Answer: 

</p>
</details>

---



### 2. this指针

---

###### 2.1 What's the output?

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

#### Answer: 

</p>
</details>

---
