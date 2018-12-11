# 浅尝 vue 双向绑定原理

vue 双向绑定基于观察者模式

1、创建一个观察着列表类（Dep）,该类里有一个数组，用于收集所有需要被监听观察的对象
2、定义一个观察类（Watcher）,代表每一个需要被监听的属性对象
3、定义一个 Observe 类，用 Object.propertity()方法绑定每一个 data 对象的属性到 this,实现 this 访问 data 中的属性
4、vue 还有一个思想就是虚拟 dom，vue 会遍历根元素下的子 dom 元素，将其先 append 到文档碎片中，等所有操作完成后再一次性 append 到根元素下，这样就只实行了一次绘制，提高了性能

讲完大致思想，可以上源码（自己模拟的）分析了

```
// html
<div id="app"><input type="text" v-model="text" /> {{ text }}</div>
```

一、定义一个 Vue 构造函数

```
class Vue{
  constructor(options={}){
    this.el = options.el;
    this.data = options.data;
    observe(this.data, this); // 绑定data属性到this引用
    const frag = nodeToFragment(this, document.getElementById(this.el)); // 遍历根元素的子元素到虚拟dom
    document.getElementById(this.el).appendChild(frag); // 将虚拟dom append到根元素下
  }
}

// create an instance
new Vue({
  el: 'app',
  data: {
    text: 'hello world!'
  }
});
```

二、定义一个 Observe 函数

```
function observe(data, vm) {
  // 遍历data对象的每一个属性，绑定到this引用中
  Object.keys(data).forEach(key => {
    defineReactive(vm, key, data[key]);
  });
}

function defineReactive(vm, key, value) {
  // const dep = new Dep();
  Object.defineProperty(vm, key, {
    enumerable: true,
    configurable: true,
    get() {
      Dep.target && dep.addSub(Dep.target); // Dep.target是data对象中属性的watcher实例
      return value;
    },
    set(newVal) {
      if (value === newVal) return;
      value = newVal;
      dep.notify(); // 更新属性值,可索引到Watcher方法
    }
  });
}
```

三、定义 nodeToFragment 函数

```
function nodeToFragment(vm, node) {
  const fragment = document.createDocumentFragment(); //创建文档碎片
  let child;
  while ((child = node.firstChild)) { 遍历根元素的子元素dom
    compile(child, vm);
    fragment.appendChild(child);
  }
  return fragment;
}

function compile(node, vm) {
  const reg = /\{\{(.*)\}\}/;
  if (node.nodeType === 1) {
    // 遍历到input元素的v-model属性，获取属性值（在该例子中为text），
    // 给input标签绑定input事件，将输入的值赋值给this属性
    const attrs = node.attributes;
    for (let i = 0; i < attrs.length; i++) {
      if (attrs[i].nodeName === 'v-model') {
        const name = attrs[i].nodeValue;
        node.addEventListener('input', e => {
          vm[name] = e.target.value; // 当input框输入值时，赋值会更新Watchcer的notify方法
        });

        new Watcher(vm, node, name, 'input');
        node.removeAttribute('v-model'); // 移除input标签的v-model属性，所在在真实dom元素中是看不到v-model属性的
      }
    }
  } else if (node.nodeType === 3) {
    if (reg.test(node.nodeValue)) {
      const name = RegExp.$1.trim();
      new Watcher(vm, node, name, 'text');
    }
  }
}
```

四、创建一个 Watcher 构造函数

```
class Watcher {
  constructor(vm, node, key, nodeType) {
    Dep.target = this; // 属性初次被绑定监听的标志
    this.vm = vm;
    this.key = key;
    this.node = node;
    this.nodeType = nodeType;
    this.update();
    Dep.target = null; // 之后设为null,避免属性多次被添加到监听队列中
  }

  update() {
    this.getVal();
    if (this.nodeType === 'input') {
      this.node.value = this.value;
    } else if (this.nodeType === 'text') {
      this.node.nodeValue = this.value;
    }
  }
  getVal() {
    this.value = this.vm[this.key]; // 触发object.propertity.get()操作
  }
}
```

五、创建一个 Dep 构造函数

```
class Dep {
  constructor() {
    this.subs = []; //该数组保存所有需要被监听的属性的Watcher对象实例
  }

  addSub(sub) {
    this.subs.push(sub);
  }

  notify() { // 更新所有属性的值
    this.subs.map(sub => {
      sub.update();
    });
  }
}
```
