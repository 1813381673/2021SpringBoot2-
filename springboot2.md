# Spring Boot 2.x

#### SpringMVC自动配置概览

Spring Boot provides auto-configuration for Spring MVC that **works well with most applications.(大多场景我们都无需自定义配置)**

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

- - 内容协商视图解析器和BeanName视图解析器

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).

- - 静态资源（包括webjars）

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.

- - 自动注册 `Converter，GenericConverter，Formatter `

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).

- - 支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).

- - 自动注册 `MessageCodesResolver` （国际化用）

- Static `index.html` support.

- - 静态index.html 页支持

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)).

- - 自定义 `Favicon`  

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).

- - 自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）

> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.9.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.
>
> **不用@EnableWebMvc注解。使用** **@Configuration** **+** **WebMvcConfigurer** **自定义规则**



> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> **声明** **WebMvcRegistrations** **改变默认底层组件**



> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.
>
> **使用** **@EnableWebMvc+@Configuration+DelegatingWebMvcConfiguration 全面接管SpringMVC**



#### 其他

##### META_INF/spring-factories位置来加载一个文件

主要是spring-boot-autoConfiguration这个jar包下面的这个文件 

##### 只要spring boot 一启动就会加载所有自动配置类

虽然默认全部加载 但是都是按需加载 在很多配置类上 都加了 **@Conditional**  之类的注解限制加载  条件装配

```java
@EnableAutoConfiguration // 这个最重要
/**
 * 点进去之后在上面标注的注解
 * @AutoConfigurationPackage
 *      其上面标注的注解
 *      @Import({Registrar.class})
 *         利用Registrar 给容器中导入一系列组件
 *         将制定的一个包下的所有组件导入进来  MainApplication所在包
 *
 * @Import({AutoConfigurationImportSelector.class})
 *
 */
```

```java
/**
 * 1.配置类使用@Bean标注在方法上给容器注册组件，默认是单实例
 * 2.配置类本身也是组件
 * 3.proxyBeanMethods: 代理bean的方法
 *      true的时候 是单例
 *      解决组件依赖关系
 *      false的时候 就不会检查容器内有无组件  启动的时候就会很快 所以容器组件没有依赖关系时推荐使用
 * 4. @Import({User.class, DBHelper.class})
 *      给容器中自动创建出这两个类型的组件,默认组件的名字是全类名
 *
 */
@Import({User.class, DBHelper.class})
@Configuration(proxyBeanMethods = true) // Since spring 5.2  spring boot 2.x 默认是true
@ImportResource("classpath:beans.xml") // 可以继续让原来的spring配置文件生效
@EnableConfigurationProperties(Car.class)
// 1.开启Car配置绑定功能
// 2.把这个Car组件自动注册到容器中
```

##### 总结：

1. SpringBoot先加载所有的自动配置类  xxxxAutoConfiguration    

2. 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值  xxxxProperties里面拿，XXXProperties和配置文件进行了绑定

3. 生效的配置类就会给容器中装配很多组件

4. 只要容器中有这些组件，相当于这些功能就有了

5. 只要用户有自己配置的，就优先用户的

6. 定制化配置

   ​	用户直接自己@Bean替换底层的组件

   ​	用户去看这个组件是获取的配置文件是什么值  去修改就行了

   xxxAutoConfiguration --> 组件 --> xxxProperties里面拿值  -- > applicaiton.properties

#### 最佳实践

查看自动配置了那些

​	配置文件中 debug=true 控制台会打印出 生效以及不生效的组件

自定义加入或替换组件

​	@Bean   @Component...

自定义器 **XXXCustomizer**

#### 静态资源访问

##### 1.静态资源目录

/static，/resources，/META-INF/resources，/public

原理：静态映射 /**

请求进来，先去找Controller看能不能处理，不能处理的所有请求又交给静态资源处理器，静态资源处理器若也找不到就会404

##### 2.静态资源访问前缀

默认无前缀，但是在做拦截器的时候  就需要一个前缀 只要是这个前缀就放行

~~~yaml
spring:
  mvc:
    static-path-pattern: /res/**
~~~

当前项目 + static-path-pattern + 静态资源名 =  静态资源文件夹下找

favicon.ico可以自定义网页前面的小图标

==这个前缀会导致自定义欢迎页面 解析不了==

##### 2- 静态资源配置原理

- springboot启动默认加载  xxxAutoConfiguration类 (自动配置)
- springMVC功能的自动配置类 WebMVCAutoConfiguration

~~~java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
~~~

- 给容器中配了什么

  ~~~java
  	@Configuration(proxyBeanMethods = false)
  	@Import(EnableWebMvcConfiguration.class)
  	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
  	@Order(0)
  	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
  ~~~

- 配置文件的相关属性和xxx进行了绑定，WebMVCProperties==spring.mvc、

  ResourceProperties==spring.resources

##### 3.配置类只有一个有参构造器

~~~java
	// 有参构造器所有参数的值都会从容器中确定
	// ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
	// WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
	// ListableBeanFactory beanFactory Spring的beanFactory IOC容器
	// HttpMessageConverters 找到所有的HttpMessageConverters
	// ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
	// DispatcherServletPath  
	// ServletRegistrationBean   给应用注册Servlet、Filter....
	public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}
~~~

##### 4.资源处理的默认规则

~~~java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			//webjars的规则
            if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            
            //
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
~~~

~~~yaml
spring:
  resources:
    add-mappings: false   禁用所有静态资源，放在哪都访问不了
~~~

##### 5.欢迎页的处理规则

~~~java
	HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。	

	@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}

	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            // 要用欢迎页功能，必须是/**
            // 这就是为什么加前缀会出现  404 的问题
			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            // 调用Controller  /index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}
~~~

#### 请求参数处理

##### 0. 请求映射

##### 1. rest使用原理

- @xxxMapping;
- Rest风格支持(使用HTTP请求方式动词来表示对资源的操作
- 核心Filter：HiddenHttpMethodFilter  在SpringMVC的时候配置过

  - 用法：表单method = post 隐藏域 _method=put
  - SpringBoot中手动开启

~~~java
    @RequestMapping(value = "/user",method = RequestMethod.GET)
    public String getUser(){
        return "GET-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String saveUser(){
        return "POST-张三";
    }


    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    public String putUser(){
        return "PUT-张三";
    }

    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
    public String deleteUser(){
        return "DELETE-张三";
    }

	// 开启rest风格 必须要配置才会开启
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}


//自定义filter
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }
~~~

Rest原理（表单提交要使用REST的时候）

- 表单提交会带上**_method=PUT**
- **请求过来被**HiddenHttpMethodFilter拦截

- - 请求是否正常，并且是POST

- - - 获取到**_method**的值。
    - 兼容以下请求；**PUT**.**DELETE**.**PATCH**
    - **原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。**
    - **过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。**

**Rest使用客户端工具，**

- 如PostMan直接发送Put、delete等方式请求，无需Filter。

~~~yaml
spring:
mvc:
hiddenmethod:
filter:
enabled: true   #开启页面表单的Rest功能 选择性开启
~~~

##### 2. 请求映射原理

![](SpringBoot2Imgs\1.png)

SpringMVC功能分析都从 

org.springframework.web.servlet.DispatcherServlet-》doDispatch（）

~~~java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 找到当前请求使用哪个Handler（Controller的方法）处理
				mappedHandler = getHandler(processedRequest);
                
                //HandlerMapping：处理器映射。/xxx->>xxxx
~~~

**RequestMappingHandlerMapping**：保存了所有@RequestMapping 和handler的映射规则

![](SpringBoot2Imgs\2.png)

所有的请求映射都在HandlerMapping中。

- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping
- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。

- - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping

- 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**

#### 普通参数与基本注解

##### 1.1、注解

@PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@MatrixVariable、@CookieValue、@RequestBody

~~~java
@RestController
public class ParamterTestController {

    @GetMapping("/car/{id}/owner/{username}")
    public Map<String, Object> getCar(@PathVariable("id") Integer id,
                                      @PathVariable("username") String name,
                                      @PathVariable Map<String, String> pv,
                                      @RequestHeader("User-Agent") String userAgent,
                                      @RequestHeader Map<String, String> header,
                                      @RequestParam("age") Integer age,
                                      @RequestParam("inters") List<String> inters,
                                      @RequestParam Map<String, String> parmas,
                                      @CookieValue("Pycharm-917e35ad") String ga,
                                      @CookieValue("Pycharm-917e35ad") Cookie cookie){
        Map<String, Object> map = new HashMap<>();
        /*map.put("id", id);
        map.put("name", name);
        map.put("pv", pv);
        map.put("user-agent", userAgent);
        map.put("header", header);*/
        map.put("age", age);
        map.put("inters", inters);
        map.put("params", parmas);
        // 注意cookieValue这个属性    查看你发送的请求中到底有没有这个cookieValue  没有就会报错
        map.put("Pycharm-917e35ad", ga);
        System.out.println(cookie);
        System.out.println(cookie.getName() + "------" + cookie.getValue());
        return map;
    }

    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String, Object> map = new HashMap();
        map.put("content", content);
        return map;
    }
    // 1. 语法 /cars/sell;low=34;brand=byd,audi,yd
    // 2. SpringBoot默认是禁用了矩阵变量的功能
    //      需要手动开启,
    //      原理：对于路径的处理 UrlPathHelper进行解析
    //      removeSemiconlonContent（移除分号内容） 支持矩阵变量的
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable String path){

        Map<String, Object> map = new HashMap();
        map.put("low", low);
        map.put("brand", brand);
        map.put("path", path);
        return map;
    }

    //   /boss/1;age=20/2;age=10
    @GetMapping("/boss/{bossid}/{empid}")
    public Map boos(@MatrixVariable(value = "age",pathVar = "bossid") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empid") Integer empAge){
        Map<String, Object> map = new HashMap();
        map.put("bossAge", bossAge);
        map.put("empAge", empAge);
        return map;
    }
}
~~~

##### 1.2、Servlet API：

WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、ZoneId

**ServletRequestMethodArgumentResolver  可以解析 以上的部分参数**

~~~java
@Override
	public boolean supportsParameter(MethodParameter parameter) {
		Class<?> paramType = parameter.getParameterType();
		return (WebRequest.class.isAssignableFrom(paramType) ||
				ServletRequest.class.isAssignableFrom(paramType) ||
				MultipartRequest.class.isAssignableFrom(paramType) ||
				HttpSession.class.isAssignableFrom(paramType) ||
				(pushBuilder != null && pushBuilder.isAssignableFrom(paramType)) ||
				Principal.class.isAssignableFrom(paramType) ||
				InputStream.class.isAssignableFrom(paramType) ||
				Reader.class.isAssignableFrom(paramType) ||
				HttpMethod.class == paramType ||
				Locale.class == paramType ||
				TimeZone.class == paramType ||
				ZoneId.class == paramType);
	}
~~~

##### 1.3、复杂参数

**Map、Model(map、model里面的数据会被放在request的请求域中  相当于 request.setAttribute)**、Errors/Bindingresult、**RedirectAttributes(重定向携带数据)**、**ServletResponse**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

~~~java
// 在这三个对象中放东西  都相当于在request域中放数据
@GetMapping("/params")
    public String testParms(Map<String, Object> map,
                            Model model,
                            HttpServletRequest request){
        map.put("map","map 123");
        model.addAttribute("model","model 123");
        request.setAttribute("request", "request 123");      
        return "forward:" + "/success";
    }
~~~

**Map、Model**类型的参数，最终都会返回  mavContainer.getModel();----->是Model  也是 Map

![](SpringBoot2Imgs\7.png)

![](SpringBoot2Imgs\8.png)

![](SpringBoot2Imgs\9.png)

##### 1.4、自定义对象参数

可以自动类型转换与格式化，可以级联封装。

~~~java
/**
 *     姓名： <input name="userName"/> <br/>
 *     年龄： <input name="age"/> <br/>
 *     生日： <input name="birth"/> <br/>
 *     宠物姓名：<input name="pet.name"/><br/>
 *     宠物年龄：<input name="pet.age"/>
 */
@Data
public class Person {
    
    private String userName;
    private Integer age;
    private Date birth;
    private Pet pet;
    
}

@Data
public class Pet {

    private String name;
    private String age;

}

result
~~~



#### 参数处理原理

- HandlerMapping中找到能处理请求的Handler（Controller.method()）

- 为当前Handler 找一个适配器 HandlerAdapter； **RequestMappingHandlerAdapter**

- 适配器执行目标方法并确定方法参数的每一个值

##### 1、HandlerAdapter

![](SpringBoot2Imgs\3.png)

0-支持方法上标注@RequestMapping   // 我们写的默认都在这里找

1-支持函数式编程

##### 2、执行目标方法

~~~java
// Actually invoke the handler.
//DispatcherServlet -- doDispatch
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
~~~

~~~java

mav = invokeHandlerMethod(request, response, handlerMethod); //执行目标方法

// ServletInvocableHandlerMethod  真正执行目标方法  会进入controller
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
// 获取方法的参数值
Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
~~~

##### 3、参数解析器--HandlerMethodArgumentResolver

确定将要执行的目标方法的每一个参数的值是什么;

SpringMVC目标方法能写多少种参数类型。取决于参数解析器。

![](SpringBoot2Imgs\4.png)

![](SpringBoot2Imgs\5.png)

- 当前解析器是否支持解析这种参数

- 支持就调用解析方法   resolveArgument

##### 4、返回值处理器

![](SpringBoot2Imgs\6.png)

##### 5、如何确定目标方法每一个参数的值

~~~java
============InvocableHandlerMethod==========================
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
		// 获取所有的参数声明
		MethodParameter[] parameters = getMethodParameters();
		if (ObjectUtils.isEmpty(parameters)) {
			return EMPTY_ARGS;
		}

		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			args[i] = findProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
			if (!this.resolvers.supportsParameter(parameter)) {
				throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
			}
			try {
				args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
			}
			catch (Exception ex) {
				// Leave stack trace for later, exception may actually be resolved and handled...
				if (logger.isDebugEnabled()) {
					String exMsg = ex.getMessage();
					if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
						logger.debug(formatArgumentError(parameter, exMsg));
					}
				}
				throw ex;
			}
		}
		return args;
	}
~~~

##### 5.1、挨个判断所有参数解析器那个支持解析这个参数

~~~java
	@Nullable
	private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
		HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
		if (result == null) {
			for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
				if (resolver.supportsParameter(parameter)) {
					result = resolver;
					this.argumentResolverCache.put(parameter, result);
					break;
				}
			}
		}
		return result;
	}
~~~

##### 5.2、解析这个参数的值

调用各自 HandlerMethodArgumentResolver 的 resolveArgument 方法即可

##### 5.3、自定义类型参数 封装POJO

**ServletModelAttributeMethodProcessor  这个参数处理器支持是否为简单类型。**

~~~java
public static boolean isSimpleValueType(Class<?> type) {
		return (Void.class != type && void.class != type &&
				(ClassUtils.isPrimitiveOrWrapper(type) ||
				Enum.class.isAssignableFrom(type) ||
				CharSequence.class.isAssignableFrom(type) ||
				Number.class.isAssignableFrom(type) ||
				Date.class.isAssignableFrom(type) ||
				Temporal.class.isAssignableFrom(type) ||
				URI.class == type ||
				URL.class == type ||
				Locale.class == type ||
				Class.class == type));
	}
~~~

~~~java
@Override
	@Nullable
	public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

		Assert.state(mavContainer != null, "ModelAttributeMethodProcessor requires ModelAndViewContainer");
		Assert.state(binderFactory != null, "ModelAttributeMethodProcessor requires WebDataBinderFactory");

		String name = ModelFactory.getNameForParameter(parameter);
		ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);
		if (ann != null) {
			mavContainer.setBinding(name, ann.binding());
		}

		Object attribute = null;
		BindingResult bindingResult = null;

		if (mavContainer.containsAttribute(name)) {
			attribute = mavContainer.getModel().get(name);
		}
		else {
			// Create attribute instance
			try {
				attribute = createAttribute(name, parameter, binderFactory, webRequest);
			}
			catch (BindException ex) {
				if (isBindExceptionRequired(parameter)) {
					// No BindingResult parameter -> fail with BindException
					throw ex;
				}
				// Otherwise, expose null/empty value and associated BindingResult
				if (parameter.getParameterType() == Optional.class) {
					attribute = Optional.empty();
				}
				bindingResult = ex.getBindingResult();
			}
		}

		if (bindingResult == null) {
			// Bean property binding and validation;
			// skipped in case of binding failure on construction.
			WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
			if (binder.getTarget() != null) {
				if (!mavContainer.isBindingDisabled(name)) {
					bindRequestParameters(binder, webRequest);
				}
				validateIfApplicable(binder, parameter);
				if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
					throw new BindException(binder.getBindingResult());
				}
			}
			// Value type adaptation, also covering java.util.Optional
			if (!parameter.getParameterType().isInstance(attribute)) {
				attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
			}
			bindingResult = binder.getBindingResult();
		}

		// Add resolved attribute and BindingResult at the end of the model
		Map<String, Object> bindingResultModel = bindingResult.getModel();
		mavContainer.removeAttributes(bindingResultModel);
		mavContainer.addAllAttributes(bindingResultModel);

		return attribute;
	}
~~~

**WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**

**WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面(attribute空对象)**

**WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**



(http 超文本协议  万物皆文本  从文本对象转为需要的类型)

**GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）** // 看哪个可以支持转换操作

**byte -- > file**  

![](SpringBoot2Imgs\11.png)

**所有的converter**

![](SpringBoot2Imgs\12.png)

未来我们可以给WebDataBinder里面放自己的Converter；

**private static final class** StringToNumber<T **extends** Number> **implements** Converter<String, T>



##### 6、目标方法执行完成

将所有的数据都放在ModelAndViewContainer；包含要去的页面地址View，还包含Model数据

![](SpringBoot2Imgs\10.png)

##### 7、处理派发结果

**核心方法**

**processDispatchResult**(processedRequest, response, mappedHandler, mv, dispatchException);

renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);

~~~java
InternalResourceView类：
@Override
	protected void renderMergedOutputModel(
			Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
    
		// Expose the model object as request attributes.
		exposeModelAsRequestAttributes(model, request);

		// Expose helpers as request attributes, if any.
		exposeHelpers(request);

		// Determine the path for the request dispatcher.
		String dispatcherPath = prepareForRendering(request, response);

		// Obtain a RequestDispatcher for the target resource (typically a JSP).
		RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
		if (rd == null) {
			throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
					"]: Check that the corresponding file exists within your web application archive!");
		}

		// If already included or response already committed, perform include, else forward.
		if (useInclude(request, response)) {
			response.setContentType(getContentType());
			if (logger.isDebugEnabled()) {
				logger.debug("Including [" + getUrl() + "]");
			}
			rd.include(request, response);
		}

		else {
			// Note: The forwarded resource is supposed to determine the content type itself.
			if (logger.isDebugEnabled()) {
				logger.debug("Forwarding to [" + getUrl() + "]");
			}
			rd.forward(request, response);
		}
	}
~~~

**关键点**

~~~java
暴露模型作为请求域属性
		// Expose the model object as request attributes.
		// 就是遍历model这个map对象 把key-value放入request域中
		exposeModelAsRequestAttributes(model, request);
~~~

~~~java
protected void exposeModelAsRequestAttributes(Map<String, Object> model,
			HttpServletRequest request) throws Exception {
    	//model中的所有数据遍历挨个放在请求域中
		model.forEach((name, value) -> {
			if (value != null) {
				request.setAttribute(name, value);
			}
			else {
				request.removeAttribute(name);
			}
		});
	}
~~~

### 4.数据响应与内容协商

![](SpringBoot2Imgs\13.jpg)



#### 1、响应JSON

##### 1.1、jackson.jar + @ResponseBody

~~~pom
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
	<!-- web场景自动引入了json场景 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>
~~~

![](SpringBoot2Imgs\13.png)

给前端自动返回json数据

##### 1、返回值解析器

![](SpringBoot2Imgs\14.png)

~~~java
try {
		this.returnValueHandlers.handleReturnValue(
		returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
	}
~~~

~~~java
	@Override
	public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

		HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
		if (handler == null) {
			throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
		}
		handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
	}
~~~

~~~java
RequestResponseBodyMethodProcessor  	
@Override
	public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
			throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

		mavContainer.setRequestHandled(true);
		ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
		ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

		// Try even with null return value. ResponseBodyAdvice could get involved.
        // 使用消息转换器进行写出操作
		writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
	}
~~~





##### 2、返回值解析器原理

![](SpringBoot2Imgs\15.png)

- 1、返回值处理器判断是否支持这种类型返回值 supportsReturnType
- 2、返回值处理器调用 handleReturnValue 进行处理
- 3、RequestResponseBodyMethodProcessor 可以处理返回值标了@ResponseBody 注解的。

- - 1.  利用 MessageConverters 进行处理 将数据写为json

- - - 1、内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
    - 2、服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据，
    - 3、SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？

- - - - 1、得到MappingJackson2HttpMessageConverter可以将对象写为json
      - 2、利用MappingJackson2HttpMessageConverter将对象转为json再写出去。

##### 1.2、SpringMVC到底支持哪些返回值

~~~
ModelAndView
Model
View
ResponseEntity 
ResponseBodyEmitter
StreamingResponseBody
HttpEntity
HttpHeaders
Callable
DeferredResult
ListenableFuture
CompletionStage
WebAsyncTask
有 @ModelAttribute 且为对象类型的
@ResponseBody 注解 ---> RequestResponseBodyMethodProcessor；
~~~
