---
layout: post
title:  "[CodeReview] Promise reduce 결과값 누적 반환 예시 코드"
subtitle:   "Promise reduce 순차처리하여 결과값 누적 반환 예시 코드 "
categories: doc
tags: nodejs
---

## Promise 를 reduce 로 순차적으로 처리하여 결과값을 누적 반환하는 예시 코드 
Promise 를 reduce 로  순차적으로 처리하여 결과값을 누적 반환해보자.
```js
(async () => {
const sleep = (n) => new Promise((resolve) => setTimeout(resolve, n * 1000));

const arr = [1,2,3,4,5]

const fn = (n) => async (result) => {
    console.log(n);
    await sleep(n);
    return [...result, n];
}

const result = await _.reduce(arr, (prev, curr) => {
    return prev.then(fn(curr));
}, Promise.resolve([]));
  
  return result;
})()
  .then((result) => {
    console.log(result);
  });
```
위의 코드를 살펴보다 해제할당을 반복 하는 것보다 인자로 결과배열을 넘겨 concat 으로 처리하는게 어떻냐는 의견이 있었다.
그래서 spread 와 concat 차이를 확인해 보았다.

## spread operator vs array.concat()
아래와 같은 코드는 같은 결과를 보여주지만 내부적으로 동작하는 방식은 다르기 때문에 차이는 존재한다.
```Js
const a = [1, 2];
const b = [3, 4, 5];
console.log([...a, ...b]); // [1, 2, 3, 4, 5]
console.log(a.concat(b)); // [1, 2, 3, 4, 5]
```

인수가 배열이 아닌 경우 spread 는 iterate 한다.

이렇게 인수가 배열이 아닐 가능성이 있을 때 concat 과 spread 간의 선택을 할 수 있을 것 같다.
```js
// 
const int = [1, 2, 3];
const str = 'hello';
console.log(int.concat(str)); // [1, 2, 3, 'hello']
console.log([...int, ...str]); // [ 1, 2, 3, 'h', 'e', 'l', 'l', 'o' ]
```

성능의 차이도 확인해 보자.
array 의 크기가 클 수록 성능 차이가 확연하다.
```js
const big = (new Array(1e5)).fill(99);
let i, x;

console.time('concat-big');
for(i = 0; i < 1e2; i++) x = [].concat(big)
console.timeEnd('concat-big');

console.time('spread-big');
for(i = 0; i < 1e2; i++) x = [...big]
console.timeEnd('spread-big');


const a = (new Array(1e3)).fill(99);
const b = (new Array(1e3)).fill(99);
const c = (new Array(1e3)).fill(99);
const d = (new Array(1e3)).fill(99);

console.time('concat-many');
for(i = 0; i < 1e2; i++) {
  x = [1,2,3].concat(a, b, c, d)
}
console.timeEnd('concat-many');

console.time('spread-many');
for(i = 0; i < 1e2; i++) {
  x = [1,2,3, ...a, ...b, ...c, ...d]
}
console.timeEnd('spread-many');

```
spread 는 메모리 사용량이 크다.

array 크기가 크면 ```Maximum call stack size exceeded``` 오류를 발생 시킬 수 있다.
```Js
const someArray = new Array(600000).fill(99);
const newArray = [];
let tempArray = [];

try {
  newArray.push(...someArray);
} catch (e) {
  console.log("Using spread operator:", e.message)
}

tempArray = newArray.concat(someArray);
console.log("Using concat function:", tempArray.length)
```

# 마무리
그럼 무조건 concat 인가?
 
그건 아니다.

당연한 이야기지만 상황에 맞게 써야지.
