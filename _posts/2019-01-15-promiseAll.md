---
layout: post
title:  "[CodeReview] Promise 병렬처리"
subtitle:   "병렬처리"
categories: doc
tags: nodejs
comments: true
---

## Promise.all 
* async-await 사용의 편리함으로 인하여 병렬로 실행해도 되는 상황에서도 순차적으로 처리하는 경우가 많다.
* 순차가 필요하지 않은 상황에서 병렬로 처리해보자

아래는 처리가 1초가 필요한 promise 이다. 이 코드를 실행하면 3초가 걸린다. 
```Js
const sleep = (num = 1) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(`${num} called`);
      resolve()
    }, 1000);
  });
};

const runPromise = async () => {
  for (let i = 0; i < 3; i++) {
    await sleep(i + 1);
  }
};

console.time('runPromise');
runPromise().then(() => console.timeEnd('runPromise'));
```

하지만 다음처럼 Promise.all 을 통해 병렬로 실행해도 아무런 문제가 없다.
```Js
const sleep = (num = 1) => {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log(`${num} called`);
      resolve()
    }, 1000);
  });
};

const runPromiseAll = async () => {
  await Promise.all(
    [1, 2, 3].map(item => sleep(item)),
  );
};

console.time('runPromiseAll');
runPromiseAll().then(() => console.timeEnd('runPromiseAll'));
``` 

이전 promise 실행이 다음 promise 실행에 영향을 받지 않는다면, Promise.all을 사용하여 병렬처리 하는 것이 속도측면에서 빠르다.
그리고 이전 promise 실행이 다음 promise 실행에 영향을 미친다 하여도 묶어서 처리 할 수 있다.
```Js
const sleep = (num = 1) => {
  return new Promise((resolve) => {
    setTimeout(
      () => resolve(num),
      1000,
    );
  });
};

const print = (num) => {
  return new Promise((resolve) => {
    console.log(num);
    resolve();
  });
};

const runPromise = async () => {
  for (let i = 0; i < 3; i++) {
    const result = await sleep(i + 1);
    await print(result);
  }
};

console.time('runPromise');
runPromise.then(() => console.timeEnd('runPromise'));
```

위 코드를 병렬 처리될 수 있도록 묶는다면 다음과 같이 실행 할 수 있다.
```Js
const sleep = (num = 1) => {
  return new Promise((resolve) => {
    setTimeout(
      () => resolve(num),
      1000,
    );
  });
};

const print = (num) => {
  return new Promise((resolve) => {
    console.log(num);
    resolve();
  });
};

const runPromiseAll = async () => {
  Promise.all(
    [1, 2, 3].map(item => sleep(item).then(print))
  );
};

console.time('runPromiseAll');
runPromiseAll.then(() => console.timeEnd('runPromiseAll'));
```

함수로 처리 하는 방법도 있다.
```Js
const sleep = (num = 1) => {
  return new Promise((resolve) => {
    setTimeout(
      () => resolve(num),
      1000,
    );
  });
};

const print = (num) => {
  return new Promise((resolve) => {
    console.log(num);
    resolve();
  });
};

const runPromise = async (item) => {
  await sleep(item);
  await print(item);
};

const runPromiseAll = async () => {
  Promise.all(
    [1, 2, 3].map(runPromise) 
  );
};

console.time('runPromiseAll');
runPromiseAll.then(() => console.timeEnd('runPromiseAll'));
```

```Js
const sleepOne = (num = 1) => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve()
    }, 1000);
  });
};

const sleepTwo = (num = 2) => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve()
    }, 1000);
  });
};

// 순서 필요없는 promise 를 반환하는 함수 
const run = async () => {
 const one = await sleepOne(1);
 const two = await sleepTwo(2); 
 
 const resultOne = one;
 const resultTwo = two;
};

// 위의 함수를 리펙토링 한다면.!
// 아래처럼 병렬처리하여 개선할 수 있다.
const run  = async () => {
const [one, two] = await Promise.all([
    sleepOne(1),
    sleepTwo(2),
  ]);  
};
```

## 마무리
충분히 성능 개선에 도움이 될 수 있음으로 최초에 코드를 작성 할 때 고려해서 개발할 수 있으면 하자.!
