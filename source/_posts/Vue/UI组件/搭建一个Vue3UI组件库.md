---
title: 搭建一个Vue3UI组件库
date: 2021/3/01 17:30:00
categories: 
- Vue
tags: 
- UI组件
comments: true
top: true
cover: false
keywords: Vue Vue3 UI组件
---

# 前言

>  本文[GIT源码](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E6%89%8B%E5%86%99Promise)

# 一、创建项目

## 1、环境准备
首先需要准备Node和npm环境，大家可以自行百度搜索安装，版本不要太低即可，以下是笔者的版本：

- node 14.10.0
- npm 6.14.8

## 2、安装脚手架
脚手架这里选择最新版的vue-cli4，如果没有安装可以用以下命令先进行安装

```bash
//安装
npm install @vue/cli -g
```

如果已经安装了vue-cli3，则可以按以下命令先卸载，再安装vue-cli4版本

```bash
// 卸载
npm uninstall vue-cli -g
```

## 3、初始化项目
使用脚手架命令创建一个新项目

```bash
//xxx为你的项目名称，不可以包含大些字母
npm create xxx
```
选择第二项，创建Vue3的项目模板
![vue-cli创建新项目](https://img-blog.csdnimg.cn/20210227143855687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsdWVCbHVlQmVycnk=,size_16,color_FFFFFF,t_70)

## 4、配置Sass
CSS预处理器使用的Sass，如果不打算使用预处理可以跳过这一步。先安装node-sass和sass-loader这两个包

> *注意：这里强烈建议大家不要安装最新版本

```bash
//安装node-sass
npm install --save node-sass@4.x
//安装sass-loader
npm install --save sass-loader@8.x
```

# 二、架构搭建
## 1、目录
对新创建的目录进行小修改，整理的目录如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210227154640547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsdWVCbHVlQmVycnk=,size_16,color_FFFFFF,t_70)

## 2、package
新建package文件夹，用来放所有的组件和组件样式。
package文件夹下新建button文件夹用来存放button组件，新建styles文件夹用来存放所有的全局样式文件。具体实现下面来讲。
例如这里我写了一个button的组件，具体代码可以[点击这里](https://github.com/JuneBlueberry/blog-post-code/tree/master/%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E6%90%AD%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%9F%BA%E4%BA%8EVue3%E7%9A%84UI%E7%BB%84%E4%BB%B6%E5%BA%93/cats-ui-demo)查看源码

## 3、package/index.js
这个文件是一个入口文件，会整合所有的组件然后export出去。具体实现如下：

```javascript 
// 导入入口样式文件
import './styles/index.scss'
// 导入所有组件
import { Button } from './button/index'
const components = {
  Button
}
// 定义 install 方法，在Vue中注册组件。如果使用 use 注册插件，则所有的组件都将被注册
const install = function (Vue) {
  //判断是否已经注册
  if (install.installed) return
  //循环注册组件
  Object.keys(components).forEach(key => {
    Vue.component(components[key].name, components[key])
  })
}
// 判断是否是直接引入文件
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}
const API = {
  install,
  ...components
}
export default API
```

## 4、vue.config.js
在vue-cli3之后，vue的配置就放到vue.config.js这个文件中，并且项目创建时目录中没有这个文件，需要手动创建。

```javascript
// vue.config.js
module.exports = {
  // 组件样式内联
  css: {
    extract:{
      filename: `lib/[name].css`,
      chunkFilename: `lib/[name].css`
    },
  },
  // 扩展 webpack 配置，使 packages 加入编译
  chainWebpack: config => {
    config.module
      .rule('js')
      .include
      .add('/packages/')
      .end()
      .use('babel')
      .loader('babel-loader')
      .tap(options => {
        // 修改它的选项...
        return options
      })
  }
}
```

# 四、组件
这里演示一个button组件的实现
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021030112013580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsdWVCbHVlQmVycnk=,size_16,color_FFFFFF,t_70)

## 1、组件编写
- button/button.vue

```html
<template>
  <button :class="['cat-btn', 'cat-btn-type-'+type, 'cat-btn-size-'+size,
  {'is-disabled': disabled,'is-plain': plain, 'is-round': round}]" 
  :disabled="disabled"
  @click="handleClick">
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'cat-button',
  props: {
    type: {
      type: String,
      default: 'default'
    },
    size: {
      type: String,
      default: 'default'
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plain: {
      type: Boolean,
      default: false
    },
    round: {
      type: Boolean,
      default: false
    }
  },
  catthods: {
    /**
     * 按钮点击事件
     */
    handleClick: function(event){
      if(this.disabled){
        return
      }
      this.$emit('click', event)
    }
  }
}
</script>
```

- button/button.scss

```css
.cat-btn{
  font-size: 12px;
  border-radius: 0px;
  padding: 8px 16px;
  height: 32px;
  line-height: 14px;

  border: 1px solid #dddddd;
  background-color: #f7f7f7;
  color: #000000;

  &:hover{
    background-color: #ffffff;
  }
}

//按钮类型--Type
.cat-btn-type-primary{
  border: 1px solid #0099cc;
  background-color: #0099cc;
  color: #ffffff;

  &:hover{
    border: 1px solid #28B5D6;
    background-color: #28B5D6;
  }
}
.cat-btn-type-success{
  border: 1px solid #4db118;
  background-color: #4db118;
  color: #ffffff;

  &:hover{
    border: 1px solid #57bc20;
    background-color: #57bc20
  }
}
.cat-btn-type-info{
  border: 1px solid #909399;
  background-color: #909399;
  color: #ffffff;

  &:hover{
    border: 1px solid #a5a9af;
    background-color: #a5a9af;
  }
}
.cat-btn-type-warning{
  border: 1px solid #e79302;
  background-color: #e79302;
  color: #ffffff;

  &:hover{
    border: 1px solid #ffa200;
    background-color: #ffa200;
  }
}
.cat-btn-type-error{
  border: 1px solid #f25721;
  background-color: #f25721;
  color: #ffffff;

  &:hover{
    border: 1px solid #fc6824;
    background-color: #fc6824;
  }
}

//按钮是否圆角--round
.cat-btn.is-round{
  border-radius: 5px;
}

//按钮是否可点击--disabled
.cat-btn.is-disabled{
  opacity: 0.7;
  cursor: not-allowed;
  background-color: #f7f7f7;
}
.cat-btn-type-primary.is-disabled{
  background-color: #0099cc;
}
.cat-btn-type-success.is-disabled{
  background-color: #4db118;
}
.cat-btn-type-info.is-disabled{
  background-color: #909399;
}
.cat-btn-type-warning.is-disabled{
  background-color: #e79302;
}
.cat-btn-type-error.is-disabled{
  background-color: #f25721;
}

//按钮是否有填充颜色--plain
.cat-btn.is-plain{
  &:hover{
    color: #0099cc;
    border: 1px solid #0099cc;
  }
}
.cat-btn-type-primary.is-plain{
  color: #0099cc;
  border: 1px solid #0099cc;
  background-color: #ecf5ff;

  &:hover{
    color: #ffffff;
    background-color: #0099cc;
    border: 1px solid #0099cc;
  }
}
.cat-btn-type-success.is-plain{
  color: #4db118;
  border: 1px solid #4db118;
  background-color: #f0f9eb;

  &:hover{
    color: #ffffff;
    background-color: #4db118;
    border: 1px solid #4db118;
  }
}
.cat-btn-type-info.is-plain{
  color: #909399;
  border: 1px solid #909399;
  background-color: #f4f4f5;

  &:hover{
    color: #ffffff;
    background-color: #909399;
    border: 1px solid #909399;
  }
}
.cat-btn-type-warning.is-plain{
  color: #e79302;
  border: 1px solid #e79302;
  background-color: #fdf6ec;

  &:hover{
    color: #ffffff;
    background-color: #e79302;
    border: 1px solid #e79302;
  }
}
.cat-btn-type-error.is-plain{
  color: #f25721;
  border: 1px solid #f25721;
  background-color: #fef0f0;

  &:hover{
    color: #ffffff;
    background-color: #f25721;
    border: 1px solid #f25721;
  }
}

//按钮的尺寸--size
.cat-btn-size-big{
  font-size: 16px;
  padding: 8px 16px;
  height: 42px;
  line-height: 26px;
}
.cat-btn-size-catdium{
  font-size: 15px;
  padding: 8px 16px;
  height: 38px;
  line-height: 22px;
}
.cat-btn-size-small{
  font-size: 13px;
  padding: 8px 16px;
  height: 28px;
  line-height: 12px;
}
.cat-btn-size-mini{
  font-size: 12px;
  padding: 8px 16px;
  height: 24px;
  line-height: 8px;
}
```

- button/index.js

```javascript
import Button from './button.vue'
export {
  Button
}
```

## 2、全局样式编写
styles文件夹里面负责入口样式和所有的全局样式，例如global，动画，函数等等。

- styles/index.scss
这个是入口样式文件，会引入所有的全局样式和组件样式

```css
@import "./global.scss";
@import "../button//button.scss";
```

- styles/global.scss

```css
* {
  padding: 0;
  margin: 0;
  position: relative;
}

input, button{
  outline: none;
}

i{
  font-style: normal;
}

*, :after, :before{
  box-sizing: border-box;
}

li{
  list-style: none;
}

table{
  display: table;
  margin: 0;
  padding: 0;
}
```

## 3、组件测试
写好组件之后，我们来测试一下。修改一下项目入口文件src/main.js，将写好的UI组件引入。

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import CatsUI from '../packages/index'

createApp(App).use(CatsUI).mount('#app')
```

```html
// src/App.vue
<template>
  <div>
      <div class="demo">
        <cat-button class="demo1">按钮</cat-button>
        <cat-button class="demo1" type='primary'>按钮</cat-button>
        <cat-button class="demo1" type='success'>按钮</cat-button>
        <cat-button class="demo1" type='info'>按钮</cat-button>
        <cat-button class="demo1" type='warning'>按钮</cat-button>
        <cat-button class="demo1" type='error'>按钮</cat-button>
    </div>
    <div class="demo">
        <cat-button class="demo1" plain>按钮</cat-button>
        <cat-button class="demo1" type='primary' plain>按钮</cat-button>
        <cat-button class="demo1" type='success' plain>按钮</cat-button>
        <cat-button class="demo1" type='info' plain>按钮</cat-button>
        <cat-button class="demo1" type='warning' plain>按钮</cat-button>
        <cat-button class="demo1" type='error' plain>按钮</cat-button>
    </div>
    <div class="demo">
        <cat-button class="demo1" round>按钮</cat-button>
        <cat-button class="demo1" type='primary' round>按钮</cat-button>
        <cat-button class="demo1" type='success' round>按钮</cat-button>
        <cat-button class="demo1" type='info' round>按钮</cat-button>
        <cat-button class="demo1" type='warning' round>按钮</cat-button>
        <cat-button class="demo1" type='error' round>按钮</cat-button>
    </div>
    <div class="demo">
        <cat-button class="demo1" disabled>按钮</cat-button>
        <cat-button class="demo1" type='primary' disabled>按钮</cat-button>
        <cat-button class="demo1" type='success' disabled>按钮</cat-button>
        <cat-button class="demo1" type='info' disabled>按钮</cat-button>
        <cat-button class="demo1" type='warning' disabled>按钮</cat-button>
        <cat-button class="demo1" type='error' disabled>按钮</cat-button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.demo {
  margin: 10px 0;
}
.demo1 {
  margin: 0 10px;
}
</style>
```

最后，结果如下图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210301143855502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsdWVCbHVlQmVycnk=,size_16,color_FFFFFF,t_70)

# 五、npm发布

## 1、编译
修改package.json文件，新增一个lib命令专门进行给组件库进行编译，同时，修改入口文件为编译后的压缩JS文件。

>  lib是vue-cli3 里的一个专门用于打包组件库

```json
// package.json
{
  "name": "cats-ui-demo",
  "version": "0.1.4",
  "private": false,
  "main": "lib/cats-ui-demo.umd.min.js",
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint",
    "lib": "vue-cli-service build --target lib --name cats-ui-demo --dest lib packages/index.js"
  }
  ...
}
```

```bash
npm run lib
```
运行后如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210301151241103.png)


## 2、发布
先在[npm官网](https://www.npmjs.com/)上注册一个账号，然后在控制台中进行登录和发布

```bash
//登录
npm login

//发布
npm publish
```

> 每次发布的版本号(package.json里面的version)不可以是一样的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210301152007480.png)

## 3、测试
最后测试一下所发布的组件库。由于同一个项目不可以安装同名的库，因此新建一个空的项目来测试。此处就省略建项的步骤，不太熟悉的同学可以看看第一步。

```bash
//安装cats-ui-demo
npm install --save cats-ui-demo
```

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import CatsUI from 'cats-ui-demo'
import CatsUICSS from 'cats-ui-demo/lib/cats-ui-demo.css';

createApp(App).use(CatsUI).use(CatsUICSS).mount('#app')
```

最后在App.vue页面应用button组件

```html
//	src/App.vue
<template>
  <div>
    <cat-button class="demo1">按钮</cat-button>
    <cat-button class="demo1" type='primary'>按钮</cat-button>
    <cat-button class="demo1" type='success'>按钮</cat-button>
    <cat-button class="demo1" type='info'>按钮</cat-button>
    <cat-button class="demo1" type='warning'>按钮</cat-button>
    <cat-button class="demo1" type='error'>按钮</cat-button>
  </div>
</template>

<script>
export default {
  name: 'App',
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.demo1{
  margin: 0 10px;
}
</style>
```

结果如下图所示

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021030116275862.png)
搞定！收工！