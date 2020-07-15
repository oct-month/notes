# Spring MVC自动配置

## Spring Boot自动配置好了Spring MVC

以下是Spring Boot对Spring MVC的默认配置：

- 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象(View)，视图对象决定如何渲染(转发/重定向)）
  - ContentNegotiatingViewResolver：组合所有的视图解析器。
  - 我们可以自己给容器中添加一个视图解析器，自动地将其组合进来。

```java
@SpringBootApplication
public class DemoApplication
{
	@Bean
	public ViewResolver myViewReolver()
	{
		return new MyViewResolver();
	}

	private static class MyViewResolver implements ViewResolver
	{
		public MyViewResolver()
		{
			super();
			System.out.println("MyViewResolver被使用");
		}

		@Override
		public View resolveViewName(String viewName, Locale locale) throws Exception
		{
			return null;
		}
	}
}

```

- 静态资源文件路径、webjars
- 静态首页访问
- favicon.ico
- 自动注册了Converter、Formatter
  - Converter：转换器，类型转换
  - Formatter：格式化器，自己添加的格式化转换器只需要放在容器中即可
- HttpMessageConverters支持
  - Spring MVC用来转换http请求和响应
  - 自己添加HttpMessageConverters，只需要放在容器中就行（@Bean）
- 定义错误代码生成规则
- ConfigurableWebBindingInitializer
  - 可以自己配置一个来替换默认的

## 如何修改Spring Boot的默认配置

模式：

​	1）Spring Boot在自动配置组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）；如果有就用用户配置的，如果没有就自动配置。如果有些组件可以有多个，将用户配置的和自己默认的组合起来。

​	2）自己编写一个配置类@Configuration，WebMvcConfigurerAdapter类型，不标注@EnableWebMvc。

```java
@Configuration
public class MyMvcConfi extends WebMvcConfigurerAdapter
{
    @Override
    public void addViewControllers(ViewControllerRegistry registry)
    {
        registry.addViewController("/hello").setViewName("success");
    }
}
```

​	这样配置后，浏览器发送 /hello 请求也会映射到 success 页面；但是Spring Boot自动配置的映射规则还在。

​	3）全面接管Spring MVC，@EnableWebMvc。

```java
@EnableWebMvc
@Configuration
public class MyMvcConfi extends WebMvcConfigurerAdapter
{
    @Override
    public void addViewControllers(ViewControllerRegistry registry)
    {
        registry.addViewController("/hello").setViewName("success");
    }
}
```

​	这样配置后，Spring Boot对Spring Mvc的自动配置就都失效了，所有的配置都需要自己来。