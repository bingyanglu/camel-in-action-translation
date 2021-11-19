# 在 Camel 中使用 Bean

------

**此章节涵盖**

> - 在 Camel 中调用 Java Bean
> - 理解服务激发器模式
> - Camel 如何通过注册查找到 Bean
> - Camel 如何选定Bean方法调用
> - 使用 Bean 参数绑定
> - 在路由中使用 Java Bean 作为谓词和表达式

​		如果你已经开发软件多年，你可能使用过多种组件模型，如：CORBA，EJB，JBI，SCA 和最近的 OSGi。它们中的一些，尤其是早期的一些，对编程模型施加了很大的影响，它决定了你能做什么，不能做什么，而且他们通常需要复杂的打包和部署过程。这给日常的程序员留下了很多需要学习和掌握的概念。在某些情况下，围绕程序限制和部署模型花费的时间比围绕业务应用程序本身要多得多。

​		由于这种日益复杂的复杂性和由此产生的挫折，从开源社区产生了一个更简单、更实用的编程模型：普通旧Java对象( POJO )模型。许多开源项目已经证明了 POJO 编程模型和轻量级容器满足了当今业务的期望。事实上，简单的编程模型和轻量级的容器概念被证明优于以前使用的重量级和过于复杂的企业应用程序和集成服务器。这一趋势一直持续到今天，我们看到了微服务和无容器部署的兴起。我们将在第7章中介绍微服务，但首先您需要了解更多关于 Camel 的基础知识，包括本章关于使用 Java Bean 和 Camel 的主题。		

​		那么 Camel 呢？Camel并不要求使用特定的组件或编程模型。它并没有要求一个你必须学习和理解才能富有成效沉重的规范。Camel 不需要您重新打包任何现有的库，也不需要您使用 Camal API 来满足你的集成需求。Camel 与 Spring 框架相同；两者都是支持 POJO 编程模型的轻量级容器。Camel 认识到了 POJO 编程模型的强大功能，并竭尽全力的使用bean。通过使用 Bean，您可以实现软件行业的一个重要目标：减少耦合。Camel 不仅通过 Bean 进行解耦，你也可以通过 Camel 路由进行解耦。例如：三个团队可以同时在他们自己的路由集上工作，它们可以很容易的组成一个系统。

​		本章首先向你介绍什么时候下不要在 Camel 中使用 Bean，这将使你更加清楚如何使用 Bean。之后，你将学习服务激发器模式背后的原理，并深入了解在 Camel 中是如何实现该模式的。接下来我们介绍 Bean 的绑定流程，它将可以让你对 Camel 中被调用方法上的参数绑定和当前路由信息进行更细粒度的控制。最后本章节介绍如何在你的路由中使用 Bean 作为谓词和表达式。



## 4.1 Bean 的好用法和坏用法

​		在本节，你将通过一个例子明白什么时候不要在 Camel 中使用 Bean（Bean 的不佳使用方式）。之后你将看到如何简洁的使用 Bean。

​		假设您有一个现有的bean，提供您需要在集成应用程序中使用的操作（服务）。例如，HelloBean 提供了 hello 方法作为其服务：

```java
public class HelloBean { 
  public String hello(String name) { 
    return "Hello " + name; 
  }
}
```

让我们通过不同的方式在你的应用中使用该 Bean。

### 4.1.1 使用 Java 调用 Bean

​		通过使用 Camel `Processor`，您可以从 Java 代码中调用 Bean，如下列表所示。

```java
public class InvokeWithProcessorRoute extends RouteBuilder {
    public void configure() throws Exception {
      	// ❶Uses a Processor
        from("direct:hello").process(new Processor() {	
                public void process(Exchange exchange) throws Exception {
                  	// ❷Extracts input from Camel message
                    String name = exchange.getIn().getBody(String.class);	
                    HelloBean hello = new HelloBean();
                    // ❸Invokes HelloBean
                    String answer = hello.hello(name);
	                  // ❹Response from HelloBean is set on OUT message 
                    exchange.getOut().setBody(answer);
                }
            });
    }
}
```

<center>清单4.1 使用 <code>Processor</code> 调用 HelloBean 的 hello 方法</center>

​		清单4.1展示了 `RouteBuilder` 如何定义路由。代码❶定义了 Camel 内部的 `Processor`类，其提供的 `process` 方法用来处理消息。在❷处必须从输入信息中提取消息体，这是你接下来调用 Bean 时使用的参数。之后在❸处你需要实例一个 Bean 并进行调用。最后在❹处将 Bean 的返回内容设置到输出消息中。

​		现在你已经通过使用 Java DSL 的方式完成了它，让我们来看看如何通过 XML DSL 的方式来完成。

### 4.1.2 使用 XML DSL 调用 Bean

​		当使用 Spring XML 或 OSGi Blueprint 来构建 Bean 容器时， Bean 都是基于 XML 文件定义的。清单4.2和4.3将展示如何通过 Spring Bean 的方式修正清单4.1中的代码。

```xml
<bean id="helloBean" class="camelinaction.HelloBean"/> 

<!--> ❶Defines HelloBean <-->
<bean id="route" class="camelinaction.InvokeWithProcessorSpringRoute"/> 

<camelContext id="camel" xmlns="http://camel.apache.org/schema/spring">
  <routeBuilder ref="route"/> 
</camelContext>
```

<center>清单4.2 配置 Spring，在 Camel 路由中使用 HelloBean</center>

​		首先在 Spring XML 文件中定义一个 id 为 “helloBean” 的 HelloBean，如果你仍想使用 Java DSL 构建路由，你需要定义一个包含路由的 Bean。最后，定义 `CamelContext`，让 Camel 和 Spring 可以一起运行。

```java
public class InvokeWithProcessorSpringRoute extends RouteBuilder {
		// ❶Injects HelloBean 
  	@Autowired 
    private HelloBean hello;

    public void configure() throws Exception {
        from("direct:hello").process(new Processor() {
                public void process(Exchange exchange)
                    throws Exception {
                    String name = exchange.getIn().getBody(String.class);
										// ❷Invokes HelloBean 
                  	String answer = hello.hello(name);  
                    exchange.getOut().setBody(answer);
                }
            });
    }
}

```

<center>清单4.3 Camel 路由使用 Processor 调用 HelloBean</center>

​		清单4.3中的路由与清单4.1中的路由几乎完全一致。它们的区别是当前的 Bean 通过 Spring 的 `@Autowird` 注解进行依赖注入（❶处）来代替原先的 Bean 实例化创建，你可以直接使用注入后的 Bean（❷处）。

​		你可以自己尝试这些例子；它们在本书源码的 chapter4/bean 目录中。运行以下 Maven 命令来尝试最后的两个案例：

```shell
mvn test -Dtest=InvokeWithProcessorTest 
mvn test -Dtest=InvokeWithProcessorSpringTest
```

​		到目前为止，你已经看到了两个 Camel 路由的案例，它需要一些管道才能正常工作。如你所见，由于以下原因，使用 Bean 是困难的：

- 你必须使用 Java 代码去调用 Bean。

- 你必须使用 Camel的 `Processor`，这使得路由变得混乱，无法理解会发生什么（路由逻辑和实现逻辑混淆在一起）。

- 你必须从 Camel 消息中提取数据将其传递给 Bean，并且您必须将bean中的返回结果放回到 Camel 消息中。 

- 您必须自己实例化bean，或者注入其依赖关系。

  
  
  现在让我们看看如何简化这些操作。

### 4.1.3 便捷的使用 Bean

