---
author: "jarvan"
title: "SpringBoot MVC 核心原理、使用、优点、配置与注意事项"
date: 2025-01-01
description: "系统整理 SpringBoot MVC 的请求流程、核心组件、常用注解、配置方式、架构分层、优点以及项目实践中的注意事项。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👤
tags:
- java
- springboot
- mvc
- web
categories:
- java
---

# SpringBoot MVC 是什么

SpringBoot MVC 是基于 Spring MVC 封装出来的 Web 开发能力。

它帮助我们快速开发 HTTP 接口、页面路由、参数接收、数据返回、异常处理、拦截器、文件上传等 Web 功能。

简单理解：

```shell
客户端请求
   ↓
SpringBoot MVC 接收请求
   ↓
找到对应 Controller 方法
   ↓
执行业务逻辑
   ↓
返回 JSON / 页面 / 文件
```

SpringBoot MVC 的核心价值是：

```shell
用统一的请求处理模型，把 Web 请求、业务方法和响应结果串起来。
```

---

# MVC 架构思想

MVC 是 Model、View、Controller 的缩写。

```shell
Model      数据模型，负责承载业务数据
View       视图层，负责展示结果
Controller 控制层，负责接收请求、调用业务、返回响应
```

在现在的前后端分离项目中，View 往往不再是 JSP、Thymeleaf 页面，而是 JSON 响应。

所以常见结构变成：

```shell
Controller  接收请求
Service     处理业务
Mapper/DAO  访问数据库
Model/DTO   承载数据
JSON        返回给前端
```

也就是：

```shell
前端 Vue / React
        ↓ HTTP
SpringBoot Controller
        ↓
Service 业务层
        ↓
Mapper 数据访问层
        ↓
MySQL / Redis / MQ
```

---

# SpringBoot MVC 请求核心流程

SpringBoot MVC 的请求处理核心是 `DispatcherServlet`。

完整流程可以理解为：

```shell
浏览器 / 客户端
   ↓
Tomcat 接收请求
   ↓
DispatcherServlet
   ↓
HandlerMapping 查找 Controller 方法
   ↓
HandlerAdapter 调用 Controller 方法
   ↓
Controller 执行业务
   ↓
HttpMessageConverter 转换返回结果
   ↓
返回 JSON / HTML / 文件
```

更细一点：

```shell
1、客户端发送 HTTP 请求
2、Tomcat 接收请求并交给 DispatcherServlet
3、DispatcherServlet 根据 URL 查找 HandlerMapping
4、HandlerMapping 找到对应 Controller 方法
5、HandlerAdapter 负责真正调用方法
6、参数解析器解析请求参数
7、Controller 调用 Service 完成业务
8、返回值处理器处理返回结果
9、HttpMessageConverter 转成 JSON
10、响应返回客户端
```

---

# 一、DispatcherServlet 前端控制器

`DispatcherServlet` 是 Spring MVC 的总入口，也叫前端控制器。

它的职责：

```shell
接收所有 Web 请求
协调各个 MVC 组件
查找处理器
调用 Controller
处理返回值
完成响应输出
```

它本身不写业务逻辑，只负责调度。

可以理解成：

```shell
DispatcherServlet = MVC 请求处理的总指挥
```

所有请求先进入它，再由它分发给真正的 Controller。

---

# 二、HandlerMapping 路由映射器

`HandlerMapping` 的职责是根据请求路径找到对应的处理方法。

例如：

```java
@RestController
@RequestMapping("/api/user")
public class UserController {

    @GetMapping("/{id}")
    public UserDTO detail(@PathVariable Long id) {
        return userService.detail(id);
    }
}
```

当请求进入：

```shell
GET /api/user/1001
```

`HandlerMapping` 会找到：

```shell
UserController.detail()
```

常见映射注解：

```shell
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

---

# 三、HandlerAdapter 处理器适配器

`HandlerAdapter` 负责真正调用 Controller 方法。

为什么需要适配器？

因为 Controller 方法可以有很多不同写法：

```java
public String list()

public UserDTO detail(Long id)

public ResponseEntity<UserDTO> detail(Long id)

public Result<UserDTO> detail(@RequestBody UserQuery query)
```

Spring MVC 要能统一调用这些不同方法，就需要适配器。

`HandlerAdapter` 主要负责：

```shell
解析方法参数
调用目标方法
处理方法返回值
```

---

# 四、参数解析器

Controller 方法中的参数，不是手动从 request 里取的，而是由 Spring MVC 参数解析器完成。

常见参数注解：

```shell
@PathVariable     获取路径参数
@RequestParam     获取 query/form 参数
@RequestBody      获取 JSON 请求体
@RequestHeader    获取请求头
@CookieValue      获取 Cookie
@ModelAttribute   绑定表单对象
```

示例：

```java
@PostMapping("/create")
public Result<Long> create(@RequestBody CreateUserRequest request,
                           @RequestHeader("X-Trace-Id") String traceId) {
    return Result.success(userService.create(request, traceId));
}
```

Spring MVC 会自动完成：

```shell
读取请求体
JSON 反序列化
参数绑定
类型转换
校验触发
```

---

# 五、HttpMessageConverter 消息转换器

前后端分离项目中，最常见的是 JSON 接口。

Spring MVC 通过 `HttpMessageConverter` 完成对象和 HTTP 内容之间的转换。

例如：

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public UserDTO user() {
        return new UserDTO(1L, "jarvan");
    }
}
```

返回对象后，Spring MVC 会通过消息转换器变成：

```json
{
  "id": 1,
  "name": "jarvan"
}
```

常见转换：

```shell
Java 对象 → JSON
JSON → Java 对象
String → text/plain
byte[] → 文件流
```

SpringBoot 默认集成 Jackson，所以大多数 JSON 接口不用手动配置。

---

# 六、ViewResolver 视图解析器

如果是传统 MVC 页面项目，会用到 `ViewResolver`。

例如：

```java
@Controller
public class PageController {

    @GetMapping("/index")
    public String index() {
        return "index";
    }
}
```

返回 `"index"` 后，视图解析器会找到对应页面。

例如 Thymeleaf：

```shell
resources/templates/index.html
```

前后端分离项目中，一般更多使用：

```shell
@RestController
@ResponseBody
```

直接返回 JSON。

---

# 常用注解

## 1、@Controller 和 @RestController

`@Controller` 适合页面返回。

`@RestController` 等于：

```shell
@Controller + @ResponseBody
```

也就是方法返回值直接写入 HTTP 响应体，一般用于 JSON API。

## 2、@RequestMapping

用于定义请求路径，可以写在类上，也可以写在方法上。

```java
@RestController
@RequestMapping("/api/order")
public class OrderController {

    @GetMapping("/{id}")
    public OrderDTO detail(@PathVariable Long id) {
        return orderService.detail(id);
    }
}
```

## 3、@RequestBody

接收 JSON 请求体。

```java
@PostMapping("/save")
public Result<Void> save(@RequestBody SaveOrderRequest request) {
    orderService.save(request);
    return Result.success();
}
```

## 4、@Valid

用于参数校验。

```java
@PostMapping("/save")
public Result<Void> save(@Valid @RequestBody SaveOrderRequest request) {
    orderService.save(request);
    return Result.success();
}
```

请求对象：

```java
public class SaveOrderRequest {

    @NotNull(message = "用户ID不能为空")
    private Long userId;

    @NotBlank(message = "商品名称不能为空")
    private String productName;
}
```

---

# SpringBoot MVC 常用配置

## 1、端口配置

```yml
server:
  port: 8080
```

## 2、上下文路径

```yml
server:
  servlet:
    context-path: /api
```

访问路径会变成：

```shell
http://localhost:8080/api/user/list
```

## 3、JSON 时间格式

```yml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: Asia/Shanghai
```

## 4、文件上传大小

```yml
spring:
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 50MB
```

## 5、静态资源路径

默认静态资源目录：

```shell
classpath:/static/
classpath:/public/
classpath:/resources/
classpath:/META-INF/resources/
```

例如：

```shell
src/main/resources/static/logo.png
```

访问：

```shell
http://localhost:8080/logo.png
```

---

# WebMvcConfigurer 扩展配置

SpringBoot 提供 `WebMvcConfigurer` 用来扩展 MVC 行为。

常见用途：

```shell
添加拦截器
配置跨域
配置静态资源
配置消息转换器
配置参数解析器
```

示例：

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/login");
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:5173")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowCredentials(true);
    }
}
```

注意：

```shell
一般使用 WebMvcConfigurer 扩展配置，不要直接继承 WebMvcConfigurationSupport。
```

因为继承 `WebMvcConfigurationSupport` 可能会让 SpringBoot 的 MVC 自动配置失效。

---

# 拦截器 Interceptor

拦截器用于在 Controller 执行前后做统一处理。

常见场景：

```shell
登录校验
权限判断
TraceId 设置
请求日志
接口耗时统计
防重复提交
```

执行流程：

```shell
preHandle       Controller 执行前
postHandle      Controller 执行后，视图渲染前
afterCompletion 请求完成后
```

示例：

```java
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) {
        String token = request.getHeader("Authorization");
        return token != null && token.startsWith("Bearer ");
    }
}
```

如果 `preHandle` 返回 `false`，请求不会继续进入 Controller。

---

# 统一异常处理

项目中不要在每个 Controller 里重复 try-catch。

推荐使用：

```shell
@RestControllerAdvice
@ExceptionHandler
```

示例：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValidException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().get(0).getDefaultMessage();
        return Result.fail("PARAM_ERROR", message);
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        return Result.fail("SYSTEM_ERROR", "系统异常");
    }
}
```

统一异常处理的好处：

- Controller 更干净。
- 错误返回格式统一。
- 参数校验错误统一处理。
- 系统异常不会直接暴露堆栈给前端。

---

# 统一返回结构

接口建议统一返回结构。

例如：

```java
public class Result<T> {
    private String code;
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.code = "SUCCESS";
        result.message = "成功";
        result.data = data;
        return result;
    }

    public static <T> Result<T> fail(String code, String message) {
        Result<T> result = new Result<>();
        result.code = code;
        result.message = message;
        return result;
    }
}
```

前端拿到的数据结构稳定，联调和排查都会更方便。

---

# 推荐项目分层

一个常见 SpringBoot MVC 项目可以这样组织：

```shell
com.example.demo
  ├── controller      接收 HTTP 请求
  ├── service         业务逻辑
  ├── service.impl    业务实现
  ├── mapper          数据访问
  ├── entity          数据库实体
  ├── dto             返回给前端的数据
  ├── request         请求参数对象
  ├── response        响应对象
  ├── config          配置类
  ├── interceptor     拦截器
  ├── exception       异常定义
  └── common          通用工具和返回结构
```

职责边界：

```shell
Controller 只做参数接收、校验触发、调用 Service、返回结果
Service 负责业务编排和事务控制
Mapper 只负责数据库访问
DTO / VO 负责返回数据
Request 负责接收请求参数
```

不要把 SQL、业务判断、远程调用全部堆在 Controller 里。

---

# SpringBoot MVC 的优点

## 1、开发效率高

SpringBoot 自动配置了大量 Web 基础能力。

例如：

```shell
内嵌 Tomcat
JSON 转换
参数绑定
静态资源
异常处理扩展
文件上传
校验框架
```

开发者只需要关注业务接口。

## 2、生态成熟

SpringBoot MVC 可以很好地和其它组件配合：

```shell
MyBatis / JPA
Redis
RabbitMQ / Kafka
Spring Security
Spring Cloud Gateway
Nacos
Sentinel
OpenFeign
Swagger / Knife4j
```

## 3、结构清晰

MVC 分层清楚，适合中后台系统、管理系统、微服务接口、开放 API。

## 4、扩展性强

可以通过：

```shell
Interceptor
Filter
ArgumentResolver
MessageConverter
ExceptionHandler
WebMvcConfigurer
```

扩展请求处理链路。

---

# Filter 和 Interceptor 的区别

Filter 是 Servlet 规范里的组件。

Interceptor 是 Spring MVC 提供的组件。

区别：

```shell
Filter 执行更早，在 DispatcherServlet 之前
Interceptor 在 DispatcherServlet 之后，Controller 之前
Filter 不依赖 Spring MVC
Interceptor 能拿到 HandlerMethod 等 MVC 信息
```

常见使用：

```shell
Filter：跨域、编码、链路 TraceId、请求包装
Interceptor：登录校验、权限控制、接口日志、业务拦截
```

执行顺序：

```shell
请求
  ↓
Filter
  ↓
DispatcherServlet
  ↓
Interceptor
  ↓
Controller
```

---

# 使用注意事项

## 1、Controller 不要写复杂业务

Controller 应该保持轻量：

```shell
接收参数
调用 Service
返回结果
```

复杂业务放到 Service。

## 2、参数一定要校验

外部请求不可信，必须校验：

```shell
必填字段
长度
格式
枚举范围
金额范围
时间范围
```

## 3、不要直接返回数据库 Entity

Entity 是数据库结构，不一定适合直接暴露给前端。

建议：

```shell
Entity → DTO / VO → 返回前端
```

避免字段泄露和接口结构被数据库表绑定。

## 4、接口要有统一错误码

不要随便返回字符串错误。

建议统一：

```shell
SUCCESS
PARAM_ERROR
AUTH_ERROR
PERMISSION_DENIED
BUSINESS_ERROR
SYSTEM_ERROR
```

## 5、注意 JSON 时间格式

时间字段要统一时区和格式。

否则前后端、数据库、日志中容易出现时间不一致。

## 6、文件上传要限制大小和类型

文件上传接口一定要限制：

```shell
文件大小
文件类型
文件后缀
存储路径
访问权限
```

不要让用户上传任意文件。

## 7、接口日志不要打印敏感信息

日志中不要直接打印：

```shell
密码
身份证
银行卡
Token
手机号完整值
支付密钥
```

敏感字段要脱敏。

## 8、跨域配置不要过度放开

开发环境可以宽松，生产环境要限制来源。

不要随意配置：

```shell
allowedOrigins("*")
allowCredentials(true)
```

这类组合容易带来安全风险。

---

# 总结

SpringBoot MVC 的核心，是围绕 `DispatcherServlet` 建立的一套请求处理机制。

它通过：

```shell
HandlerMapping
HandlerAdapter
参数解析器
返回值处理器
HttpMessageConverter
ViewResolver
Interceptor
ExceptionHandler
```

把 HTTP 请求转成 Controller 方法调用，再把方法结果转成 HTTP 响应。

项目实践中，建议坚持：

```shell
Controller 保持轻量
Service 承载业务
Mapper 只管数据
DTO 隔离返回结构
统一参数校验
统一异常处理
统一返回格式
统一日志和 TraceId
```

SpringBoot MVC 不是只会写 `@GetMapping` 和 `@PostMapping`，真正理解它的核心流程和扩展点后，才能写出清晰、稳定、可维护的 Web 服务。
