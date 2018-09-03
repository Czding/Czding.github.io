---
title: VUE数据双向绑定
date: 2018-09-01 21:45:07
tags:
---

数据双向绑定就是视图上的变化能够反映到数据上，数据上的变化也能反映到视图上。

### **实现双向数据绑定的方式**


#### **发布者/订阅者模式**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;假定存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，而知道什么时候自己可以开始执行，这就叫做"发布/订阅模式"。一般通过publish和subscribe的方式来实现。

#### **脏值检查**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;angular.js就是利用脏值检查的方式来实现数据双向绑定。利用setInterval()定时轮询检测数据变动比对数据是否有变更，来决定是否更新视图。通过监听事件来触发定时器，譬如：DOM事件、浏览器事件、http响应等。

#### **数据劫持**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vue.js 则是采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。


### **Object.defineProperty()**
语法：

Object.defineProperty(obj, prop, descriptor)

参数说明：

obj：必需。目标对象 
prop：必需。需定义或修改的属性的名字
descriptor：必需。目标属性所拥有的特性

兼容性：

IE9+，在ie8下只能在DOM对象上使用，**这就是为什么vue只兼容IE9及以上版本**。里我们主要先来研究下它对应的两个描述属性get和set。
我们平时打印一个对象的数据：
```
var person = {
    name: '小芳'
};
console.log(person.name); // 小芳

```

如果要在打印出人名是同时给人名加修饰词呢？这时候就要用上Object.defineProperty()了，代码如下：

```
var person = {};
var name = '';
Object.defineProperty( person, name, {
    set: function(val) {
        name = val;
        console.log('名字叫' + name);
    },
    get: function() {
        return '漂亮的' + name;
    }
})

person.name = '小芳'; // 小芳
console.log(person.name); // 漂亮的小芳

```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过Object.defineProperty( )设置了对象person的name属性，对其get和set进行重写操作，顾名思义，get就是在读取name属性这个值触发的函数，set就是在设置name属性这个值触发的函数，所以当执行 person.name = '小芳' 这个语句时，控制台会打印出 "名字叫小芳"，紧接着，当读取这个属性时，就会输出 "漂亮的小芳"，因为我们在get函数里面对该值做了加工了。

### **实现过程**

#### **简单的双向绑定实现**

```

<html>
    <body>
        <input type="text" id="input">
        <div id="div"></div>
    </body>
    <script>

        var obj = {};
        Object.defineProperty(obj,'hello',{
            set: function(val) {
                document.getElementById('input').value = val;
                document.getElementById('div').innerHTML = val;
            }
        });

        document.addEventListener('keyup',function(e) {
            obj.hello  = e.target.value;
        })

    </script>
</html>

```
随文本框输入文字的变化，div中会同步显示相同的文字内容；在js或控制台显式的修改obj.name的值，视图会相应更新。这样就实现了model =>view以及view => model的双向绑定，并且是响应式的,这就是vue实现数据双向绑定的基本原理。

![百度](Object-defineProperty/property.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实现数据的双向绑定，首先要对数据进行劫持监听，所以我们需要设置一个监听器Observer，用来监听所有属性。如果属性发上变化了，就需要告诉订阅者Watcher看是否需要更新。因为订阅者可能有多个，所以我们需要有一个消息订阅器Dep来专门收集这些订阅者，然后在监听器Observer和订阅者Watcher之间进行统一管理的。接着，我们还需要有一个指令解析器Compile，对每个节点元素进行扫描和解析，将相关指令对应初始化成一个订阅者Watcher，并替换模板数据或者绑定相应的函数，此时当订阅者Watcher接收到相应属性的变化，就会执行对应的更新函数，从而更新视图。因此，总结出了以下3个步骤，实现数据的双向绑定：

#### **监听器Observer**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用来劫持并监听所有属性，如果有变动的，就通知订阅者。

#### **订阅者Watche**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以收到属性的变化通知并执行相应的函数，从而更新视图。

#### **解析器Compile**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以扫描和解析每个节点的相关指令，并根据初始化模板数据以及初始化相应的订阅器。


