---
author: "jarvan"
title: "vue3框架"
date: 2023-06-01
description: "Vue.js是一种流行的JavaScript前端框架，用于构建用户界面。它是一种渐进式框架，可以逐步应用到项目中，也可以与其他库或现有项目进行整合。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- javascript
- categories:

---
# 前期准备
## 插件
+ VScode + Vue Language Features (Volar) -- 高亮
+ edge 浏览器插件 vue devtools
## 创建项目
```vue
npm init vue@latest
# 一路回车
npm install
npm run dev
npm run build
```
+ docker外网访问，解决方法
```vue
# package.json 中修改 dev --host
  "scripts": {
    "dev": "vite --host 172.19.0.13",
    "build": "vite build",
    "preview": "vite preview"
  },

```
`package.json`
`index.html`
`main.js`
`app.vue`
## MVVM的演示
```vue
<!--脚本-->
<script>
  export default{
    data:()=>({
      account:'abc'

    })
  }
</script>
<!--视图-->
<template>
  <input type="text" v-model="account">
  <h1>这是我的第一个vue</h1>
</template>
<!--样式-->
<style scoped>

</style>
```

## vue的组件风格
vue的组件可以按两种不同的风格书写
+ 选项式API
+ 组合式API
### 选项式API
可以用包含多个选项的 对象 来描述组件的逻辑，如： `data` 、 `methods` 和 `mounted`
选项所定义的属性都会暴露在函数内部的 `this` 上，它会指向当前的组件实例
```vue
<script>
  export default{
    data:()=>({
      account:'abc'

    })
  }
</script>
```

+ 响应式数据的声明
```vue
<script>
export default {
  //data选项是一个函数返回的对象
  data: () => ({
    account: "ABc",
    student: {
      name: "jack",
      age: 30,
    },
  }),
</script>
{{ account }}

```

### 组合式API
可以使用导入的API函数来描述组件逻辑
在单文件组件中，组合式API通常会与`<script setup>`搭配使用
`setup` 属性是一个标识，告诉vue需要在编译时进行一些处理，可以更简洁地使用组合式API
`<script setup>`可以直接使用
```vue
<script setup>
//引入API函数
import { ref } from "vue";

//数据源
let account = ref("afdsaf");

//方法
function changeAccount() {
  account.value += "??????";
}
</script>

<input type="text" v-model="account" />

<button @click="changeAccount">点我更改account</button>
```
+ 响应式数据的声明
* 普通变量的数据源，不具备响应式对象
* 普通变量使用`ref`函数后的数据源，具备响应式对象
```vue
let account = ref("xyz");
account.value //获取
```
* 使用reactive函数声明`原始类型`的数据源，不具备响应式对象
* 使用reactive函数声明`对象类型`的数据源，具备响应式对象

```vue
//reactive把对象类型数据源，变为响应式对象
<script setup>
import { reactive } from "vue";
import { ref } from "vue";
//普通对象类型的数据源，具备响应式对象
let emp = reactive({
salary: 88000,
name: "wjb",
});
function changeEmpSalary() {
emp.salary += 100;
console.log(emp);
}
//ref把变通变量数据源，变为响式对旬，要用.value来获取值
let account = ref("xyz");
function changeAccount() {
  account.value += "+";
  console.info(account);
}
</script>
```
## vue的常用指令
### v-text 指令
1.采用纯文字的方式填充数据
2.加在空元素上

### v-html 指令
1.以html语法显示数据
2.加在空元素上

### {{  }}插值表达式
1.可以放任意元素中

### v-model 指令
```vue
<script>
<div>账号：{{ student.name }}</div>
<!-- 单行文本框 -->
<input type="text" v-model="student.name" />
<!-- 多行 -->
<textarea v-model="student.name"></textarea>
<!-- 复选 框true/false -->
<input type="checkbox" v-model="student.open" />灯
<!-- 复选 框true-value/false-value -->
<input
    type="checkbox"
    true-value="确定"
    false-value="不确定"
    v-model="student.open"
/>选择

<input type="checkbox" value="zq" v-model="student.likes" />足球
<input type="checkbox" value="lq" v-model="student.likes" />篮球
<input type="checkbox" value="ymq" v-model="student.likes" />羽毛球
<!-- 单选框 -->
<input type="radio" value="man" v-model="student.sex" />男
<input type="radio" value="woman" v-model="student.sex" />女
<!-- 下拉框 -->
<select v-model="student.level">
<option value="C">初级</option>
<option value="B">中级</option>
<option value="A">高级</option>
</select>
<!-- 多选下拉框 -->
<select multiple v-model="student.city">
<option value="吉C">通化</option>
<option value="吉B">吉林</option>
<option value="吉A">长春</option>
</select>
</script>
```
#### 转换数据类型
v-model.number
v-model.trim
v-model.lazy

```vue
<!-- 用户输入的值自动转成数值 -->
  <input type="text" v-model.number="student.age" />

  <!-- 用户输入的值自动过滤空白字符-->
  <input type="text" v-model.trim="student.nickname" />

  <!-- 懒、用户输入的值change后自动转成数值 -->
  <input type="text" v-model.lazy.number="student.age" />
```

### v-bind 绑定指令
#### 1.绑定class类
PS ： 如果将属性绑定的值为null值，会直接迁移掉这个属性
```vue
<script>
import { reactive } from "vue";
export default {
  data: () => ({
    picture: {
      with: 200,
      src: "http://localhost:1313/images/whoami/avatar.jpg",
    },
  }),
};
</script>

<template>
  <!-- v-bind -->
  <img v-bind:src="picture.src" v-bind:width="picture.with" />
  <!-- 精简 :-->
  <img :src="picture.src" :width="picture.with" />
  <input type="text" v-model="picture.with" />
</template>
```
绑定多个属性/值
```vue
<script setup>
import { reactive } from "vue";
let attrs = reactive({
  class: "error",
  id: "borderBlue",
});
</script>

<template>
  <!-- 绑定多个属性及值 -->
  <button v-bind="attrs">我是一个普通按钮</button>
</template>
<style>
.error {
  background-color: red;
  color: white;
}
#borderBlue {
  border: 2px solid rgb(44, 67, 167);
}
</style>

```
class数组
```vue
let btnTheme=ref([])
<input type="checkbox" value="error" v-model="btnTheme"> error

<input type="checkbox" value="flat" v-model="btnTheme"> flat
<button :class="btnTheme">按钮</button>

<!-- 数组和对象可以一起使用 -->
<button :class="[{'rounded':capsule},widthTheme]">按钮</button>
```

#### 2.绑定style样式
1. 绑定数据源对象
```vue
<script>
export default {
  data: () => ({
    btnTheme: {
      backgroundColor: "#FF0000",
      color: "#000000",
    },
    backColor: "#0000FF",
    textColor: "#FFFFFF",
    borRadius: 20,
  }),
};
</script>
背景色：<input type="color" v-model="btnTheme.backgroundColor" />error
  字体色：<input type="color" v-model="btnTheme.color" />flat
  <br />
  <br />
  <!-- style 可以直接绑定对象数据源，但是对象数据源的属性名称不能随便写，(驼峰命名法替代'-') -->
  <button :style="btnTheme">普通按钮</button>
```
2. 直接传属性对象
```vue
<!-- style可以传对象 -->
背景色：<input type="color" v-model="backColor" />

字体色：<input type="color" v-model="textColor" />

边框圆角：<input type="range" min="0" max="20" v-model="borRadius" />

<button
    :style="{
      backgroundColor: backColor,
      color: textColor,
      'border-radius': borRadius + 'px',
    }"
>
普通按钮
</button>

```

### v-if/v-else-if/v-else 判断指令
```vue
<script>
export default {
  data: () => ({
    isShow: false,
  }),
};
</script>

<template>
  是否显示：<input type="checkbox" v-model="isShow" />
  <h3 v-if="isShow">这是一个标签</h3>
</template>
```

### v-show 是否显示
```vue
export default {
data: () => ({
isShow: false,
show: true,
}),
};
</script>

<template>
  是否显示：<input type="checkbox" v-model="isShow" />
  <h3 v-if="isShow">这是一个标签</h3>
  <h3 v-show="show">显示</h3>
</template>
```

### v-on 事件绑定指令(缩写：@)
```vue
<script>
export default {
  data: () => ({
    volume: 5,
  }),
  methods: {
    addVolume() {
      if (this.volume !== 10) {
        this.volume++;
      }
    },
  },
};
</script>

<template>
  <h3>当前音量：{{ volume }}</h3>
  <button v-on:click="addVolume">添加音量</button>
  <button @:click="addVolume">添加音量</button>
</template>
```

### 事件的修饰符

1. `.prevent` 阻止默认行为
```vue
<script>
export default {
  data: () => ({
    volume: 5,
  }),
  methods: {
    say() {
      window.alert("hi");
    },
  },
};
</script>

<template>
  <a href="www.baidu.com" @click.prevent="say">百度</a>
</template>
```
2. `.stop`阻止事件冒泡
```vue
<script>
export default {
  data: () => ({
    volume: 5,
  }),
  methods: {
    say(name) {
      console.log("hi" + name);
    },
  },
};
</script>

<template>
  <div class="divArea" @click="say('DIV')">
    <button @click.stop="say('BUTTON')">按钮</button>
  </div>
</template>
```

3. `.capture` 以捕获模式触发当前的事件处理函数
给该元素添加了一个监听器
先触发该元素的事件，都有修饰符，由外向内触发
4. `.once`绑定事件只触发1次
5. `.self`只有在event.target是当前元素自身时触发事件处理函数
对只该元素上触发事件有效
6. `.passive` 向浏览器表明了不想阻止事件的默认行为

### 按键的修饰符
`.enter` / `.tab` / `.shift` / `.exact`
```vue
按键的键盘中包含enter事件<input
    type="text"
    @keydown.enter="showMessage('按下了enter键')"
  />
  按键的键盘中包含enter+shift事件<input
    type="text"
    @keydown.enter.shift="showMessage('按下了enter+shift键')"
  />
```
### 鼠标按键的修饰符
`.left` / `.right` / `.middle`

### v-for渲染数组
```vue
<script setup>
import { ref } from "vue";
let subject = ref([
  { id: 1, name: "vue" },
  { id: 2, name: "python" },
  { id: 3, name: "java" },
  { id: 4, name: "php" },
]);
</script>

<template>
  <!-- 对象 -->
  <ul>
    <li v-for="sub in subject">编号：{{ sub.id }} -- 名称：{{ sub.name }}</li>
  </ul>
  <!-- 解构 -->
  <ul>
    <li v-for="{ id, name } in subject">编号：{{ id }} -- 名称：{{ name }}</li>
  </ul>
  <!-- 索引 -->
  <ul>
    <li v-for="(sub, index) in subject">
      编号：{{ sub.id }} -- 名称：{{ sub.name }} -- 索引：{{ index }}
    </li>
  </ul>
</template>
```

### v-for渲染对象
```vue
 <ul>
    <li v-for="value in student">{{ value }}</li>
  </ul>
  <ul>
    <li v-for="(value, name) in student">
      属性名：{{ name }}--属性值：{{ value }}
    </li>
  </ul>
  <ul>
    <li v-for="(value, name, index) in student">
      属性名：{{ name }}--属性值：{{ value }}--索引：{{ index }}
    </li>
  </ul>
```

### key管理状态
```vue
<script setup>
import { ref } from "vue";
let subject = ref([
  { id: 1, name: "java" },
  { id: 2, name: "python" },
  { id: 3, name: "php" },
  { id: 4, name: "go" },
]);
function addSub() {
  subject.value.unshift({ id: 5, name: "hadoop" });
}
</script>

<template>
  <button @click.once="addSub">添加课程</button>
  <ul>
    <li v-for="sub in subject" :key="sub.id">
      <input type="checkbox" />
      课程：{{ sub }}
    </li>
  </ul>
</template>
```

## 侦听器
### 选项式API中的侦听器
选项式API中使用 `watch` 选项在每次响应式属性发生变化是触发一个函数
1. 函数式侦听器
```vue
<script>
export default {
  data: () => ({
    age: 30,
    emp: {
      name: "jack",
      sex: 1,
    },
  }),
  watch: {
    /**
     * 侦听age数据源是否变化
     * @param {*} newData
     * @param {*} oldData
     */
    age(newData, oldData) {
      console.log("oldData:" + oldData);
      console.log("newData:" + newData);
    },
    "emp.name"(newData, oldData) {
      console.log("oldData:" + oldData);
      console.log("newData:" + newData);
    },
  },
};
</script>

<template>
  <input type="text" v-model="age" />
  <input type="text" v-model="emp.name" />
</template>
```
2. 对象式侦听器
+ deep 深度侦听
```vue
deep:true
```
+ immediate 创建时立即触发
```vue
immediate:true
```
+ flush 改回调机制(DOM更新后)
```vue
flush:'post'
```

3. this.$watch 侦听器
允许提交停止该侦听器
语法：`this.$watch(data,method,object)`
data:侦听数据源，类型为string
method:回调函数，参数一新值，参数二旧值
object:配置
    a. deep:深度侦听
    b. immediate:创建时立即触发
    c. flush:'post'：要改回调机制(DOM更新后)
4. 停止侦听器
```vue
accountStop: null,
...
this.accountStop = this.$watch(
"age",
(newData, oldData) => {
console.log("年龄的新旧值1");
console.log(newData);
console.log(oldData);
},
{ immediate: true }
);
```

### 组合式API中的侦听器

#### watch函数
```vue
//侦听对象的某个属性，要以对象的方式
watch(
  () => emp.salary,
  (newData, oldData) => {
    console.log("=== 员工薪资新旧值 ===");
    console.log(newData);
    console.log(oldData);
  }
)
```
#### watchEffect函数
立即回调
```vue
<script setup>
import { reactive, ref, watch, watchEffect } from "vue";

let account = ref("Abc");

watchEffect(() => {
  console.log(account.value);
});
</script>
```
* 默认情况下，回调触发机制，在DOM更新之前
* flush:'post' 在DOM更新之后
#### watchPostEffect函数
`在DOM更新之后`
watchEffect函数 + flush:'post'

## 计算属性
`computed(()=>{})`
```vue
<script setup>
import { computed, ref } from "vue";

let age = ref(20);
//计算属性
let ageState = computed(() => {
  if (age.value < 18) {
    return "未成年";
  } else if (age.value < 60) {
    return "中年";
  } else {
    return "老年";
  }
});
</script>
<template>
  年龄：<input type="number" v-model="age" />
  <p>年龄阶段：{{ ageState }}</p>
</template>
```
* 计算属性与方法的区别：
1.两种方式的结果确实是完全相同的，不同之处在于计算属性值会基于响应式依赖被缓存。
2.计算属性：仅会在响应式依赖数据更新时才会重新计算
3.方法：总是会在页面渲染发生时再次执行函数

## 组件
一个vue组件在使用前需要先被'注册'，这样vue才能在渲染模板时找到其对应的实现；组件注册有两种方式：`全局注册` `局部注册`
### 全局注册组件
可使用`app.component(name,Component)` 注册组件的方法，在此应用的任意组件的模板中使用
+ name:注册的名字
+ Component:需要注册的组件
```vue
//在main.js中
...
import App from './App.vue'
//1、引入需要注册的组件
import LoginVue from './components/Login.vue'
//
let app = createApp(App)

//--
//2.全局注册组件
app.component('MLogin',LoginVue)
//--

app.mount('#app')
...
```
```vue
//App.vue中
<script></script>
<template>
  <!-- 使用全局注册的组件 -->
  <MLogin/>
</template>

```
### 局部注册组件
局部注册的组件需要在使用它的父组件中显式导入，并且只能在该父组件中使用。
1. 在选项式API中，可以使用`components`选项来局部注册组件
```vue
<script>
//1、引入需要注册的组件
import LoginVue from './components/Login.vue'
export default {
  //2.注册组件选项
  components:{
      "MLogin":LoginVue //名字一样的话，直接写组件名即可LoginVue
  }
}
</script>

<template>
  <!-- 使用局部注册的组件 -->
  <MLogin/>
</template>
```
2. 在组合式API中的`<script setup>`内，直接导入的组件就可以在模板中直接可用，无需注册
```vue
<script setup>
//1、引入需要注册的组件
import LoginVue from './components/Login.vue'
</script>
<template>
  <LoginVue/>
</template>
```

### 数据传递
如果父组件向子组件进行传递数据，那么我们需要在子组件中声明`props`来接收传递数据的属性，可采用
1、字符串数组式 或
2、对象式来声明`props`
#### 方式一：字符串数组式
+ 定义 `defineProps`
```vue
<script setup>
defineProps(['title',''error','flat'])
</script>
<template>
  <button :class="{error,flat}">
    {{title}}
  </button>
</template>
button {
  border:none;
  padding:12px 25px;
}
.error {
  background-color:rgb(197,75,75);
  color:white;
}
.flat {
  box-shadow:0 0 10px grey;
}
```
+ 使用
```vue
<script setup>
import { ref } from 'vue';
import ButtonVue from './components/Button.vue'
let isError =ref(false)
let isFlat = ref(false)
let btnText = ref('普通按钮')

</script>
<template>
  主题：<input type="checkbox" v-model="isError">
  阴影：<input type="checkbox" v-model="isFlat">
  按钮文本：<input type="text" v-model="btnText">
  <ButtonVue title="提交" :error="true" :flat="false" />
</template>
```
注意``不能直接修改 props 的数据，因为是只读的，只能父组件修改``
#### 方式二：对象式来声明
注意：
1. 所有`prop`默认都是可选的，除非声明了`required:true`
2. 除`Boolean`外的传递的可选`prop`将会有一个默认值`undefined`
3. `Boolean`类型的未传递`prop`将被转换为`false`
4. 当`prop`的校验失败后，Vue会抛出一个控制台警告
5. 注意`prop`的校验是在组件实例被创建之前
    a.在选项式API中，实例的属性（比如 `data` 、` computed` 等）将在 `default` 或 `validator` 函数中不可用
    b.在组合式API中，`defineProps`宏中的参数不可以访问`<script setup>`中定义的其他变量。
```vue
<script setup>
let propsData = defineProps({
  title:{
    type:String,
    required:true
  },
  error:Boolean,
  flat:Boolean,
  tips:{
    type:String,
    default:'我是一个普通的按钮'
  }
})
</script>
```
对象中的属性：
+ `type`:类型，如 String,Number,Boolean,Array,Object,Date
+ `default`:默认值：对象或者数组应当用工厂函数返回
+ `required`:是否必填，布尔值
+ `validator`:自定义校验，函数类型


注意：
```vue
关于Boolean类型转换：
为了更贴近原生 boolean attributes 行为，声明为 Boolean 类型的 props 有特别的类型转换规则
如声明时： defineProps{{ error:Boolean}}
传递数据时：
 <MyComponent error/> 相当于 <MyComponent :error="true" />
 <MyComponent /> 相当于 <MyComponent :error="false" />
```
### 自定义事件 emits
子组件中如下：
```vue
<script>
export default {
  //自定义事件选项
  emits:['changeAge','changeAgeAndName'],
  methods:{
    emitEventAge(){
      //选项式通过 this.$emit 触发自定义事件
      this.$emit('changeAge',30)
    }
  }
}
</script>
<template>
  <button @click="emitEventAge">更改年龄</button>
  <br>
  <button @click="$emit('changeAgeAndName',10,'Annie')">更改年龄</button>
</template>
```
父组件中如下
```vue
<script>
import StudentVue from './compoments/Student.vue'
export default {
  components:{StudentVue},
  data:()=>({
    student:{
      name:'Jack',
      age:18,
      sex:'男',
    }
  }),
  methods:{
    getNewAge(newAge){
      console.log(('年龄的新值：'+newAge))
      this.student.age=newAge
    }
  }
}
</script>
<template>
  {{student}}
  <hr>
  <StudentVue @change-age="getNewAge"/>
</template>
```

触发自定义组件事件：
+ 在选项式API中，可通过组件当前实例 `this.$emit(event,...args)` 来触发当前组件自定义的事件
+ 在组合式API中，可调用 `defineEmits` 宏返回的 `emit(event,...args)` 函数来触发当前组件自定义的事件
其中上方两个参数分别为：
    `event`：触发事件名，字符串类型
    `...args`：传递参数，可没有，可多个

*注意事项：
+ 模板上可以用$event变量
```vue
$event.target.checked
```
+ 组合式API中变量的值.value
```vue
shopCar.value
```

### 透传属性和事件
父组件在使用子组件的时候：
1. 透传属性和事件并没有在子组件中用 `props` 和 `emits` 声明
2. 透传属性和事件最常见的如 `@click` 和 `class` 、 `id` 、`style`
3. 当子组件只有一个元素时，透传属性和事件会自动添加到该根元素上；如果根元素已有 `class`或`style`属性，它会自动合并
 
组止透传属性和事件自动透传给唯一的根组件
在子组件中添加
1. 在选项式API中，可以在组件选项中设置 `inheritAttrs:false` 来组上

2. 在组合式API中，`<script setup>` 中，你需要一个额外的`<script>`块来写`inheritAttrs:false` 选项声明禁止
```vue
<script>
export default{
  inheritAttrs:false
}
</script>
```
多根元素的透传属性和事件
在子组件中元素添加 `v-bind="$attrs"`以说明接受透传属性和事件
```vue
<button v-bind="$attrs"></button>
```
访问透传属性和事件
1. 在选项式API中，可以用 `this.$attrs` 或 `$attrs`
```vue
this.$attrs
this.$attrs.onClick()
<template>
  //在模板中可以直接使用 $attrs
  <h6>{{ $attrs }}</h6>
  <h6>{{ $attrs.title }}</h6>
</template>
```
2. 在组合式API中，可以用 `useAttrs()` 来获取
```vue
<script setup>
  let attrs = useAttrs()
</script>
<template>
  //在模板中可以直接使用 attrs
  <h6>{{ attrs }}</h6>
  <h6>{{ attrs.title }}</h6>
</template>
```

### 插糟
在封装组件时，可以使用 `<slot>` 元素把不确定的、希望由用户指定的部分定义为插槽；插槽可以理解为给
预留的内容提供点位符，插槽也可以提供默认内容，如果组件的使用者没有为插槽提供任何内容，则插槽的
默认内容会生效。
```vue
//父组件中
<template>
  <CardVue>
    <button>关闭</button>
  </CardVue>
</template>

//子组件中
<template>
  <slot>卡片功能区域</slot>
</template>
```
插槽的默认内容只有父组件没有提供内容时才会显示

### 具名插槽
```vue
//父组件
<template v-solt:test>
  指定name
</template>
<template #test>
  #号
</template>
<template #default>
  默认
</template>
//子组件
<template>
  <!-- 具名插槽 -->
  <solt name="test"></solt>
  <!-- 默认插槽 -->
  <solt>卡片功能区域</solt>
</template>
```

### 作用域插槽
带有数据的插槽称为`作用域插槽`
```vue
//子组件中
<script>
let blog = reactive({
  title:'Java实现上传',
  time:'2020-10-10 15:33:33',
})
let author = ref('爱思考的飞飞')
</script>
<slot name="cardContent" :cardBlog="blog" :cardAuthor="author"></slot>

//父组件中
<template #cardContent="dataProps">
  <li>{{ dataProps }}</li>
  <li>博客的标题：{{  dataProps.cardBlog.title }}</li>
  <li>博客的时间：{{  dataProps.cardBlog.time }}</li>
  <li>博客的作者：{{  dataProps.cardAuthor }}</li>
</template>
```
默认插槽使用属性值
```vue
//如果使用子组件时用到了 `v-slot` ，则该子组件标签中将无法向其他具名插槽中提供内容
<CardVue v-slot="dataProps">
  //错误
  <button #XXX="XXX"></button>
  <button>{{ dataProps.close }}</button>
  <button>{{ dataProps.sure }}</button>
</CardVue>
```
### scoped 属性
**默认情况下，写在`.vue`组件中的样式会全局生效，很容易造成多个组件之间的样式冲突问题导致
组件之间样式冲突的根本原因是：
1. 单页面应用程序中，所有组件的`DOM`结构，都是基于唯一的`index.html`页面进行呈现的
2. 每个组件中的样式，都会影响整个`index.html`页面中的DOM元素

scoped 属性
让下方的样式只作用在该组件中，或者子组件的根元素上
该组件中的所有元素及子组件中的根元素会加上固定的属性(data-v-1a2j3i6)
该css选择器都自动添加固定的属性选择器[data-v-1a2j3i6]
```vue
<style>
//可以作用在当前页面元素上
h3{
  border:1px solid red;
}
//可以作用在子组件的根元素上
.error{
  border:1px solid black;;
}
</style>
```
### :deep()深度选择器
```vue
.error.deep(button){

}
```
### css中的v-bind()
```vue
<script setup>
let btnTheme = reactive({
  backColor:'#000000',
  textColor:'#FFFFFF',
})

</script>
<style>
  button{
    //v-bind() 数据源值变化，样式也变化
    background-color:v-bind('btnTheme.backColor');
    color:v-bind('btnTheme.textColor');
  }
</style>
```

## 注入inject
main.js中引入提供数据
```vue
app.provice('message','登录成功')
```
在子组件中
```vue
<script>
export default{
  inject:['message','title','subtitle','changeSubtitle']
}
</script>
<template>
  <h2>应用层提供的数据 {{ message }}</h2>
</template>
```

**选项式API中，访问组件实例`this`，`provide`必须采用函数的方式（不能用箭头函数）
```vue
<script>
export default{
  data:()=>({
    title:'博客',
    subtitle:'百万博主分享经验'
  }),
  methods:{
    changeSubtitle(sub){
      this.subtitle = sub
    }
  }
  //provide:{title:this.title} //无法访问组件实例 this
  //如果想访问组件实例 this，必须采用函数的方式
  provicde(){
    return {
      title:this.title, //这种并注入方 与 提供方 没有响应式连接
      subtitle:computed(()=>this.subtitle),
      changeSubtitle:this.changeSubtitle   
    }
  }
}
</script>
```
+ title:this.title, //这种并注入方 与 提供方 没有响应式连接
+ subtitle:computed(()=>this.subtitle)  //响应式连接借助组合式API中的 `computed()` 函数提供计算属性

## 生命周期

### beforeCreate
1. `beforeCreate` 选项式声明周期函数,可以访问`props`的数据
2. 在组件实例初始化之前调用（`props` 解析已解析、`data` 和`computed`等选项还未处理）
3. 不能访问组件的实例 `this` 中的`数据源`和`函数`等
4. 不能访问组件中的视图`DOM`元素
5. 组合式API中的setup()钩子会在所有选项式API钩子之前调用
```vue
beforeCreate(){
    console.log('beforeCreate 组件实例话之前')
    console.log(this.$props.subtitle)
  }
```
### created
组件实例初始化之后
```vue
created(){
    console.log('created 组件实例话之后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
  }
```
### beforeMount（onBeforeMount）
组件`视图渲染`之前
```vue
beforeMount(){
    console.log('beforeMount 组件视图渲染之前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
  },
```
组合式API
```vue
onBeforeMount(()=>{
    console.log('beforeMount 组件视图渲染之前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
})
```
### mounted
组件`视图渲染`之后
```vue
mounted(){
    console.log('mounted 组件视图渲染之后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  }
```
组合式API：
```vue
onMounted(()=>{
    console.log('mounted 组件视图渲染之后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
})
```
### beforeUpdate
`数据源`发生改变，视图重新渲染`前`
```vue
beforeUpdate(){
    console.log('beforeUpdate 数据源发生改变，视图重新渲染前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  }
```
在组合式API：
```vue
onBeforeUpdate(()=>{
    console.log('beforeUpdate 数据源发生改变，视图重新渲染前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
})
```
### update
`数据源`发生改变，视图重新渲染`后`
```vue
updated(){
    console.log('updated 数据源发生改变，视图重新渲染后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  }
```
在组合式API：
```vue
onUpdated(()=>{
    console.log('updated 数据源发生改变，视图重新渲染后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
})
```
### beforeUnmount
组件在卸载之前
```vue
beforeUnmount(){
    console.log('beforeUnmount 组件在卸载之前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
}
```
在组合式API:
```vue
onBeforeUnmount(()=>{
    console.log('beforeUnmount 组件在卸载之前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
})
```
### unmounted
组件在卸载之后
```vue
unmounted(){
    console.log('unmounted 组件已卸载')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    // console.log(document.getElementById('title').innerHTML)
}
```
在组合式API：
```vue
onUnmounted(()=>{
    console.log('unmounted 组件已卸载')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    // console.log(document.getElementById('title').innerHTML)
})
```
完整如下：
```vue
<script>
export default {
  props:['subtitle'],
  data:()=>({
    age:30
  }),
  methods:{
    showMessage(){
      console.log('函数 Hello')
    }
  },
  beforeCreate(){
    console.log('beforeCreate 组件实例话之前')
    console.log(this.$props.subtitle)
  },
  created(){
    console.log('created 组件实例话之后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
  },
  beforeMount(){
    console.log('beforeMount 组件视图渲染之前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
  },
  mounted(){
    console.log('mounted 组件视图渲染之后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  },
  beforeUpdate(){
    console.log('beforeUpdate 数据源发生改变，视图重新渲染前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  },
  updated(){
    console.log('updated 数据源发生改变，视图重新渲染后')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  },
  beforeUnmount(){
    console.log('beforeUnmount 组件在卸载之前')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    console.log(document.getElementById('title').innerHTML)
  },
  unmounted(){
    console.log('unmounted 组件已卸载')
    console.log(this.$props.subtitle)
    console.log(this.age)
    this.showMessage()
    // console.log(document.getElementById('title').innerHTML)
  },
}
</script>
```

## 访问DOM元素模板引用
如果想访问组件中的底层 DOM 元素，可使用 vue 提供特殊的 ref 属性来访问
### 选项式API中ref
1. 字符声明的：
ref="account" 字符串，获取 this.$refs.account
```vue
this.$refs.account
```
2. 函数声明的：
:ref="passwordRef" ，获取 passwordRef(el){}
```vue
 //注意函数式声明的ref，不会在 this.$refs 中获取
passwordRef(el){
  this.passwordEl = el

}
```
*注意： 函数式声明的ref，不会在 this.$refs 中获取

### 组合API中ref
字符声明的：
```vue
<script settup>
let account = ref(null) //ref变量名一定要和输入框中的ref属性值一样
function changeAccount(){
  console.log(account.value)
  account.value.style='padding:10px'
  account.value.className='rounded'
  account.value.focus()
}
</script>
<template>
  <input type="text" ref="account"><button @click="changeAccount">改变账号</button>
</template>
```
函数声明的：
```vue
<script settup>
let password = ref(null) //ref变量名一定要和输入框中的ref属性值一样
function passwordRef(el){
  password.value=el
}
function changePassword(){
  console.log(password.value)
  password.value.style='padding:10px'
  password.value.className='rounded'
  password.value.focus()
}
</script>
<template>
  <input type="text" :ref="passwordRef"><button @click="changePassword">改变p密码</button>
</template>
```

### v-for 中的模板引用
前提条件：
注意：vue的版本在`3.2.25`及以上版本
vue版本查看：在项目根目录中的 `package.json`
```vue
"dependencies": {
    "vue": "^3.3.4"
}
```
引用方法：
1. 如果 ref 值是字符串形式，在元素被渲染后包含对应整个列表的所有元素【数组】
2. 如果 ref 值是函数形式，则会每渲染一个列表元素则会执行对应的函数【不推荐使用，影响性能】
```vue
<script>
import { url } from "inspector";

export default {
  data: () => ({
    books: [
      { id: 1, name: "红楼梦" },
      { id: 2, name: "三国演义" },
      { id: 3, name: "水浒传" },
      { id: 4, name: "西游记" },
    ],
  }),
  methods: {
    changeBookListStyle() {
      console.log(this.$refs.booklist);
      this.$refs.booklist[2].style = "color:red";
    },
  },
};
</script>
<template>
  <ul>
    <li v-for="b in books" :key="b.id" ref="booklist">
      {{ b.name }}
    </li>
  </ul>
  <button @click="">点击查看 booklist</button>
</template>
```
组合式API：
```vue
<script setup>
import { VueElement } from 'vue';
import {ref} from VueElement
import { Script } from 'vm';

let students = ref([
  {id:1,name:'Jack'},
  {id:2,name:'Annie'},
  {id:3,name:'Tom'},
])
function changeStyle(){
  console.log(students.value)
  students.value[2].style='color:red'
}
</script>
<template>
  <ul>
    <li v-for="s in students" :key="s.id" ref="students">
      {{ s.name }}
    </li>
  </ul>
  <button @click="changeStyle">点击查看 students</button>
</template>
```
### 组件上的ref
模板引用也可以被用在一个子组件上；这种情况下引用中获得的值是组件实例
1. 如果子组件使用的是 选项式API，默认情况下父组件可以随意访问子组件的数据和函数， 除非在子组件使用 `expose` 选项来暴露特定的数据或函数
例子：
---- 子组件中：
```vue

<script>
//选项式API，默认情况父组件可以随意访问该子组件的数据和函数
export default {
  data: () => ({
    account: "abc123",
    password: "123321",
  }),
  methods: {
    toLogin() {
      console.log("登录中...");
    },
  },
  //只暴露指定的数据、函数等
  expose: ["account", "toLogin"],
};
</script>

<template>
  账号：<input type="text" v-model="account" />

  密码：<input type="password" v-model="password" />
  <button @click="toLogin">登录</button>
</template>
<style></style>
```

---- 父组件中：
```vue

<script>
import LoginVue from "./components/Login.vue";
export default {
  components: { LoginVue },
  data: () => ({
    login_vue: null,
  }),
  methods: {
    showSonData() {
      console.log("页面渲染后的方法");
      console.log(this.login_vue.account);
      console.log(this.login_vue.password);
    },
  },
  mounted() {
    this.login_vue = this.$refs.loginView;
  },
};
</script>
<template>
  <h3>登陆界面</h3>
  <hr />
  <!-- 组件上的ref值为该组件的实例 -->
  <LoginVue ref="loginView" />
  <hr />
  <button @click="showSonData">查看子组件中的信息</button>
</template>
```
2. 如果子组件使用的是组合式API`<script setup>` ，那么该子组件默认是私有的，则父组件无法访问该子组件，除非子组件在其中通过`defineExpose`宏显式暴露特定的数据或函数
组合式API的例子：
---- 子组件
```vue

<script setup>
import { ref } from "vue";

let account = ref("abc123");
let password = ref("123321");

function toLogin() {
  console.log("登录中...");
}

defineExpose({
  account,
  toLogin,
});
</script>

<template>
  账号：<input type="text" v-model="account" />

  密码：<input type="password" v-model="password" />
  <button @click="toLogin">登录</button>
</template>
```
---- 父组件中
```vue

<script setup>
import { ref } from "vue";
import LoginVue from "./components/Login.vue";

let loginView = ref(null);

function showSonData() {
  console.log("页面渲染后的方法");
  console.log(loginView.value.account);
  console.log(loginView.value.password);
}
</script>
<template>
  <h3>登陆界面</h3>
  <hr />
  <!-- 组件上的ref值为该组件的实例 -->
  <LoginVue ref="loginView" />
  <hr />
  <button @click="showSonData">查看子组件中的信息</button>
</template>
```

## 路由 Vue Rounter
`vue-router` 是 `vue.js` 官方给出的路由解决方案，能名轻松的管理 `SPA` 项目中组件的切换
安装： 
```vue
npm install vue-router@4
```
+ src下面创建views目录，用于放置视图页面
//src/views/HomeViews.vue 内容
```vue

<template>
  <div class="home">网站首页界面</div>
</template>
<style>
div.home {
  background-color: cyan;
}
</style>
```

//src/views/BlogHomeView.vue 内容
```vue
<template>
  <div class="blog">网站首页界面</div>
</template>
<style>
div.blog {
  background-color: yellow;
}
</style>
```
### 创建路由模块
1. 在项目中的src文件夹中创建一个`router` 文件夹，在其中创建 `index.js` 模块
2. 采用 `createRouter()` 创建路由，并暴露出去
3. 在`main.js`文件中初始化路由模块 `app.use(routerin)`

router/index.js内容如下：
```vue
import { createRouter } from "vue-router";

//创建路由对象
const router=createRouter({})

//将路由对象暴露出去
export default router
```

main.js内容如下：
```vue
import './assets/main.css'

import { createApp } from 'vue'
import App from './App.vue'
//引入路由模块(如果文件名字为index. 则默认只需要写在目录地址即可)
import router from './router'

const app=createApp(App)

//初始化路由模块
app.use(router)

app.mount('#app')
```

### 规定路由模式
history路由模式可采用：
1. `createWebHashHistory()` ：Hash模式
  a. 它在内部传递的实际URL之前使用了一个哈希字符 `#` ,如 https://example.com/#/user/id
  b. 由于这部分URL从未被发送到服务器，所以它不需要在服务器层面上进行任何特殊处理
2. `createWebHistory()`：html5模式（推荐）
   a. 当使用这种历史模式时,URL会看起来很'正常'，如 https://example.com/user/id
   b. 由于我们的应用是一个单面的客户端应用，如果没有适当的服务器配置，用户浏览https://example.com/user/id就会404；
解决这个问题：在服务器上添加 一个简单的回退路由 `try_files $uri $uri/ /index.html;（二级目录用：try_files $uri $uri/ /report/index.html;）`，如果URL不匹配任何资源，应提供与你的应用程序中的index.html相同的页面

### 使用路由规则
routes 配置路由规则
+ path :路由分配的 URL
+ name :当路由指向此页面时显示的名字
+ component ：路由调用这个页面时加载的组件

router/index.js中：
```vue
import { createRouter, createWebHistory } from "vue-router";
import BlogHomeView from '@/views/BlogHomeView.vue'
//路由规则
const routes=[
{
path:'/home',//路由地址
name:'home',//路由名称
component:()=>import('@/views/HomeView.vue') //切换路由地址时展示的组件

},
{
path:'/blog',//路由地址
name:'blog',//路由名称
component:BlogHomeView

}
]

//创建路由对象
const router=createRouter({
history:createWebHistory(), //采用html5路由模式
routes
})

//将路由对象暴露出去
export default router
```

App.vue 中：
```vue
<script setup>
import { ref } from "vue";
import router from "./router";
</script>
<template>
  <router-link to="/home">首页</router-link>
  |
  <router-link to="/blog">博客</router-link>
  <hr />
  <router-view />
</template>
```
#### 重定向
可采用 `redirect` 重定向
```vue
const routes=[
    {
        path:'/',
        redirect:'/home'//重定向地址
    },
】
```
#### 嵌套路由(二级目录)
三点：
+ 父级页面添加 `<router-view />`
+ 父级页面连接地址 `/school/math`
+ 路由配置children=>path、name、component
例子如下：
  SchoolHomeView.vue 中内容：
```vue
<template>
  <div class="school">
    学堂首页界面

    <router-link to="/school/english">英文</router-link>
    |
    <router-link to="/school/math">数学</router-link>

    <router-view />
  </div>
</template>
<style>
div.school {
  padding: 20px;
  background-color: orange;
}
</style>

```
路由配置文件 router/index.js地址的内容：
```vue
import { createRouter, createWebHistory } from "vue-router";
import BlogHomeView from '@/views/BlogHomeView.vue'
import SchoolHomeView from '@/views/SchoolHomeView.vue'
import EnglishView from '@/views/school/EnglishView.vue'
import MathView from '@/views/school/MathView.vue'
//路由规则
const routes=[
    {
        path:'/school',//路由地址
        name:'school',//路由名称
        component:SchoolHomeView,
        //嵌套路由，下面要展示的组件需要在父级路由的组件中(router-view)进行展示
        children:[
            {
                path:'english',//注意：嵌套路由中的path不能以“/”开头
                name:'school-english',
                component:EnglishView
            },
            {
                path:'math',//注意：嵌套路由中的path不能以“/”开头
                name:'school-math',
                component:MathView
            },
        ]
    }
]

//创建路由对象
const router=createRouter({
    history:createWebHistory(), //采用html5路由模式
    routes
})

//将路由对象暴露出去
export default router
```

#### 获取url路由的参数
路由index.js中，路由加 `:id：
```vue
{
    path:'/blog-content/:id',//获取参数
    name:'blog-content',
    component:BlogContent
}
```
1. 在选项式API JS中采用 `this.$route.params` 来访问，试图模板上采用 `$route.params`来访问
```vue
<script>
console.log(this.$route.params)
console.log(this.$route.params.id)
</script>
<template>
  {{ $route.params }}
</template>
```
2. 在组合式API中，需要 `import { useRoute } from 'vue-router'` ，JS和视图模板上通过userRoute().params 来访问
```vue
<script setup>
import { useRoute } from "vue-router";
const routObj = useRoute();
const paramsData = defineProps(["id"]);
function getParams() {
  console.log(routObj.params);
  console.log(routObj.params.id);
  console.log(paramsData.id);
}
</script>
<template>
  <div class="home">博客的详情</div>
  <ul>
    <li>{{ routObj.params }}</li>
    <li>{{ routObj.params.id }}</li>
    <li>{{ id }}</li>
  </ul>
  <button @click="getParams">获取js中的内容</button>
</template>
<style>
div.home {
  padding: 20px;
  background-color: cyan;
}
</style>
```
3. 还可以通过在路由规则上添加 `props:true`，将路由参数传递给组件的 `props` 中
```vue
//props:true,
{
        path:'/blog-content/:id',//路由地址
        name:'blog-content',//路由名称
        props:true, //将路由参数动态传给组件的 `props`选项
        component:BlogContent 
    }
//在模板中直接可以用id
{{ id }}
```

#### 声明式和编程式导航
+ 声明式：<router-link>
```vue
<router-link to="/school/math">数学</router-link>
```
+ 编程式：`push`
1. 选项式API：this.router.push(...) 或者模板中 $router.push(...)
```vue
<template>
  <button @click="$router.push('/school/math')"></button>
  <button @click="$router.push({ path:'/blog' })")></button>
  <button @click="$router.push({ name:'school'})")></button>
  <button @click="$router.push({ name:'school' , params:{ id:119 } })"></button>
</template>

```
2. 组合式API：
```vue
<script setup>
import { useRouter } from "vue-router";
const routerObj = useRouter();
</script>
<template>
  <button @click="routerObj.push('/school')">学堂</button>
</template>
```

#### 声明式和编程式 替换
+ 声明式：
返回上一页是当前页面
<router-link replace>
```vue
<router-link to="/school/math" replace>数学</router-link>
```
+ 编程式：`replace`
1. 选项式API：this.router.push(...) 或者模板中 $router.push(...)
```vue
<template>
  <button @click="$router.replace('/school/math')"></button>
  <button @click="$router.replace({ path:'/blog' }")></button>
  <button @click="$router.replace({ name:'school'}")></button>
  <button @click="$router.replace({ name:'school' , params:{ id:119} })"></button>
  //replace:true
  <button @click="$router.push({ name:'school' , params:{ id:119 }, replace:true})"></button>
</template>
```
2. 组合式API：
```vue
<script setup>
import { useRouter } from "vue-router";
const routerObj = useRouter();
</script>
<template>
  <button @click="routerObj.replace('/school')">学堂</button>
</template>
```
#### 路由历史
参数 `n` 表示前进或后退步数
n : 
router.go(1) 前进一步  ，相当于 `router.forward()`
router.go(-1) 后退一步  ，相当于 `router.back()`
如果没有历史记录，什么都不发生
1. 选项式API
```vue
this.$router.go(n)
```
2. 组合式API
```vue
useRouter().go(n)
```
#### 导航守卫
种类：
1. 全局`前置`守卫
2. 全局`解析`守卫
3. 全局`后置`守卫
4. `路由独享`守卫
5. `组件内的`守卫

##### 全局前置守卫
注册全局 前置守卫
```vue
<script>
//创建路由对象
const router=createRouter({
  history:createWebHistory(), //采用html5路由模式
  routes
})
//..........

//注册全局 前置守卫
//to :将要访问的路由信息对象
//from :将要离开的路由信息对象
router.beforeEach((to,from,next)=>{
  //判断要访问的路由是否需要用户登录
  if(to.meta.isLogin){
    //获取存储对象
    let userLogin=localStorage.getItem('loginUser')

    //判断用户是否已经登录了
    if(userLogin==null){
      //未登录 --> 跳转至登录页
      return next({path:'/login'})
    }
  }

  return next()
})

//将路由对象暴露出去
export default router
//..........
</script>
```
存储对象到本地
```vue
<script>
localStorage.setItem('loginUser',JSON.stringify(user))
localStorage.getItem('loginUser')
localStorage.removeItem('loginUser')
</script>
```
```vue
<script>
localStorage.setItem('user',JSON.stringify(user))
</script>
```

是不是登录判断跳转>>>
src/views/LoginView.vue 内容如下：
```vue
<script setup>
import { reactive } from "vue";

const user = reactive({
  account: "",
  password: "",
});
function toLogin() {
  localStorage.setItem("loginUser", JSON.stringify(user));
  alert("登录成功");
}
function loginOut() {
  localStorage.removeItem("loginUser");
  alert("注销成功");
}
</script>
<template>
  <div class="login">登录界面</div>
  <hr />
  账号：<input type="text" v-model="user.account" /><br />

  密码:<input type="text" v-model="user.password" /><br />

  <button @click="toLogin">登录</button>
  <button @click="loginOut">注销</button>
</template>
<style>
div.login {
  padding: 20px;
  background-color: rgb(138, 112, 242);
}
</style>

```

src/router/index.js 内容如下：
```vue
<script>
import { createRouter, createWebHistory } from "vue-router";
import BlogHomeView from '@/views/BlogHomeView.vue'
import SchoolHomeView from '@/views/SchoolHomeView.vue'
import EnglishView from '@/views/school/EnglishView.vue'
import MathView from '@/views/school/MathView.vue'
import BlogContent from '@/views/BlogContentView.vue'
import LoginView from '@/views/LoginView.vue'

//路由规则
const routes=[
  {
    path:'/',
    redirect:'/home'//重定向地址
  },
  {
    path:'/home',//路由地址
    name:'home',//路由名称
    component:()=>import('@/views/HomeView.vue') //切换路由地址时展示的组件

  },
  {
    path:'/blog',//路由地址
    name:'blog',//路由名称
    meta:{isLogin:true},
    component:BlogHomeView

  },
  {
    path:'/school',//路由地址
    name:'school',//路由名称
    meta:{isLogin:true},
    component:SchoolHomeView,
    //嵌套路由，下面要展示的组件需要在父级路由的组件中(router-view)进行展示
    children:[
      {
        path:'english',//注意：嵌套路由中的path不能以“/”开头
        name:'school-english',
        component:EnglishView
      },
      {
        path:'math',//注意：嵌套路由中的path不能以“/”开头
        name:'school-math',
        component:MathView
      },
    ]
  },
  {
    path:'/blog-content/:id',//路由地址
    name:'blog-content',//路由名称
    props:true,
    meta:{isLogin:true},
    component:BlogContent
  },
  {
    path:'/login',//路由地址
    name:'login',//路由名称
    component:LoginView

  },
]

//创建路由对象
const router=createRouter({
  history:createWebHistory(), //采用html5路由模式
  routes
})

//注册全局 前置守卫
//to :将要访问的路由信息对象
//from :将要离开的路由信息对象
router.beforeEach((to,from,next)=>{
  console.log(to)
  console.log(to.path)
  //判断要访问的路由是否需要用户登录
  if(to.meta.isLogin){
    //获取存储对象
    let userLogin=localStorage.getItem('loginUser')

    //判断用户是否已经登录了
    if(userLogin==null){
      //未登录 --> 跳转至登录页
      return next({path:'/login'})
    }
  }

  return next()
})

//将路由对象暴露出去
export default router
</script>
```

## 状态管理库Pinia
### 安装与使用
安装：
```vue
npm install pinia
```
使用：
在`src/main.js`中：
```vue
import {createPinia} from 'pinia'
//创建pinia(根存储)，并且应用到整个应用中
app.use(createPinia())
```
### Store
1. `store` 是一个保存状态和业务逻辑的实体，它并不与组件树绑定，它承载着全局状态；它永远存在，每个组件都可以读取和写入它
2. `store` 有三个概念，`state`、`getters` 和 `actions` ，我们可以理解成组件中的 `data`、`computed` 和 `methods`
3. 在项目中的 `src\store` 文件夹下不同的 `store.js` 文件

语法：
defineStore(name,function|options) 定义的，建议其函数返回的值命名为 use...Store 方便理解
选项式API写法：
```vue
import { defineStore } from 'pinia'

//创建一个store，并暴露出去
//参数 1：名称，保证唯一
//参数 2：对象形式（选项式）
export const useStor=defineStore('main',{
    state:()=>({}), //共享的数据
    getters:{}, //通过计算得到的共享数据
    actions:{} //共享的函数
})
```
组件式API写法：
```vue
export const useStore=defineStore('main',()=>{
    //ref 变量 --> 共享的数据
    //computed() -->通过计算得到的共享数据
    //function() --> 共享的函数
    
    return {
        //返回组件需要的：变量、计算属性、函数
    }
})
```

#### state
语法：
`mapState( storeObj , array | object )`
获取state中的值：`mapState`

<font color="cyan">**思路整理:**</font>
1. 步骤一：在xxxStore.js中设置一些共享数据 export const useUserStore = defineStore('user',{xxx})
2. 步骤二：vue页面引入 `useUserStore` , 动态获取 const u = useUserStore();
3. 步骤三：使用 `computed` 获取对象中的变量值

```vue
//--步骤一--
export const useUserStore = defineStore('user',{
    state:()=>({
        age:10,
        level:5,
        nickname:'wjb',
        account:'SCD273'
    }),
    //通过计算得到的新的共享的数据，只读
    //如果依赖的数据发生变化，则会重新计算
    getters:{
        month(){
            return this.birthday.split('-')

        },
        ageStage:(state)=>{
            
        }
    }
})

//--步骤二--
<script>
import { mapState, mapWritableState } from "pinia";
import { useUserStore } from "./store/useUserStore";
import { computed } from "vue";
const u = useUserStore();
const u_age = computed(() => {
  u.age;
});
const u_level = computed(() => {
  u.level;
});
const u_nickname = computed(() => {
  u.nickname;
});

const { age, level } = storeToRefs(u);
</script>
```
demo例子如下：
src/store/useUserStore.js内容：
```vue
import {defineStore} from 'pinia'

export const useUserStore = defineStore('user',{
    state:()=>({
        age:10,
        level:5,
        nickname:'wjb',
        account:'SCD273'
    })
})
```
src/App.vue内容如下：
```vue
<script>
import { mapState } from "pinia";
import { useUserStore } from "./store/useUserStore";
export default {
  computed: {
    ...mapState(useUserStore, ["age", "level"]),
    ...mapState(useUserStore, {
      user_account: "account",
      user_nickname: "nickname",
    }),
  },
};
</script>
<template>
  <ul>
    <li>{{ this.age }}</li>
    <li>{{ level }}</li>
    <li>{{ user_account }}</li>
    <li>{{ user_nickname }}</li>
  </ul>
</template>
```
修改state 映射的值`
`mapWritableState`
`...mapWritableState(useUserStore, ["account", "nickname"]),`
```vueexport default {
  computed: {
    //mapState：将store的state映射成当前组件的计算属性
    //具有响应式，但是是只读
    //字符串数组形式，不能自定义计算属性名
    //对象形式：可以自定义计算属性名
    ...mapState(useUserStore, ["age", "level"]),
    ...mapState(useUserStore, {
      user_account: "account",
      user_nickname: "nickname",
    }),
    ...mapWritableState(useUserStore, ["account", "nickname"]),
  },
};

```

从 store 解构想要的 state:
`storeToRefs` : 既有自己的属性，又是响应式
```vue
const { age, level,account:userAccount } = storeToRefs(u);
```

#### getters
<font color="cyan">**思路整理:**</font>
1. 步骤一：在xxxStore.js中设置一些共享计算的数据 
```vue
getters:{
        month(){
            return this.birthday.split('-')

        },
        ageStage:(state)=>{
            
        }
    }
```
2. 步骤二：vue页面引入 `useUserStore` , 动态获取
```vue
export default {
  computed: {
    //mapState：将store的state映射成当前组件的计算属性
    //具有响应式，但是是只读
    //字符串数组形式，不能自定义计算属性名
    //对象形式：可以自定义计算属性名
    ...mapState(useUserStore, ["age", "level"]),
    ...mapState(useUserStore, {
      user_account: "account",
      user_nickname: "nickname",
    }),
    ...mapWritableState(useUserStore, ["account", "nickname"]),
  },
};
```
3. 步骤三：使用 `computed` 获取对象中的变量值

js中内容：
```vue
<script>
//通过计算得到的新的共享的数据，只读
//如果依赖的数据发生变化，则会重新计算
getters:{
  month(){
    return this.birthday.split('-')

  },
  ageStage:(state)=>{

  }
}
</script>
```
vue中内容：
从 store 取getters 和 取 state 用法相同，都可以使用 mapState
具有响应式
```vue
..mapState(useUserStore, ["month", "ageStage"]),
```

#### actions
<font color="cyan">**思路整理:**</font>
1. 定义共享数据
2. 设置actions修改共享数据
```vue
export const useUserStore = defineStore('user',{
        state:()=>({
            age:10,
            nickname:'wjb',
        }),
        actions:{
            setUserStore(nickname,age){
                //当前state中的数据
                this.nickname=nickname
                this.age=age
            }
        }
})
```

+ 选项式API中访问js中的共享数据actions
1. 使用 mapActions 将 store 中的 actions 映射为自己的函数
2. 字符串数组模式，不可以自定义函数名
3. 对象模式，可以自定义函数名
```vue
methods:{
  //使用 mapActions 将 store 中的 actions 映射为自己的函数
  //字符串数组模式，不可以自定义函数名
  //对象模式，可以自定义函数名
  ...mapActions(useUserStore,['setUserInfo'],
  ...mapActions(useUserStore,{
    set_user_object:'setUserInfoByObject'  
  })
}
```

+ 组合式API中访问js中的共享数据actions
  将 store 中的 actions 解构成自己的函数 可自定义函数(不得使用storeToRefs)
```vue
//store 实例
const user_store = useUserStore();

const { nickname, age } = storeToRefs(user_store);

//将 store 中的 actions 解构成自己的函数 可自定义函数(不得使用storeToRefs)
const { setUserStore } = useUserStore;
```

## 请求库Axios
+ Axios 是一个基于 `promise` 网络请求库，作用于 `node.js` 和 `浏览器` 中
+ Axios 在服务端它使用原生 `node.js` `http` 模块，而在客户端(浏览端) 则使用 `XMLHttpRequests`
+ Axios 可以拦截请求和响应、转换请求和响应数据、取消请求、自动转换 JSON 数据
+ Axios 安装方式： `npm install axios`

### Axios配置项
这些是创建请求时最常用的配置选项； 详细的配置项请前往 Axios 官网
```vue
{
    url: "/user", //请求的服务器地址URL
    method: "GET", //请求方式，默认值GET
    baseURL: "https://some-domain.com/api", //如果url不是绝对地址，则会发送
    headers: { "X-Requested-With": "XMLHttpRequest" }, //自定义请求头
    params: { ID: 12345 }, //与请求一起发送的URL参数
    data: { firstName: "Fred" }, //作为请求体被发送的数据，仅适用'PUT','POST','DELETE'和'PATCH'请求方法
    timeout: 1000, //请求超时的毫秒数，默认为0（永不超时）
    responseType: "json", //期望服务器返回的数据类型，选项包括：'arraybuffer','document','json','text','stream',浏览器专属：'blob',默认'json'
    //允许在向服务器发送前，修改请求数据，它只能用于'PUT','POST','PATCH'这几个请求方法
    transformRequest: [
    function (data, headers) {
    return data; //对发送的 data 进行任意转换处理
    },
    ],
    //在传递给 then/catch 前，允许修改响应数据
    transformResponse: [
    function (data) {
    return data; //对接收的data进行任意转换处理
    },
    ],
}
```
### 跨域问题
同源策略：是指协议，域名，端口都要相同，其中有一个不同都会产生跨域
vite.config.js中配置
```vue
export default defineConfig({
  plugins: [
    vue(),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
  server:{
    proxy:{
      '/apisace':{
        target:'https://eolink.o.apispace.com/history-weather/query',
        changeOrigin:true,
        rewrite:path=>path.replace(/^\/apispace/,'')
      }
    }
  }
})
```
### 使用方法
```vue
<script setup>
import { axios } from "axios";
function getCityWeather() {
  axios({
    method: "GET",
    url: "/apisace",
  })
    .then((response) => {
      console.log();
    })
    .catch((error) => {
      alert("服务器异常");
    });
}
</script>
```
