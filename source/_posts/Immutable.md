---
title: Immutable
date: 2019-11-30 13:50:31
categories: '前端'
tags: 'React'
---

#### 1. 来源
> Immutable.js出自Facebook，是最流行的不可变数据结构的实现之一。它实现了完全的持久化数据结构，通过使用像tries这样的先进技术来实现结构共享。所有的更新操作都会返回新的值，但是在内部结构是共享的，来减少内存占用(和垃圾回收的失效)。
#### 2. immutable.js三大特性：
- Persistent data structure （持久化数据结构）
- structural sharing （结构共享）
- support lazy operation （惰性操作）
###### 2.1. 持久化数据结构
这里说的持久化是用来描述一种数据结构，指一个数据，在被修改时，仍然能够保持修改前的状态，即不可变类型。immutable.js提供了十余种不可变的类型（List，Map，Set，Seq，Collection，Range等）
###### 2.2. 结构共享
> Immutable使用先进的tries(字典树)技术实现结构共享来解决性能问题，当我们对一个Immutable对象进行操作的时候，ImmutableJS会只clone该节点以及它的祖先节点，其他保持不变，这样可以共享相同的部分，大大提高性能。

![图片来自网络](https://upload-images.jianshu.io/upload_images/3817318-c338d48d6547c7bf.gif?imageMogr2/auto-orient/strip)
###### 2.3. 惰性操作
```
const oddSquares = Immutable.seq.of(1, 2, 3, 4, 5, 6, 7, 8)
  .filter(item => {
    console.log('immutable对象的filter执行');
    return item % 2;
  })
  .map(x => x * x);

console.log(oddSquares.get(1)); // 9

const jsSquares = [1, 2, 3, 4, 5, 6, 7, 8]
  .filter(item => {
    console.log('原生数组的filter执行');
    return item % 2;
  })
  .map(x => x * x);

console.log(jsSquares[1]); // 9
```
![执行结果](https://upload-images.jianshu.io/upload_images/3817318-af7140b7ccbb4b16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用seq创建的对象，其实代码块没有被执行，只是被声明了，代码在get(1)的时候才会实际被执行，取到index=1的数之后，后面的就不会再执行了，所以在filter时，第三次就取到了要的数，从4-8都不会再执行。如果在实际业务中，数据量非常大，一个array的长度是几百，要操作这样一个array，如果应用惰性操作的特性，会节省非常多的性能。

#### 3. 常用API介绍
```
// Map(): 原生object转Map对象 (只会转换第一层，注意和fromJS区别)
immutable.Map({ name: 'graceji', age: 18 });

// List(): 原生array转List对象 (只会转换第一层，注意和fromJS区别)
immutable.List([1, 2, 3, 4, 5]);

// fromJS(): 原生js转immutable对象  (深度转换，会将内部嵌套的对象和数组全部转成immutable)
immutable.fromJS([1, 2, 3, 4, 5]);    // 原生array  --> List
immutable.fromJS({ name: 'graceji', age: 18 });   // 原生object  --> Map

// toJS(): immutable对象转原生js  (深度转换，会将内部嵌套的Map和List全部转换成原生js)
immutableData.toJS();

// 查看List或者map大小  
immutableData.size  或者 immutableData.count()

// is(): 判断两个immutable对象是否相等
const objA = { name: 'graceji', age: 18 };
const objB = { name: 'graceji', age: 18 };
const imA = immutable.Map({ name: 'graceji', age: 18 });
const imB =immutable.Map({ name: 'graceji', age: 18 });
objsA === objB // false； 比较的是地址
immutable.is(imA, imB); // true; hashcode相同

// merge()  对象合并
const imA = immutable.fromJS({ a: 1,b: 2 });
const imA = immutable.fromJS({ c: 3 });
const imC = imA.merge(imB);
console.log(imC.toJS())  // { a: 1,b: 2,c: 3 }

// 增删改查（所有操作都会返回新的值，不会修改原来值）
const immutableData = immutable.fromJS({
    a: 1,
    b: 2，
    c: {
        d: 3
    }
});
const data1 = immutableData.get('a') //  data1 = 1  
const data2 = immutableData.getIn(['c', 'd']) // data2 = 3; getIn用于深层结构访问
const data3 = immutableData.set('a' , 2);   // data3中的 a = 2
const data4 = immutableData.setIn(['c',  'd'],  4);   // data4中的 d = 4
const data5 = immutableData.update('a' , function(x) { return x+4 })   // data5中的 a = 5
const data6 = immutableData.updateIn(['c', 'd'], function(x) { return x+4 })   // data6中的 d = 7
const data7 = immutableData.delete('a')   // data7中的 a 不存在
const data8 = immutableData.deleteIn(['c',  'd'])   // data8中的 d 不存在
```

#### 4. 优缺点

**优点：**
- 降低mutable带来的复杂度
- 节省内存
- 历史追溯性（时间旅行）
> 时间旅行指的是，每时每刻的值都被保留了，想回退到哪一步只要简单的将数据取出就行。如果现在页面有个撤销的操作，撤销前的数据被保留了，只需要取出就行，这个特性在redux或者flux中特别有用
- 拥抱函数式编程：immutable本来就是函数式编程的概念，纯函数式编程的特点就是，只要输入一致，输出必然一致，相比于面向对象，这样开发组件和调试更方便

**缺点：**
- 需要重新学习api
- 资源包大小增加（源码5000行左右）
- 容易与原生对象混淆：由于api与原生不同，混用的话容易出错。

#### 5. 在react+redux中集成immutable.js

react有个重要的性能优化的点就是shouldComponentUpdate，返回true代码该组件要re-render，false则不重新渲染。简单的场景可以直接使用===去判断this.props和nextProps是否相等，但当props是一个复杂的结构时，===肯定是没用的。

##### 5.1 集成前的准备

首先需要确定哪些数据需要使用不可变数据，哪些数据要使用原生js数据结构，哪些地方需要做互相转换。
- 在redux中，全局state必须是immutable的，这是使用immutable来优化redux的核心
- 组件props是通过redux的connect从state中获得的，并且引入immutablejs的另一个目的是减少组件shouldComponentUpdate中不必要渲染，shouldComponentUpdate中比对的是props，如果props是原生js就失去了优化的意义
- 组件内部state如果需要提交到store的，必须是immutable，否则不强制
- view提交到action中的数据必须是immutable
- action提交到reducer中的数据必须是immutable
- reducer中最终处理state必须是以immutable的形式处理并返回
- 与服务端ajax交互的数据为原生js数据，需要转换成immutable数据
从上面这些点可以看出，几乎整个项目都是必须使用immutable的，只有在少数与外部依赖有交互的地方使用了原生js。这么做的目的其实就是为了防止在大型项目中，原生js与immutable混用，导致自己都不清楚一个变量中存储的到底是什么类型的数据。
`Note：fromJS() 和 toJS() 是深层的互转immutable对象和原生对象，性能开销大，尽量不要使用。`

##### 5.2 具体实现

- **redux-immutable**
redux中利用combineReducers来合并reducer并初始化state，redux自带的combineReducers只支持state是原生js形式的，所以需要使用redux-immutable提供的combineReducers来替换原来的方法。
```
import { combineReducers } from 'redux-immutable';
import reducer1 from './reducer1';
import reducer2 from './reducer2';
import reducer3 from './reducer3';

const rootReducer = combineReducers({
    reducer1,
    reducer2,
    reducer3,
});

export default rootReducer;
```
- **reducer中的initialState也需要初始化成immutable类型:**
```
const initialState = Immutable.Map({});
const reducer1 = (state = initialState, action) => {
    switch (action.type) {
    case ILOVEYOU:
        return state.set('smile', true);
    }
}
export default reducer1;
```
- **state成为了immutable类型, 所以通过mapStateToProps传入组件的属性写法也需要做相应改变**
```
// connect
function mapStateToProps (state) {
    return {
        lovers: state.getIn(['man', 'list']),  
        abhorrers: state.getIn(['women', 'list']) // 使用get或者getIn来获取state中的变量
    }
}
```
- **服务端交互ajax**
```
// 伪代码
$.ajax({
    type: 'GET',
    url: 'XXX',
    dataType: 'json',
    success(res) {
        res = immutable.fromJS(res || {});
        callback && callback(res);
    },
    error(e) {
        e = immutable.fromJS(e || {});
        callback && callback(e);
    },
});
```
-  **shouldComponentUpdate**
```
// BaseComponent.js 基类

import React, { Component } from 'react';
import { is } from 'immutable';

class BaseComponent extends Component {
    constructor(props, context) {
        super(props, context);
    }

    shouldComponentUpdate (nextProps, nextState) {
        const thisProps = this.props || {};
        const thisState = this.state || {};
        nextState = nextState || {};
        nextProps = nextProps || {};

        if (Object.keys(thisProps).length !== Object.keys(nextProps).length ||
            Object.keys(thisState).length !== Object.keys(nextState).length) {
            return true;
        }

        for (const key in nextProps) {
            if (!is(thisProps[key], nextProps[key])) {
                return true;
            }
        }

        for (const key in nextState) {
            if (!is(thisState[key], nextState[key])) {
                return true;
            }
        }
        return false;
    }
}

export default BaseComponent;
```
组件中如果需要使用统一封装的shouldComponentUpdate，则直接继承基类, 如果不想组件使用封装的方法，那直接在该组件中重写shouldComponentUpdate就行了。

```
import BaseComponent from './BaseComponent'; 
class People extends BaseComponent {
    constructor() {
        super();
    }
    …………
}
```

#### 6. immutable.js使用过程中的一些注意点

- fromJS和toJS会深度转换数据，随之带来的开销较大，尽可能避免使用，单层数据转换使用Map()和List()
- Map类型的key必须是string
- 所有针对immutable变量的增删改必须左边有赋值，因为所有操作都不会改变原来的值，只是生成一个新的变量
```
// js
const arr = [1,2,3,4];
arr.push(5);
console.log(arr); // [1,2,3,4,5]
//immutable
const arr = immutable.fromJS([1,2,3,4])
// 错误用法
arr.push(5);
console.log(arr) //[1,2,3,4]
// 正确用法
arr = arr.push(5);
console.log(arr) // [1,2,3,4,5]
```
- 引入immutablejs后，不应该再出现对象数组拷贝的代码
```
// es6对象复制
const state = Object.assign({}, state, {
    key: value
});

// array复制
const newArr = [].concat([1, 2, 3]);
```
- immutable对象直接可以转JSON.stringify(),不需要显式手动调用toJS()转原生
- 判断对象是否是空可以直接用size
- 调试过程中要看一个immutable变量中真实的值，可以chrome中加断点，在console中使用.toJS()方法来查看

#### 7. 总结

总的来说immutable.js完美的契合了react+redux的state流处理，redux的宗旨就是单一数据流，可追溯，这两点恰恰是immutable.js的优势。当然也不是所有使用react+redux的场景都需要使用immutable.js，**建议满足项目足够大，state结构足够复杂的原则**，小项目可以手动处理shouldComponentUpdate，不建议使用，得不偿失。

