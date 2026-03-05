---
author: "jarvan"
title: "vue3实战项目"
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
>PS: 查看所有npm全局变量
> <font color="lightgreen">npm list -global</font>

```vue
npm cache clean --force //清除npm缓存 
npm init vue@latest 或 npm create vue@latest
✔ Project name: … vue3-project
```
package.json中修改：
```vue
"dev": "vite --host 172.19.0.13"
```
总结关键点：
1. `vite.config.js` - 项目的配置文件 基于vite的配置
2. `package.json` - 项目包文件 核心依赖变成了 vue3.x 和 vite 
3. `main.js` - 入口文件 createApp 函数创建应用实例
4. `app.vue` - 根组件 SFC单文件组件 script-template-style
+ 变化一：脚本script和模板template顺序调整
+ 变化二：模板template不再要求唯一根元素
+ 变化三：脚本script添加setup标识支持组合式API
5. `index.html` - 单页入口 提供id为app的挂载点

> <font color="orange">报错信息</font>
> npm install 时报错：code ERESOLVE
> <font color="lightgreen">解决方法：</font>
> 清除npm缓存 <font color="lightgreen">npm cache clean --force</font>

## reactive 和 ref
生成响应式对象
reactive() //只能传入对象返回对象
ref()//传入字符或对象
总结关键点：
1. reactive和ref函数的共同作用是什么？
答：生成响应对象数据
2. reactive vs ref ？
答：
1.reactive 不能处理简单类型的数据 
2.ref参数类型支持更好但是必须通过.value访问修改 
3.ref函数的内部实现依赖于reactive函数
3. 在实际工作中推荐使用哪个？
答：推荐使用ref函数，更加灵活

## computed计算属性函数
和vue2的完全一致，只是修改了写法
核心步骤：
1. 导入computed函数
2. 执行函数在回调参数中return基于响应式数据做计算的值，用变量接收
```vue
<script setup>
//导入
import { VueElement } from 'vue';
import {computed} from VueElement
//执行函数 变量接收 在回调参数中return计算值
const computedState=computed(()=>{
  return '基于响应式数据做计算后的值'
})
</script>
```
### filter
过滤数组
list.value.filter((item) => item > 2);
```vue
const computedState = computed(() => {
return list.value.filter((item) => item > 2);
});
```
### setTimeout
setTimeout(()=>{},3000)
```vue
 setTimeout(() => {
    list.value.push(9, 10);
    }, 2000);
```

### watch函数
作用：侦听一个或多个数据的变化，数据变化时执行回调函数
2个参数：1. immediate (立即执行) 2.deep（深度侦听）
```vue
watch(count, (newValue, oldValue) => {
  console.log("count发生了变化", oldValue, newValue);
});
```
侦听多个数据
```vue
watch([count, name], ([newCount, newName], [oldCount, oldName]) => {
  console.log("count或者name变化", [newCount, newName], [oldCount, oldName]);
});
```

** watch 立即执行
immediate: true,
```vue
watch(
  count,
  () => {
    console.log("立即变化");
  },
  {
    immediate: true,
  }
);
```

** watch 深度侦听 deep侦听所有
```vue
watch(
  count,
  () => {
    console.log("深度侦听");
  },
  {
    deep: true,
  }
);
```
** 针对侦听
```vue
watch(
  () => state.value.age,
  () => {
    console.log("深度侦听");
  }
);
```
## 生命周期
基本使用
1. 导入生命周期函数
2. 执行生命周期函数 传入回调

PS:可多次顺序执行
```vue
<script setup>
import { onMounted } from "vue";
onMounted(() => {
  //自定义逻辑
});
</script>
```
## 父传子参数
基本思想：
1. 父组件中给`子组件绑定属性`
2. 子组件内部通过`props选项接收`

子组件中宏函数：defineProps()
```vue
defineProps({
  message:String
})
```
## 子传父参数
基本思想：
1. 父组件中给`子组件标签通过@绑定事件`
2. 子组件内部通过`$emit方法触发事件`

子组件中宏函数：defineEmits()
```vue
defineEmits(['get-message'])
```
总结关键点：
1. 父传子的过程中通过什么方式接收props？
defineProps({属性名：类型})
2. setup语法糖中如何使用父组件传过来的数据？
const props = defineProps({属性名：类型})
3. 子传父的过程中通过什么方式得到emit方法？
defineEmits(['事件名称'])

## 模板引用
通过 `ref标识` 获取真实的 `dom对象或者组件实例对象`
基本思想：
1. 调用ref函数生成一个ref对象(空对象ref(null)) `const h1Ref=ref(null)`
2. 通过ref标识绑定ref对象到标签 `<h1 ref=""h1Ref">我是dom标签h1</h1>`

PS:注意：组件挂载完成之后才能获取
```vue
onMounted(()=>{
  console.log(h1Ref.value)
})
```
### 暴露宏函数defineExpose()
默认在`<script setup>`语法糖下组件内部的属性和方法是不开放给父组件访问的，可以通过 defineExpose() 宏函数指定哪些属性和方法允许访问
```vue
const testMessage = ref('this is test msg')
defineExpose({
  testMessage
})
```
总结关键点：
1. 获取模板引用的时机是什么？
组件挂载完毕
2. defineExpose 宏函数的作用是什么？
显式暴露组件内部的属性和方法

## 组件通信
顶层组件向任意组件的底层组件传递数据和方法，实现跨层组件通信
核心思想：
1. 顶层组件通过 provide 函数提供数据
```vue
provide('key',顶层组件中的数据)
```
2. 底层组件通过 inject 函数获取数据
```vue
const message = inject('key')
```
响应式数据：
顶层组件中的数据  --> ref()变量

PS:跨层传递方法：和传递数据一样,`(原则誰的数据谁负责修改)`

总结关键点：
1. provide和inject的作用？
跨层组件通信
2. 如何在传递的过程中保持数据响应式？
第二个参数传递ref对象
3. 底层组件想要通知顶层组件做修改，如何做？
传递方法，底层组件调用方法
4. 一颗组件树中只有一个顶层或底层组件吗？
相对概念，存在多个顶层和底层的关系

## pinia
状态管理库
### pinia的引入
`main.js` 使用方法：
1. 导入 `createPinia`
```vue
import {createPinia} from 'pinia'
```
2. 执行方法得到`实例`
```vue
const pinia = createPinia()
```
3. 把`pinia`实例加入到`app应用中`
```vue
createApp(App).use(pinia).mount('#app')
```

### pinia的定义(state + action)
核心思想：
1. 引入 `defineStore`
```vue
import {defineStore} from 'pinia'
```
2. 定义 `defineStore别名实例`的`变量`和`方法`,以`对象形式返回`
   `export` : 表示 `导出` 的意思
```vue
export const useCounterStore = defineStore('counter',()=>{
  //数据(state)
  const count = ref(0)
  //修改数据的方法（action）
  const increment = ()=>{
    count.value++
  }

  //以对象形式返回
  return { count , increment }
})
```
### pinia的使用
核心思想：
1. 导入 `useCounterStore` 方法
```vue
import { useCounterStore } from '@/stores/counter'
```
2. 执行方法得到 `useCounterStore`对象
```vue
const useCounterStore = useCounterStore()
//... useCounterStore.increment
```


#### getters
pinia中的getters直接使用`computed`函数进行模拟
```vue
//数据（state）
const count = ref(0)
//getter
const doubleCount = computed(()=>count.value*2)
```
#### action 异步
核心思想：
1. 定义变量
2. 请求修改变量
3. 挂载变量
4. 使用变量
```vue
const list = ref([])
const getList = async ()=>{
  const res = await axios.get(API_URL)
  list.value = res.data.data.channels
}
```

#### storeToRefs
使用 `storeToRefs` 函数可以辅助保持数据 `(state+getter)` 的响应式解构
场景：想使用`解析赋值`
```vue
//响应式丢失 视图不更新
const { count , doubleCount } = counterStore
```
解决方法：
```vue
const { count , doubleCount } = storeToRefs(counterStore)
```

PS:注意：storeToRefs 只修饰数据，不能修饰方便，方法还要从之前变量中解构赋值
```vue
//数据
const { count , doubleCount } = storeToRefs(counterStore)
//方法直接从原来的 counterStore 中解构赋值
const { increment } = counterStore
```

## 创建项目
```vue
npm init vue@latest
✔ Project name: … vue-rabbit
✔ Add Vue Router for Single Page Application development? … No / `Yes`
✔ Add Pinia for state management? … No / `Yes`
✔ Add ESLint for code quality? … No / `Yes`
Done. Now run:

cd vue-rabbit
npm install
npm run dev
```
PS:docker环境中增加外网访问：
```vue
"scripts": {
   "dev": "vite --host 172.19.0.13",
   ...
}
```
### 添加几个文件夹
apis：API接口文件夹
composables:组合函数文件夹
directives:全局指令文件夹
styles:全局样式文件夹
utils:工具函数文件夹

### 配置联想提示
1. 在根目录下新增 jsconfig.json文件
2. 添加json格式的配置项目：如下：
```vue
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "@/*":[
                "src/*"
            ]
        }
    }
}
```
3. 之后输入"@/"就会有提示当前目录了

### 安装element Plus
+ 步骤一： npm install element-plus --save
```vue
# 选择一个你喜欢的包管理器

# NPM
$ npm install element-plus --save
```
+ 步骤二：安装 `unplugin-vue-components` 和 `unplugin-auto-import` 这两款插件
`-D` ： 代表只在开发环境安装
```vue
npm install -D unplugin-vue-components unplugin-auto-import
```

+ 步骤三：修改 `vite.config.js` 按需导入
```vue
//...
//elementPlus 按需导入
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
//...
plugins: [
   //...
   AutoImport({
   resolvers: [ElementPlusResolver()],
   }),
   Components({
   resolvers: [ElementPlusResolver()],
   }),
],
```

+ 步骤四：验证element-plus
```vue
<el-button type="primary">Primary</el-button>
```
### 主题定制
如何定制（scss变量替换方案）
1. 安装scss 
==> `npm i sass -D`
2. 准备定制样式文件 
==> `styles/element/index.scss`
```vue
//index.scss的内容如下：
@forward 'element-plus/theme-chalk/src/common/var.scss' with(
    $colors: (
        'primary':(
            //主色
            'base':#27ba9b,
        ),
        'success':(
            //成功色
            'base':#1dc779,
        ),
        'warning':(
            'base':#ffb302,
        ),
        'danger':(
            'base':#e26237,
        ),
        'error':(
            'base':#cf4444,
        )
    )
)
```
3. 对elementPlus样式进行覆盖 ==> 通知 element 采用scss语言 -> 导入定制scss文件覆盖
```vue
//vite.config.js的内容如下：
Components({
   //1.配置elementPlus采用sass样式配色系统
   resolvers: [
    ElementPlusResolver({importStyle:'sass'})
   ],
}
//...
//vite样式自动导入及覆盖
css:{
   preprocessorOptions:{
      scss:{
         //2.自动导入定制化样式文件进行样式覆盖
         additionalData:`
           @use "@/styles/element/index.scss" as *;
         `,
      }
   }
}
```

### axios基本配置
1. 安装axios
```vue
npm i axios
```
2. 配置基础实例（统一接口配置）
```vue
1、接口基地址
2、接口超时时间
3、请求拦截器
4、响应拦截器
```
1. 步骤一：添加http.js封装文件：
src/utils/http.js
```vue
//axios基础的封装
import axios from "axios";

const httpInstance = axios.create({
baseURL: 'http://pcapi-xiaotuxian-front-devtest.itheima.net',
timeout: 5000
})

//拦截器

//axios请求拦截器
httpInstance.interceptors.request.use(config => {
return config
}, e => Promise.rejecte)

//axios响应式拦截器
httpInstance.interceptors.response.use(res => res.data, e => {
return Promise.reject(e)
})

export default httpInstance
```
2. 步骤二：添加src/apis/testAPI.js文件
```vue
import httpInstance from "@/utils/http";

export function getCategory() {
    return httpInstance({
        url: 'home/category/head'
    })
}
```
3. 步骤三：App.vue中添加请求
```vue
import { getCategory } from "@/apis/testAPI";
getCategory().then((res) => {
  console.log(res);
});
```

### eslint使用
1. 步骤一：下载 eslint包
```vue
npm i -D eslint
```
2. 步骤二：初始化Eslint配置->生成.eslintrc.js文件
```vue
npx eslint --init
或者 'npm init @eslint/config'
//回车如下：
# npx eslint --init
You can also run this command directly using 'npm init @eslint/config'.
✔ How would you like to use ESLint? · problems
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · vue
✔ Does your project use TypeScript? · No / Yes
✔ Where does your code run? · browser, node
✔ What format do you want your config file to be in? · JavaScript
The config that you've selected requires the following dependencies:

@typescript-eslint/eslint-plugin@latest eslint-plugin-vue@latest @typescript-eslint/parser@latest
✔ Would you like to install them now? · No / Yes
✔ Which package manager do you want to use? · npm
Installing @typescript-eslint/eslint-plugin@latest, eslint-plugin-vue@latest, @typescript-eslint/parser@latest
```
3. 步骤三：若想配置 parser: 'babel-eslint' 需要安装依赖如下：
```vue
npm install --save-dev babel-eslint
```
babel-eslint 解析器是一种使用频率很高的解析器，现在很多公司的很多项目目前都使用了es6，为了兼容性考虑基本都使用babel插件对代码进行编译。
**PS: `rules` 一般是开发中关闭的一些报错规则 ，默认规则在 `extends` 中

#### rules配置：
```vue
"no-alert": 0,//禁止使用alert confirm prompt
"no-array-constructor": 2,//禁止使用数组构造器
"no-bitwise": 0,//禁止使用按位运算符
"no-caller": 1,//禁止使用arguments.caller或arguments.callee
"no-catch-shadow": 2,//禁止catch子句参数与外部作用域变量同名
"no-class-assign": 2,//禁止给类赋值
"no-cond-assign": 2,//禁止在条件表达式中使用赋值语句
"no-console": 2,//禁止使用console
"no-const-assign": 2,//禁止修改const声明的变量
"no-constant-condition": 2,//禁止在条件中使用常量表达式 if(true) if(1)
"no-continue": 0,//禁止使用continue
"no-control-regex": 2,//禁止在正则表达式中使用控制字符
"no-debugger": 2,//禁止使用debugger
"no-delete-var": 2,//不能对var声明的变量使用delete操作符
"no-div-regex": 1,//不能使用看起来像除法的正则表达式/=foo/
"no-dupe-keys": 2,//在创建对象字面量时不允许键重复 {a:1,a:1}
"no-dupe-args": 2,//函数参数不能重复
"no-duplicate-case": 2,//switch中的case标签不能重复
"no-else-return": 2,//如果if语句里面有return,后面不能跟else语句
"no-empty": 2,//块语句中的内容不能为空
"no-empty-character-class": 2,//正则表达式中的[]内容不能为空
"no-empty-label": 2,//禁止使用空label
"no-eq-null": 2,//禁止对null使用==或!=运算符
"no-eval": 1,//禁止使用eval
"no-ex-assign": 2,//禁止给catch语句中的异常参数赋值
"no-extend-native": 2,//禁止扩展native对象
"no-extra-bind": 2,//禁止不必要的函数绑定
"no-extra-boolean-cast": 2,//禁止不必要的bool转换
"no-extra-parens": 2,//禁止非必要的括号
"no-extra-semi": 2,//禁止多余的冒号
"no-fallthrough": 1,//禁止switch穿透
"no-floating-decimal": 2,//禁止省略浮点数中的0 .5 3.
"no-func-assign": 2,//禁止重复的函数声明
"no-implicit-coercion": 1,//禁止隐式转换
"no-implied-eval": 2,//禁止使用隐式eval
"no-inline-comments": 0,//禁止行内备注
"no-inner-declarations": [2, "functions"],//禁止在块语句中使用声明（变量或函数）
"no-invalid-regexp": 2,//禁止无效的正则表达式
"no-invalid-this": 2,//禁止无效的this，只能用在构造器，类，对象字面量
"no-irregular-whitespace": 2,//不能有不规则的空格
"no-iterator": 2,//禁止使用__iterator__ 属性
"no-label-var": 2,//label名不能与var声明的变量名相同
"no-labels": 2,//禁止标签声明
"no-lone-blocks": 2,//禁止不必要的嵌套块
"no-lonely-if": 2,//禁止else语句内只有if语句
"no-loop-func": 1,//禁止在循环中使用函数（如果没有引用外部变量不形成闭包就可以）
"no-mixed-requires": [0, false],//声明时不能混用声明类型
"no-mixed-spaces-and-tabs": [2, false],//禁止混用tab和空格
"linebreak-style": [0, "windows"],//换行风格
"no-multi-spaces": 0,//不能用多余的空格
"no-multi-str": 2,//字符串不能用\换行
"no-multiple-empty-lines": [1, {"max": 3}],//空行最多不能超过2行
"no-native-reassign": 2,//不能重写native对象
"no-negated-in-lhs": 2,//in 操作符的左边不能有!
"no-nested-ternary": 0,//禁止使用嵌套的三目运算
"no-new": 1,//禁止在使用new构造一个实例后不赋值
"no-new-func": 1,//禁止使用new Function
"no-new-object": 2,//禁止使用new Object()
"no-new-require": 2,//禁止使用new require
"no-new-wrappers": 2,//禁止使用new创建包装实例，new String new Boolean new Number
"no-obj-calls": 2,//不能调用内置的全局对象，比如Math() JSON()
"no-octal": 2,//禁止使用八进制数字
"no-octal-escape": 2,//禁止使用八进制转义序列
"no-param-reassign": 2,//禁止给参数重新赋值
"no-path-concat": 0,//node中不能使用__dirname或__filename做路径拼接
"no-plusplus": 0,//禁止使用++，--
"no-process-env": 0,//禁止使用process.env
"no-process-exit": 0,//禁止使用process.exit()
"no-proto": 2,//禁止使用__proto__属性
"no-redeclare": 2,//禁止重复声明变量
"no-regex-spaces": 2,//禁止在正则表达式字面量中使用多个空格 /foo bar/
"no-restricted-modules": 0,//如果禁用了指定模块，使用就会报错
"no-return-assign": 1,//return 语句中不能有赋值表达式
"no-script-url": 0,//禁止使用javascript:void(0)
"no-self-compare": 2,//不能比较自身
"no-sequences": 0,//禁止使用逗号运算符
"no-shadow": 2,//外部作用域中的变量不能与它所包含的作用域中的变量或参数同名
"no-shadow-restricted-names": 2,//严格模式中规定的限制标识符不能作为声明时的变量名使用
"no-spaced-func": 2,//函数调用时 函数名与()之间不能有空格
"no-sparse-arrays": 2,//禁止稀疏数组， [1,,2]
"no-sync": 0,//nodejs 禁止同步方法
"no-ternary": 0,//禁止使用三目运算符
"no-trailing-spaces": 1,//一行结束后面不要有空格
"no-this-before-super": 0,//在调用super()之前不能使用this或super
"no-throw-literal": 2,//禁止抛出字面量错误 throw "error";
"no-undef": 2,//不能有未定义的变量
"no-undef-init": 2,//变量初始化时不能直接给它赋值为undefined
"no-undefined": 2,//不能使用undefined
"no-unexpected-multiline": 2,//避免多行表达式
"no-underscore-dangle": 1,//标识符不能以_开头或结尾
"no-unneeded-ternary": 2,//禁止不必要的嵌套 var isYes = answer === 1 ? true : false;
"no-unreachable": 2,//不能有无法执行的代码
"no-unused-expressions": 2,//禁止无用的表达式
"no-unused-vars": [2, {"vars": "all", "args": "after-used"}],//不能有声明后未被使用的变量或参数
"no-use-before-define": 2,//未定义前不能使用
"no-useless-call": 2,//禁止不必要的call和apply
"no-void": 2,//禁用void操作符
"no-var": 0,//禁用var，用let和const代替
"no-warning-comments": [1, { "terms": ["todo", "fixme", "xxx"], "location": "start" }],//不能有警告备注
"no-with": 2,//禁用with

"array-bracket-spacing": [2, "never"],//是否允许非空数组里面有多余的空格
"arrow-parens": 0,//箭头函数用小括号括起来
"arrow-spacing": 0,//=>的前/后括号
"accessor-pairs": 0,//在对象中使用getter/setter
"block-scoped-var": 0,//块语句中使用var
"brace-style": [1, "1tbs"],//大括号风格
"callback-return": 1,//避免多次调用回调什么的
"camelcase": 2,//强制驼峰法命名
"comma-dangle": [2, "never"],//对象字面量项尾不能有逗号
"comma-spacing": 0,//逗号前后的空格
"comma-style": [2, "last"],//逗号风格，换行时在行首还是行尾
"complexity": [0, 11],//循环复杂度
"computed-property-spacing": [0, "never"],//是否允许计算后的键名什么的
"consistent-return": 0,//return 后面是否允许省略
"consistent-this": [2, "that"],//this别名
"constructor-super": 0,//非派生类不能调用super，派生类必须调用super
"curly": [2, "all"],//必须使用 if(){} 中的{}
"default-case": 2,//switch语句最后必须有default
"dot-location": 0,//对象访问符的位置，换行的时候在行首还是行尾
"dot-notation": [0, { "allowKeywords": true }],//避免不必要的方括号
"eol-last": 0,//文件以单一的换行符结束
"eqeqeq": 0,//必须使用全等
"func-names": 0,//函数表达式必须有名字
"func-style": [0, "declaration"],//函数风格，规定只能使用函数声明/函数表达式
"generator-star-spacing": 0,//生成器函数*的前后空格
"guard-for-in": 0,//for in循环要用if语句过滤
"handle-callback-err": 0,//nodejs 处理错误
"id-length": 0,//变量名长度
"indent": [2, 2],//缩进风格
"init-declarations": 0,//声明时必须赋初值
"key-spacing": [0, { "beforeColon": false, "afterColon": true }],//对象字面量中冒号的前后空格
"lines-around-comment": 0,//行前/行后备注
"max-depth": [0, 4],//嵌套块深度
"max-len": [0, 80, 4],//字符串最大长度
"max-nested-callbacks": [0, 2],//回调嵌套深度
"max-params": [0, 3],//函数最多只能有3个参数
"max-statements": [0, 10],//函数内最多有几个声明
"new-cap": 2,//函数名首行大写必须使用new方式调用，首行小写必须用不带new方式调用
"new-parens": 2,//new时必须加小括号
"newline-after-var": 2,//变量声明后是否需要空一行
"object-curly-spacing": [0, "never"],//大括号内是否允许不必要的空格
"object-shorthand": 0,//强制对象字面量缩写语法
"one-var": 1,//连续声明
"operator-assignment": [0, "always"],//赋值运算符 += -=什么的
"operator-linebreak": [2, "after"],//换行时运算符在行尾还是行首
"padded-blocks": 0,//块语句内行首行尾是否要空行
"prefer-const": 0,//首选const
"prefer-spread": 0,//首选展开运算
"prefer-reflect": 0,//首选Reflect的方法
"quotes": [1, "single"],//引号类型 `` "" ''
"quote-props":[2, "always"],//对象字面量中的属性名是否强制双引号
"radix": 2,//parseInt必须指定第二个参数
"id-match": 0,//命名检测
"require-yield": 0,//生成器函数必须有yield
"semi": [2, "always"],//语句强制分号结尾
"semi-spacing": [0, {"before": false, "after": true}],//分号前后空格
"sort-vars": 0,//变量声明时排序
"space-after-keywords": [0, "always"],//关键字后面是否要空一格
"space-before-blocks": [0, "always"],//不以新行开始的块{前面要不要有空格
"space-before-function-paren": [0, "always"],//函数定义时括号前面要不要有空格
"space-in-parens": [0, "never"],//小括号里面要不要有空格
"space-infix-ops": 0,//中缀操作符周围要不要有空格
"space-return-throw-case": 2,//return throw case后面要不要加空格
"space-unary-ops": [0, { "words": true, "nonwords": false }],//一元运算符的前/后要不要加空格
"spaced-comment": 0,//注释风格要不要有空格什么的
"strict": 2,//使用严格模式
"use-isnan": 2,//禁止比较时使用NaN，只能用isNaN()
"valid-jsdoc": 0,//jsdoc规则
"valid-typeof": 2,//必须使用合法的typeof的值
"vars-on-top": 2,//var必须放在作用域顶部
"wrap-iife": [2, "inside"],//立即执行函数表达式的小括号风格
"wrap-regex": 0,//正则表达式字面量用小括号包起来
"yoda": [2, "never"]//禁止尤达条件

```

核心思想： 
+ 路由设计的依据是什么？
答：内容切换的方式
+ 默认二级路由如何进行设置？
答：path配置项为空

### scss自动导入
1. 新建var.scss 文件
目录：./styles/var.scss
内容：
```vue
$xtxColor:#27ba9b;
$helpColor:#e26237;
$sucColor:#1dc779;
$warnColor:#ffb302;
$priceColor:#cf4444;
...
```
2. 配置 vite.config.js 自动加载
```vue
//vite样式自动导入及覆盖
  css:{
    preprocessorOptions:{
      scss:{
        //2.自动导入定制化样式文件进行样式覆盖
        additionalData:`
          @use "@/styles/element/index.scss" as *; 
          @use "@/styles/var.scss" as *; 
        `,
      }
    }
  }
```
3. 使用 lang="scss" 加载
```vue
<style scoped lang="scss">
.test{
   color:$priceColor
}
</style>
```
## Layout布局搭建
+ Nav区域
```vue
<template>
  <div class="g-container">
    <div>
      <div>周杰伦</div>
      <div>| 退出登录</div>
      <div>| 我的订单</div>
      <div>| 会员中心</div>
    </div>
  </div>
</template>
<style lang="scss">
.g-container {
  width: 100%;
  height: 45px;
  margin: 0 auto;
  line-height: 3;
  color: white;
  background-color: #4d4a4a;
  // position: relative;
  overflow: hidden;

  div {
    float: right;
    display: flex;
    width: 400px;
    // padding-right: 100px;
    div {
      justify-content: center;
      display: flex;
      width: 80px;
    }
  }
  & > div {
    // margin-bottom: 50px;
  }
}
</style>
```
+ Header区域
```vue
<script setup>
import { useCategoryStore } from "@/stores/category";
const categoryStore = useCategoryStore();
</script>
<template>
  <div class="h-container">
    <div class="d-logo">
      <img
        src="https://static.canva.cn/web/images/e1787d84651689b6f20de2ebd8d86baf.svg"
      />
    </div>
    <div
      class="d-nav"
      v-for="item in categoryStore.categoryList"
      :key="item.id"
    >
      {{ item.name }}
    </div>

    <div class="d-search">搜索</div>
  </div>
</template>
<style lang="scss">
.h-container {
  width: 100%;
  height: 130px;
  margin: 0 auto;
  line-height: 2;
  color: black;
  background-color: white;

  // position: relative;
  overflow: hidden;
  .d-logo {
    height: 130px;
    width: 300px;
    float: left;
    display: flex;

    margin-left: 200px;
    img {
      width: 100%;
      height: 100%;
    }
  }
  .d-nav {
    margin-left: 50px;
    height: 130px;
    float: left;
    display: flex;
    width: 50px;
    justify-content: center;
    align-items: center;
  }
  .d-search {
    margin-right: 200px;
    justify-content: center;
    align-items: center;
    height: 130px;
    float: right;
    display: flex;
    width: 100px;
  }
}
</style>

```
+ 二级路由出口区域
```vue
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      component: Layout,
      children: [
        {
          path: '',
          component: Home
        },
        {
          path: 'category',
          component: Category
        }
      ]
    },
    {
      path: '/login',
      component: Login
    }
  ]
})
```
+ Footer区域
使用阿里的图标：
地址：`https://www.iconfont.cn/`
1. 帮助 -> 代码应用 -> font-class 引用
2. 首页 -> 搜索 -> 要的图标 -> 添加购物车
3. index.html -> link ->  iconfont
```vue
//index.html
<link rel="stylesheet" href="//at.alicdn.com/t/font_8d5l8fzk5b87iudi.css">
//vue
<i class="iconfont icon-weixin"></i>
```

## 吸顶导航
1. 关键样式
```vue
<style lang="scss">
.hd-container {
   //关键样式：上平移自身高度+透明
   transform: translateY(-100%);
   opacity: 0;
   position: fixed;
   z-index: 10;
   overflow: hidden;
   
   //移除平移+完全不透明
   &.show {
      transition: all 0.3s linear;
      transform: none;
      opacity: 1;
   }
}
</style>

```
2. 插件@vueuse/core的useScroll
```vue
import { useScroll } from "@vueuse/core";

const { y } = useScroll(window);
```
3. 绑定show类，判断y>130显示
```vue
<div class="hd-container" :class="{ show: y > 130 }">
```

## pinia重复请求优化
1. layout组件请求接口
在apis/layout.js中：
```vue
import httpInstance from "@/utils/http";

export function getCategoryAPI() {
    return httpInstance({
        url: 'home/category/head'
    })
}
```
2. 利用pinia的store存储和action
在stores/category.js中：
```vue
import { ref } from 'vue'
import { defineStore } from 'pinia'
import { getCategoryAPI } from '@/apis/layout';

export const useCategoryStore = defineStore('category', () => {
  //state
  const categoryList = ref([]);

  //action
  const getCategory = async () => {
    const res = await getCategoryAPI();
    categoryList.value = res.result;
  };


  return {
    categoryList, getCategory
  }
})
```
3. 各模块组件的父类渲染
在Layout/index.vue中：
```vue
import { useCategoryStore } from "@/stores/category";
import { onMounted } from "vue";

const categoryStore = useCategoryStore();
onMounted(() => categoryStore.getCategory());
```
4. 各模块组件中获取state
在Layout/compoments/LayoutHeader.vue中
```vue
<script setup>
import { useCategoryStore } from "@/stores/category";
const categoryStore = useCategoryStore();
</script>
<template>
   <div
           class="d-nav"
           v-for="item in categoryStore.categoryList"
           :key="item.id"
   >
      {{ item.name }}
   </div>
</template>
```

## ElementPlus插件的使用
1. 安装
```vue
npm install element-plus --save
```
2. 加载
main.js中：
```vue
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
...
app.use(ElementPlus)

app.mount('#app')
```
3. 使用
```vue
<el-button>message</el-button>
```

## 左导航+轮播图
element的 menu 菜单(el-menu) + Carousel 走马灯(el-carousel)
```vue
<script lang="ts" setup>
import { useCategoryStore } from "@/stores/category";
const categoryStore = useCategoryStore();
</script>
<template>
  <div class="c-left">
    <!-- 侧边导航 -->
    <el-col
      :span="12"
      style="position: absolute; top: 0px; z-index: 10; width: 300px"
    >
      <el-menu
        default-active="2"
        class="el-menu-vertical-demo"
        style="background-color: black; opacity: 0.8"
        background-color="rgb(21, 249, 195)"
      >
        <el-menu-item v-for="i in categoryStore.categoryList" :key="i">
          <!-- <el-icon><setting /></el-icon> -->
          <span class="s-parent">{{ i.name }} </span>
          <span class="s-children" v-for="ic in i.children.slice(0, 2)">{{
            ic.name
          }}</span>
        </el-menu-item>
      </el-menu>
    </el-col>

    <!-- 轮播图 -->
    <div class="block text-center">
      <el-carousel height="500px">
        <el-carousel-item
          v-for="item in 4"
          :key="item"
          style="position: absolute; top: 0px; z-index: 9"
        >
          <img
            src="https://m.360buyimg.com/babel/jfs/t20260925/7569/37/22520/40263/65122aa8F7db4b1cc/4690718474f0e4b7.jpg"
          />
        </el-carousel-item>
      </el-carousel>
    </div>
  </div>
</template>
<style lang="scss">
.c-left {
  z-index: 8;
  position: relative;
  width: 1480px;
  height: 500px;
  // background: #000;
  margin: auto;
  .el-menu-vertical-demo {
    border: 0;
    .s-parent {
      font-size: 20px;
      color: white;
      margin-left: 40px;
    }
    .s-children {
      color: white;
      font-size: 15px;
      margin-left: 10px;
    }
  }
  .block {
    img {
      width: 100%;
      height: 500px;
    }
  }
  .text-center {
  }

  h3 {
  }
  img {
    width: 100%;
    height: 500px;
  }
}
.demonstration {
  color: var(--el-text-color-secondary);
}

.el-carousel__item h3 {
  color: #475669;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
  text-align: center;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
}
</style>
```

## 组件封装

组件参数：
1. props
2. 插槽

### 实现步骤
1. 不做任何抽象，准备静态模板 
- 定义属性 defineProps，占位模板
- 加入<slot / >
2. 抽象可变的部分
- 主标题和副标题是`纯文本`，可以抽象成`prop`传入
- 主体内容是`复杂的模版`，抽象成`插槽`
```vue
<script setup>
//定义props
defineProps({
  //主标题
  title: {
    type: String,
  },
  //副标题
  subTitle: {
    type: String,
  },
});
</script>
<template>
  <div class="container">
    <div class="panel-item">
      <div class="panel-head">
        <!-- 主标题和副标题 -->
        <h3>
          {{ title }}<small>{{ subTitle }}</small>
        </h3>
      </div>
      <!-- 主体内容 -->
      <slot />
    </div>
  </div>
</template>

//其它面板加载组件
<HomePanel title="新鲜好物" sub-title="新鲜好物 好多商品">
...复杂的静态文件
</HomePanel>
```

## 图片懒加载
实现步骤：
1. 定义全局指令，并绑定图片
在main.js中：
```vue
//定义全局指令
app.directive('img-lazy', {
    mounted(el, binding) {
        //el : 指令绑定的那个元素
        //binding: binding.value 指令等于号后面绑定的表达式的值 图片url
        // console.log(el, binding.value)

        useIntersectionObserver(
            el,
            ([{ isIntersecting }]) => {
                if (isIntersecting) {
                    //进入视口区域
                    el.src = binding.value
                }
                console.log(isIntersecting)
            },
        )
    }
})
```
在图片组件中：
```vue
<img v-img-lazy="item.picture" />
```
2. useIntersectionObserver 监听视口区
判断isIntersecting=true
```vue
if (isIntersecting) {
   //进入视口区域
   el.src = binding.value
}
```
3. 进入视口区域 图片src绑定
   监听对象的src=binding.value
   //el : 指令绑定的那个元素
   //binding: binding.value 指令等于号后面绑定的表达式的值 图片url

## 封装指定插件
实现步骤：
1. directives/index.js中加插件
```vue
//定义懒加载插件
import { useIntersectionObserver } from '@vueuse/core'

export const lazyPlugin = {
    install(app) {
        //懒加载指令逻辑

        //定义全局指令
        app.directive('img-lazy', {
            mounted(el, binding) {
                //el : 指令绑定的那个元素
                //binding: binding.value 指令等于号后面绑定的表达式的值 图片url
                // console.log(el, binding.value)

                const { stop } = useIntersectionObserver(
                    el,
                    ([{ isIntersecting }]) => {
                        if (isIntersecting) {
                            //进入视口区域
                            el.src = binding.value
                            stop()
                        }
                        
                        // console.log(isIntersecting)
                    },
                )
            }
        })

    }
}
```
2. main.js中加载
```vue
// 引入懒加载指令插件并且注册
import { lazyPlugin } from '@/directives'

app.use(lazyPlugin)

app.mount('#app')
```

## 路由配置RouterLink
```:to="" 中加 `` 符号，和变量${参数}```
```vue
<div class="dd-nav">
<RouterLink to="/" >首页</RouterLink>
</div>
<RouterLink :to="`/category/${item.id}`">{{
        item.name
      }}</RouterLink>
```

## 获取url参数
useRoute 插件
```vue
import { useRoute } from "vue-router";
//（params.id 来至 API请求接收的参数设置）
route.params.id
```

## 激活元素显示
```vue
active-class="active"
.active {
counter-reset: $xtxColor;
border-bottom: 1px solid $xtxColor;
}

```

## 路由缓存问题处理

+ 方式一：添加key
```vue
  <!-- 添加key 破坏复用机制 强制销毁重建-->
  <RouterView :key="$route.fullPath" />
```
+ 方式二：onBeforeRouteUpdate
```vue
<script setup>
//默认id = route.params.id
const getCategory = async (id = route.params.id) => {
   const res = await getCategoryAPI(id);
   cateGoryData.value = res.result;
};
onMounted(() => getCategory());

//目录：路由参数变化的时候，可以把分类数据接口重新发送
onBeforeRouteUpdate((to) => {
   console.log("路由变化了");
   //存在问题：使用最新的路由参数请求最新的分类数据
   console.log(to);
   getCategory(to.params.id);
});
</script>
```

## 函数封闭，拆分业务，利于维护
基于逻辑函数拆分业务是把同一个组件中独立的业务代码通过函数做封装处理，提升代码的可维护性
实现步骤：
1. 把代码集中放于composables中并命名
2. 方法中最后return对象结果
3. 在模板中引用该引用


## 无限加载数据
实现步骤：
1. 容器的div中添加v-infinite-scroll="load"
```vue
<div class="sub-body" v-infinite-scroll="load"></div>
```
2. 为load添加方法获取数据
```vue
<script>
//加载更多
const disabled = ref(false);
const load = async () => {
   console.log("加载更多数据");
   //获取下一页数据
   reqData.value.page++;
   const resNew = await getSubCategoryAPI(reqData.value);
   goodList.value = [...goodList.value, ...resNew.result.items];
   //加载完毕 停止监听
   if (res.result.items.length === 0) {
      disabled.value = true;
   }
};
</script>
```
3. 拼接第二页的数据
```vue
<script>
goodList.value = [...goodList.value, ...resNew.result.items]
</script>
```

## 路由切换滚动条置顶
在router/index.js中：
```vue
  routes:[
  ...
  ],
  //路由滚动顶部
  scrollBehavior() {
    return { top: 0 }
  }
```
## 渲染模板遇到对象的多层属性访问为空
`good.details.pictures`
`TypeError:Cannot read properties of undefined (reading 'properties')`
解决方法：
1. 可选链 `?.`
```vue
//？号前端有值，才继续运算
${goods.categories?.[1].id}
```
2. v-if控制渲染
```vue
<div class="container" v-if="goods.details">

</div>
```

## tab类样式切换
绑定class，判断当前元素的id和激动状态activeInde是不是相等
```vue
.active 是样式
:class="{ active: i === activeIndex }"
```

## 扩大镜

## 注册全局组件
1. 在目录components目录下新建文件index.js 内容如下：
```vue
//把components中的所有组件都进行全局化注册
//通过插件的方式
import ImageView from './ImageView/index.vue'
import Sku from './XtxSku/index.vue'

export const componentPlugin = {
   install(app) {
      // app.component('组件名字',组件配置对象)
      app.component('XtxImageView', ImageView)
      app.component('XtxSku', Sku)
   }
}
```
2. main.js 增加内容如下：
```vue
//引入全局组件插件
import { componentPlugin } from '@/components'

app.use(componentPlugin)
app.mount('#app')
```
3. 之后就可以把一些引入加载的组件去掉之后直接使用`XtxImageView` 和 `XtxSku`

## 静态跳转
```vue
<a href="javascript:;" @click="$router.push(`/login`)">请先登录</a>
```
## 验证功能
el-form => 绑定`表单对象`和`规则对象`
el-form-item => 绑定使用的`规则字段`
el-input => 双向绑定`表单数据`
步骤：
1. 准备表单对象
```vue
const form = ref({
account: "",
password: "",
});
```
2. 准备规则对象
```vue
const rules = {
account: [{ required: true, message: "用户名不能为空", trigger: "blur" }],
password: [
{ required: true, message: "密码不能为空", trigger: "blur" },
{ min: 6, max: 14, message: "密码长度6-14", trigger: "blur" },
],
};
```
3. 指定表单域的检验字段名
4. 双向绑定输入对象
```vue
<el-form :model="form" :rules="rules">
   <el-form-item prop="account" label="账户">
      <el-input v-model="form.account" />
   </el-form-item>
   <el-form-item prop="password" label="密码">
      <el-input v-model="form.password" />
   </el-form-item>
   <el-button>点击登录</el-button>
</el-form>
```

*** PS:vue处理复杂功能 ***
思想：当功能很复杂时，通过 多个组件各自负责某个小功能，再组合成一个大功能是组件设计中的常用方法

## 自定义验证规则 
方法：
```vue
{
  validator:(rule,val,callback) = >{
    //自定义检验逻辑
    //value：当前输入的数据
    //callback:检验处理函数 校验通过调用 
  }
}
```
添加内容如下：
```vue
agree: [
    {
      validator: (rule, value, callback) => {
        console.log(value);
        //自定义梳校验逻辑
        //勾选就通过 不勾选就不通过
        if (value) {
          callback();
        } else {
          callback(new Error("请勾选协议"));
        }
      },
    },
  ],
.
.
.
<el-form-item prop="agree" label="协议">
<el-input v-model="form.agree" />
</el-form-item>
```

## 整个表单的内容验证
对所有需要检验的表单进行统一梳校验
```vue
formEl.validator((valid) => {
  if (valid) {
    console.log("submit");
  } else {
    console.log("error submit");
    return false;
  }
});
```
增加内容如下：
```vue
//获取form实例做统一校验
const formRef=ref(null)
const doLogin=()=>{
  //调用实例方法
  formRef.value.validator(()=>{
    //valid:所有表单都通过校验 才为true
    console.log(valid)
    //以valid做为判断条件 如果通过校验才执行登录逻辑
    if(valid){
      //TODO LOGIN
    }
  })
}
//vue 获取实例 :ref=""
<el-form :ref="formRef" :model="form" :rules="rules">
```

### vue获取实例：
1. 先定义空变量
   `const formRef=ref(null)`
2. 再用:ref绑定空变量
   `:ref="formRef"`
### 结构赋值
 有三个变量，但现在只取2个
```vue
const form = ref({
   account: "",
   password: "",
   agree: true,
});
const { account , password } =form.value
```

### 登录操作
```vue
import { useRouter } from "vue-router";

const res = await loginAPI({account,password})
//1.提示用户
ElMessage({type:'success',message:'登录成功'})
//2.跳转首页
const router=useRouter()
router.replace({path:'/'})
```

### 存储localStorage
挺久化存储插件：pinia-plugin-persistedstate
1. 安装插件
```vue
npm i pinia-plugin-persistedstate
```
2.注册插件
在main.js中，添加：
```vue
const pinia = createPinia()
//注册持久化插件
pinia.use(PiniaPluginPersistedstate)
```
3. 使用插件
创建store时，将 persistent 选项设置为 true
```vue
import { defineStore } from 'pinia'
export const useStore = defineStore('main', () => {
   const someState = ref('你好token')
   return { someState }
   },
   { persist: true }
)
```


## 登录和非登录状态的模版乱配
核心思想：`v-if="false  v-else`
```vue
<template v-if="userStore.userInfo.token">
   //登录时显示第一块
</template>
<template v-else>
   //非登录时显示第二块
</template>
```

## 请求拦截器携带token
核心思想：
```vue
httpInstance.interceptors.request.use(config => {
    const userStore = userUserStore()
    const token = userStore.userInfo.token
    if (token) {
        config.headers.Authorization = `Bearer ${token}`
    }
    return config
}, e => Promise.rejecte)
```

## 退出登录
1. 步骤一：pinia中写清除方法
```vue
  //退出时清除用户信息
  const clearUserInfo = () => {
    clearUserInfo.value = {}
  }
```
2. 步骤二：登录页面处理逻辑
```vue
const confirm = () => {
   console.log("用户要退出登录了");
   //退出登录逻辑
   //1 清除用户信息 触发action
   useCounterStore.clearUserInfo();
   //2 跳转路由地址
   router.push("/login");
};
```
## 统一错误提示
```vue
//axios响应式拦截器
httpInstance.interceptors.response.use(res => res.data, e => {
    //统一错误提示
    ElMessage({
        type:'warning',
        message:e.response.data.message
    })

    return Promise.reject(e)
})
```
## token失效401
```vue
//axios响应式拦截器
httpInstance.interceptors.response.use(res => res.data, e => {
    //统一错误提示
    ElMessage({
        type: 'warning',
        message: e.response.data.message
    })

    //401 token失效处理
    //1 清除本地用户数据
    //2 跳转登录页
    if(e.response.status==401){
        userStore.clearUserInfo()
        router.push('/login')
    }

    return Promise.reject(e)
})
```

## 添加本地购物车
stores/cartStore.js 的内容：
```vue
//封装财物车模块
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useCartStore=defineStore('cart',()=>{
  //1 定义state - cartList
  const cartList = ref([])
  //2 定义action - addCart
  const addCart=(goods)=>{
    //添加购物车逻辑
    //思路：通过匹配传递过来的商品对象中的skuId能不能在cartList中找到，找到了就是添加过
cosnt item = cartList.value.find((item)=>goods.skuId === item.skuId )
    //已添加过 count+1
    if(item){
    //找到了
    item.count++
    }else{
    //没找到
    cartList.value.push(goods)
    }
    //没有添加过 直接push
    
  }
  return {
    cartList,
    addCart
  }
},{
  persist:true,
})
```

```vue
//sku规格被操作时
let skuObj={}
const skuChange=()=>{
  console.log(sku)
  skuObj=sku
}

const count=ref(1)
const countChange = (count)=>{
  console.log(count)
  
}


//添加财物车
const cartStore = useCartStore()
const addCart = ()=>{
  if(skuObj.skuId){
  //规则已经选择 触发action
    cartStore.addCart({
       id:goods.value.id,
       name:goods.value.name,
       picture:goods.value.picture,
       price:goods.value.price,
       count:count.value,
       skuId:skuObj.skuId,
       attrsText:skuObj.attrsText,
       selected:true
    })
  }else{
  //规则没有选择 提示用户
  ElMessage.warning('请选择规则')
  }
}
```

## 删除购物车
思路：
1. 找到要删除项的下标值 - splice
2. 使用数组的过滤方法 - filter
```vue
<script>
const delCart = (skuId)=>{
  const idx=cartList.value.findIndex((item)=> skuId === item.skuId)
  cartList.value.splice(idx,1)
}
</script>
```

## 计算属性
1. 总的数量 所有项的count之和
```vue
const allCount = computed( ()=>cartList.value.reduce( (a,c)=>a + c.count,0))
```
2. 总价 所有项的count * price 之和
```vue
const allPrice = computed( ()=>cartList.value.reduce( (a,c)=>a + c.count * c.price ,0))
```
//`保留两位小数`
cartStore.allPrice.`toFixed(2)`

## 列表购物车-单选功能
核心思路：单选的核心思路就是始终把 `单行框的状态和Pinia中store对应的状态保持同步`
注意事项：`v-model` 双向绑定指令不方便进行命令式的操作（因为后续还要调用接口），所以把 `v-model` 回退到一般模式，也就是: `model-value` 和 `@change` 的配合实现
步骤：
+ 步骤一：pinia（store）-- (使用store渲染单行框) --> :model-value
+ 步骤二：@change -- 单选框切换时修改store中对应的状态 --> pinia（store）

## 全选
1. 是否全选：
方法: `every`
```vue
const isAll=computed(()=>cartList.value.every((item) => item.selected ))
...
:model-value="cartStore.isAll"
```
2. 全选功能
方法： `forEach`
```vue
const allCheck = (selected)=>{
//把cartLst中的每一项的selected都设置为当前的全选框状态
cartList.value.forEach(item => item.selected = selected)
}
```
## 列表购物车-统计数据实现
核心方法： `filter` 和 `reduce`
1. 已选择数量 = cartList中所有selected字段为true项的count之和
2. 商品合计 = cartList中所有selected字段为true项的count*price之和
```vue
//已选择数量:
const selectedCount = computed( ()=>cartList.value.filter(item=>item.selected).reduce( (a,c)=>a + c.count,0))
//已选择商品价钱合计:
const selectedPrice = computed( ()=>cartList.value.filter(item=>item.selected).reduce( (a,c)=>a + c.count*c.price,0))
```

## tab切换类
核心思想：
1. 点击时记录一个当前激活地址对象activeAddress，点击哪个地址就把哪个地址对象记录下来
2. 通过 动态类名 `:class` 控制激活样式类型 active 是否存在，判断条件为：`激活地址对象的id === 当前项id`
```vue
<template>
   <div class="text item" :class="{active: activeAddress.id === item.id }"> TAB </div>
</template>
```

## 倒计时函数
```vue
export const useCountDown = ()=>{
   //1 响应式的数据
  let timer = null
   const time = ref(0)
   //格式化时间为 XX分XX秒
   const formatTime = computed(()=>dayjs.unix(time.value).format('mm分ss秒'))
   //2 开启倒计时的函数
   const start= (currentTime)=>{
     //开始倒计时的逻辑
     //核心逻辑的编写：每隔1s就--
     time.value = currentTime
     timer = setInterval(()=>{
        time.value--
     },1000)
   }
//组件销毁时清除定时器
  onUnmounted(()=>{
    timer && clearInterval(timer)
  })

   return {
     formatTime,
     start
   }
}
```
使用如下：
```vue
const { formatTime,start } = useCountDown()
{{ formatTime }}
start(60)
```

## 分页逻辑实现
1. 使用列表数据生成分页（页数=总条数/每页条数）
```vue
<template>
   <el-pagination :total="total" :page-size="params.pageSize" /> 
</template>
```
2. 切换分页修改page参数，再次获取订单列表数据
```vue
```vue
<script>
const pageChange = (page)=>{
  console.lgo(page)
   params.value.page=page
   //发送请求
   getOrderList()
}
</script>
<template>
   <el-pagination :total="total" :page-size="params.pageSize" @current-change="pageChange"/> 
</template>
```
```