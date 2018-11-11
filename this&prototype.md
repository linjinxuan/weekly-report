# 关于this
### 强调点：
this在任何情况下都不指向函数的词法作用域；
this的指向是在函数调用时确定的而不是在函数声明时确定的

### this绑定规则（优先级由低到高）
#### 1、默认绑定
 
函数调用类型： 独立函数调用

    function foo() {
      console.log(this);
    }

    foo(); // global window...

#### 2、隐式绑定
函数调用类型：obj.foo()

    function foo() {
      console.log(this);
    }

    var obj = {
      foo: foo
    }

    obj.foo();
但是在隐式绑定中存在着隐式丢失绑定对象的情况

    一种情况 
    function foo() {console.log(this);}
     var obj = {foo: foo}
     var bar = obj.foo // 函数别名
     bar()  //调用位置在全局作用域下，为默认绑定 
    
    另一种情况：函数作为参数传入时
    function foo() {console.log(this);}
    function doFoo(fn) {
      // fn其实引用的是foo
      fn(); // <--调用位置
    }
    var obj ={foo: foo}
    doFoo(obj.foo);

#### 3、显式绑定
call、apply、bind

#### 4、new 绑定

    var p = new Person();

    var obj = {};   // 创建一个全新的对象
    obj.__proto__ = Person.prototype;  // 这个新对象会被执行[[prototype]]连接
    Person.call(obj);  // 这个新对象会绑定到函数的this
    return obj; // 返回这个新对象

##### es6箭头函数中的this值不能被绑定，只属于外层（函数或全局）作用域
```
function test() {
  setTimeout(() => {
    console.log(this); // input global window
}, 500);  
};
test.apply(this);
```

# 理解原型对象

#### 构造函数(只是普通函数) 
function Person(){
}

#### 创建实例
const person = new Person();

#### 原型对象
Person.prototype.name = 'ivy';

Person.prototype.will = 'become excellent';

#### 三者关系
（1）、只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性。这个属性指向函数的原型对象;

（2）、所有原型对象都会获得constructor属性，这个属性指向prototype属性所在函数的指针，即构造函数

（3）、实例对象有属性_proto_指向原型对象（only firefox、safary、chrome support）

#### isPrototypeOf()
Person.prototype.isPrototypeOf(person)   // return true

#### Object.getPrototypeOf()  
 该方法返回实例原型对象  
 Object.getPrototypeOf(person) === Person.prototype

#### hasOwnProperty()
该方法检测一个属性是否存在于实例中
person.hasOwnProperty('name') 

#### 原型与in操作符
检测属性是否存在于原型中或实例中

#### hasPrototypeProperty()
该方法检测属性是否存在于原型中

#### 更简单的原型语法(对象字面量的形式)
Person.prototype = {
name: 'ivy',
age: '25',
will: 'become excellent'
}