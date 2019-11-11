# 全干工程师的自我修养---vue第二部分

## 1、组件

### 1.1 创建组件

在components里面放置我们自己写的组件，例如在components文件夹里面创建一个Home组件。

创建好组件后，我们来看看组件由哪些组成呢？

有模板 template  script style

Home.vue

```html
<template>
  <div class="header">
    <h2 class="word">{{msg}}</h2>
    <button class="btn" @click="run()">点击显示msg</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msg: '我是一个header组件'
    }
  },
  methods: {
    run(){
      alert(this.msg);
    }
  },
}
</script>

```

### 1.2使用Home组件

在app.vue中引入组件并使用：

第一步：引入组件

第二部：挂载组件

第三部： 在模板中使用

app.vue

```html
<template>
  <div id="app">
    <!-- <img src="./assets/logo.png"> -->
    <!--使用组件-->
    <v-home></v-home>
   <!-- <router-view/>  -->
  </div>
</template>

<script>
// 引入组件
import Home from './components/Home.vue'
export default {
  name: 'App',
  data() {
    return {

    }
  },
  // 挂载组件
  components:{
    "v-home": Home
  }

}
</script>

<style>

</style>

```

接下来引入样式

```css
<style scoped>
  .header{
    background: pink;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }
  .btn{
    border: 1px solid skyblue;
    padding: 2px;
    border-radius: 10px;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;

  }
  .word{
    color: red;
  }
</style>

```

css样式上面加上scoped,表示该样式是在局部作用域上起作用

### 1.3 组件运用

1、创建一个News.vue新闻组件

```html
<template>
  <div>

     <h2>这是一个新闻组件</h2>
     <ul>
       <li>11111</li>
       <li>22222</li>
        <li>33333</li>
     </ul>
  </div>
</template>

<script>

</script>

<style  scoped>

</style>

```



2、将Header组件引入News组件

```html
<template>
  <div>
     <h2>这是一个新闻组件</h2>
     <ul>
       <li>11111</li>
       <li>22222</li>
        <li>33333</li>
     </ul>
  </div>
</template>

<script>
//引入header组件
import Header from './Home'
export default {
  data() {
    return {

    }
  },
  //挂载header组件
  components:{
    "v-header": Header,
  }
}
</script>

<style  scoped>

</style>

```



3、使用Header组件

```
<template>
  <div>
  <!--使用header组件-->
    <v-header></v-header>
     <h2>这是一个新闻组件</h2>
     <ul>
       <li>11111</li>
       <li>22222</li>
        <li>33333</li>
     </ul>
  </div>
</template>
```



4、将news组件引入并挂载到app.vue组件

```vue
<script>
// 引入组件
// import Home from './components/Home.vue'
import News from './components/News.vue'
export default {
  name: 'App',
  data() {
    return {

    }
  },
  // 挂载组件
  components:{
    // "v-home": Home,
    "v-news": News
  }

}
</script>
```

6、使用组件

```
<template>
  <div id="app">
   <v-news></v-news>
  </div>
</template>
```

## 2、生命周期函数

定义：组件挂载及组件更新组件销毁时触发的一系列方法

示意图：![Vue 实例生命周期](https://cn.vuejs.org/images/lifecycle.png)

共有8个声明周期函数，在这里，我们选取一些出来：

```
  mounted() {
    console.log("mounted")
  },
  beforeCreate() {
    console.log("before create")
  },
  beforeMount() {
    console.log("before mount")
  },
  beforeUpdate() {
    console.log("before update")
  },
  destroyed() {
    console.log("destoryed")
  },
  beforeDestroy() {
    console.log("before destroy")
  },
  created() {
    console.log("created")
  },
  updated(){
       console.log("updated")
  }
```

一运行这些函数就会运行：

```
before create
Life.vue?3ece:40 created
Life.vue?3ece:28 before mount
Life.vue?3ece:22 mounted
```

一修改数据：

```
before update
Life.vue?3ece:43 updated
```

最后还有销毁，这个函数在销毁的时候调用。

news，vue

```vue
<template>
  <div>
    <v-header></v-header>
     <h2>这是一个新闻组件</h2>
     <ul>
       <li>11111</li>
       <li>22222</li>
        <li>33333</li>
     </ul>
     <hr>
     <v-life v-if="flag"></v-life>
     <button @click="flag=!flag">挂载及卸载life组件</button>
  </div>
</template>

<script>
import Header from './Home.vue'
import Life from './Life.vue'
export default {
  data() {
    return {
      flag: true,

    }
  },
  components:{
    "v-header": Header,
    "v-life": Life
  }
}
</script>

<style  scoped>

</style>

```

life.vue

```
<template>
  <div>
      <p>生命周期的演示 ------{{msg}}</p>
      <button class="btn" @click="setMsg()" >change msg</button>
  </div>

</template>

<script>
export default {
  data() {
    return {
      msg: 'msg',

    }
  },
  methods: {
    setMsg(){
      this.msg = "我是改变后的msg"
    }
  },
  mounted() {
    console.log("mounted")
  },
  beforeCreate() {
    console.log("before create")
  },
  beforeMount() {
    console.log("before mount")
  },
  beforeUpdate() {
    console.log("before update")
  },
  destroyed() {
    console.log("destoryed")
  },
  beforeDestroy() {
    console.log("before destroy")
  },
  created() {
    console.log("created")
  },
    updated(){
       console.log("updated")
  }
}
</script>

<style scoped>

</style>

```

点击卸载时，就会调用destory及beforeDestory方法。

## 3、请求数据模块vue-source

### 3.1安装vue-resource

第一步： 打开<https://github.com/>

第二步： 搜索vue-resource

第三部： 进入<https://github.com/pagekit/vue-resource>

第四部： 查看文档

进入文档后，我们可以看到要使用这个模块得先安装它，具体安装如下：

**You can install it via [yarn](https://yarnpkg.com/) or [NPM](http://npmjs.org/).**

```
$ yarn add vue-resource
$ npm install vue-resource --save
```

**CDN**

Available on [jsdelivr](https://cdn.jsdelivr.net/npm/vue-resource@1.5.1), [unpkg](https://unpkg.com/vue-resource@1.5.1) or [cdnjs](https://cdnjs.com/libraries/vue-resource).

```
<script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>
```

并且还有实例：

**Example**

```
{
  // GET /someUrl
  this.$http.get('/someUrl').then(response => {

    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });
}
```

### 3.2引入vue-resource

在main.js文件中:

```
import VueResouce from 'vue-resource'
```

接下来还要使用：

```
Vue.use(VueResouce)
```

然后创建我们自己的实例，帮助我们理解vue-resource的用法：

创建requestData.vue

```vue
<template>
  <div>
    <button class="btn" @click="getApiData()">请求api数据</button>
  </div>
</template>
<script>
export default {
  data() {
    return {
     
    }
  },
  methods: {

    getApiData(){
      var api= 'http://www.phonegap100.com/appapi.php?a=getPortalList&catid=20&page=1';
       // GET /someUrl
      this.$http.get(api).then(response => {

       
        console.log(response)

      }, response => {
        // error callback
        console.log("未访问到数据")
      });
    }
  },
}
</script>

<style scoped>

</style>

```

将它注册到app.vue组件上面去。

点击请求按钮：获得json数据:

```
Response {url: "http://www.phonegap100.com/appapi.php?a=getPortalList&catid=20&page=1", ok: true, status: 200, statusText: "OK", headers: Headers, …}
body: {result: Array(20)}
bodyText: "{"result":[{"aid":"499","catid":"20","username":"admin","title":"\u3010\u56fd\u5185\u9996\u5bb6\u3011\u5fae\u4fe1\u5c0f\u7a0b\u5e8f\u89c6\u9891\u6559\u7a0b\u514d\u8d39\u4e0b\u8f7d","pic":"portal\/201610\/13\/211832yvlbybpl3rologrr.jpg","dateline":"1476364740"},{"aid":"498","catid":"20","username":"admin","title":"ionic\u57df\u8d44\u6e90\u5171\u4eab CORS \u8be6\u89e3","pic":"","dateline":"1472952906"},{"aid":"497","catid":"20","username":"admin","title":"\u79fb\u52a8\u7aef\u89e6\u6478\u6ed1\u52a8js\u63d2\u4ef6_html5\u624b\u673a\u7aef\u8f6e\u64ad\u63d2\u4ef6","pic":"portal\/201606\/28\/211604ullzo5arr4iurnum.jpg","dateline":"1467119820"},{"aid":"496","catid":"20","username":"admin","title":"\u672a\u6765\u7a0b\u5e8f\u5458\u4f1a\u88ab\u673a\u5668\u4eba\u53d6\u4ee3\u5417\uff1f","pic":"portal\/201606\/02\/221818eafffffm4srfdf4s.jpg","dateline":"1464874140"},{"aid":"495","catid":"20","username":"admin","title":"\u9524\u5b50\u5b89\u5168\u9524_\u9524\u5b50\u771f\u7684\u51fa\u4e86\u4e2a\u201c\u9524\u5b50\u201d\uff1a\u8f66\u5145\uff0b\u5b89\u5168\u9524","pic":"portal\/201605\/20\/213752f6i56f1e0hbfzhkb.jpg","dateline":"1463751505"},{"aid":"494","catid":"20","username":"admin","title":"html5\u80fd\u505a\u4ec0\u4e48_html5\u80fd\u505a\u54ea\u4e9b\u5f00\u53d1\uff1f","pic":"","dateline":"1463664540"},{"aid":"493","catid":"20","username":"admin","title":"\u5e73\u5b89\u53e3\u888b\u94f6\u884cApp\u91c7\u7528-Cordova\u6df7\u5408\u5f00\u53d1","pic":"","dateline":"1463294580"},{"aid":"492","catid":"20","username":"admin","title":"JavaScript Emoji \u8868\u60c5\u5e93_js \u7c7b\u4f3c\u4e8eqq\u5fae\u4fe1\u7684\u8868\u60c5\u5e93","pic":"portal\/201604\/25\/084907r2e3im3dvd1q3f7z.jpg","dateline":"1461545392"},{"aid":"491","catid":"20","username":"admin","title":"cordova\u70ed\u66f4\u65b0\u63d2\u4ef6-\u4e0d\u53d1\u5e03\u5e94\u7528\u5e02\u573a\u52a8\u6001\u66f4\u65b0APP\u6e90\u7801","pic":"portal\/201604\/12\/152638zaxz5xz3t58bfts2.png","dateline":"1460446140"},{"aid":"490","catid":"20","username":"admin","title":"\u592e\u884c\u65b0\u89c4\uff01\u652f\u4ed8\u5b9d\u3001\u5fae\u4fe1\u7528\u6237\u522b\u5fd8\u505a\u8fd9\u4ef6\u4e8b","pic":"portal\/201603\/29\/144942tcnnenueefagukfk.jpg","dateline":"1459234206"},{"aid":"471","catid":"20","username":"admin","title":"HTML5 \u79fb\u52a8app\u5f00\u53d1\u6846\u67b6\u8be5\u5982\u4f55\u9009\u62e9","pic":"portal\/201511\/15\/163112q4kz6k2rgcgpi1tc.jpg","dateline":"1457771160"},{"aid":"488","catid":"20","username":"admin","title":"\u7eafCSS3\u52a8\u753b\u6309\u94ae\u6548\u679c,\u53ef\u7528\u4e8e\u79fb\u52a8wap app\u5f00\u53d1","pic":"portal\/201603\/09\/202742r1kngyt17na7n1nk.jpg","dateline":"1457526780"},{"aid":"487","catid":"20","username":"admin","title":"\u4eac\u4e1c\u6bcf\u5929\u4e8f\u4e0a\u4ebf_\u4e0d\u4f1a\u6284\u88ad\u3001\u527d\u7a83?\u5fc5\u5c06\u6b7b\u5728\u4e92\u8054\u7f51\u4e0b\u4e00\u7ad9\u7684\u8d77\u70b9\u4e0a! ...","pic":"portal\/201603\/02\/155825h28zxs2vsxjccv4c.jpg","dateline":"1456905746"},{"aid":"486","catid":"20","username":"admin","title":"ionic react-native\u548cnative\u5f00\u53d1\u79fb\u52a8app\u90a3\u4e2a\u597d","pic":"portal\/201602\/25\/193433dtzfvlzl1oavhljy.jpg","dateline":"1456398960"},{"aid":"484","catid":"20","username":"admin","title":"\u8fd912\u884c\u4ee3\u7801\u5206\u5206\u949f\u8ba9\u4f60\u7535\u8111\u5d29\u6e83\u624b\u673a\u91cd\u542f","pic":"","dateline":"1453426595"},{"aid":"483","catid":"20","username":"admin","title":"\u7f57\u632f\u5b87\u7f57\u6c38\u6d69\u96f7\u519b\u4eec\u7684\u6f14\u8bb2 \u4f60\u559c\u6b22\u54ea\u4e00\u4e2a","pic":"","dateline":"1452226800"},{"aid":"482","catid":"20","username":"admin","title":"ionic-native-transitions\u8ba9\u4f60\u7684Ionic\u5e94\u7528\u6bd4\u539f\u751f\u8fd8\u5feb","pic":"portal\/201601\/07\/135529z4r7gwglv4rw8l74.jpeg","dateline":"1452145500"},{"aid":"481","catid":"20","username":"admin","title":"ionic 1.2.4 \u53d1\u5e03\uff0c\u6700\u597d\u7684html5\u79fb\u52a8app\u5f00\u53d1\u6846\u67b6","pic":"portal\/201601\/05\/132107h9bllr7li74zoh49.jpg","dateline":"1451971293"},{"aid":"480","catid":"20","username":"admin","title":"phonegap\u53d1\u5e03\u5e94\u7528\u5230appstore","pic":"portal\/201601\/05\/122115yhh22i77sqn2ijc6.jpg","dateline":"1451967910"},{"aid":"479","catid":"20","username":"admin","title":"HTML5\u4eff\u82f9\u679c\u5e94\u7528\u7684\u52a8\u753b","pic":"portal\/201601\/04\/220252ycyddectvivr55pq.png","dateline":"1451916189"}]}"
headers: Headers {map: {…}}
ok: true
status: 200
statusText: "OK"
url: "http://www.phonegap100.com/appapi.php?a=getPortalList&catid=20&page=1"
data: (...)
__proto__: Object
```

最后，对数据进行封装展示：

在data属性里面添加一个集合接收数据：list:[]

在response里获取数据列表赋值给list

```
  data() {
    return {
      resultList: []
    }
  },
```

```
  // console.log(response)
        this.resultList = response.data.result;
```

将数据展示到页面上：

```
 <hr>
    <div class="details">
      <h2>api获取数据展示</h2>
        <ul>
          <li v-for="(item, index) in resultList" :key="index">title : {{item.title}}</li>
        </ul>
    </div>
```

显示数据如下：

```json
api获取数据展示
title : 【国内首家】微信小程序视频教程免费下载
title : ionic域资源共享 CORS 详解
title : 移动端触摸滑动js插件_html5手机端轮播插件
title : 未来程序员会被机器人取代吗？
title : 锤子安全锤_锤子真的出了个“锤子”：车充＋安全锤
title : html5能做什么_html5能做哪些开发？
title : 平安口袋银行App采用-Cordova混合开发
title : JavaScript Emoji 表情库_js 类似于qq微信的表情库
title : cordova热更新插件-不发布应用市场动态更新APP源码
title : 央行新规！支付宝、微信用户别忘做这件事
title : HTML5 移动app开发框架该如何选择
title : 纯CSS3动画按钮效果,可用于移动wap app开发
title : 京东每天亏上亿_不会抄袭、剽窃?必将死在互联网下一站的起点上! ...
title : ionic react-native和native开发移动app那个好
title : 这12行代码分分钟让你电脑崩溃手机重启
title : 罗振宇罗永浩雷军们的演讲 你喜欢哪一个
title : ionic-native-transitions让你的Ionic应用比原生还快
title : ionic 1.2.4 发布，最好的html5移动app开发框架
title : phonegap发布应用到appstore
title : HTML5仿苹果应用的动画
```

这一节的代码：

requestData.vue:

```
<template>
  <div>
    <button class="btn" @click="getApiData()">请求api数据</button>
    <hr>
    <div class="details">
      <h2>api获取数据展示</h2>
        <ul>
          <li v-for="(item, index) in resultList" :key="index">title : {{item.title}}</li>
        </ul>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      resultList: []
    }
  },
  methods: {

    getApiData(){
      var api= 'http://www.phonegap100.com/appapi.php?a=getPortalList&catid=20&page=1';
       // GET /someUrl
      this.$http.get(api).then(response => {

        // get body data

        // console.log(response)
        this.resultList = response.data.result;

      }, response => {
        // error callback
        console.log("未访问到数据")
      });
    }
  },
}
</script>

<style scoped>

</style>

```

app.vue:

```
<template>
  <div id="app">
    <!-- <img src="./assets/logo.png"> -->
    <!--使用组件-->
    <!-- <v-home></v-home>  -->
   <!-- <router-view/>  -->
  <!-- <v-news></v-news> -->
  <request-data></request-data>
  </div>
</template>

<script>
// 引入组件
// import Home from './components/Home.vue'
// import News from './components/News.vue'
import RequestData from './components/requestData.vue'
export default {
  name: 'App',
  data() {
    return {

    }
  },
  // 挂载组件
  components:{
    // "v-home": Home,
    // "v-news": News
    "request-data": RequestData
  }

}
</script>

<style>

</style>

```

### 3.3本节总结

使用vue-resource的步骤：

1、安装vue-resouce

```
cnpm install vue-source --save
```

2、main.js引入及使用vue-resouce

```
import VueResource from 'vue-source'
Vue.use(VueResource)
```

3、组件中使用

```
this.$http.get(api).then(res=>{
    //成功后返回的res
},res=>{
    //失败后的callback
})
```

## 4、axios的使用

### 4.1安装axios

```
npm install axios --save
```



### 4.2哪里引入哪里使用

具体使用请看: <https://github.com/axios/axios>

我们这里根据官网编写了一个测试实例:

header.vue

```

```

app.vue

```
<template>
  <div id="app">
    <v-header></v-header>
  </div>
</template>

<script>
import Header from './components/Header.vue'
export default {
  name: 'App',
  components:{
    "v-header": Header
  }
  
}
</script>

<style>

</style>

```

点击按钮,显示可得到和vue-resource一样的结果

## 5 父向子组件传值

### 5.1 传值

首先创建2个组件,一个时Home.vue,一个时Header.vue,Home.vue里面引入Header组件,即形成了父子组件关系,

在模板里传入给子组件的值,然后在子组件里使用prop['值'],然后就可以在子组件里使用父组件传的值了.

好了,话不多说,直接上代码:

Home.vue

```
<template>
    <div>
        <v-header :title="title" :run="run"></v-header>
        <p>我是home组件</p>
        
    </div>
</template>
<script>
import Header1 from './Header1.vue'
export default {
    data(){
        return{
            msg: '我是一个home组件',
            title: '首页'
        }
    },
    methods:{
        run(data){
            alert('我是父组件的run方法'+data)
        }
    },
    components:{
        "v-header": Header1,
    }


}
</script>
```



Header.vue

```
<template>
    <div>
        <p>我是Header组件 ----{{title}}</p>
        <button @click="run('123')">执行父组件的run方法</button>
    </div>
</template>
<script>
export default {
    data() {
        return {
            
        }
    },
    props:['title','run']
}
</script>
```

显示如下:

```

```

### 5.2 小节

父组件给子组件传值:

1. 绑定动态属性  :   如上面的  :title
2. 子组件通过props接收父组件传来的值

## 6 父组件自动获取子组件的值

父组件自动获取子组件的数据的方法:

1.调用子组件的时候定义一个ref

```

```

2.在父组件里通过

```
this.$ref.header.属性
this.$ref.header.方法
```

子组件自动获取父组件的数据和方法:

```
this.$parent.数据
this.$parent.方法
```

代码如下:

Home.vue

```
<template>
    <div>
        <v-header ref="header"></v-header>
        <p>我是home组件</p>
        <button @click="getChildData()">获取子组件的数据</button>
        <button @click="getChildrun()">获取子组件的方法</button>
    </div>
</template>
<script>
import Header1 from './Header1.vue'
export default {
    data(){
        return{
            msg: '我是一个home组件',
            title: '首页'
        }
    },
    methods:{
        // 获取子组件的数据
        getChildData(){
            alert(this.$refs.header.msg);
        },
        getChildrun(){
            this.$refs.header.run();
        }
    },
    components:{
        "v-header": Header1,
    }


}
</script>
```

header.vue

```
<template>
    <div>
        <p>我是Header组件</p>
        
    </div>
</template>
<script>
export default {
    data() {
        return {
            msg: '子组件的msg'
        }
    },
    methods:{
        run(){
            alert("子组件的run方法")
        }
    }

}
</script>
```

这样就可以使父组件自动获取子组件的值了.

同理,要使子组件自动获取父组件的值,只需要在子组件里使用this.$parent.数据/方法就行了.

还有非父子组件传值:

## 7 路由

参考:<https://router.vuejs.org/zh/>

1. 安装路由组件

   cnpm install vue-router --save

2. 引入

   创建一个router目录,在里面放置一个index.js

   ```
   import Vue from 'vue'
   import VueRouter from 'vue-router'

   Vue.use(VueRouter)
   ```

3. 使用方法

   ```
   // 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

   // 1. 定义 (路由) 组件。
   // 可以从其他文件 import 进来
   const Foo = { template: '<div>foo</div>' }
   const Bar = { template: '<div>bar</div>' }

   // 2. 定义路由
   // 每个路由应该映射一个组件。 其中"component" 可以是
   // 通过 Vue.extend() 创建的组件构造器，
   // 或者，只是一个组件配置对象。
   // 我们晚点再讨论嵌套路由。
   const routes = [
     { path: '/foo', component: Foo },
     { path: '/bar', component: Bar }
   ]

   // 3. 创建 router 实例，然后传 `routes` 配置
   // 你还可以传别的配置参数, 不过先这么简单着吧。
   const router = new VueRouter({
     routes // (缩写) 相当于 routes: routes
   })

   // 4. 创建和挂载根实例。
   // 记得要通过 router 配置参数注入路由，
   // 从而让整个应用都有路由功能
   const app = new Vue({
     router
   }).$mount('#app')

   // 现在，应用已经启动了
   ```



还有导航守卫等等知识点,具体青去https://router.vuejs.org/zh/

