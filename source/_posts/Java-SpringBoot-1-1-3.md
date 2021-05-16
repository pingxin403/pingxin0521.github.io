---
title: Spring boot API 接口文档 Swagger
date: 2019-07-11 09:19:59
tags:
 - Java
 - 框架
categories:
 - Java
 - Spring boot
---

相信无论是前端还是后端开发，都或多或少地被接口文档折磨过。前端经常抱怨后端给的接口文档与实际情况不一致。后端又觉得编写及维护接口文档会耗费不少精力，经常来不及更新。其实无论是前端调用后端，还是后端调用后端，都期望有一个好的接口文档。但是这个接口文档对于程序员来说，就跟注释一样，经常会抱怨别人写的代码没有写注释，然而自己写起代码起来，最讨厌的，也是写注释。所以仅仅只通过强制来规范大家是不够的，随着时间推移，版本迭代，接口文档往往很容易就跟不上代码了。

<!--more-->

**Swagger是什么？它能干什么？**

发现了痛点就要去找解决方案。解决方案用的人多了，就成了标准的规范，这就是Swagger的由来。通过这套规范，你只需要按照它的规范去定义接口及接口相关的信息。再通过Swagger衍生出来的一系列项目和工具，就可以做到生成各种格式的接口文档，生成多种语言的客户端和服务端的代码，以及在线接口调试页面等等。这样，如果按照新的开发模式，在开发新版本或者迭代版本的时候，只需要更新Swagger描述文件，就可以自动生成接口文档和客户端服务端代码，做到调用端代码、服务端代码以及接口文档的一致性。

但即便如此，对于许多开发来说，编写这个yml或json格式的描述文件，本身也是有一定负担的工作，特别是在后面持续迭代开发的时候，往往会忽略更新这个描述文件，直接更改代码。久而久之，这个描述文件也和实际项目渐行渐远，基于该描述文件生成的接口文档也失去了参考意义。所以作为Java届服务端的大一统框架Spring，迅速将Swagger规范纳入自身的标准，建立了Spring-swagger项目，后面改成了现在的Springfox。通过在项目中引入Springfox，可以扫描相关的代码，生成该描述文件，进而生成与代码一致的接口文档和客户端代码。这种通过代码生成接口文档的形式，在后面需求持续迭代的项目中，显得尤为重要和高效。

#### 框架说明及使用

现在SWAGGER官网主要提供了几种开源工具，提供相应的功能。可以通过配置甚至是修改源码以达到你想要的效果。

**Swagger Codegen**: 通过Codegen 可以将描述文件生成html格式和cwiki形式的接口文档，同时也能生成多钟语言的服务端和客户端的代码。支持通过jar包，docker，node等方式在本地化执行生成。也可以在后面的Swagger Editor中在线生成。

**Swagger UI**:提供了一个可视化的UI页面展示描述文件。接口的调用方、测试、项目经理等都可以在该页面中对相关接口进行查阅和做一些简单的接口请求。该项目支持在线导入描述文件和本地部署UI项目。

**Swagger Editor**: 类似于markendown编辑器的编辑Swagger描述文件的编辑器，该编辑支持实时预览描述文件的更新效果。也提供了在线编辑器和本地部署编辑器两种方式。

**Swagger Inspector**: 感觉和postman差不多，是一个可以对接口进行测试的在线版的postman。比在Swagger UI里面做接口请求，会返回更多的信息，也会保存你请求的实际请求参数等数据。

**Swagger Hub**：集成了上面所有项目的各个功能，你可以以项目和版本为单位，将你的描述文件上传到Swagger Hub中。在Swagger Hub中可以完成上面项目的所有工作，需要注册账号，分免费版和收费版。

**Springfox Swagger**: Spring 基于swagger规范，可以将基于SpringMVC和Spring Boot项目的项目代码，自动生成JSON格式的描述文件。本身不是属于Swagger官网提供的，在这里列出来做个说明，方便后面作一个使用的展开。

#### 基于Spring框架的Swagger流程应用

这里讲的是如何结合现有的工具和功能，设计一个流程，去保证一个项目从开始开发到后面持续迭代的时候，以最小代价去维护代码、接口文档以及Swagger描述文件。

**项目开始阶段**

一般来说，接口文档都是由服务端来编写的。在项目开发阶段的时候，服务端开发可以视情况来决定是直接编写服务端调用层代码，还是写Swagger描述文件。建议是如果项目启动阶段，就已经搭好了后台框架，那可以直接编写服务端被调用层的代码（即controller及其入参出参对象），然后通过Springfox－swagger 生成swagger json描述文件。如果项目启动阶段并没有相关后台框架，而前端对接口文档追得紧，那就建议先编写swagger描述文件，通过该描述文件生成接口文档。后续后台框架搭好了，也可以生成相关的服务端代码。

**项目迭代阶段**

到这个阶段，事情就简单很多了。后续后台人员，无需关注Swagger描述文件和接口文档，有需求变更导致接口变化，直接写代码就好了。把调用层的代码做个修改，然后生成新的描述文件和接口文档后，给到前端即可。真正做到了一劳永逸。

**流程**

总结一下就是通过下面这两种流程中的一种，可以做到代码和接口文档的一致性，服务端开发再也不用花费精力去维护接口文档。

1. 流程一

   ![QzU3jJ.png](https://s2.ax1x.com/2019/12/22/QzU3jJ.png)

2. 流程二

   ![QzUGu9.png](https://s2.ax1x.com/2019/12/22/QzUGu9.png)

#### 示例

在 pom.xml 文件中，引入相关依赖

```xml
  <dependencies>
    ...
    <!-- swagger -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.5.0</version>
    </dependency>
    <!-- swagger-ui -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.5.0</version>
    </dependency>
  </dependencies>
```

然后进行配置

```java
@Configuration
@EnableSwagger2 // 标记项目启用 Swagger API 接口文档
public class SwaggerConfiguration {

    @Bean
    public Docket createRestApi() {
        // 创建 Docket 对象
        return new Docket(DocumentationType.SWAGGER_2) // 文档类型，使用 Swagger2
                .apiInfo(this.apiInfo()) // 设置 API 信息
                // 扫描 Controller 包路径，获得 API 接口
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.hyp.learn.w.controller"))
                .paths(PathSelectors.any())
                // 构建出 Docket 对象
                .build();
    }

    /**
     * 创建 API 信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试接口文档示例")
                .description("我是一段描述")
                .version("1.0.0") // 版本号
                .contact(new Contact("平心", "http://hanyunpeng0521.github.io", "m13839441583@163.com")) // 联系人
                .build();
    }

}
```

UserController

```java
@RestController
@RequestMapping("/users")
@Api(tags = "用户 API 接口")
public class UserController {

    @GetMapping("/list")
    @ApiOperation(value = "查询用户列表", notes = "目前仅仅是作为测试，所以返回用户全列表")
    public List<UserVO> list() {
        // 查询列表
        List<UserVO> result = new ArrayList<>();
        result.add(new UserVO().setId(1).setName("yudaoyuanma"));
        result.add(new UserVO().setId(2).setName("woshiyutou"));
        result.add(new UserVO().setId(3).setName("chifanshuijiao"));
        // 返回列表
        return result;
    }

    @GetMapping("/get")
    @ApiOperation("获得指定用户编号的用户")
    @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
    public UserVO get(@RequestParam("id") Integer id) {
        // 查询并返回用户
        return new UserVO().setId(id).setName(UUID.randomUUID().toString());
    }

    @PostMapping("add")
    @ApiOperation("添加用户")
    public Integer add(UserAddDTO addDTO) {
        // 插入用户记录，返回编号
        Integer returnId = UUID.randomUUID().hashCode();
        // 返回用户编号
        return returnId;
    }

    @PostMapping("/update")
    @ApiOperation("更新指定用户编号的用户")
    public Boolean update(UserUpdateDTO updateDTO) {
        // 更新用户记录
        Boolean success = true;
        // 返回更新是否成功
        return success;
    }

    @PostMapping("/delete")
    @ApiOperation(value = "删除指定用户编号的用户")
    @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
    public Boolean delete(@RequestParam("id") Integer id) {
        // 删除用户记录
        Boolean success = false;
        // 返回是否更新成功
        return success;
    }
}
```

相比我们之前使用 SpringMVC 来说，我们在类和接口上，额外增加了 Swagger 提供的注解。

因为已经使用了 Swagger 的注解，所以类和方法上的注释，一般可以删除了，除非有特殊诉求。

执行 Application 启动项目。然后浏览器访问 `http://127.0.0.1:8080/web/swagger-ui.html` 地址，就可以看到 Swagger 生成的 API 接口文档。

**建议**

- controller方法指定请求方法，不然swagger会默认创建所有请求方法的接口，会造成文档可读性差
- 同名问题：@Api(同名的问题) 因为swagger会根据tags 的名称查找对象，有同名对象的时候，swagger的文档就会出现问题。如果swagger的某个API下出现不属于该API的请求，这个就是API的同名的问题，查找相同的API名称替换即可
- 类上的注解“/”的问题：@ApiModel(不能使用“/”)
- 使用map作为返回类型报错：升级到2.8

#### 注解

**@Api**

@Api 注解，添加在 Controller 类上，标记它作为 Swagger 文档资源。

`@Api` 注解的**常用属性**，如下：

- tags属性：用于控制 API 所属的标签列表。[]数组，可以填写多个。
  - 可以在**一个** Controller 上的 `@Api` 的 `tags` 属性，设置**多个**标签，那么这个 Controller 下的 API 接口，就会出现在这**两个**标签中。
  - 如果在**多个** Controller 上的 `@Api` 的 `tags` 属性，设置**一个**标签，那么这些 Controller 下的 API 接口，仅会出现在这**一个**标签中。
  - 本质上，`tags` 就是为了分组 API 接口，和 Controller 本质上是一个目的。所以绝大数场景下，我们只会给一个 Controller 一个**唯一**的标签。例如说，UserController 的 `tags` 设置为 `"用户 API 接口"` 。

`@Api` 注解的**不常用属性**，如下：

- `produces` 属性：请求请求头的**可接受类型**( [Accept](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept) )。如果有多个，使用 `,` 分隔。
- `consumes` 属性：请求请求头的**提交内容类型**( [Content-Type](https://juejin.im/post/5cb34fc06fb9a068a75d3555) )。如果有多个，使用 `,` 分隔。
- `protocols` 属性：协议，可选值为 `"http"`、`"https"`、`"ws"`、`"wss"` 。如果有多个，使用 `,` 分隔。
- `authorizations` 属性：授权相关的配置，`[]` 数组，使用 [`@Authorization`](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/Authorization.java) 注解。
- `hidden` 属性：是否隐藏，不再 API 接口文档中显示。

`@Api` 注解的**废弃属性**，不建议使用，有 `value`、`description`、`basePath`、`position` 。

**@ApiOperation**

@ApiOperation 注解，添加在 Controller 方法上，标记它是一个 API 操作。

`@ApiOperation` 注解的**常用属性**，如下：

- `value` 属性：API 操作名。
- `notes` 属性：API 操作的描述。

`@ApiOperation` 注解的**不常用属性**，如下：

- `tags` 属性：和 `@API` 注解的 `tags` 属性一致。
- `nickname` 属性：API 操作接口的唯一标识，主要用于和第三方工具做对接。
- `httpMethod` 属性：请求方法，可选值为 `GET`、`HEAD`、`POST`、`PUT`、`DELETE`、`OPTIONS`、`PATCH` 。因为 Swagger 会解析 SpringMVC 的注解，所以一般无需填写。
- `produces` 属性：和 `@API` 注解的 `produces` 属性一致。
- `consumes` 属性：和 `@API` 注解的 `consumes` 属性一致。
- `protocols` 属性：和 `@API` 注解的 `protocols` 属性一致。
- `authorizations` 属性：和 `@API` 注解的 `authorizations` 属性一致。
- `hidden` 属性：和 `@API` 注解的 `hidden` 属性一致。
- `response` 属性：响应结果类型。因为 Swagger 会解析方法的返回类型，所以一般无需填写。
- `responseContainer` 属性：响应结果的容器，可选值为 `List`、`Set`、`Map` 。
- `responseReference` 属性：指定对响应类型的引用。这个引用可以是本地，也可以是远程。并且，当设置了它时，会覆盖 `response` 属性。说人话，就是可以忽略这个属性，哈哈哈。
- `responseHeaders` 属性：响应头，`[]` 数组，使用 [`@ResponseHeader`](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/ResponseHeader.java) 注解。
- `code` 属性：响应状态码，默认为 200 。
- `extensions` 属性：拓展属性，`[]` 属性，使用 [`@Extension`](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/Extension.java) 注解。
- `ignoreJsonView` 属性：在解析操作和类型，忽略 JsonView 注释。主要是为了向后兼容。

`@ApiOperation` 注解的**废弃属性**，不建议使用，有 `position` 。

**@ApiImplicitParam**

@ApiImplicitParam 注解，添加在 Controller 方法上，声明每个请求参数的信息。

`@ApiImplicitParam` 注解的**常用属性**，如下：

- `name` 属性：参数名。
- `value` 属性：参数的简要说明。
- `required` 属性：是否为必传参数。默认为 `false` 。
- `dataType` 属性：数据类型，通过字符串 String 定义。
- `dataTypeClass` 属性：数据类型，通过 `dataTypeClass` 定义。在设置了 `dataTypeClass` 属性的情况下，会覆盖 `dataType` 属性。**推荐采用这个方式**。
- paramType属性：参数所在位置的类型。有如下 5 种方式：
  - `"path"` 值：对应 SpringMVC 的 `@PathVariable` 注解。
  - 【*默认值*】`"query"` 值：对应 SpringMVC 的 `@PathVariable` 注解。
  - `"body"` 值：对应 SpringMVC 的 `@RequestBody` 注解。
  - `"header"` 值：对应 SpringMVC 的 `@RequestHeader` 注解。
  - `"form"` 值：Form 表单提交，对应 SpringMVC 的 `@PathVariable` 注解。
  - 😈 **绝大多数情况下，使用 `"query"` 值这个类型即可。**
- `example` 属性：参数值的简单示例。
- `examples` 属性：参数值的复杂示例，使用 [`@Example`](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/Example.java) 注解。

`@ApiImplicitParam` 注解的**不常用属性**，如下：

- `defaultValue` 属性：默认值。
- allowableValues属性：允许的值。如果要设置多个值，有两种方式：
  - 数组方式，即 `{value1, value2, value3}` 。例如说，`{1, 2, 3}` 。
  - 范围方式，即 `[value1, value2]` 或 `[value1, value2)` 。例如说 `[1, 5]` 表示 1 到 5 的所有数字。如果有无穷的，可以使用 `(-infinity, value2]` 或 `[value1, infinity)` 。
- `allowEmptyValue` 属性：是否允许空值。
- `allowMultiple` 属性：是否允许通过多次传递该参数来接受多个值。默认为 `false` 。
- `type` 属性：搞不懂具体用途，对应英文注释为 `Adds the ability to override the detected type` 。
- `readOnly` 属性：是否只读。
- `format` 属性：自定义的格式化。
- `collectionFormat` 属性：针对 Collection 集合的，自定义的格式化。

当我们需要添加在方法上添加多个 `@ApiImplicitParam` 注解时，可以使用 [@ApiImplicitParams](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/ApiImplicitParams.java) 注解中添加多个。示例如下：

```java
@ApiImplicitParams({ // 参数数组
        @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024"), // 参数一
        @ApiImplicitParam(name = "name", value = "昵称", paramType = "query", dataTypeClass = String.class, required = true, example = "芋道"), // 参数二
})
```

**@ApiModel**

@ApiModel 注解，添加在 POJO 类，声明 POJO 类的信息。而在 Swagger 中，把这种 POJO 类称为 Model 类。

`@ApiModel` 注解的**常用属性**，如下：

- `value` 属性：Model 名字。
- `description` 属性：Model 描述。

`@ApiModel` 注解的**不常用属性**，如下：

- `parent` 属性：指定该 Model 的父 Class 类，用于继承父 Class 的 Swagger 信息。
- `subTypes` 属性：定义该 Model 类的子类 Class 们。

**@ApiModelProperty**

@ApiModelProperty 注解，添加在 Model 类的成员变量上，声明每个成员变量的信息。

`@ApiModelProperty` 注解的**常用属性**，如下：

- `value` 属性：属性的描述。
- `dataType` 属性：和 `@ApiImplicitParam` 注解的 `dataType` 属性一致。不过因为 `@ApiModelProperty` 是添加在成员变量上，可以自动获得成员变量的类型。
- `required` 属性：和 `@ApiImplicitParam` 注解的 `required` 属性一致。
- `example` 属性：`@ApiImplicitParam` 注解的 `example` 属性一致。

`@ApiModelProperty` 注解的**不常用属性**，如下：

- `name` 属性：覆盖成员变量的名字，使用该属性进行自定义。
- `allowableValues` 属性：和 `@ApiImplicitParam` 注解的 `allowableValues` 属性一致。
- `position` 属性：成员变量排序位置，默认为 0 。
- `hidden` 属性：`@ApiImplicitParam` 注解的 `hidden` 属性一致。
- `accessMode` 属性：访问模式，有 `AccessMode.AUTO`、`AccessMode.READ_ONLY`、`AccessMode.READ_WRITE` 三种，默认为 `AccessMode.AUTO` 。
- `reference` 属性：和 `@ApiModel` 注解的 `reference` 属性一致。
- `allowEmptyValue` 属性：和 `@ApiImplicitParam` 注解的 `allowEmptyValue` 属性一致。
- `extensions` 属性：和 `@ApiImplicitParam` 注解的 `extensions` 属性一致。

`@ApiModelProperty` 注解的**废弃属性**，不建议使用，有 `readOnly` 。

**@ApiResponse**

在大多数情况下，我们并不需要使用 `@ApiResponse` 注解，因为我们会类似 `UserController#get(id)` 方法这个接口，返回一个 Model 即可

[`@ApiResponse`](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/ApiResponse.java) 注解，添加在 Controller 类的方法上，声明每个响应参数的信息。

`@ApiResponse` 注解的属性，基本已经被 `@ApiOperation` 注解所覆盖，如下：

- `message` 属性：响应的提示内容。
- `code` 属性：和 `@ApiOperation` 注解的 `code` 属性一致。
- `response` 属性：和 `@ApiOperation` 注解的 `response` 属性一致。
- `reference` 属性：和 `@ApiOperation` 注解的 `responseReference` 属性一致。
- `responseHeaders` 属性：和 `@ApiOperation` 注解的 `responseHeaders` 属性一致。
- `responseContainer` 属性：和 `@ApiOperation` 注解的 `responseContainer` 属性一致。
- `examples` 属性：和 `@ApiOperation` 注解的 `examples` 属性一致。

当我们需要添加在方法上添加多个 `@ApiResponse` 注解时，可以使用 [@ApiResponses](https://github.com/swagger-api/swagger-core/blob/1.5/modules/swagger-annotations/src/main/java/io/swagger/annotations/ApiResponse.java) 注解中添加多个。

#### 测试接口

在 Swagger 的 UI 界面上，提供了简单的测试接口的工具。我们仅仅需要点开某个接口，点击「Try it out」按钮。

Swagger 给我们提供了：

- 提供了 [curl](https://curl.haxx.se/) 命令，让我们可以直接在命令行执行。
- 提供了 Request URL 地址，方便我们在浏览器中访问。
- 提供了执行结果，我们可以人肉看看，是否符合我们希望的结果。

#### 更好看的 Swagger UI 界面

[knife4j](https://doc.xiaominfo.com)是为Java MVC框架集成Swagger生成Api文档的增强解决方案

swagger-bootstrap-ui的最后一个版本是1.9.6,已更名为knife4j

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案,前身是`swagger-bootstrap-ui`,取名knife4j是希望她能像一把匕首一样小巧,轻量,并且功能强悍!

后续版本请使用knife4j,关于knife4j的使用方法请[参考文档](https://doc.xiaominfo.com/knife4j/)

使用knife4j的引用配置如下：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <!--在引用时请在maven中央仓库搜索最新版本号-->
    <version>2.0.2</version>
</dependency>
```

`knife4j-spring-boot-starter`主要为我们引用的相关jar包：

- knife4j-spring:Swagger增强处理类
- knife4j-spring-ui:swagger的增强ui文档
- springfox-swagger:springfox最新2.9.2版本
- springfox-swagger-ui:springfox提供的ui
- springfox-bean-validators：springfxo验证支持组件

此时,位于包路径`com.github.xiaoymin.knife4j.spring.configuration.Knife4jAutoConfiguration.java`

类会为我们开启Swagger的增强注解,您只需要在项目中创建Swagger的Docket对象即可

```java
@Configuration
@EnableSwagger2
@EnableKnife4j
@Import(BeanValidatorPluginsConfiguration.class)
public class SwaggerConfiguration {
    
}
```

项目可以参考示例[knife4j-spring-boot-single-demo](https://gitee.com/xiaoym/swagger-bootstrap-ui-demo/tree/master/knife4j-spring-boot-single-demo)

#### 更强大的 YApi

虽然说 Swagger 已经挺强大了，可以很好的完成提供后端 API 接口文档的功能，但是实际场景下，我们还是会碰到很多问题：

- Swagger 没有内置 Mock 功能。在实际的开发中，在后端定义好 API 接口之后，前端会根据 API 接口，进行接口的 Mock ，从而实现前后端的并行开发。
- 多个项目的 API 接口文档的整合。随着微服务的流行，一个产品实际是拆分成多个微服务项目，提供 API 接口。那么，一个微服务项目，一个接口文档，肯定会气死前端。气死一个前端小哥哥没事，如果是小姐姐那多可惜啊。

所以，我们需要更加强大的 API 接口管理平台。目前艿艿团队采用的解决方案是：

- 后端开发，还是使用 Swagger 注解，生成 API 接口文档。
- 使用 [YApi](https://github.com/YMFE/yapi) 可视化接口管理平台，自动调用 Swagger 提供的 `v2/api-docs` 接口，采集 Swagger 注解生成的 API 接口信息，从而录入到 YApi 中。

YApi 是**高效**、**易用**、**功能强大**的 api 管理平台，旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API，YApi 还为用户提供了优秀的交互体验，开发人员只需利用平台提供的接口数据写入工具以及简单的点击操作就可以实现接口的管理。

- 基于 Json5 和 Mockjs 定义接口返回数据的结构和文档，效率提升多倍
- 扁平化权限设计，即保证了大型企业级项目的管理，又保证了易用性
- 类似 postman 的接口调试
- 自动化测试, 支持对 Response 断言
- MockServer 除支持普通的随机 mock 外，还增加了 Mock 期望功能，根据设置的请求过滤规则，返回期望数据
- 支持 postman, har, swagger 数据导入
- 免费开源，内网部署，信息再也不怕泄露了

可以访问 http://yapi.demo.qunar.com/ 地址，快速体验下 Yapi 的功能。

**使用 Docker 构建 Yapi**

1. 启动 MongoDB

   ```shell
   docker run -d --name mongo-yapi mongo
   ```

2. 获取 Yapi 镜像，版本信息可在 [阿里云镜像仓库](https://links.jianshu.com/go?to=https%3A%2F%2Fdev.aliyun.com%2Fdetail.html%3Fspm%3D5176.1972343.2.26.I97LV8%26repoId%3D139034) 查看

   ```shell
   docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
   ```

3. 初始化 Yapi 数据库索引及管理员账号

   ```shell
   docker run -it --rm \
     --link mongo-yapi:mongo \
     --entrypoint npm \
     --workdir /api/vendors \
     registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
     run install-server
   ```

   > 自定义配置文件挂载到目录 `/api/config.json`，官方自定义配置文件 -> [传送门](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FYMFE%2Fyapi%2Fblob%2Fmaster%2Fconfig_example.json)

4. 启动 Yapi 服务

   ```shell
   docker run -d \
     --name yapi \
     --link mongo-yapi:mongo \
     --workdir /api/vendors \
     -p 3000:3000 \
     registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
     server/app.js
   ```

5. 使用 Yapi

   访问 [http://localhost:3000](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A3000)   登录账号 **[admin@admin.com](https://links.jianshu.com/go?to=mailto%3Aadmin%40admin.com)**，密码 **ymfe.org**

至此，帅气的 Yapi 就可以轻松使用啦！更多文档信息，请参考

- [Yapi 官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fyapi.ymfe.org%2Fdocuments%2Findex.html)
- [Yapi 版本更新记录](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FYMFE%2Fyapi%2Fblob%2Fmaster%2FCHANGELOG.md)

### 参考

1. [顶尖 API 文档管理工具 (Yapi)](https://www.jianshu.com/p/a97d2efb23c5)