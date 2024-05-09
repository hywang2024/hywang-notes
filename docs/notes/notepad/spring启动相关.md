# Spring启动

## Servlet去web.xml

spring在3.0版本之前使用的是web.xml方式启动web

spring从3.1版本开始基于Servlet3.0实现无web.xml方式启动web项目

关键是spring的WebApplicationInitializer类，继承这个类，在启动项目的时候，容器会调用onStartup方法来启动项目加载配置

## Spring启动

在Servlet中为了支持可以不使用web.xml，所以提供了`ServletContainerInitializer`，它可以通过SPI机制，当启动web容器的时候，会自动加载`META-INF/services`下以`META-INF/services/javax.servlet.ServletContainerInitializer`中配置的类`ServletContainerInitializer`

启动后调用`onStartup`方法

```java
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {
	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)throws ServletException { 
		List<WebApplicationInitializer> initializers = Collections.emptyList(); 
		if (webAppInitializerClasses != null) {
			initializers = new ArrayList<>(webAppInitializerClasses.size());
			for (Class<?> waiClass : webAppInitializerClasses) {
				// Be defensive: Some servlet containers provide us with invalid classes,
				// no matter what @HandlesTypes says...
				if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
						WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
					try {
						initializers.add((WebApplicationInitializer)
								ReflectionUtils.accessibleConstructor(waiClass).newInstance());
					}
					catch (Throwable ex) {
						throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
					}
				}
			}
		}

		if (initializers.isEmpty()) {
			servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
			return;
		}

		servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
		AnnotationAwareOrderComparator.sort(initializers);
		for (WebApplicationInitializer initializer : initializers) {
			initializer.onStartup(servletContext);
		}
	}
}
```

通过`@HandlesTypes`可以将感兴趣的一些类注入到`ServletContainerInitializerde`的`onStartup()`方法作为参数传入 

在`onStartup()`方法中会将所有感兴趣的类注入到参数`webAppInitializerClasses`中，使用迭代器将所有感兴趣的类加入到集合`initializers`中，最后会循环调用`initializers`中类的`onStartup()`方法

Spring中注入的类是`WebApplicationInitializer`，在他的抽象实现类中`AbstractDispatcherServletInitializer`重写父类`onStartup()`

`onStartup()`中调用`registerDispatcherServlet`，首先通过`createDispatcherServlet`创建了`DispatcherServlet`对象，并将`DispatcherServlet`加入到servlet容器中 

```java
public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {

	/**
	 * The default servlet name. Can be customized by overriding {@link #getServletName}.
	 */
	public static final String DEFAULT_SERVLET_NAME = "dispatcher";


	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		super.onStartup(servletContext);
		registerDispatcherServlet(servletContext);
	}
    protected void registerDispatcherServlet(ServletContext servletContext) {
		String servletName = getServletName();
		Assert.hasLength(servletName, "getServletName() must not return null or empty");

		WebApplicationContext servletAppContext = createServletApplicationContext();
		Assert.notNull(servletAppContext, "createServletApplicationContext() must not return null");

		FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);
		Assert.notNull(dispatcherServlet, "createDispatcherServlet(WebApplicationContext) must not return null");
		dispatcherServlet.setContextInitializers(getServletApplicationContextInitializers());

		ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
		if (registration == null) {
			throw new IllegalStateException("Failed to register servlet with name '" + servletName + "'. " +
					"Check if there is another servlet registered under the same name.");
		}

		registration.setLoadOnStartup(1);
		registration.addMapping(getServletMappings());
		registration.setAsyncSupported(isAsyncSupported());

		Filter[] filters = getServletFilters();
		if (!ObjectUtils.isEmpty(filters)) {
			for (Filter filter : filters) {
				registerServletFilter(servletContext, filter);
			}
		}

		customizeRegistration(registration);
	}
    protected FrameworkServlet createDispatcherServlet(WebApplicationContext servletAppContext) {
		return new DispatcherServlet(servletAppContext);
	}
    protected String getServletName() {
		return DEFAULT_SERVLET_NAME;
	}
}
```

提到`DispatcherServlet`大家都很熟悉了吧，接下来就是Spring中MVC的启动流程了

为了方便理解，自己写了一个上面的启动过程

先引入相关依赖

```xml
 	  <!-- 引入servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <!-- jetty  启动servlet -->
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-server</artifactId>
            <version>11.0.2</version>
        </dependency>
```

创建SPI加载类

```java
/**
 * @className: javax.servlet.ServletContainerInitializer-> HYServletContainerInitializer
 * @description: servlet 启动加载 onStatup
 * 使用@HandlesTypes加入webAppInitializerClasses
 * 注意 @HandlesTypes 必须是父类，子类加入不进去
 * @author: hywang
 * @createDate: 2021-04-27 19:49
 * @version: 1.0
 */
@HandlesTypes(HYWebApplicationInitializer.class)
public class HYServletContainerInitializer implements ServletContainerInitializer {
    @Override
    public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {

        if (webAppInitializerClasses != null) {
            Iterator<Class<?>> iterator = webAppInitializerClasses.iterator();
            if (iterator.hasNext()) {
                Class<?> clazz = iterator.next();
                try {
                    HYWebApplicationInitializer webApplicationInitializer = (HYWebApplicationInitializer) ReflectionUtils.accessibleConstructor(clazz).newInstance();
                    webApplicationInitializer.onStartup(servletContext);
                } catch (Throwable ex) {
                    throw new ServletException("Failed to instantiate HYWebApplicationInitializer class", ex);
                }
            }
        }
    }
}
```

并在resource下的META-INF/services中创建javax.servlet.ServletContainerInitializer

```
com.hywang.springframework.web.HYServletContainerInitializer
```

注入到servlet中的启动类

```java
/**
 * @description: servlet 启动加载, web.xml的功能一樣
 * @author: hywang
 * @createDate: 2021-04-27 19:39
 * @version: 1.0
 */
public interface HYWebApplicationInitializer {
    void onStartup(ServletContext servletContext) throws ServletException;
}
/**
 * @description: servlet容器中注册自定义servlet
 * @author: hywang
 * @createDate: 2021-04-27 19:45
 * @version: 1.0
 */
public class HYWebInitializer implements HYWebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        String servletName = "test";
        String mapping = "/test";
        String servlet = "com.hywang.springframework.servlet.MyServlet";
        ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, servlet);
        if (registration == null) {
            throw new IllegalStateException("Failed to register servlet with name '" + servletName + "'. " +
                    "Check if there is another servlet registered under the same name.");
        }

        registration.setLoadOnStartup(1);
        registration.addMapping(mapping);
        registration.setAsyncSupported(true);
    }
}

```

```java
/**
 * @description: 自定义servlet
 * @author: hywang
 * @createDate: 2021-04-27 19:43
 * @version: 1.0
 */
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("HYSetvlet");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

}

```

最后通过jetty启动容器，浏览器输入http://127.0.0.1:8180/test

