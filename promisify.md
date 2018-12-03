# 浅析 node util.promisify 函数

node v8.0.0 新加了一个方法，就是 util.promisify(original)方法

#### node 官方文档是如下解释的

![1543299596928](/assets/1543299596928.jpg)

original 参数是一个将 error-first node 回调作为最后参数的函数;
该函数返回一个 promise 实例，以供链式回调

虽然 promise 函数在 js 中已经很成熟了，但是 node 中仍有很多将回调函数做为最后参数的异步方法，比如 fs.readFile().

```
var fs = require('fs');

fs.readFile(path, (err, value) => {
  if(err) {return;}
  console.log(value);
  // code here...
})
```

若要将以上方法 promise 化，或者可以这样做

```
readFile(path) {
  return new Promise((res, rej) => {
    fs.readFile(path, (err, value) => {
      if(err) {
        rej(err);
      }
      res(value)
    })
  })
}

readFile(path).then(res => {
  console.log(res);
})
.catch(err => {
  console.log(err);
});
```

但如果每个函数都去这样包装一次，那就忒麻烦了，所以 node 新加的这个函数很及时可靠呢~
恩，现在只需要这样写就好了呢

```
var {promisify} = require('util');
var readFile = promisify(fs.readFile);

readFile(path).then(res => {
  console.log(res);
})
.catch(err => {
  console.log(err);
});

```

是不是很便捷简单呢。。。

那么，问题来了，如果某函数不符合 err-first 回调风格呢，是不是就不可以用 util.promisify 方法来转化了呢？

#### of course not

promisify 方法提供了一个函数，util.promisify.custom,可以将该函数转化成 promise 函数，promisify 函数的结果会被该属性值覆盖。

```
var testAsync = str => {
  return str;
};

testAsync[util.promisify.custom] = str =>
  new Promise((res, rej) => {
    res(str);
  });

var testAsyncPromisify = promisify(testAsync);
testAsyncPromisify('hello world!').then(res => {
  console.log(res);
});

console.log(testAsync[util.promisify.custom] === promisify(testAsync));
```

#### 源码剖析

```
var kCustomPromisifiedSymbol = Symbol('util.promisify.custom'); //定义symbol属性
var kCustomPromisifyArgsSymbol = Symbol('customPromisifyArgs');

function promisify(original) {
  if (typeof original !== 'function')
    throw new ERR_INVALID_ARG_TYPE('original', 'Function', original);

  if (original[kCustomPromisifiedSymbol]) {
    const fn = original[kCustomPromisifiedSymbol];
    if (typeof fn !== 'function') {
      throw new ERR_INVALID_ARG_TYPE('util.promisify.custom', 'Function', fn);
    }
    Object.defineProperty(fn, kCustomPromisifiedSymbol, {
      value: fn,
      enumerable: false,
      writable: false,
      configurable: true
    });
    return fn;
  }

  // Names to create an object from in case the callback receives multiple
  // arguments, e.g. ['stdout', 'stderr'] for child_process.exec.
  const argumentNames = original[kCustomPromisifyArgsSymbol];

  function fn(...args) {
    return new Promise((resolve, reject) => {
      original.call(this, ...args, (err, ...values) => {
        if (err) {
          return reject(err);
        }
        if (argumentNames !== undefined && values.length > 1) {
          const obj = {};
          for (var i = 0; i < argumentNames.length; i++)
            obj[argumentNames[i]] = values[i];
          resolve(obj);
        } else {
          resolve(values[0]);
        }
      });
    });
  }

  Object.setPrototypeOf(fn, Object.getPrototypeOf(original));

  Object.defineProperty(fn, kCustomPromisifiedSymbol, {
    value: fn,
    enumerable: false,
    writable: false,
    configurable: true
  });
  return Object.defineProperties(
    fn,
    Object.getOwnPropertyDescriptors(original)
  );
}

promisify.custom = kCustomPromisifiedSymbol;
```

从源码中可知，如果 original 参数不是个函数，会抛出错误；
函数会返回 fn 函数，在 fn 中调用 original 原函数，err-first 回调作为最后一个参数传送，在 err-first 回调中，返回 promise 实例，以供链式调用。

哈哈，标题既然为“浅析”，那就真的只是浅浅剖析一下了，太深的我也不懂
