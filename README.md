Unknown things of javascript, inspired by [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS) and [javascript-questions](https://github.com/anyexinglu/javascript-questions)

BTW, all the following examples are running in non-strict mode.

- [react](https://github.com/anyexinglu/javascript-unknown/blob/master/README.md#react)
- [javascript](https://github.com/anyexinglu/javascript-unknown/blob/master/README.md#javascript)

## react

All the following codes can run within https://codesandbox.io/s/new

---

###### 1 What's the output?

Click `Console.log`, then `Click me`. Wait for 3 seconds, what's the output?

```javascript
import React from "react";
import ReactDOM from "react-dom";
const { useState } = React;

function Demo() {
  const [count, setCount] = useState(0);

  function log() {
    setTimeout(() => {
      console.log("count: " + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
      <button onClick={log}>Console.log</button>
    </div>
  );
}
function App() {
  const [list, setList] = useState([]);
  const handleChange = list => setList(list);
  return <Demo list={list} onChange={handleChange} />;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

<details><summary><b>Answer</b></summary>
<p>

```
count: 0
```

Why?

Within every rerender, the `count` is new.

</p>
</details>

---

###### 2 What's the output?

Click `add`, what's the output?

```javascript
import React from "react";
import ReactDOM from "react-dom";
const { useState, useRef } = React;

function Demo(props) {
  const { list, onChange } = props;

  const cloneList = [...list];
  const cloneListRef = useRef(cloneList);
  console.log("...render", cloneListRef.current === cloneList);
  return (
    <div>
      <button onClick={() => onChange([...list, 1])}>add</button>
      list: {JSON.stringify(list)}
    </div>
  );
}

function App() {
  const [list, setList] = useState([]);
  const handleChange = list => setList(list);
  return <Demo list={list} onChange={handleChange} />;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

<details><summary><b>Answer</b></summary>
<p>

```
...render true // initial render
...render false
```

Why?

Within every rerender, the `cloneList` is new.
And only in the first render, the `cloneList` equals `cloneListRef.current`.

</p>
</details>

---

###### 3 What's the output?

```javascript
import React from "react";
import ReactDOM from "react-dom";
const { useState } = React;

let id = 0;

function Demo(props) {
  const { list, onChange } = props;
  const onStatus = payload => {
    if (payload.status === "posting") {
      onChange([...list, payload]);
    } else if (payload.status === "done") {
      const newlist = list.map(item => {
        if (item.id === payload.id) {
          return payload;
        }
        return item;
      });
      onChange(newlist);
    }
  };

  const post = async () => {
    const payload = { id: id };
    onStatus({ ...payload, status: "posting" });
    return new Promise((resolve, reject) => {
      id++;
      setTimeout(() => {
        resolve(payload);
      }, 3000);
    }).then(payload => {
      onStatus({ ...payload, status: "done" });
    });
  };

  console.log("list", list);

  return (
    <div>
      <button onClick={() => post()}>post</button>
      list: {JSON.stringify(list)}
    </div>
  );
}

function App() {
  const [list, setList] = useState([]);
  const handleChange = list => setList(list);
  return <Demo list={list} onChange={handleChange} />;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

<details><summary><b>Answer</b></summary>
<p>

```
list: []
list: [{id: 1, status: 'posting'}]
list: []
```

Why?

Within `else if (payload.status === "done") { const newlist = list.map(item => {`, the `list` is the original value `[]`.

</p>
</details>

---

###### 4 What's the output? (use a `cloneList` base on #3)

Double click the button `post` quickly

```javascript
import React from "react";
import ReactDOM from "react-dom";
const { useState } = React;

let id = 0;

function Demo(props) {
  const { list, onChange } = props;

  // Begin different with question 1
  const cloneList = [...list];
  const onStatus = payload => {
    if (payload.status === "posting") {
      cloneList.push(payload);
      onChange(cloneList);
    } else if (payload.status === "done") {
      const newlist = cloneList.map(item => {
        if (item.id === payload.id) {
          return payload;
        }
        return item;
      });
      onChange(newlist);
    }
  };
  // End different with question 1

  const post = async () => {
    const payload = { id: id };
    onStatus({ ...payload, status: "posting" });
    return new Promise((resolve, reject) => {
      id++;
      setTimeout(() => {
        resolve(payload);
      }, 3000);
    }).then(payload => {
      onStatus({ ...payload, status: "done" });
    });
  };

  console.log("list", list);

  return (
    <div>
      <button onClick={() => post()}>post</button>
      list: {JSON.stringify(list)}
    </div>
  );
}

function App() {
  const [list, setList] = useState([]);
  const handleChange = list => setList(list);
  return <Demo list={list} onChange={handleChange} />;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

<details><summary><b>Answer</b></summary>
<p>

```
list: []
list: [{id: 1, status: 'posting'}]
list: [{id: 1, status: 'posting'}, {id: 2, status: 'posting'}]
list: [{id: 1, status: 'posting'}, {id: 2, status: 'done'}]
```

Why?

Within every rerender, the `cloneList` is new.
How to fix it? Use `useRef`?

</p>
</details>

---

###### 5 What's the output? (`useRef` base on #4)

Double click the button `post` quickly

```javascript
import React from "react";
import ReactDOM from "react-dom";
const { useState, useRef } = React;

let id = 0;

function Demo(props) {
  const { list, onChange } = props;

  const cloneListRef = useRef(list);

  const onStatus = payload => {
    let cloneList = cloneListRef && cloneListRef.current.slice();
    if (payload.status === "posting") {
      cloneList.push(payload);
      onChange(cloneList);
    } else if (payload.status === "done") {
      const newlist = cloneList.map(item => {
        if (item.id === payload.id) {
          return payload;
        }
        return item;
      });
      onChange(newlist);
    }
  };

  const post = async () => {
    const payload = { id: id };
    onStatus({ ...payload, status: "posting" });
    return new Promise((resolve, reject) => {
      id++;
      setTimeout(() => {
        resolve(payload);
      }, 3000);
    }).then(payload => {
      onStatus({ ...payload, status: "done" });
    });
  };
  console.log("list", list);

  return (
    <div>
      <button onClick={() => post()}>post</button>
      list: {JSON.stringify(list)}
    </div>
  );
}

function App() {
  const [list, setList] = useState([]);
  const handleChange = list => setList(list);
  return <Demo list={list} onChange={handleChange} />;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

<details><summary><b>Answer</b></summary>
<p>

```
list: []
list: [{id: 0, status: 'posting'}]
list: [{id: 1, status: 'posting'}]
list: []
list: []
```

Why?

`cloneListRef` cached the value of `list` in first render, and then never change.

</p>
</details>

---

###### 6 What's the output? (update the ref value base on #5)

Double click the button `post` quickly

```javascript
import React from "react";
import ReactDOM from "react-dom";
const { useState, useRef, useEffect } = React;

let id = 0;

function Demo(props) {
  const { list, onChange } = props;

  const cloneListRef = useRef(list);
  useEffect(() => {
    cloneListRef.current = list;
  }, [list]);

  const onStatus = payload => {
    let cloneList = cloneListRef && cloneListRef.current.slice();
    if (payload.status === "posting") {
      cloneList.push(payload);
      onChange(cloneList);
    } else if (payload.status === "done") {
      const newlist = cloneList.map(item => {
        if (item.id === payload.id) {
          return payload;
        }
        return item;
      });
      onChange(newlist);
    }
  };

  const post = async () => {
    const payload = { id: id };
    onStatus({ ...payload, status: "posting" });
    return new Promise((resolve, reject) => {
      id++;
      setTimeout(() => {
        resolve(payload);
      }, 3000);
    }).then(payload => {
      onStatus({ ...payload, status: "done" });
    });
  };
  console.log("list", list);

  return (
    <div>
      <button onClick={() => post()}>post</button>
      list: {JSON.stringify(list)}
    </div>
  );
}

function App() {
  const [list, setList] = useState([]);
  const handleChange = list => setList(list);
  return <Demo list={list} onChange={handleChange} />;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

<details><summary><b>Answer</b></summary>
<p>

```
list: []
list: [{id: 0, status: 'posting'}]
list: [{id: 0, status: 'posting'}, {id: 1, status: 'posting'}]
list: [{id: 0, status: 'done'}, {id: 1, status: 'posting'}]
list: [{id: 0, status: 'done'}, {id: 1, status: 'done'}]
```

Why?

Sync the value list to `cloneListRef.current` immediately, and the `onStatus` will use the updated value.

</p>
</details>
 
---

## javascript


---

## 1. js basic types

---

###### 1.1 What's the output?

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

###### 1.2 What's the output?

```javascript
let list = [1, 2, 3, 4];
let alreadyList = [2, 3];

let cloneList = [...list];

for (let i = 0; i < list.length - 1; i++) {
  let item = list[i];
  if (alreadyList.includes(item)) {
    cloneList.splice(i, 1);
  }
}

console.log("...", cloneList);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
[1, 3];
```

Why?

- After `delete 2 - cloneList[1]`, cloneList becomes `[1, 3, 4]`
- When `delete 3 - cloneList[2]`, cloneList becomes `[1, 3]`

</p>
</details>

---

###### 1.3 What's the output?

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

###### 1.4 What's the output?

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

###### 1.5 What's the output?

```javascript
a = "12" + 9;
console.log(a, typeof a);
b = "12" - 9;
console.log(b, typeof b);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
129 string
3 "number"
```

Why?

- `string + number` will transform number to string, outputs string
- `string - number` will transform string to number, outputs number

</p>
</details>

---

###### 1.6 What's the output?

Run seperately:

```javascript
JSON.stringify(undefined);

JSON.stringify(function() {});

JSON.stringify([1, undefined, function() {}, 4, new Date()]);

JSON.stringify({ a: 2, b: function() {}, c: Symbol.for("ccc"), d: 1 });
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
undefined
undefined
[1,null,null,4,"2019-08-14T01:52:25.428Z"]
{"a":2,"d":1}
```

Why?

JSON.stringify will ignore `undefined`, `function`, `symbol`

</p>
</details>

---

###### 1.7 What's the output?

```javascript
a = Array(3);
b = new Array(3);
c = Array.apply(null, { length: 3 });
d = [undefined, undefined, undefined];

console.log(
  a.map(function(v, i) {
    return i;
  })
);
console.log(
  b.map(function(v, i) {
    return i;
  })
);
console.log(
  c.map(function(v, i) {
    return i;
  })
);
console.log(
  d.map(function(v, i) {
    return i;
  })
);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

Different browsers may behave differently, while within current Chrome, the output is:

```javascript
[empty × 3]
[empty × 3]
[0, 1, 2]
[0, 1, 2]
```

Why?

- `Array(num)` is as same as `new Array(num)`, since the browser will auto add `new` in before of `Array(num)`
- `new Array(3)` create a array, in which every member is `empty` unit (`undefined` type).
- `a.map(..)` & `b.map(..)` will be failed, as the array is full of `empty`, `map` will not iterate them.

</p>
</details>

---

###### 1.8 What's the output?

```javascript
x = [1, 2, { a: 1 }];
y = x;
z = [...x];
y[0] = 2;
(y[2].b = 2), (z[2].a = 4);
console.log(x, y, z);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
[2, 2, { a: 4, b: 2 }][(2, 2, { a: 4, b: 2 })][(1, 2, { a: 4, b: 2 })];
```

Why?

- `z = [...x]` is shallow copy

</p>
</details>

---

###### 1.9 What's the output?

```javascript
a = new Array(3);
b = [undefined, undefined, undefined];

console.log(a.join("-"));
console.log(b.join("-"));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

Different browsers may behave differently, while within current Chrome, the output is:

```javascript
--
--
```

Why?

`join` works differently with `map`:

```javascript
function fakeJoin(arr, connector) {
  var str = "";
  for (var i = 0; i < arr.length; i++) {
    if (i > 0) {
      str += connector;
    }
    if (arr[i] !== undefined) {
      str += arr[i];
    }
  }
  return str;
}
var a = new Array(3);
fakeJoin(a, "-"); // "--"
```

</p>
</details>

---

## 2. this

---

###### 2.1 What's the output?

```javascript
obj = {
  a: 1,
  getA() {
    console.log("getA: ", this.a);
  }
};
obj.getA();
x = obj.getA;
x();
setTimeout(obj.getA, 100);
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
getA:  1
getA:  undefined
（a timerId number）
getA:  undefined
```

Why：

- It's `Implicitly Lost`
- Even though getA appears to be a reference to obj.getA, in fact, it's really just another reference to getA itself. Moreover, the call-site is what matters, and the call-site is getA(), which is a plain, un-decorated call and thus the `default binding` applies.
- `default binding` makes `this` the global(`Window`) or undefined (depends on if this is `strict mode`).

[reference](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md#implicitly-lost)

Question：How to change `x(); setTimeout(obj.getA, 100);`, make it output `getA: 1`.

</p>
</details>

---

###### 1.2 What's the output?

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

###### 2.3 What's the output?

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

###### 2.4 What's the output?

```javascript
let boss1 = { name: "boss1" };
let boss2 = { name: "boss2" };
let boss1returnThis = function() {
  return this.name;
}.bind(boss1);
console.log(boss1returnThis.bind(boss2)());
console.log(boss1returnThis.apply(boss2));
console.log(boss1returnThis.call(boss2));
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
boss1;
boss1;
boss1;
```

Why?

- For binded this, it cannot be reassigned, even with .bind(), .apply() or .call()

[reference](https://medium.com/javascript-scene/what-is-this-the-inner-workings-of-javascript-objects-d397bfa0708a)

</p>
</details>

---

###### 2.5 What's the output?

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

###### 2.6 What's the output?

```javascript
var value = 1;
var foo = {
  value: 2,
  bar: function() {
    return this.value;
  }
};

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

###### 2.7 What's the output?

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
console.log((foo.bar = foo.bar)());
console.log((false || foo.bar)());
console.log((foo.bar, foo.bar)());
```

<details><summary><b>Answer</b></summary>
<p>

Output:

```javascript
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

## 3. property

---

###### 3.1 What's the output?

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

###### 3.2 What's the output?

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

###### 3.3 What's the output?

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

###### 3.4 What's the output?

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

###### 3.5 What's the output?

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

## 4. `__proto__` && `prototype`

---

###### 4.1 What's the output?

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

###### 4.2 What's the output?

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

###### 4.3 What's the output?

```javascript
function Test(oo) {
  function F() {}
  F.prototype = oo;
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

###### 4.4 What's the output?

```javascript
function Animal(name) {
  this.name = name || "Animal";
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
