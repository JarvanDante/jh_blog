---
author: "karson"
title: "响应式布局"
date: 2022-07-03
description: "一个网站能够兼容多个终端——而不是为每个终端做一个特定的版本。这个概念是为解决移动互联网浏览而诞生的"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags:
- javascript
- categories:

---
## 缺点
1. 当一个网内容特别多(如：淘宝)的情况下，要一套代码响应多端这个工作比较繁琐，最关键的问题是移动端网站速度太慢了
2. SEO的问题


>PS:问题：什么样的网站适合"完全"响应式？[一个url响应多端]
> 答：展示类的网站 ==> 文档类、网站官网
> *** 其它的不建议完全的响应式，但可以写一点点@media

### PC端跳转移动端
```vue
<script>
function isMobile(){
  var userAgentInfo = navigator.userAgent;
  var Agents = ["Android","iPhone","SymbianOS","Windows Phone","iPad","iPod"];
  var flag = false;
  for (var v=0;v<Agents.length;v++){
    if(userAgentInfo.indexOf(Agents[v])>0){
      flag=true;
      break;
    }
  }
  return flag;
}
var flag = isMobile();
if(flag){
  window.location.href="xxx.html"
}
</script>
```

## 响应式网页设计
### 对移动设备开启响应式
在html头部中添加 `meta` 元素：
```vue
<meta name="viewport" content="width=device-width,initial-scale=1.0"/>
```
### 宽度设计(纵向)
1. 方式一：百分比
```vue
.container{
  width:100%;
}
```
2. 方式二：max-width
```vue
.container{
  max-width:80%;
  margin:0 auto;
} 
```
3. 方式三：固定宽度
```vue
@media(max-width:900px){
  .container{
    width:600px;
  }
}

@media(max-width:700px){
    .container{
        width:500px;
    }
}
```

### 宽度设计(横向)
#### flex横向多列布局：
```vue
/* flex */
.container{
  display:flex;
}
//解决方案如下：
/* 当屏幕过窄， 会影响阅读，设置换行 */
.container{
  display:flex;
  flex-wrap:wrap;
}
/* 每个 flex item 最小宽度 */
.container p{
  flex:250px;
}
```

#### grid横向多列布局：
```vue
/* Grid */
.container{
  display:grid;
}

//解决方案一如下：
//在一行中容纳最多数量的列
.container{
  display:grid;
  grid-template-columns:repeat(auto-fill,minmax(250px,1fr))
}

//解决方案二如下：
//使用media query 手动控制列数
.container{
  display:grid;
  grid-template-columns:1fr 1fr 1fr;
}
@media(max-width:900px){
  .container{
    grid-template-columns:1fr 1fr;
  }
}
@media(max-width:700px){
    .container{
      grid-template-columns:1fr;
    }
}
```

### 图片设计
```vue
img {
  max-width:100%;
}
```
#### 不同宽度加载不同图片
1. 方式一：`srcset` + `sizes`
```vue
<img 
    src="../image-300.png"
    srcset="
      ../image-1240.png 1240w,
      ../image-600.png 600w,
      ../image-300.png 300w,
    "
    sizes="(max-width:400px) 300px,(max-width:900px) 600px,1240px"
/>
```
2.方式二：`picture`
```vue
<picturn>
  <sorce media="(max-width:400px)" srcset="../image-300.png" />
  <sorce media="(max-width:900px)" srcset="../image-600.png" />
  <img src="../image-1240.png" />
</picturn>
```

### 字体设计
1. 方式一：百分比
```vue
h1 {
  font-size:6vw;
}
或
h1 {
  font-size:6rem;
}
//升级：最小2rem
h1 {
  font-size: calc(2rem+2vw);
} 
```
2. 方式二：media quary
```vue
@media(max-width:900px){
  h1{
    font-size:3rem;
  }
}
@media(max-width:700px){
  h1{
    font-size:2rem;
  }
} 
```