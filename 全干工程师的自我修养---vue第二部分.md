# 全干工程师的自我修养---vue第二部分

## 1、组件

### 1.1 创建组件

在components里面放置我们自己写的组件，例如在components文件夹里面创建一个Home组件。

创建好组件后，我们来看看组件由哪些组成呢？

有模板 template  script style

Home.vue

```vue
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

## 1.2使用Home组件

在app.vue中引入组件并使用：

第一步：引入组件

第二部：挂载组件

第三部： 在模板中使用

app.vue

```vue
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

## 1.3 组件运用

1、创建一个News.vue新闻组件

```vue
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

```
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



