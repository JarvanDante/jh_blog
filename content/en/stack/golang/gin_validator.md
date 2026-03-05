---
author: "karson"
title: "gin框架验证器validator"
date: 2022-12-03 00:00:01
description: "只需定义结构体使用binding或validate tag标识校验规则，就可以进行参数校验"
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

## 安装
地址：github.com/go-playground/validator/v10
命令：
```shell
go get github.com/go-playground/validator/v10
```

## 模式绑定
若要将请求主体绑定到结构体中，请使用模型绑定，目前支持JSON、XML、YAML和标准表单值(foo=bar&boo=baz)的绑定。

需要在绑定的字段上设置tag，比如，绑定格式为json，需要这样设置 json:“fieldname” 

gin提供了两套绑定方法：
### 1.Must bind
+ Methods 
支持：Bind, BindJSON, BindXML, BindQuery, BindYAML
+ Behavior
这些方法底层使用 MustBindWith，如果存在绑定错误，请求将被以下指令中止 c.AbortWithError(400, err).SetType(ErrorTypeBind)，响应状态代码会被设置为400，请求头Content-Type被设置为text/plain; charset=utf-8。注意，如果你试图在此之后设置响应代码，将会发出一个警告 [GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422，如果你希望更好地控制行为，请使用ShouldBind相关的方法

来看看MustBindWith方法实现：
```go
//MustBindWith使用指定的绑定引擎绑定传递的结构指针。
//如果发生任何错误，它将中止HTTP 400的请求。
func (c *Context) MustBindWith(obj any, b binding.Binding) error {
	if err := c.ShouldBindWith(obj, b); err != nil {
		c.AbortWithError(http.StatusBadRequest, err).SetType(ErrorTypeBind) // nolint: errcheck
		return err
	}
	return nil
}
```
### 2.Should bind
+ Methods 
支持：ShouldBind, ShouldBindJSON, ShouldBindXML, ShouldBindQuery, ShouldBindYAML
+ Behavior 
这些方法底层使用 ShouldBindWith，如果存在绑定错误，则返回错误，开发人员可以正确处理请求和错误。

来看看ShouldBindWith方法实现:
```go
//使用指定的绑定引擎绑定传递的struct指针。
func (c *Context) ShouldBindWith(obj any, b binding.Binding) error {
	return b.Bind(c.Request, obj)
}
```

#### ShouldBindJSON方法
ShouldBindJSON是c.ShouldBindWith(obj, binding.JSON)的快捷方式。

JSON绑定结构体：
```go
type PostParams struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
	Sex  bool   `json:"sex"`
}
```
ShouldBindJSON代码示例：
```go
func main() {
	r := gin.Default()
	r.POST("/testBind/", func(c *gin.Context) {
		//声明一个PostParams结构体
		var p PostParams
		//通过ShouldBindJSON方法绑定结构体的对应属性
		err := c.ShouldBindJSON(&p)
		fmt.Printf("p: %v\n", p)
		if err != nil {
			fmt.Println(err)
			c.JSON(404, gin.H{
				"msg":  "报错了",
				"data": gin.H{},
			})
		} else {
			c.JSON(200, gin.H{
				"msg":  "成功了",
				"data": p,
			})
		}
	})
	r.Run(":8080")
}
```

验证：
POST请求，访问http://localhost:8080/testBind
json参数：
```text
{
    "name": "admin",
    "age": 66,
    "sex": true
}
```
#### ShouldBindUri方法
ShouldBindUri使用指定的绑定引擎绑定传递的struct指针。

Uri绑定结构体：
```go
type PostParams struct {
	Name string `uri:"name"`
	Age  int    `uri:"age"`
	Sex  bool   `uri:"sex"`
}
```
ShouldBindUri代码示例：
```go
func main() {
	r := gin.Default()
	//路由路径变为uri形式获取参数
	r.POST("/testBind/:name/:age/:sex", func(c *gin.Context) {
		//声明一个PostParams结构体
		var p PostParams
		//通过ShouldBindUri方法绑定结构体的对应属性
		err := c.ShouldBindUri(&p)
		fmt.Printf("p: %v\n", p)
		if err != nil {
			fmt.Println(err)
			c.JSON(404, gin.H{
				"msg":  "报错了",
				"data": gin.H{},
			})
		} else {
			c.JSON(200, gin.H{
				"msg":  "成功了",
				"data": p,
			})
		}
	})
	r.Run(":8080")
}
```

验证：
POST请求，访问http://localhost:8080/testBind/linzy/23/true

参数在uri上面

#### ShouldBindQuery方法
ShouldBindQuery是c.ShouldBindWith(obj, binding.Query)的快捷方式

Query绑定结构体：
```go
type PostParams struct {
	Name string `form:"name"`
	Age  int    `form:"age"`
	Sex  bool   `form:"sex"`
}
```

ShouldBindQuery代码示例：

```go
func main() {
	r := gin.Default()
	r.GET("/testBind", func(c *gin.Context) {
		//声明一个PostParams结构体
		var p PostParams
		//通过ShouldBindQuery方法绑定结构体的对应属性
		err := c.ShouldBindQuery(&p)
		fmt.Printf("p: %v\n", p)
		if err != nil {
			fmt.Println(err)
			c.JSON(404, gin.H{
				"msg":  "报错了",
				"data": gin.H{},
			})
		} else {
			c.JSON(200, gin.H{
				"msg":  "成功了",
				"data": p,
			})
		}
	})
	r.Run(":8080")
}
```

验证：
GET请求，访问http://localhost:8080/testBind?name=linzy&age=23&sex=true

参数用query

## 参数验证器
我们可以给字段指定特定规则的修饰符，如果一个字段用binding:"required"修饰，并且在绑定时该字段的值为空，那么将返回一个错误

### 结构体验证器
用gin框架数据验证，可以不用解析数据，来if-else判断，整体使代码精简了很多。
`binding:"required" `就是gin自带的数据验证，表示数据不为空，为空则返回错误

定义结构体：
```go
type PostParams struct {
	Name string `json:"name"`
	//age不为空并且大于10
	Age int  `json:"age" binding:"required,gt=10"`
	Sex bool `json:"sex"`
}
```

代码示例：
```go
func main() {
	r := gin.Default()
	r.POST("/testBind", func(c *gin.Context) {
		var p PostParams
		err := c.ShouldBindJSON(&p)
		fmt.Printf("p: %v\n", p)
		if err != nil {
			fmt.Println(err)
			c.JSON(404, gin.H{
				"msg":  "报错了",
				"data": gin.H{},
			})
		} else {
			c.JSON(200, gin.H{
				"msg":  "成功了",
				"data": p,
			})
		}
	})
	r.Run(":8080")
}
```

### 自定义数据验证

对绑定解析到结构体上的参数，自定义验证功能。比如我们想name不为空的同时，不能为admin的时候，就无法 binding 现成的方法

结构体：
```go
type PostParams struct {
	//在参数binding上使用自定义的校验方法函数注册时候的名称
	//name不为空且不能为admin
	Name string `json:"name" binding:"required,notAdmin"`
	Age  int    `json:"age"`
	Sex  bool   `json:"sex"`
}
```

自定义的校验方法：

```go
//自定义的校验方法
func notAdmin(v validator.FieldLevel) bool {
	//Field字段返回当前字段进行验证
	//返回的字段需要转为接口用断言获取底层数据进行校验
	if v.Field().Interface().(string) == "admin" {
		return false
	}
	return true
}
```

代码示例：
```go
func main() {
	r := gin.Default()
	//将我们自定义的校验方法注册到 validator中
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		//这里的 key 和 fn 可以不一样最终在 struct 使用的是 key
		v.RegisterValidation("notAdmin", notAdmin)
	}
	r.POST("/testBind", func(c *gin.Context) {
		var p PostParams
		err := c.ShouldBindJSON(&p)
		fmt.Printf("p: %v\n", p)
		if err != nil {
			fmt.Println(err)
			c.JSON(404, gin.H{
				"msg":  "name 不能为admin",
				"data": gin.H{},
			})
		} else {
			c.JSON(200, gin.H{
				"msg":  "成功了",
				"data": p,
			})
		}
	})
	r.Run(":8080")
}
```