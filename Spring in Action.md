
##Spring in Action##

###Spring AOP###

Spring‘s AOP configuration elements enable non-invasive declaration of aspects

- <aop:advisor>
- <aop:after>
- <aop:after-returning>
- <aop:after-throwing>
- <aop:around>
- <aop:aspect>
- <aop:aspectj-autoproxy>
- <aop:before>
- <aop:config>
- <aop:declare-parents>
- <aop:pointcut>

----
Declaaring before and after adivce
	
	<aop:config>
		<aop:aspect ref="audience">
			<aop:before
				pointcut="execution(** concert.Performance.perform(..))"
				method="silenceCellPhones" />
			<aop:before
				pointcut="execution(** concert.Performance.perform(..))"
				method="takeSeats" />
			<aop:after-returning
				pointcut="execution(** concert.Performance.perform(..))
				method="applause" />
			<aop:after-throwing
				pointcut="execution(** concert.Performance.perform(..))"
				method="demandRefund" />
		</aop:aspect>
	</aop:config>


Introducing new functionality with aspects

	<aop:aspect>
		<aop:declare-parents
			types-matching="concert.Performance+"
			implement-interface="concert.Encoreable"
			default-impl="concert.DefaultEncoreable"
			/>
	
	</aop:aspect>

*default-impl* attribute explicitly identify the implementation by its fully qualified class name, which also can be instead by *delegate-ref* attribute

Injection AspectJ aspects

	<bean class="com.springaction.springidol.CriticAspect"
			factory-method="aspectOf">
		<property name="criticismEngine" ref = "criticismEngine" />
	</bean>

Spring donesn't use the <bean> declaration from earlier to create an instance of the CriticAspect, it has already been created by the AspectJ runtime. Instead, Spring retrieves a reference to the aspect through the *aspectOf()* factory method and then performs dependency injection on it as prescribed by the <bean> element.

### spring on the web ###

In a Servlet 3.0 environment, the container looks for any classes in the classpath that implement the **javax.servlet.ServletContainerInitializer** interface, if any are found, they're used to configure the servlet container. Spring suppilies an implementation of that interface called **SpringServletContainerInitializer** and since Spring 3.2 it introduced a convenient base implementation of **WebApplicationInitializer** called **AbstractAnnotationConfigDispatcherServletInitializer**, so any Class implments **AbstractAnnotationConfigDispatcherServletInitializer** will be automatically discovered when deployed in servlet 3.0 container and be used to configure the servlet context. It's an alternative to the **web.xml** file.

	package spittr.config
	import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
	
	public class SpittrWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{
		
		@Override
		protected String[] getServletMapping(){
			return new String[]{"/"};
		}

		@Override
		protected Class<?>[] getRootConfigClasses(){
			return new Class<?>[]{RootConfig.class};
		}

		@Override
		protected Class<?>[] getServletConfigClassess(){
			return new Class<?>[] { WebConfig.class};
		}

	}

`AbstractAnnotationConfigDispatchServletInitializer` creates both a `DispatcherServlet` and a `ContextLoaderListener`


#### Enable Spring MVC ####

<mvc:annotation-driven> element enable annotation-driven Spring　MVC.

@EnableWebMvc
