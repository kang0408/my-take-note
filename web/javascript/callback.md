## 1. What is a callback function?

> A **callback function** ís a function that is passed as _an argument_ to another function, to be "called back"

A function accepts other function as arguments is called a **higher-order function**

```javascript
function sum(a, b, callback) {
  let sum = a + b;
  callback(sum);
}

function alertSum(sum) {
  console.log("The sum: " + sum);
}

sum(2, 3, alertSum());

// Result in console:
// The sum: 5
```

## 2. Synchronous and Asynchronous

### 2.1. Synchronous

> **Synchronous** : every statement of the code gets executed in a sequence, one by one. This mean a statement has to wait for the earlier statement to get executed

```javascript
function hello() => {
    console.log("hello");
}

function world() => {
    console.log("world!");
}

hello();
world();

// result in console
// hello
// world!
```

### 2.2. Asynchronous

>

## 1. Hàm callback là gì?

> Một **hàm callback** là hàm được truyền vào như một đối số của hàm khác

> Hàm nhận những hàm khác như một đối số được gọi là **hàm bậc cao**

```javascript
function sum(a, b, callback) {
  let sum = a + b;
  callback(sum);
}

function alertSum(sum) {
  console.log("The sum: " + sum);
}

sum(2, 3, alertSum());

// Result in console:
// The sum: 5
```
