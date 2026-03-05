---
author: "karson"
title: "学习go(第二部分)"
date: 2021-01-04 01:01:00
description: "go语言起源、安装运行环境、编辑器、集成等"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- golang
- categories:

---


## 一、基本结构与基本数据类型
### 1、文件名、关键字与标识符
#### 1.1 文件名
+ 文件名 均由小写字母组成，如 scanner.go。
+ 如果文件名由多个部分组成，则使用下划线 _ 对它们进行分隔，如 scanner_test.go
+ 文件名不包含 空格 或 其他特殊字符。

<font color="red">_</font> 本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用这个这个标识符作为变量对其它变量的进行赋值或运算。
#### 1.2 关键字
Go 代码中会使用到的 25 个关键字或保留字：
|||||
|-------:|-------:|-------:|-------:|
|  break |  default  |  func  |  interface   |
|  case |  defer  |  go  |  map   |
|  chan |  else  |  goto  |  package   |
|  const |  fallthrough  |  if  |  range   |
|  continue |  for  |  import  |  return   |

#### 1.3 标识符
Go 语言还有 36 个预定义标识符:
|||||||
|-------:|-------:|-------:|-------:|-------:|-------:|
|append|	bool	|byte	|cap	|close	|complex
|copy	|false	|float32	|float64|	imag	|int
|int32	|int64	|iota	|len	|make	|new
|print	|println|	real	|recover|	string	|true

程序一般由关键字、常量、变量、运算符、类型和函数组成。

程序中可能会使用到这些分隔符：括号 ()，中括号 [] 和大括号 {}。

程序中可能会使用到这些标点符号：.、,、;、: 和 …。

程序的代码通过语句来实现结构化。每个语句不需要像 C 家族中的其它语言一样以分号 ; 

结尾，因为这些工作都将由 Go 编译器自动完成。

如果你打算将多个语句写在同一行，它们则必须使用 ; 人为区分，但在实际开发中我们并不鼓励这种做法。

### 2、Go 程序的基本结构和要素
#### 2.1 包的概念、导入与可见性
包是结构化代码的一种方式：每个程序都由包（通常简称为 pkg）的概念组成，可以使用自身的包或者从其它包中导入内容。
一个应用程序可以包含不同的包，而且即使你只使用 main 包也不必把所有的代码都写在一个巨大的文件里：你可以用一些较小的文件，并且在每个文件非注释的第一行都使用 package main 来指明这些文件都属于 main 包。
标准库:
在 Go 的安装文件里包含了一些可以直接使用的包，即标准库。
一般情况下，标准包会存放在 <font color="red">$GOROOT/pkg/$GOOS_$GOARCH/</font> 目录下。
<font color="jade"><strong>如果对一个包进行更改或重新编译，所有引用了这个包的客户端程序都必须全部重新编译。</strong></font>
Go 中的包模型采用了显式依赖关系的机制来达到快速编译的目的，编译器会从后缀名为 .o 的对象文件（需要且只需要这个文件）中提取传递依赖类型的信息。

如果 A.go 依赖 B.go，而 B.go 又依赖 C.go：

编译 C.go, B.go, 然后是 A.go.
为了编译 A.go, 编译器读取的是 B.o 而不是 C.o.
这种机制对于编译大型的项目时可以显著地提升编译速度。
<font color="jade">每一段代码只会被编译一次</font>
如果需要多个包，它们可以被分别导入：
更短且更优雅的方法（被称为因式分解关键字，该方法同样适用于 const、var 和 type 的声明或定义）：
```golang
import (
"fmt"
"os"
)
```
+ 如果包名不是以 . 或 / 开头，如 "fmt" 或者 "container/list"，则 Go 会在 全局文件 进行查找；
+ 如果包名以 ./ 开头，则 Go 会在 相对目录 中查找；
+ 如果包名以 / 开头（在 Windows 下也可以这样使用），则会在系统的绝对路径中查找。

可见性规则:

+ 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个<font color="jade">大写字母开头</font>，如：Group1，那么使用这种形式的标识符的对象就可以<font color="jade">被外部包</font>的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；
+ 标识符如果以<font color="jade">小写字母开头</font>，则<font color="jade">对包外是不可见的</font>，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）。

包的分级声明和初始化:

可以在使用 import 导入包之后定义或声明 0 个或多个常量（const）、变量（var）和类型（type），这些对象的 <font color="jade"> 作用域 </font>都是全局的（在本包范围内），所以可以<font color="jade">  被本包中所有的函数调用 </font>,然后声明一个或多个函数（func）

#### 2.2 函数
这是定义一个函数最简单的格式：
{{<boxmd>}}func functionName(){{</boxmd>}}
函数里的代码（函数体）使用大括号 {} 括起来。

左大括号 { 必须与方法的声明放在同一行，这是编译器的强制规定，否则你在使用 gofmt 时就会出现错误提示：
{{<boxmd>}}
`build-error: syntax error: unexpected semicolon or newline before {`
{{</boxmd>}}

<font color="jade">Go 语言虽然看起来不使用分号作为语句的结束，但实际上这一过程是由编译器自动完成，因此才会引发像上面这样的错误</font>

#### 2.3 注释
{{<boxmd>}}
package main

import "fmt" // Package implementing formatted I/O.

func main() {
fmt.Printf("Καλημέρα κόσμε; or こんにちは 世界\n")
}
{{</boxmd>}}
上面这个例子通过打印 Καλημέρα κόσμε; or こんにちは 世界 展示了如何在 Go 中使用国际化字符，以及如何使用注释。
+ 单行注释是最常见的注释形式，你可以在任何地方使用以 <font color="jade"><b> // </b></font> 开头的单行注释。
+ 多行注释也叫块注释，均已以 <font color="jade"><b>/* 开头，并以 */ 结尾</b></font>，且不可以嵌套使用，多行注释一般用于包的文档描述或注释成块的代码片段。

#### 2.4 类型
可以包含数据的变量（或常量）可以使用不同的数据类型或类型来保存数据。使用 var 声明的变量的值会自动初始化为该类型的零值。类型定义了某个变量的值的集合与可对其进行操作的集合。

|  基本类型 |
|------:|
|    int   |
|    float   |
|    bool   |
|    string   |

|  复合类型 |
|------:|
|    struct   |
|    array   |
|    slice   |
|    map   |
|    channel   |

|  描述类型 |
|------:|
|    interface   |

<font color="jade"><b>Go 语言中不存在类型继承</b></font>

#### 2.5 Go 程序的一般结构
Go 程序的执行（程序启动）顺序如下：

1. 按顺序导入所有被 main 包引用的其它包，然后在每个包中执行如下流程：
2. 如果该包又导入了其它的包，则从第一步开始递归执行，但是每个包只会被导入一次。
3. 然后以相反的顺序在每个包中初始化常量和变量，如果该包含有 init 函数的话，则调用该函数。
4. 在完成这一切之后，main 也执行同样的过程，最后调用 main 函数开始执行程序。

#### 2.6 类型转换
在必要以及可行的情况下，一个类型的值可以被转换成另一种类型的值。由于 Go 语言不存在隐式类型转换，因此所有的转换都必须显式说明，就像调用一个函数一样（类型在这里的作用可以看作是一种函数）：
{{<boxmd>}}
valueOfTypeB = typeB(valueOfTypeA)
{{</boxmd>}}
<font color="jade"><b>类型 B 的值 = 类型 B(类型 A 的值)</b></font>
```golang
a := 5.0
b := int(a)
```
但这只能在定义正确的情况下转换成功，例如从一个取值范围较小的类型转换到一个取值范围较大的类型（例如将 int16 转换为 int32）。当从一个取值范围较大的转换到取值范围较小的类型时（例如将 int32 转换为 int16 或将 float32 转换为 int），会发生精度丢失（截断）的情况。当编译器捕捉到非法的类型转换时会引发编译时错误，否则将引发运行时错误。

具有相同底层类型的变量之间可以相互转换：
```golang
var a IZ = 5
c := int(a)
d := IZ(c)
```

### 3、常量
常量使用关键字 <font color="jade"><b>const</b></font> 定义，用于存储 不会改变 的数据。
如：
```golang
const Pi = 3.14159

显式类型定义： const b string = "abc"

隐式类型定义： const b = "abc"
```
常量的值必须是能够在编译时就能够确定的；你可以在其赋值表达式中涉及计算过程，但是所有用于计算的值必须在编译期间就能获得。
```golang
正确的做法：const c1 = 2/3

错误的做法：const c2 = getNumber() // 引发构建错误: getNumber() used as value
```
<font color="jade"><b>因为在编译期间自定义函数均属于未知，因此无法用于常量的赋值，但内置函数可以使用，如：len()。</b></font>

在这个例子中，iota 可以被用作枚举值：
```golang
const (
   a = iota
   b = iota
   c = iota
)
```

第一个 iota 等于 0，每当 iota 在新的一行被使用时，它的值都会自动加 1；所以 a=0, b=1, c=2 可以简写为如下形式：
```golang
const (
      a = iota
      b
      c
)
```


简单地讲，每遇到一次 const 关键字，iota 就重置为 0


### 4、变量
这种语法能够按照从左至右的顺序阅读，使得代码更加容易理解。

示例：
```golang
var a int
var b bool
var str string
```

你也可以改写成这种形式：
```golang
var (
   a int
   b bool
   str string
)
```

### 5、基本类型和运算符

### 6、字符串

### 7、strings 和 strconv 包

### 8、时间和日期

### 9、指针