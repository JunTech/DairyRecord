---
title: 全干工程师的自我修养---vue第一部分
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: vue学习第一部分
categories: js
tags: vue
keywords: vue
abbrlink: e2b329d4
date: 2019-09-06 14:56:46
password:
copyright: true
---

# 全干工程师的自我修养---vue第一部分

## 1、环境搭建

主要是搭建nodejs环境，设置淘宝源及安装cnpm，在之前的文章中有提到过，就不再赘述。

接下来全局安装vue脚手架： 

```
cnpm install vue-cli -g
```

这样就可以成功安装了，安装完成后，我们就可以尽情的食用了。

也可以使用在线加速cdn:

```
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
```

## 2、初始化一个项目

我们这里安装的是vue2.x,所以使用一下命令

```
vue init webpack 
```

最好不适用es lint6，所以前面4个都回车，后面填no.

这样cd进目录，使用

```
cnpm run dev
```

打开localhost:8080就可以看到我们的第一个vue hello,world项目了。

## 3、从爬行到飙车

### 3.1声明式渲染

![Vue 实例生命周期](https://cn.vuejs.org/images/lifecycle.png)

Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统：如下

```
<div id="app">
  {{ message }}
</div>
```

```
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

输出： ```hello,vue```

### 3.2 数据绑定 v-bind

```vue
<div :title="title">鼠标绑定事件</div>
```

```vue
  data() {
      return {
        msg: "你好vue",
        title: "这是一个title!",
        message: '',
        todos: [{ text: '学习 JavaScript' },{ text: '学习 Vue' }, { text: '整个牛项目' }],
        formData: 'username',
    }
  },
```

鼠标放在上面时，会显示： 这是一个title! 这就是v-bind，简写方式为  ```:```

### 3.3事件绑定 v-on

```vue
 <div id="app-1">
        <p>{{msg}}</p>
        <button v-on:click="reverseMessage">反转消息</button>
    </div>
//methods里面
    reverseMessage(){
      this.msg = this.msg.split('').reverse().join('');
    },
```

点击反转消息按钮就会显示： euv好你 ，这就是v-on，简写为@

### 3.4数据双向绑定v-model

```vue
    <div id="app-6">
      <p>message : {{message}}</p>
      <input type="text" v-model="message" name="message" id="message">
    </div>
```

数据同步更新

### 3.5v-for

```vue
<div>
      <ul>
          <li v-for="item in todos">
            {{item.text}}
          </li>
      </ul>
</div>
```

显示todos内容

### 3.6 v-if

```
<p v-if="seen">现在你看到我了</p>
```

这里，`v-if` 指令将根据表达式 `seen` 的值的真假来插入/移除 `<p>` 元素。

### 3.7 其他一些修饰符

```
v-html
v-on的修饰符
.stop - 调用 event.stopPropagation()。
.prevent - 调用 event.preventDefault()。
.capture - 添加事件侦听器时使用 capture 模式。
.self - 只当事件是从侦听器绑定的元素本身触发时才触发回调。
.{keyCode | keyAlias} - 只当事件是从特定键触发时才触发回调。
.native - 监听组件根元素的原生事件。
.once - 只触发一次回调。
.left - (2.2.0) 只当点击鼠标左键时触发。
.right - (2.2.0) 只当点击鼠标右键时触发。
.middle - (2.2.0) 只当点击鼠标中键时触发。
.passive - (2.3.0) 以 { passive: true } 模式添加侦听器
```

### 3.8ref操作dom

```
<div id="app-5">
      <input type="text"  ref="userInfo">
      <div id="box" ref="box">我是一个box</div>
      <button  @click="getInputValue()">获取input value</button>

  </div>
  
  
  //methods里
   // ref操作dom节点
    getInputValue(){
      alert(this.$refs.userInfo.value);
      this.$refs.box.style.background= 'red';
    }
```

ref指定对象，在方法里通过this.$refs.对象来获取对象值。 

### 3.9方法传值

```
  <div id="app-7">
    <button @click="requestData()">请求数据</button>
    <ul>
      <li v-for="item in list">{{item}}</li>
    </ul>
  </div>
  
  //data
  list: []
  
  //methods
      requestData(){
      for(var i=0;i<10;i++){
        this.list.push(i);
      }
    }
```

点击按钮，就会显示0-9的列表

### 3.10小结--todolist

第一步====需求： todolist的增添和删除

```js
<template>
  <div>
    //表单
    <input type="text" v-model="todolist" placeholder="请输入todo list" class="inputText">
    <button @click="pushData()" class="btn">+</button>
	//打印to'do'list
    <ul>
      <li v-for="(item,key) in list">
        {{item}}  ----- <button @click="del(key)">删除</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'helloworld',
  data() {
    return {
      todolist: '',
      list: []
    }
  },
  methods: {
      //添加元素操作
    pushData(){
      this.list.push(this.todolist);
      this.todolist = ''
    },
        //删除元素操作  splice是js数组操作的方法可以去看一下，这里的意思表示从key开始删除第一个，即key代表的值,如果是splice(1,3,'wiinfud')表示从第一个开始删除3个，并添加一个‘wiidfd'
    del(key){
        this.list.splice(key,1)
    }
  },
}
</script>
//样式
<style>
  .inputText{
    border-radius: 20px;
    border: solid 1px gray;
    padding: 20px 10px;
    display: inline;
  }
  .btn{
    background: skyblue;
    border: 2px red solid;
    padding: 20px 10px;
    display: inline;
  }
</style>


```

第二部====需求：可以点击来更改状态

```js
<template>
  <div>
    <input type="text" v-model="todolist" placeholder="请输入todo list" class="inputText">
    <button @click="pushData()" class="btn">+</button>

  <h2>进行中</h2>
    <ul>
      <li v-for="(item,key) in list" v-if="!item.checked">
        <input type="checkbox" name="" id="" v-model="item.checked">{{item.title}}  ----- <button @click="del(key)">删除</button>
      </li>
    </ul>
    <br>
    <hr>
    <h2>已完成</h2>
    <ul>
      <li v-for="(item,key) in list" v-if="item.checked">
        <input type="checkbox" name="" id="" v-model="item.checked">{{item.title}}  ----- <button @click="del(key)">删除</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'helloworld',
  data() {
    return {
      todolist: '',
      checked: false,
      list: [
        {
          title: 'vue',
          checked: true
        },
        {
          title: 'nodejs',
          checked: false
        }
      ]
    }
  },
  methods: {
    pushData(){
      this.list.push({
        title: this.todolist,
        checked: this.checked
      });
      this.todolist = ''
    },
    del(key){
        this.list.splice(key,1)
    }
  },
}
</script>

<style>
  .inputText{
    border-radius: 20px;
    border: solid 1px gray;
    padding: 20px 10px;
    display: inline;
  }
  .btn{
    background: skyblue;
    border: 2px red solid;
    padding: 20px 10px;
    display: inline;
  }
</style>


```

### 3.11todolist升级

需求：保存状态，使用enter点击事件

需要使用到html5的storage技术

```
<template>
  <div>
    <input type="text" v-model="todolist" placeholder="请输入todo list" class="inputText">
    <button @click="pushData()" class="btn">+</button>

  <h2>进行中</h2>
    <ul>
      <li v-for="(item,key) in list" v-if="!item.checked">
        <input type="checkbox" name="" id="" v-model="item.checked" @change="saveList()">{{item.title}}  ----- <button @click="del(key)">删除</button>
      </li>
    </ul>
    <br>
    <hr>
    <h2>已完成</h2>
    <ul>
      <li v-for="(item,key) in list" v-if="item.checked">
        <input type="checkbox" name="" id="" v-model="item.checked" @change="saveList()">{{item.title}}  ----- <button @click="del(key)">删除</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'helloworld',
  data() {
    return {
      todolist: '',
      checked: false,
      list: [
        {
          title: 'vue',
          checked: true
        },
        {
          title: 'nodejs',
          checked: false
        }
      ]
    }
  },
  methods: {
    pushData(){

        this.list.push({
        title: this.todolist,
        checked: this.checked
        });
      this.todolist = ''
      localStorage.setItem('list',JSON.stringify(this.list))
    },
    del(key){
        this.list.splice(key,1);
        localStorage.setItem('list',JSON.stringify(this.list))
    },
    saveList(){
      localStorage.setItem('list',JSON.stringify(this.list))
    }

  },
  mounted() {
    var list = JSON.parse(localStorage.getItem('list'))
    if(list){
      this.list = list;
    }
  },
}
</script>

<style>
  .inputText{
    border-radius: 20px;
    border: solid 1px gray;
    padding: 20px 10px;
    display: inline;
  }
  .btn{
    background: skyblue;
    border: 2px red solid;
    padding: 20px 10px;
    display: inline;
  }
</style>


```

这里可以将storage封装成一个模块，暴露出来引用就行。





