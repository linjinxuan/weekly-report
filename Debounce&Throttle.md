<!--
  -- --------------------------------------------------------
  -- @file Debounce&Throttle.md
  -- @author jinxuan_lin <jinxuan_lin@kingdee.com>
  -- @date 2019-02-01 15:00:27
  -- @last_modified_by jinxuan_lin <jinxuan_lin@kingdee.com>
  -- @last_modified_date 2019-02-01 17:38:01
  -- @copyright (c) 2019 @yfe/weekly-report
  -- --------------------------------------------------------
 -->

# Debounce & Throttle

当某一事件触发频率很高时， 我们需要采取一定措施优化页面性能，减少事件回调函数的触发次数，这就是防抖和节流的应用场景了。

## Debounce

**防抖动技术**是指某事件高频触发时，回调触发时间推后。就是说事件快速重复触发时，取消之前的注册的定时器，只有最后一次事件注册的定时器生效。回调函数被调用的时间是一直在推后的。也就是说 debounce 函数将多次的事件触发合并成了一次

```
function debounce(fn, wait) {
      let timer = null;
      return function() {
            clearTimeOut(timer);
            timer = setTimeOut(fn, wait);
      };
}

const scrollHandle = function() {
      console.log('scroll');
}
window.addEventLisener('scroll', debounce(scrollHandle, 400))
```

就是 400ms 内，滚动事件快速多次触发，只有在最后一次触发时的定时器有效。

## Throttle

**节流阀技术**是指将多次事件合并成一次事件。就是说在规定时间内，该事件必会触发一次。
它和防抖的是区别就是，无论事件触发多频繁，一定会触发一次。

```
function throttle(fn, wait) {
      let start = new Date().getTime();
      let timer = null;
      return function(...args) {
            const end = new Date().getTime();
            const remaining = wait - (end-start);
            clearTimeOut(timer);

            if (remaining <=0 ) {
                  fn.apply(this, args);
                  start = new Date().getTime();
            } else {
                  timer = setTimeOut(fn, remaining);
            }
      }
}

const scrollHandle = function() {
      console.log('scroll');
}
window.addEventLisener('scroll', throttle(scrollHandle, 400))
```

就是说在 400ms 内，滚动事件回调函数一定会触发一次。
场景：无限滚动加载数据
