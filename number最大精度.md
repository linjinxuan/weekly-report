#关于 Number 的最大数问题
在 js 中，Number 对象有 MAX_VALUE、MIN_VALUE、MAX_SAFE_INTEGER、MIN_SAFE_INTEGER 等属性，Number 对象是有范围限制的，如下图：

![Number](/assets/Number.png)

MAX_VALUE:js 中最大可以表示的数，即 1.7976931348623157e+308，这个数虽然能表示出来，但是精度不准确
例如：

```
11111111111111110000 === 11111111111111110001
// true
```

以上两个数明显是不相等的，但是结果是 true

MAX_SAFE_INTEGER：js 中可以表示精确到个位的的最大的数，Math.pow(2, 53)，十进制即 9007199254740992.

```
9007199254740992 === 9007199254740993
// true
9007199254740992 === 9007199254740991
// false
```

超过 MAX_SAFE_INTEGER 的值是无法保证精度的。

要精确地实现两个大数相加，可以用以下按位相加的方法(非原创，理解了就是自己的哈哈哈)

```
function bigNumSum(a, b) {
  let temp = 0;
  let all = '';
  const arrA = a.split('');
  const arrB = b.split('');
  while (arrA.length || arrB.length || temp) {
    temp += ~~arrA.pop() + ~~arrB.pop();
    all = (temp % 10) + all;
    temp = temp > 9;
  }
  return all.replace(/^0+/, '');
}
```
