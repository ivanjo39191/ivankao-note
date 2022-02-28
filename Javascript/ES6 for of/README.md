# ES6 for of

Created: February 1, 2022 5:22 PM
Tags: Javascript, Vue

for...of 語法：

```
for (variableof iterable) {
    statement
}
```

其中 variable 直接是值，而不是索引。

遍歷陣列：

```
let iterable = [10, 20, 30];

for (let valueof iterable) {
    value += 1;
    console.log(value);
}

// 依序輸出// 11// 21// 31
```

遍歷字串：

```
let iterable = 'boo';

for (letvalue of iterable) {
    console.log(value);
}

// 依序輸出// "b"// "o"// "o"
```

[https://www.fooish.com/javascript/ES6/for-of.html](https://www.fooish.com/javascript/ES6/for-of.html)