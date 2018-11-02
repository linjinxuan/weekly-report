# 关于 node.assert

断言，表示一些布尔表达式，一般用于单元测试中，当需要在一个值为 false 时中断当前操作的话，可以使用断言。

比较常见的有以下几个方法，归类列举如下
1、判断值是否为真值

#### assert(value[,message])

这个测试函数在 Boolean(value)为 true 时通过断言测试，否则抛出 AssertionError，message 缺省时，则抛出默认信息，value == true

```
assert('ok', 'throw me when this value is not true'); //断言通过
assert(false, 'throw me when this value is not true');
// AssertionError [ERR_ASSERTION]: throw me when this value is not true
```

#### assert.ok(value[, message])

这个测试函数同上

2、判断预期值和实际值相等（==）

#### assert.equal(actual, expected[, message])

这个测试函数用于判断预期值与实际值是否相等；
情况有二：基础类型转化为同一类型再比较值；引用类型只比较其引用值

```
assert.equal(12, '12') //断言通过
const obj = {a:1}; const obj1 = obj; const obj2 = obj;
assert.equal(obj1, obj2) //断言通过
assert.equal({}, {})
// AssertionError [ERR_ASSERTION]: {} == {}
```

#### assert.deepEqual(actual, expected[, message])

这个测试函数也是判断预期值与实际值是否相等，与 equal 不同的是，引用类型比较的是其可枚举的属性值，不测试对象的原型、连接符、或不可枚举的属性

```
assert.equal(12, '12') //断言通过
const obj = {a:1}; const obj1 = obj; const obj2 = obj;
assert.equal(obj1, obj2) //断言通过
***assert.equal({}, {})*** //断言通过
```

3、判断预期值和实际值全等（===）

#### assert.deepStrictEqual(actual, expected[, message])

基础数据类型会比较类型，引用数据类型比较对象的原型，值的类型

```
const obj1 = {a: 1};
const obj2 = {a: 1};
const son1 = Object.create(obj1);
const son2 = Object.create(obj2);
son1.a = 12;
son2.a = 12;
assert.deepEqual(son1, son2) //断言通过
assert.deepStrictEqual(son1, son2)
//AssertionError [ERR_ASSERTION]:son1 deepStrictEqual son2
```

#### assert.strictEqual(actual, expected[, message])

该测试函数是对 equal 的强化，基础类型比较了数据类型
引用类型比较引用值

```
assert.strictEqual(1, '1');
//AssertionError [ERR_ASSERTION]:son1 === son2
```

4、判断实际值与预期值不相等（!=）

#### assert.notEqual(actual, expexted[, message])

该函数为 equal 的逆运算,如果 actual != expected,则断言通过
对于基础类型，转换为相同类型后再比较
对于引用类型，比较引用值

```
assert.notEqual(1, '1')
// AsserttionError [ERR_ASSERTION]: 1 != '1'
assert.notEqual({}, {})
const obj = {a:0};
const obj1 = obj;
const obj2 = obj;
assert.notEqual(obj1, obj2);
// AsserttionError [ERR_ASSERTION]: {a:0} != {a:0}
```

#### assert.notDeepEqual(actual, expected[, message])

该函数为 deepEqual()的逆运算，不相等，则通过断言
对于基础类型，转换为相同类型后再比较
对于引用类型，比较可枚举的属性值

```
assert.notDeepEqual(1, '1')
// AsserttionError [ERR_ASSERTION]: 1 != '1'
const obj1 = {a:0};
const obj2 = {a:0};
assert.notDeepEqual(obj1, obj2);
// AsserttionError [ERR_ASSERTION]: {a:0} notDeepEqual {a:0}
```

5、判断实际值与预期值严格不相等 （!==）

#### assert.notStrictEqual(actual, expected[, message])

该函数是 strictEqual()的逆运算，不相等，则通过断言
对于基础数据类型，考虑类型
对于引用类型？？？？

#### assert.notDeepStrictEqual(actual, expected[, message])

6、断言错误并抛出

#### assert.fail(message)

该函数用于主动抛出带有 message 属性的 AssertionError 对象

```
assert.fail('throw error~');
// AsserttionError [ERR_ASSERTION]: throw error
```

#### assert.fail(actual, expected[, message[, operator[, stackStartFunction]])

该函数同上，用于主动抛出错误信息
当 message 有值时，actual,operator,expected 属性会被列入错误对象属性中，只抛出 message 信息错误
当 message 为 undefined 或 null 时，会检测 operator 值，operator?operator:undefined
当 message 与 operator 都为空时，actual != expected

#### assert.throws(block, error, message)

block: Function
error: RegExp | Function | 构造函数
message: any
该函数表明如果 block 抛出的错误满足 error 参数，即抛出的错误与期望的一致，则断言通过，否则抛出 block 中的错误，如果 block 不抛出错误，则抛出 AssertionError

```
assert.throws(() => {throw new Error('错误信息')}, Error); //断言通过
assert.throws(() => {throw new Error('错误信息')}, /错误/); //断言通过
```

#### assert.doesNotThrow(block, error, message)

该函数表明如果 block 抛出的错误与期望的错误一致，不抛出实际错误，抛出 AssertionError（message 有值时，作为 AssertionError 的参数抛出）;
不一致时，抛出 block 的实际错误

#### assert.ifError(value)

该函数表明如果 Boolean(value)为 true，则抛出 value,反之则通过
