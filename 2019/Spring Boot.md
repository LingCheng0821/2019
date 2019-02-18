Spring Boot

## 四、 MVC 自动配置

Web开发的自动配置类为：**WebMvcAutoConfiguration**

对应的配置类：**ResourceProperties** 

```java
// xxxxAutoConfiguration
		1.自动配置类,帮我们给容器中自动配置组件；
		2.其中有大量的 @EnableConfigurationProperties(xxxxProperties.class)
// xxxxProperties
        1. 配置类来封装配置文件的内容；
```

###1.  官方文档

https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-spring-mvc

Spring Boot 自动配置好了MVC，以下是SpringBoot对SpringMVC的默认配置：

(1)  包含 `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans；

- 自动配置了 ViewResolver【视图解析器：根据方法的返回值得到视图对象】；
- ContentNegotiatingViewResolver：组合所有的视图解析器；
- 如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来；

```java
// main方法类中，定制 视图解析器
@Bean
public ViewResolver myViewReslover(){
    return new MyViewReslover();
}
public static class MyViewReslover implements ViewResolver{
    @Override
    public View resolveViewName(String viewName, Locale locale) throws Exception {
        return null;
    }
}
```

(2) 静态资源文件夹路径 和 webjars的支持；

(3) Static `index.html` support：静态首页访问；

(4) Custom `Favicon` support：图标 favicon.ico

(5) 自动注册 `Converter`, `GenericConverter`, and `Formatter` beans.

- Converter：转换器。例如页面转入 年龄为文本类型的18，封装bean的时候，类型转换为 int。
- `Formatter` ：格式化器。例如：2019-01-01 转换为 Date。

(6)  `HttpMessageConverters` 

- HttpMessageConverter：SpringMVC用来转换Http请求和响应的；User---Json。

(7) 自动注册 `MessageCodesResolver`

- 定义错误代码生成规则。

(8) 自动使用 `ConfigurableWebBindingInitializer`

- 初始化WebDataBinder，即 请求数据=====JavaBean。



### 2. 获取静态资源方法

添加资源映射源码：

```java
// WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#addResourceHandlers
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
	... ...
	if (!registry.hasMappingForPattern("/webjars/**")) {
		customizeResourceHandlerRegistration(registry
			.addResourceHandler("/webjars/**")
			.addResourceLocations("classpath:/META-INF/resources/webjars/")
			...;
	}
    //静态资源文件夹映射                                  
	String staticPathPattern = this.mvcProperties.getStaticPathPattern();   // /**
	if (!registry.hasMappingForPattern(staticPathPattern)) {
		customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
			.addResourceLocations(getResourceLocations(
						this.resourceProperties.getStaticLocations()))
			.setCachePeriod(getSeconds(cachePeriod))
			.setCacheControl(cacheControl));
	}
}
```

#### 2.1  /webjars/\**

webjars：以jar包的方式引入静态资源；

所有webjars方式导进来 **classpath:/META-INF/resources/webjars/** 的资源， 以“  /webjars/\**  **  ”访问；

例如：localhost:8080/webjars/jquery/3.3.1/jquery.js

```xml
<dependency>                                   <!--【webjars：http://www.webjars.org/】-->
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
```

#### 2.2   /**

以下路径的资源，都可以以 "   **/****   " 访问；

```xml
<!--ResourceProperties 类中配置，可以修改配置文件的路径：spring.resources.static-locations-->
classpath:/META‐INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
/                                               <!--当前项目的根路径-->
```

例如：ocalhost:8080/js/Chart.min.js

  #### 2.3 /

**(1) 欢迎页**

静态资源文件下的 **index.html** 页面，被"  **/****  "映射；

例如：localhost:8080/

```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext app) {
	... ...
}
```

**(2)  图标**

静态资源文件下的 favicon.ico 图标，被"  **/****  "映射；



### 3. 扩展配置

```xml
<mvc:view‐controller path="/hello" view‐name="success"/>        <!--MVC-->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 3.1 方法

编写一个配置类**@Configuration**，是 **WebMvcConfigurerAdapte** 类型；不能标注**@EnableWebMvc**

```java
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {         
         //浏览器发送 /soufun 请求来到 success        
         registry.addViewController("/soufun").setViewName("success");
    }   
}
```

#### 3.2 实现原理

(1) WebMvcAutoConfiguration是SpringMVC的自动配置类；

(2) 在做其他自动配置时会导入 **@Import(EnableWebMvcConfiguration.class)；**

```java
@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration 
```

(3)  DelegatingWebMvcConfiguration：从容器中获取所有的 WebMvcConfigurer；

```java
@Autowired(required = false)
public void setConfigurers(List<WebMvcConfigurer> configurers) {
	if (!CollectionUtils.isEmpty(configurers)) {
		this.configurers.addWebMvcConfigurers(configurers);
	}
}
```

(4) 容器中所有的WebMvcConfigurer都会一起起作用，我们的配置类也会被调用。

### 4. 不使用自动配置

如果添加 注解 **@EnableWebMvc，SpringBoot对SpringMVC的自动配置失效**，所有都是我们自己配置。

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@EnableWebMvc
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
   @Override
   public void addViewControllers(ViewControllerRegistry registry) {
      registry.addViewController("/atguigu").setViewName("success");
   }
}
```

#### 4.1 为什么会失效？

(1) @EnableWebMvc：导入了组件 DelegatingWebMvcConfiguration

```java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc 
```

(2)  DelegatingWebMvcConfiguration：是一个 WebMvcConfigurationSupport

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

(3)  WebMvcAutoConfiguration 在容器中有 WebMvcConfigurationSupport时，自动配置类失效

```java
//容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
public class WebMvcAutoConfiguration 
```

### 5. 修改默认配置

(1) SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如
果有就用用户配置的，如果没有，才自动配置；

(2) 如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

(3) 在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置；

(4) 在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置。



## 五、模板引擎-thymeleaf 

JSP、Velocity、Freemarker、Thymeleaf。

SpringBoot 推荐的**Thymeleaf**，英[taim li:f]， like time leaf

### 1. 引入

``` xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring‐boot‐starter‐thymeleaf</artifactId> <!--默认版本：2.1.6-->
</dependency>

<!--切换thymeleaf版本,参考官方文档-->
<properties>
      <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
      <!--布局功能的支持程序 thymeleaf3主程序 layout2以上版本-->
      <thymeleaf‐layout‐dialect.version>2.2.2</thymeleaf‐layout‐dialect.version>
</properties>
```

### 2. 用法

只要我们把HTML页面放在 **classpath:/templates/**，thymeleaf就能自动渲染；

```java
private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
public static final String DEFAULT_PREFIX = "classpath:/templates/";
public static final String DEFAULT_SUFFIX = ".html";
```

(1)  导入thymeleaf的名称空间

```xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

(2) 使用 thymeleaf语法

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Spring Boot</title>
</head>
<body>
    <h1>Welcome to Spring Boot Page!!!</h1>
    <div th:text="${hello}">这是显示欢迎信息</div>    <!-- th:text 将div里面的文本内容替换 ‐‐>
</body>
</html>
```

### 3. 语法

| Feature                         | Attributes                               | 对应jsp标签     |
| ------------------------------- | ---------------------------------------- | ----------- |
| Fragment inclusion              | th:insert <br> th:replace                | jsp:include |
| Fragment iteration              | th:each                                  | c:forEach   |
| Conditional evaluation          | th:if <br> th:unless <br> th:switch <br> th:case | c:if        |
| Local variable definition       | th:object <br> th:with                   | c:set       |
| General attribute modification  | th:attr <br> th:attrprepend <br> th:attrappend |             |
| Specific attribute modification | th:value <br> th:href <br> th:src        | 任意属性修改      |
| Text (tag body modification)    | th:text <br> th:utext                    | 修改标签体内容     |
| Fragment specification          | th:fragment                              |             |
| Fragment removal                | th:remove                                |             |

### 4. 表达式

```properties
(1) Simple expressions:（表达式语法）
(2) Literals:（字面量）
(3) Text operations:（文本操作）
(4) Arithmetic operations:（数学运算）
(5) Boolean operations:（布尔运算）
(6) Comparisons and equality:（比较运算）
(7) Conditional operators:条件运算（三元运算符）
(8) Special tokens
```

#### 4.1  Simple expressions

```properties
(1) Variable Expressions: ${...}                 ## 获取变量值
(2) Selection Variable Expressions: *{...}       ## 选择表达式，同 ${}，还可以 配合 th:object 使用
(3) Message Expressions: #{...}                  ## 获取国际化内容
(4) Link URL Expressions: @{...}                 ## 定义URL
(5) Fragment Expressions: ~{...}                 ## 
```

-  获取变量值：**${...}**

  -  获取对象的属性、调用方法
  -  使用内置的基本对象：#ctx,#vars,#locale,#request,#response,#session,#servletContext
  -  内置的一些工具对象

  ```properties
  ${#ctx.session}
  ${#request.getAttribute('foo')
  ${#session.getAttribute('foo')}
  ${#servletContext.getAttribute('foo')}
  ${#request.getContextPath()}
  ${#request.getRequestName()} 
  ```

- 选择表达式：***{...}**

       ​```html
   <div th:object="${session.user}">
     <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
   </div>
   等价于
   <div>
     <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
   </div
       ​```

### 5. 实例

#### 5.1 直接返回字符串

```java
@Controller
public class HelloController {
  @RequestMapping("/hello")
  @ResponseBody
  public String sayHello(){
    return "hello world";
  }
}
```

访问：http://localhost:8080/hello，页面显示 hello world。

#### 5.2 返回page页

```java
@Controller
public class HelloController {
  @RequestMapping("/success")
  public String success(){
    return "success";
  }
}
```

返回 classpath:/templates/success.html

#### 5.3 获取值

(1) 获取属性值

```java
@RequestMapping("/success")
public String success(Map<String,Object> map){
    map.put("success","成功了！！");
    return "success";
}
```

```html
<div th:text="${success}"></div>
<!-- 用th:text不会解析html，  用th:utext会解析html，在页面中显示相应的样式 -->
```

(2) 获取数组

```java
map.put("users",Arrays.asList("zhangsan","lisi","wangwu"));
```

```html
<tr th:each="user : ${users}">
    <td th:text="${user}"></td>
 </tr>
```



## 六、CRUD

### 1. 默认访问首页

```java
@Configuration
public class MyConfig implements WebMvcConfigurer {
   // 第一种方法：重写方法 addViewControllers
   @Override
   public void addViewControllers(ViewControllerRegistry registry) {
      registry.addViewController("/").setViewName("login");
   }
  
   // 第二种方法：在容器中注册一个 WebMvcConfigurer
   @Bean 
   WebMvcConfigurer get(){
      WebMvcConfigurer web = new WebMvcConfigurer(){
         @Override
         public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/index").setViewName("login");
         }
      };
      return web;
   }
}
```

### 2. 修改资源路径

修改 webjars 样式：

```html
<link href="asserts/css/bootstrap.min.css"  th:href="@{/webjars/bootstrap/4.2.1/css/bootstrap.css}" >
```

修改本地资源：

```html
<link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
```



### 3. 国际化

#### 3.1 实现方法

- mvc做法

  (1) 编写国际化配置文件；

  (2) 使用ResourceBundleMessageSource管理国际化资源文件；

  (3) 在页面使用fmt:message取出国际化内容；

- spring boot

  (1) 编写国际化配置文件，抽取页面需要显示的国际化消息；

  在类路径下，新建文件夹：i18n，然后 NEW->Resource Bundle，书写三个文件，

```properties
## login; login_zh_CN; login_en_US
login.tip=请登录~
login.password=密码~
login.rem=记住我~
login.username=用户名~
login.btn=登录~
```

​	(2) 在配置文件中，配置 basename

```properties
spring.messages.basename=i18n/login
```

​	(3) 去页面获取国际化的值

```html
<h1 class="h3 mb‐3 font‐weight‐normal" th:text="#{login.tip}">Please sign in</h1>

<label class="sr‐only" th:text="#{login.username}">Username</label>

<label class="sr‐only" th:text="#{login.password}">Password</label>

<!--行内表达式：-->
<input type="checkbox" value="remember‐me"/> [[#{login.remember}]] 

<button class="btn btn‐lg btn‐primary btn‐block" type="submit" th:text="#{login.btn}">Sign in</button>
```



#### 3.2 实现原理

自动配置类：**MessageSourceAutoConfiguration**.java

注入了类：MessageSource

```java
// 1. 可以在配置文件中进行配置
// private String basename = "messages";
// 我们的配置文件可以直接放在类路径下叫messages.properties； 
@Bean
@ConfigurationProperties(prefix = "spring.messages")
public MessageSourceProperties messageSourceProperties() 
  
 
// 2. 注入了 MessageSource
// 设置国际化资源文件的基础名（去掉语言国家代码的）
@Bean
public MessageSource messageSource() 	
```

#### 3.3 点击链接切换

(1) WebMvcAutoConfigurationAdapter 提供了 默认的区域信息解析器：根据请求头带来的 locale

```java
// WebMvcAutoConfigurationAdapter
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
  if (this.mvcProperties
      .getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
    return new FixedLocaleResolver(this.mvcProperties.getLocale());
  }
  AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
  localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
  return localeResolver;
}
```

(2) 自定义一个区域信息解析器；

- 页面上添加链接：

```java
<a class="btn btn-sm" th:href="@{/index(l='zh_CN')}" >中文</a>
<a class="btn btn-sm"  th:href="@{/index(l='en_US')}" >English</a>
```

- 自定义区域解析器：

```java
// 根据链接参数获取
public class MyLocaleResolver  implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String url = request.getParameter("l");
        // 默认的
        Locale locale = Locale.getDefault();
        if(!StringUtils.isEmpty(url)){
              String[] s = url.split("_");
              locale = new Locale(s[0],s[1]);
        }
        return locale;
      }

      @Override
     public void setLocale(HttpServletRequest request, HttpServletResponse response,
                          Locale locale) {}
}
```

- 将自定的区域解析器，注册到容器：

```java
// Application.java
@Bean
public LocaleResolver localeResolver(){
   return new MyLocaleResolver();
}
```

### 4. 登录

#### 4.1 实现方法

(1) 页面添加 请求链接 和 参数：

```html
<!--表单提交信息-->
<form class="form-signin" action="dashboard.html" th:action="@{/user/login}" method="post">
<input name="username" ...>       <!--用户名-->
<input name="password" ...>       <!--密码-->
```

```html
<!--错误提示信息--->
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

(2) Controller

```java
@Controller
@RequestMapping("/user")
public class UserController { 
  @PostMapping("/login")
  public String login(@RequestParam("username") String username,
                      @RequestParam("password") String password,Map<String, Object> map){
    final String PASSWORD = "123456";
    if(!StringUtils.isEmpty(username) && PASSWORD.equals(password)) {   // login success
      return "dashboard";
    } else {
      map.put("msg", "username or password error!!!");
      return "login";
    }
  }
}
```

#### 4.2 重启

(1) 开发期间模板引擎页面修改以后，要实时生效：

在配置文件中添加配置，页面修改完成以后 **ctrl+f9**：重新编译；

```properties
# 禁用缓存
spring.thymeleaf.cache=false
```

(2) application 重启

Ctrl + F9

```xml
<!--添加依赖-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <optional>true</optional>
</dependency>
```

#### 4.3 防止重复提交

防止页面刷新导致表单重复提交：使用重定向

(1) 添加视图映射：

```java
// MyConfig
@Bean
WebMvcConfigurer get(){
  WebMvcConfigurer web = new WebMvcConfigurer(){
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
      registry.addViewController("index").setViewName("login");
      registry.addViewController("main").setViewName("dashboard");
    }
  };
  return web;
}
```

(2) 使用重定向

```java
if(!StringUtils.isEmpty(username) && PASSWORD.equals(password)) {
      // return "dashboard";
      return "redirect:/main";
}
```

### 5. 拦截器

如果直接直接使用上述重定向，可能导致：**直接输入重定向地址也能直接访问，导致登录失效；**

所以使用 拦截器 进行登录检查。

#### 5.1 添加Session

```java
if(!StringUtils.isEmpty(username) && PASSWORD.equals(password)) {
    // login success
    session.setAttribute("loginUser",username);
    return "redirect:/main";
}
```

#### 5.2 自定义拦截器

```java
public class MyLoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        Object loginUser = request.getSession().getAttribute("loginUser");
        if(loginUser != null){  // 登录，直接放行         
            return true;
        } else {           // 没有登录，到登录页面
            request.setAttribute("msg","没有权限，请先登录");
            request.getRequestDispatcher("/").forward(request,response);
             return false;
        }
    }
}
```

#### 5.3 注册拦截器

```java
// MyConfig
@Bean
WebMvcConfigurer get(){
    WebMvcConfigurer web = new WebMvcConfigurer(){
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
          // 静态资源：SpringBoot 已经做好了静态资源映射
            registry.addInterceptor(new MyLoginHandlerInterceptor())
                    .addPathPatterns("/**")                              // 拦截路径
                    .excludePathPatterns("/","/index","/user/login");    // 放行路径
        }
    };
    return web;
}
```

### 6. Restful CRUD

#### 6.1 Restful

URI： /资源名称/资源标识

 HTTP请求方式区分对资源CRUD操作

|      | 普通CRUD                  | Restful CRUD      |
| ---- | ----------------------- | ----------------- |
| 查询   | getEmp                  | emp---GET         |
| 添加   | addEmp?xxx              | emp---POST        |
| 修改   | updateEmp?id=xxx&xxx=xx | emp/{id}---PUT    |
| 删除   | deleteEmp?id=1          | emp/{id}---DELETE |

#### 6.2 请求请求架构 

| 实验功能               | uri      | method |
| ------------------ | -------- | ------ |
| 查询所有员工             | emps     | GET    |
| 来到添加页面             | emp      | GET    |
| 添加员工               | emp      | POST   |
| 来到修改页面（查出员工进行信息回显） | emp/{id} | GET    |
| 修改员工               | emp      | PUT    |
| 删除员工               | emp/{id} | DELETE |

#### 6.3 抽取页面公共元素方法

```html
<!--第一种方式：~{templatename::fragmentname}:模板名::片段名-->
	<!--footer页面 ：抽取-->
	<div th:fragment="copy">
    	... ...
	</div>
	<!--引入-->
	<div th:insert="~{footer :: copy}"></div>  或者
    <div th:insert="footer :: copy"></div>

<!--第二种方式：~{templatename::selector}：模板名::选择器-->
	<!--footer页面 ：抽取-->
	<div id="copy-section">
    	... ...
    </div>
	<!--引入-->
	<div th:insert="~{footer :: #copy-section}"></div>   或者
    <div th:insert="footer :: #copy-section"></div>
```

三种引入公共片段的方式及不同：

- **th:insert**

```html
<!--th:insert：将公共片段整个插入到声明引入的元素中-->
<div>
  <footer>
  	&copy; 2011 The Good Thymes Virtual Grocery
  </footer>
</div>
```

- th:replace

```html
<!--th:replace：将声明引入的元素替换为公共片段-->
<footer>
	&copy; 2011 The Good Thymes Virtual Grocery
</footer>
```

- h:include：将被引入的片段的内容包含进这个标签中

```html
<div>
	&copy; 2011 The Good Thymes Virtual Grocery
</div>
```

#### 6.4 设置选择为高亮

(1) 原理：

```html
<div th:replace="::frag (${value1},${value2})">...</div>
<div th:replace="::frag (onevar=${value1},twovar=${value2})">...</div>
```

(2) 实现

```html
<!--1. commons-->
<a class="nav-link" href="#" th:href="@{/emps}" 
   th:class="${uri == 'emps' ? 'nav-link active' : 'nav-link'}">

<!--2. insert -->
<div th:replace="commons/bar :: #sidebar(uri='emps')"></div>   
```

#### 6.5 查询All

```html
<tr th:each="emp : ${emps}">
  <td th:text="${emp.id}"></td>
  <td th:text="${emp.lastName}"></td>
  <td th:text="${emp.email}"></td>
  <td th:text="${emp.gender==1?'男':'女'}"></td>
  <td th:text="${emp.department.departmentName}"></td>
  <td th:text="${#dates.format(emp.birth, 'yyyy-MM-dd')}"></td>
</tr>			
```

#### 6.6  添加

(1) 添加页面

- 请求参数的名称 和 Javabean 入参的对象里面的属性名 相同；
- 时间格式：**默认为 dd/MM/yyyy，可以在配置文件中修改：spring.mvc.date-format=yyyy-MM-dd**

```html
<form th:action="@{/emp}" method="post">
  <div class="form-group">
    <label>姓名</label>
    <input type="text" class="form-control"  name="lastName" placeholder="zhangsan">
  </div>
  <div class="form-group">
    <label>邮箱</label>
    <input type="email" class="form-control" name="email" placeholder="zhangsan@qq.com">
  </div>
  <div class="form-group">
    <label>性别</label><br/>
    <div class="form-check form-check-inline">
      <input type="radio" class="form-check-input" name="gender" value="1">
      <label class="form-check-label">男</label>
    </div>
    <div class="form-check form-check-inline">
      <input type="radio" class="form-check-input" name="gender" value="0">
      <label class="form-check-label">女</label>
    </div>
  </div>
  <div class="form-group">
    <label>部门</label>
    <select class="form-control" name="department.id">
      <option th:each=" dept : ${depts}" th:text="${dept.departmentName}" th:value="${dept.id}"></option>
    </select>
  </div>
  <div class="form-group">
    <label>生日</label>
    <input class="form-control" name="birth" placeholder="2019/01/01"/>
  </div>
  <button type="submit" class="btn btn-primary">添加</button>
</form>
```

(2) Controller

```java
/**
 * 去 添加页面
 */
@GetMapping("/emp")
public String toAddPage (Model model){
  Collection<Department> departments = departmentDao.getDepartments();
  model.addAttribute("depts",departments);
  return "emp/add";
}

/**
 * 添加员工
 *    SpringMVC 自动将请求参数和入参对象的属性进行一一绑定
 * 要求：
 *    请求参数的名称 和 Javabean 入参的对象里面的属性名 相同
 */
@PostMapping("/emp")
public String add(Employee employee){
  employeeDao.save(employee);
  // 转发 forward
  // 重定向 redirect
  return "redirect:/emps";
}
```

#### 6.7 修改

(1) 页面复制 add 页面，然后修改

```html
<!--1. 如果是修改页面：提交方法修改为PUT-->
<input type="hidden" name="_method" th:if="${emp!=null}" value="put"/>
<input type="hidden" name="id" th:if="${emp!=null}" th:value="${emp.id}"/>

<!--2. 如果是修改页面：回显文本-->
<input type="text" class="form-control"  name="lastName" placeholder="zhangsan" 
          th:value="${emp!=null}?${emp.lastName}">

<!--3. 如果是修改页面：回显单选-->
<input type="radio" class="form-check-input" name="gender" value="1" 
           th:checked="${emp!=null}?${emp.gender==1}">
 <input type="radio" class="form-check-input" name="gender" value="0" 
            th:checked="${emp!=null}?${emp.gender==1}">

<!--4. 如果是修改页面：回显下拉列表-->
<select class="form-control" name="department.id">
    <option th:each=" dept : ${depts}" th:text="${dept.departmentName}" th:value="${dept.id}"
            th:selected="${emp!=null}?${emp.department.id==dept.id}"></option>
</select>

<!--5. 如果是修改页面：回显时间-->
<input class="form-control" name="birth" placeholder="2019/01/01" 
       th:value="${#dates.format(emp.birth,'yyyy-MM-dd')}"/>
<!--6. 修改 button-->
<button type="submit" class="btn btn-primary" th:text="${emp!=null}?'修改':'添加'">添加</button>
```

(2) Controller

```java
//去 修改页面
@GetMapping("/emp/{id}")
public String toEditPage(@PathVariable("id") Integer id, Model model){
  Employee employee = employeeDao.get(id);
  model.addAttribute("emp",employee);
  Collection<Department> departments = departmentDao.getDepartments();
  model.addAttribute("depts",departments);
  return "emp/edit";
}

// 修改员工:使用 put 方法
@PutMapping("/emp")
public String edit(Employee employee){
  employeeDao.save(employee);
  return "redirect:/emps";
}
```

#### 6.8 删除

(1) 页面

```html
<!--1. 按钮 自定义 属性-->
<button class="btn btn-sm btn-danger deleteBtn" th:attr="delUri=@{/emp/}+${emp.id}"> 删除</button>

<!--2. 原列表外，添加一个表单-->
<form id="deleteEmpForm"  method="post">
  <input type="hidden" name="_method" value="delete"/>
</form>

<!--3. 绑定删除操作-->
<script>
  $(".deleteBtn").click(function () {
      //删除当前员工的：绑定 action
      $("#deleteEmpForm").attr("action",$(this).attr("delUri")).submit();
      return false;
  });
</script>
```

### 7. servlet

SpringBoot默认使用Tomcat作为嵌入式的Servlet容器。

SpringBoot帮我们自动SpringMVC的时候，自动的注册SpringMVC的前端控制器；DIspatcherServlet；

```java
// DispatcherServletAutoConfiguration.class
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name =
DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration( DispatcherServlet dispatcherServlet) {
   ServletRegistrationBean registration = new ServletRegistrationBean(    
    //默认拦截： /  所有请求；包静态资源，但是不拦截jsp请求；   注意：/*会拦截jsp
    //可以通过spring.mvc.servlet.path来修改SpringMVC前端控制器默认拦截的请求路径
   ）
}
```



#### 7.1 修改Servlet配置 

**第一种方法：配置文件**

配置类：ServerProperties

```properties
server.port=8081
server.context-path=/crud
server.tomcat.uri-encoding=UTF-8

//通用的Servlet容器设置
server.xxx
//Tomcat的设置
server.tomcat.xxx
```

**第二种方法：编写配置类**

**Springboot 1.0：编写EmbeddedServletContainerCustomizer接口**

```java
@Bean  //一定要将这个定制器加入到容器中
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {     
        @Override   //定制嵌入式的Servlet容器相关的规则
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8083);
        }
    };
}
```



**Springboot 2.0：编写 WebServerFactoryCustomizer接口，重写customize方法**

```java
@Configuration
public class MyConfig {
  @Bean
  public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer(){
    return new WebServerFactoryCustomizer<ConfigurableWebServerFactory>() {
      @Override
      public void customize(ConfigurableWebServerFactory factory) {
        factory.setPort(8085);
      }
    };
    // return factory -> factory.setPort(8085);
  }
}
```

#### 7.2 注册三大组件：Servlet、Filter、Listener

原配置：webapp/WEB-INF/web.xml

- Servlet----ServletRegistrationBean

  第一步：**自定义MyServlet，继承HttpServlet，重写doGet，doPost方法**；

  第二步：在配置类中，注册

```java
//注册三大组件
@Configuration
public class MyServletConfig{
	@Bean
	public ServletRegistrationBean myServlet(){
    	return new ServletRegistrationBean(new MyServlet(),"/myServlet");
	}
}
```

- Filter----FilterRegistrationBean

  第一步：**自定义MyFilter实现Filter，重写 init，doFilter，destroy方法**；

  ​              【注意doFilter方放行：chain.doFilter(request,response); 】

  第二步：注册；

```java
@Bean
public FilterRegistrationBean myFilter(){
  FilterRegistrationBean filter = new FilterRegistrationBean();
  filter.setFilter(new MyFilter());
  filter.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
  return filter;
}
```

- ServletListener----ServletListenerRegistrationBean

  第一步：**自定义MyListener实现ServletContextListener**，重写contextInitialized、contextDestroyed；

  第二步：注册；

```java
@Bean
public ServletListenerRegistrationBean myListner(){
  return new ServletListenerRegistrationBean(new MyServletListener());
}
```

#### 7.3 替换为其他Servlet 

- Tomcat（默认使用）

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- Jetty--适合长连接

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

- Undertow---不支持JSP

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

#### 7.4 嵌入式Servlet自动配置原理

**(1) Springboot 1.0 自动配置类：EmbeddedServletContainerAutoConfiguration**

① 根据导入不同的依赖，给容器中添加相应的嵌入式容器工程(EmbeddedServletContainerFactory)，用来创建嵌入式的Servlet容器；

② 容器中导入了后置处理器：EmbeddedServletContainerCustomizerBeanPostProcessor；

③ 后置处理器从容器中获取所有的EmbeddedServletContainerCustomizer，调用定制器的定制方法。

```java 
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration                
@ConditionalOnWebApplication  // web引用上生效
@Import(BeanPostProcessorsRegistrar.class)
//导入BeanPostProcessorsRegistrar：Spring注解版；给容器中导入一些组件
//导入了EmbeddedServletContainerCustomizerBeanPostProcessor：
//后置处理器：bean初始化前后（创建完对象，还没赋值赋值）执行初始化工作
public class EmbeddedServletContainerAutoConfiguration {
  
    /**
   	* Tomcat    
   	*/
  	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })//判断当前是否引入了Tomcat依赖； 
    //判断当前容器没有用户自己定义嵌入式的Servlet容器工厂
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class)  
	public static class EmbeddedTomcat { 
      
    /**
   	* Jetty
   	*/ 
    @Configuration    
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class, WebAppContext.class })
    @ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class)   
	public static class EmbeddedJetty {   
      
  /*     
   * Undertow   
   */    
    @Configuration    
    @ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })    
    @ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class)   
	public static class EmbeddedUndertow { 
} 
```

(2) EmbeddedServletContainerCustomizerBeanPostProcessor：

```java
//初始化之前
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) {
   if (bean instanceof ConfigurableEmbeddedServletContainer) {  
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}
```

(3) postProcessBeforeInitialization()：

```java 
private void postProcessBeforeInitialization(
ConfigurableEmbeddedServletContainer bean) {            
    //获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值；
    // getCustomizers(): 从容器中获取所有EmbeddedServletContainerCustomizer类型的组件
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------

**Springboot 2.0自动配置类：ServletWebServerFactoryAutoConfiguration**

```java
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@ConditionalOnClass(ServletRequest.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@EnableConfigurationProperties(ServerProperties.class)
@Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
public class ServletWebServerFactoryAutoConfiguration {
```

**后置处理器：WebServerFactoryCustomizerBeanPostProcessor**，方法与1.0一样。

#### 7.5 启动原理

- SpringApplication#run()启动；
- createApplicationContext()：创建IOC容器对象，并初始化容器
- refreshContext(context)：刷新IOC容器；
  - refresh(context)：刷新刚才创建好的ioc容器；
    - onRefresh()：web的ioc容器重写了onRefresh方法。
      - createWebServer()：web IOC 容器会创建嵌入式的Servlet容器；
        - 首先 获取 ServletWebServerFactory 组件【1.0-EmbeddedServletContainerFactory 】；
        - 接着 后处理器就定制Servlet容器的相关配置；
        - 然后  获取 嵌入式的Servlet容器，创建对象并启动



**IOC启动创建嵌入式Servlet容器：先启动嵌入式的Servlet容器，再将剩下没有创建出的对象获取出来。**


#### 7.6 使用外部Servlet容器

1. 创建 一个 war  项目，还是使用 spring boot创建项目；

2. Project Structure-->Modules -->Web ：创建  webapp 文件夹 和 Web.xml；

3. 加入 Tomcat：运行窗口--> + -->Tomcat --> Server -->Deployment；

4. Controller 返回 “success”；

5. success页面在webapp/WEB-INF下，需要再 配置文件配：

   ```properties
   spring.mvc.view.prefix=/WEB-INF/
   spring.mvc.view.suffix=.jsp
   ```

6. 将嵌入式的Tomcat指定为provided；provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring‐boot‐starter‐tomcat</artifactId>
      <scope>provided</scope>
   </dependency>
   ```

7.  SpringBootServletInitializer的子类必须有，并调用configure方法。**自动生成**

### 8. 数据访问 

#### 8.1 整合Druid数据源

- 引入 Druid数据源

  ```xml
  <!--数据库连接池-->
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>${com.alibaba.druid}</version>
  </dependency>
  ```

- 配置文件：修改数据源

  ```yaml
  spring:
    datasource:
      data-username: ebcen_test_admin
      data-password: aEF3t6wH
      url: jdbc:mysql://192.168.5.159:3306/ebcenter_test
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.jdbc.Driver
      filters: stat
      maxActive: 20
      initialSize: 1
      maxWait: 60000
      minIdle: 1
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: select 'x'
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true
      maxOpenPreparedStatements: 20
  ```

- 导入druid数据源

  ```java
  @Configuration
  public class DruidConfig {
      @ConfigurationProperties(prefix = DB_PREFIX)
      @Bean
      public DataSource druid(){
          return new DruidDataSource();
      }
    
      //配置Druid的监控
      //1、配置一个管理后台的Servlet
    	@Bean
      public ServletRegistrationBean druidServlet() {
          logger.info("init Druid Servlet Configuration ");
          ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");     
          bean.addInitParameter("allow", "*");              // IP白名单
          bean.addInitParameter("deny", "192.168.1.100");   // IP黑名单
          bean.addInitParameter("loginUsername", "admin");
          bean.addInitParameter("loginPassword", "admin");
          return bean;
      }
    
      //2、配置一个web监控的filter
      @Bean
      public FilterRegistrationBean webStatFilter(){
          FilterRegistrationBean bean = new FilterRegistrationBean();
          bean.setFilter(new WebStatFilter());
          bean.addUrlPatterns("/*");
          bean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
          return bean;
      }
  }
  ```

#### 8.2 整合MyBatis

- 添加依赖

  ```xml
  <dependency>        
  <groupId>org.mybatis.spring.boot</groupId>            
  <artifactId>mybatis‐spring‐boot‐starter</artifactId>            
  <version>1.3.1</version>            
  </dependency
  ```

- 配置文件版

  ```yaml
  mybatis:
    # 指定全局配置文件位置
    config-location: classpath:mybatis/mybatis-config.xml
    # 指定sql映射文件位置
    mapper-locations: classpath:mybatis/mapper/*.xml

  ```

- 注解版

  ```java
  @Mapper
  public interface DepartmentMapper {
      @Select("select * from department where id=#{id}")
      public Department getDeptById(Integer id);
    
      @Options(useGeneratedKeys = true,keyProperty = "id")
      @Insert("insert into department(departmentName) values(#{departmentName})")
      public int insertDept(Department department);
  }
  ```

  **注意：文件或者注解版都需要**

   	1. **可以使用 @Mapper标注在 接口上；**
   	2. **可以使用 @MapperScan(value = "xxx.xxx") 批量扫描，放到 主程序类上；**

## 9. 自定义starter

### 9.1 方法

```java 
@Configuration        //指定这个类是一个配置类
@ConditionalOnXXX     //在指定条件成立的情况下自动配置类生效
@AutoConfigureAfter   //指定自动配置类的顺序
@Bean                 //给容器中添加组件
@ConfigurationPropertie           // 结合相关xxxProperties类来绑定相关的配置
@EnableConfigurationProperties    // 让xxxProperties生效加入到容器中

//  将需要启动就加载的自动配置类，配置在META‐INF/spring.factories
```

### 9.2 实例

1. 创建一个空项目

   new --> Project -->Empty Project

2. 添加一个 maven模块，作为启动类；添加一个Spring boot 模块，作为配置类；

3. 删除配置类中的 test，启动类，配置文件；

4. **在 启动器模块 的 pom 文件中，添加 配置类的依赖：**

   ```xml
   <!--启动器-->
   <dependencies>
     <!--引入自动配置模块-->
     <dependency>
       <groupId>com.fang.starter</groupId>
       <artifactId>cl-spring-boot-starter-autoconfiger</artifactId>
       <version>0.0.1-SNAPSHOT</version>
     </dependency>
   </dependencies>
   ```

5. **在 自动配置模块**的 pom 文件中，只保留以下依赖：

   ```xml
   <dependencies>
     <!‐‐引入spring‐boot‐starter；所有starter的基本配置‐‐>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring‐boot‐starter</artifactId>
     </dependency>
   </dependencies>
   ```

6. 在 自动配置模块 添加配置：

   ```java
   // 1. 配置文件类
   @ConfigurationProperties(prefix = "cl.hello")
   public class HelloProperties {
     // 省略 getter setter
     private String prefix;
     private String suffix;
   }
   ```

   ```java
   // 2. 场景描述
   public class HelloService {
       HelloProperties helloProperties;
       public HelloProperties getHelloProperties() {
           return helloProperties;
       }
       public void setHelloProperties(HelloProperties helloProperties) {
           this.helloProperties = helloProperties;
       }
       public String sayHellAtguigu(String name){
           return helloProperties.getPrefix()+"‐" +name + helloProperties.getSuffix();
       }
   }
   ```

   ```java
   // 3. 自动配置类
   Configuration
   @ConditionalOnWebApplication  //web应用才生效
   @EnableConfigurationProperties(HelloProperties.class)  // 让属性文件生效
   public class HelloServiceAutoConfiguration {

     @Autowired
     HelloProperties helloProperties

     @Bean
     public HelloService helloService(){
       HelloService service = new HelloService();
       service.setHelloProperties(helloProperties);
       return service;
     }
   }
   ```

7. **自动配置类要能加载**：在类路径下 复制 autoconfig包下的，/META‐INF/spring.factories，只保留 

   ```xml
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   com.fang.starter.HelloServiceAutoConfiguration
   ```

8. **测试**

   ```xml
   <!-- 1.加入依赖 -->
   <dependency>
     <groupId>com.fang.starter</groupId>
     <artifactId>cl-spring-boot-starter</artifactId>
     <version>1.0-SNAPSHOT</version>
   </dependency>
   ```

   ```java
   @RestController
   public class HelloController {
     @Autowired
     private HelloService service;
     @GetMapping("/hello")
     public String sayHello(){
       return service.sayHello("cc");
     }
   }
   ```

   ​

## 10. 缓存

### 10.1 使用

- 开启SQL打印

  ```yaml
  logging:
    level:
      com.fang.cache.dao : debug
  ```

- 开启缓存

  ```java
  // 启动类
  @EnableCaching
  public class Application {}
  ```

- 缓存

  ```java
  @Cacheable(cacheNames = "dept")
    public Department getDepartment(Integer id) {
      return mapper.selectByPrimaryKey(id);
    }
  ```

### 10.2 属性

- **cacheNames/value**：指定缓存组件的名字；
- **key/keyGenerator**：缓存数据使用的key，**默认是使用方法参数的值**
  - 例如：#id；#a0； #p0；#root.args[0]
- **cacheManager**：指定缓存管理器；
- **condition**：指定符合条件的情况下才缓存；
- **unless**：否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；
  -  unless = "#result == null"
- **sync**：是否使用异步模式



### 10.3 原理

1. 自动配置类：CacheAutoConfiguration
2. 缓存的配置类：默认生效 SimpleCacheConfiguration

    org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
    org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
    org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
    org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
    org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
    org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
    org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
    org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
    org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
    org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
    org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
    
3. 给容器中注册了一个CacheManager：ConcurrentMapCacheManager
4. 可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；

### 10.4 流程

- 先调用 ConcurrentMapCacheManager 的 getCache 方法查询Cache（缓存组件），按照cacheNames指定的名字获取（例如上述例子中的“dept”）；如果是第一次获取缓存会自动创建。
- 去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
  - key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key：
    - 如果没有参数；key=new SimpleKey()；
    - 如果有一个参数：key=参数的值
    -  如果有多个参数：key=new SimpleKey(params)；
- 没有查到缓存就调用目标方法；
- 将目标方法返回的结果，放进缓存中

**核心：**

```java
// 1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件；
// 2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator；
```
### 10.5 集成Redis

- 下载镜像：docker pull registry.docker-cn.com/library/redis；

- 启动镜像：docker run -d -p 6379:6379 --name myredis registry.docker-cn.com/library/redis

- 引入starter

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

- application.yml配置redis连接地址：**spring.redis.host**

- 使用 **ReditTemplate** 或者 **StringRedisTemplate**  操作redis

  ```java
  redisTemplate.opsForValue();          //操作字符串
  redisTemplate.opsForHash();           //操作hash
  redisTemplate.opsForList();           //操作list
  redisTemplate.opsForSet();            //操作set
  redisTemplate.opsForZSet();           //操作有序set
  ```

- 如果存储的是对象，需要序列化：可以手动转为json。

- ​

  ​

  ​

















