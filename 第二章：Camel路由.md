# 第二章：Camel路由

- Camel路由介绍
- 引入“骑手摩配”场景
- FTP和JMS端点基础用法
- 使用Java DSL创建路由
- 在XML中定义路由
- 在路由中使用EIP

Camel的最重要的特性是路由，没有路由，Camel就只是一个传输连接库。在本章中，你将深入了解如何使用Camel路由数据。

在日常生活中，路由的理念无处不在。例如，你寄出一封信，辗转好几个城市，最终到达目的地；发送的电子邮件，在到达目的地之前，也会经过多个网络。通常来说，路由都是指有选择地推动消息向前移动。

在企业软件系统间进行消息传递的场景下，路由就是将消息从输入队列中取出并根据一组预设的条件发送到多个输出队列中的过程，如图2.1所示。输入和输出队列并不知道消息传递的条件。消息传递的逻辑与消息的生产者和消费者之间是解耦的。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114408)

图2.1 消息路由接收输入队列传来的消息，并根据一组条件将消息发送到输出通道中的一个队列

在Camel中，路由是一个更普通的概念。消息，从我们称之为消费者的端点进入路由，Camel指挥消息一步一步地移动，这就是路由。消费者端点可以从外部服务接收消息、也可以从外部数据源上轮询得到消息、甚至可以直接创建一个消息。这些消息会在Camel的路由定义中流经处理节点，处理节点可以是企业集成模式(EIP)、处理器、拦截器或另一个自定义组件。消息最终被发送到称之为生产者的目标端点。一个路由可以包含多个处理节点，每个处理节点都可以对消息进行修改或者将其发送到另一个位置。路由也可以没有处理组件，没有处理节点时，路由就是一个简单的、接通数据源和数据目标的管道。

本章我们将介绍一个虚构公司，这是一个贯穿全书的例子，我们会为这家公司解决一些实际问题。在本章中，我们将了解如何使用Camel端点与FTP和Java Message Service(JMS)进行通信。然后，我们将深入研究用于创建路由的基于java的领域特定语言(DSL)和基于xml的DSL。我们还将简要介绍如何使用EIP及Camel来设计和实现针对企业集成问题的解决方案。在本章结束时，你将能够熟练地使用Camel来创建能够解决实际问题的路由应用。

首先，让我们看看这家贯穿全书的虚拟公司。



## 2.1 “骑手摩配”

我们假设有一家摩托车配件生产销售为主营业务的公司，名叫“骑手摩配”，这家公司是摩托车制造厂的零配件供应商。这家公司发展了很多年，“骑手摩配”的技术架构已经迭代了数轮，接收订单的方式一变再变。最初，客户需要将CSV文件上传到FTP服务器来下订单，消息格式后来更改为XML。再后来，公司提供了一个网站，通过该网站，订单可以通过HTTP以XML消息的形式提交。

“骑手摩配”现在要求新客户使用HTTP接口下单，但是由于之前与老客户之间签订了服务水平协议（SLA），公司必须保持所有老的数据交换接口可以以老的数据格式继续正常运行。公司在处理这些订单之前，都会将其转换为一个普通Java对象（POJO）之后再进行处理。订单处理系统的简单架构图如图2.2所示。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114419)

图2.2 客户有两种方法向“骑手摩配”订单处理系统提交订单：将原始订单文件上传到FTP服务器，或者通过“骑手摩配”在线商城提交订单。所有订单最终都将通过JMS发送到“骑手摩配”的后端业务系统进行处理。

“骑手摩配”和很多公司一样面临着同一个问题：经过多年的运营，每个版本的数据传输方式和数据格式就成了现在的技术包袱。好在使用像Camel这样的集成框架可以轻而易举的解决这些问题。在本章以及本书后续章节中，你将使用Camel帮助“骑手摩配”满足现有需求、实现新的功能。

首先，你将在“骑手摩配”公司的前置系统中，实现一个FTP模块。然后，你将了解后端服务的实现细节。

实现一个FTP前置模块包括以下步骤：

1. 从FTP服务器上检查并下载新订单文件
2. 将订单文件转换为JMS消息
3. 将消息发送到JMS的incomingOrders队列

要完成第1步和第3步，你需要了解如何使用Camel端点建立与FTP和JMS的通信。要完成整个任务，你还需要了解如何使用Java DSL进行路由。首先，让我们看看如何使用Camel端点。

## 2.2 理解端点

正如你在第1章中所读到的，端点是一种抽象，它是Camel对消息通道末端的建模，软件系统可以通过这些通道发送或接收消息。本节将解释如何使用uri配置Camel端点，建立FTP和JMS的通信通道。让我们先看看FTP。

### 2.2.1 从FTP端点消费数据

Camel简单易用的一个主要原因是端点URI的设计。通过端点URI，你可以标识要使用的组件和对该组件的相关配置。然后可以决定将改组件作为消息生产者，发送消息到由该URI配置的组件，还是将该组件作为消息消费者，从该组件中获取消息。

我们来看第一个“骑手摩配”的业务场景。要从FTP服务器下载新订单，你需要执行以下操作：

1. 使用默认端口21连接到`rider.com`的FTP服务器。
2. 提供用户名`rider`和密码`secret`。
3. 更改FTP当前目录为`orders`文件夹。
4. 如果有新的订单文件，则进行下载。

如图2.3所示，你可以非常轻松地使用一串URI来配置Camel实现这一点。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114427)

图2.3 Camel端点的URI由三个部分组成：方案（Scheme）、上下文路径（Context Path）和参数（Option）列表

Camel首先在组件的注册表中查找FTP的连接方案，最终通过注册表解析得到使用为FTP组件（`FtpComponent`）。然后，Camel使用FTP组件作为工厂，根据上下文路径和参数来创建FTP端点（`FtpEndpoint`）。

上下文路径rider.com/orders告诉FTP组件它应该通过默认FTP端口登录到rider.com的FTP服务器，并将目录更改为orders。最后，选项指定了用户名和密码，它们用于登录FTP服务器。

> 注意，对于FTP组件，还可以在URI的上下文路径中指定用户名和密码，[ftp://rider:secret@rider.com/orders](https://links.jianshu.com/go?to=ftp%3A%2F%2Frider%3Asecret%40rider.com%2Forders)这串URI与图2.3中的URI作用相同。说到密码，用明文定义它们通常不是一个好主意！你将在第14章中了解如何使用加密的密码。

Ftp组件并不在camel-core模块内，所以你需要给你的工程添加一个额外的依赖包。使用maven的时候，需要如下的依赖到pom文件：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-ftp</artifactId>
    <version>2.20.1</version>
</dependency>
```

这个端点URI在无论是作为使用者还是作为生产者都有效，在当前的场景下，我们是用它从FTP服务器上下载订单。要做到这一点，你需要在Camel DSL的from节点中使用它：

```java
from("ftp://rider.com/orders?username=rider&password=secret")
```

如果你要从FTP服务器消费文件，这一行代码就够了。

你可以回顾一下图2.2，接下来我们需要做的是把FTP服务器上下载下来的订单发送到JMS队列。这个过程需要更多的设置，但仍然很容易。

### 2.2.2 发送消息到JMS端点

所有支持JMS的中间件Camel都支持，相关内容我们将在第6章中详细介绍。现在，到目前为止，你只需要了解最基本的内容，以便你可以完成“骑手摩配”的第一个应用场景，即从FTP服务器下载订单并将其发送到JMS队列。

**什么是JMS？**

Java Message Service (JMS)是一个Java API，它可以创建、发送、接收和读取消息。JMS要求消息传递是异步的，具有一些可靠性的设计，例如JMS可以提供一次消息发送仅有一次消息接收的方案。JMS可能是Java社区中部署最广泛的消息传递解决方案。

在JMS中，消息使用者和生产者通过一个中介（JMS目的地）相互通信。如图2.4所示，目的地可以是队列也可以是主题。队列是严格的点对点消息通讯，每条消息只会被一个消费者消费。主题基于发布/订阅模式，如果有多个用户订阅了该主题，那么一条消息将发送给多个用户。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114435)

图2.4 JMS目的地有两种类型:队列和主题。该队列是点对点通道，每封邮件只有一个收件人。主题将消息的副本发送给所有订阅了该消息的客户端。

JMS同样会提供一个`ConnectionFactory`，以便于客户端（例如Camel）可以使用它来创建一个与JMS服务的通讯连接。JMS服务有时又被称之为JMS消息代理，因为它们代替业务系统来实现消息生产者和消费者之间的消息通讯。

**如何配置Camel以连接JMS服务**

要将Camel连接到特定的JMS提供程序，你需要使用适当的`ConnectionFactory`来配置Camel的JMS组件。Apache ActiveMQ是最流行的开源JMS提供者之一，它是Camel团队用来测试JMS组件的主要JMS代理。因此，我们将使用它来进行演示本书中的JMS概念。

更多关于Apache ActiveMQ的信息，我们推荐Bruce Snyder等人的ActiveMQ in Action (Manning, 2011)。

在使用Apache ActiveMQ的情况下，你可以创建一个`ActiveMQConnectionFactory`，由它来指定正在运行的ActiveMQ代理地址：

```java
ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("vm://localhost");
```

URI`vm://localhost`意味着你应该连接到运行在当前JVM中，一个名为localhost的嵌入式代理。如果代理还没有运行，ActiveMQ中的vm传输连接器将按需创建一个代理，因此非常适合拿来快速构建一个测试用的JMS应用程序。对于生产场景，建议连接到已经运行的代理。此外，在生产场景中，我们建议在连接到JMS代理时使用连接池。有关这些额外配置的详细信息，请参阅第6章。

接下来，在创建`CamelContext`时，可以添加JMS组件，如下所示:

```java
CamelContext context = new DefaultCamelContext();
context.addComponent("jms",JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
```

JMS组件和Activemq的ConnectionFactory不是camel-core模块的一部分。要使用它们，需要将依赖项添加到maven项目中。对于普通JMS组件，你只需添加以下内容：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-jms</artifactId>
    <version>2.20.1</version>
</dependency>
```

JMS的连接工厂来自于ActiveMQ的相关API，所以你还需要以下依赖项：

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.2</version>
</dependency>
```

现在，你已经将JMS组件配置为连接到特定的JMS代理，下面就来看看如何使用uri来指定目的地。

**使用URI来指定目的地**

配置好JMS组件后，你就可以随心所欲地发送和接收JMS消息了。因为使用的是uri，所以配置起来非常简单。

假设你希望向名为`incomingOrders`的队列发送一条JMS消息，URI如下:

```css
jms:queue:incomingOrders
```

这串URI本身就已经说明了它的用途。jms前缀表示你正在使用之前配置的jms组件。通过指定queue, JMS组件知道目的地是名为`incomingOrders`的队列。你甚至可以省略queue那一部分，因为JMS组件的默认行为就是发送到队列而不是主题。

> 注意，有些端点可能具有巨量的URI参数列表。例如，JMS组件有80多个参数，其中许多参数仅用于特定的JMS场景。大多数情况下，Camel的默认值就已经可以适应业务场景，你可以查看Camel的官方文档中的组件页面来确认这些参数的默认值。例如这里讨论JMS组件，官方文档在：[http://camel.apache.org/jms.html](https://links.jianshu.com/go?to=http%3A%2F%2Fcamel.apache.org%2Fjms.html)。

使用Camel的Java DSL，你可以使用to关键字像这样发送消息到`incomingOrders`队列：

```java
... to("jms:queue:incomingOrders")
```

这一串URI可以读作：发送到`to`名为`incomingOrders`的JMS队列`queue`。

现在你已经了解了使用Camel与FTP和JMS进行通信的基础知识，现在我们言归正传，开始路由数据。

# 2.3 在Java中创建一个数据路由

在第一章中，你了解了可以使用`RouteBuilder`来创建一个路由，并且每个`CamelContext`可以包含多个路由。不过有一点需要明确的，`CamelContext`在运行时并不会使用`RouteBuilder`作为最终路由定义，`RouteBuilder`只是一个路由的构建器，由它构建出的一个或者多个路由，会添加到`CamelContext`中进行运行，如图2.5所示。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114448)

图2.5 `RouteBuilders`被Camel用来创建路由。每个`RouteBuilder`都可以创建一个或者多个路由。

> 特别注意，RouteBuilder和Route在概念上的区别非常重要。在RouteBuilder中编写的DSL代码，无论是Java DSL还是XML DSL，都只是一个设计时的构造，Camel在启动时只使用一次。例如，你可以在IDE中调试从RouteBuilder构建路由。我们将在第8章中介绍更多关于调试Camel应用程序的内容。

`CamelContext`的`addRoutes`方法接收一个`RoutesBuilder`，注意，这个方法接收的不是`RouteBuilder`。`RoutesBuilder`接口里面只有一个简单的方法定义：

```java
void addRoutesToCamelContext(CamelContext context) throws Exception;
```

理论上，你可以使用自己的自定义类来构建Camel路由。但并不建议你这么做。Camel提供了`RouteBuilder`类，它实现了`RoutesBuilder`接口，并且还可以使用Camel的Java DSL来创建路由。

在下一节中，你将学习如何使用`RouteBuilder`和Java DSL创建简单路由。然后，你将在2.4节中学习使用XML DSL、在2.6节中学习使用EIP进行路由。

### 2.3.1 使用RouteBuilder

Camel的`org.apache.camel.builder.RouteBuilder`抽象类会非常频繁的出现，你都需要使用它来使用Java DSL构建路由。

要使用`RouteBuilder`类，你可以自己写有一个类来继承它，然后实现其中的`configure`方法，例如：

```java
public class MyRouteBuilder extends RouteBuilder {
    public void configure() throws Exception {
        ...
    }
}
```

然后你需要将它的实例通过`addRoutes`方法添加到`CamelContext`中：

```java
CamelContext context = new DefaultCamelContext();
context.addRoutes(new MyRouteBuilder());
```

或者，你也可以通过直接在`CamelContext`中添加一个匿名`RouteBuilder`类来合并`RouteBuilder`和`CamelContext`配置，如下：

```java
CamelContext context = new DefaultCamelContext();
context.addRoutes(new RouteBuilder() {
    public void configure() throws Exception {
        ...
    }
});
```

在`configure`方法中，可以使用Java DSL定义路由。我们将在下一节详细介绍Java DSL，现在我们将简单介绍它是如何工作的。

在第1章中，你应该在GitHub上从该书的源代码下载源代码并设置过Apache Maven。如果你没有这样做，请现在去做。我们还将使用Eclipse来演示Java DSL概念。

> 注，Eclipse是一个流行的开放源码IDE，你可以在[http://eclipse.org](https://links.jianshu.com/go?to=http%3A%2F%2Feclipse.org)上找到它。在书的开发过程中，乔恩使用了Eclipse，克劳斯使用了IDEA。当然也可以使用其他Java IDE，甚至不使用IDE，但是使用IDE确实使Camel的开发更加容易。如果你不想看到与IDE相关的设置，请直接跳到下一节。在第19章中，你将看到一些可以安装在Eclipse或IDEA中的额外的Camel工具，这些工具可以使Camel开发更加出色。

在设置Eclipse之后，你应该将本书源代码的chapter2/ftp-jms目录作为Maven项目导入。

在Eclipse中加载ftp-jms项目时，打开`src/main/java/camelinaction/RouteBuilderExample.java`文件。如图2.6所示，当你在configure方法中尝试自动完成(Eclipse中的Ctrl+空格)时，你将看到几个方法。要开始一个路由，你应该使用from方法。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114458)

图2.6 可以使用IDE的自动完成功能来写路由。所有的路由都以一个from节点开始。

from方法接受端点URI作为参数，你可以添加一个FTP端点URI来连接到“骑手摩配”的订单服务器，如下所示：

```java
from("ftp://rider.com/orders?username=rider&password=secret")
```

from方法会返回一个`RouteDefinition`对象，你可以在该对象上调用实现EIP和其他消息传递概念中的各种方法。

祝贺你，你已经开始使用Camel的Java DSL了！让我们仔细看看这里发生了什么。

### 2.3.2 使用Java DSL

领域限定语言（Domain-specific languages）即DSL，是针对**特定问题领域**的计算机语言，它与大多数编程语言不同，多数编程语言，例如Java，是针对**通用领域**的计算机语言。例如，你可能已经使用正则表达式DSL来匹配文本字符串，正则是一种匹配字符串的简洁方法，而在Java中不同正则实现相同的功能就没那么容易。正则表达式DSL是一个外部的DSL、它有自定义的语法，因此需要一个单独的编译器或解释器来执行。与外部DSL相对的是内部DSL，它使用现有的通用语言（如Java），使DSL感觉像是来自特定领域的语言。最简单的方法是通过特定的方法命令和参数来匹配相关领域的概念。

实现内部DSL的另一种比较流行的方法是使用链式编程接口（也称为链式构建器）。在使用链式编程接口时，可以将一串方法调用链接在一起来构建对象。最终执行一个操作，返回构建对象实例。

> 注，有关内部DSL的更多信息，请参见Martin Fowler在他的bliki(博客+ wiki)上的“领域特定语言”条目:[www.martinfowler.com/bliki/DomainSpecificLanguage.html](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.martinfowler.com%2Fbliki%2FDomainSpecificLanguage.html)。他还在[www.martinfowler.com/bliki/FluentInterface.html](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.martinfowler.com%2Fbliki%2FFluentInterface.html)上有一个关于“链式接口”的条目。关于DSL的更多信息，我们推荐Debasish Ghosh (Manning, 2010)的《DSLs in Action》。

Camel的领域是企业集成，它的Java DSL是一组链式构建器，这个链式构建器包含各种EIP的术语命名的方法。在Eclipse编辑器中，可以在RouteBuilder中的from方法之后使用自动完成功能来看后续可以执行哪些操作。你应该会看到如图2.7所示的内容。屏幕截图显示了两个EIP——Enricher和Recipient List，还有许多其他的EIP，我们将在后面讨论。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114506)

图2.7 使用from方法后，使用IDE的自动完成特性来获得EIP列表（比如Enricher和Recipient List）和其他有用的集成功能。

现在，选择to方法，传入字符串“jms:incomingOrders”，并用分号完成路由。在RouteBuilder中，每个以from方法开始的Java语句都会创建一个新路由。这个新路由现在完成了“骑手摩配”的第一个业务场景：使用FTP服务器上的订单并将其发送到名为incomingOrders的JMS队列。如果你愿意，可以从本书的源代码中以chapter2/ftpjms的形式加载完成的示例，并打开src/main/java/camelinaction/FtpToJMSExample.java。

代码如下所示：

***清单2.1拉取FTP消息，并发送到incomingOrders队列\***

```java
import javax.jms.ConnectionFactory;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.component.jms.JmsComponent;
import org.apache.camel.impl.DefaultCamelContext;
public class FtpToJMSExample {
public static void main(String args[]) throws Exception {
    CamelContext context = new DefaultCamelContext();
    ConnectionFactory connectionFactory =
    new ActiveMQConnectionFactory("vm://localhost");
    context.addComponent("jms",
    JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
    context.addRoutes(new RouteBuilder() {
        public void configure() {
                from("ftp://rider.com/orders" 
                     + "?username=rider&password=secret") 
                .to("jms:incomingOrders"); // ❶使用java语言构建路由
            }
        });
        context.start();
        Thread.sleep(10000);
        context.stop();
    }
}
```

> 注意，因为你是从[ftp://rider.com](https://links.jianshu.com/go?to=ftp%3A%2F%2Frider.com)消费数据，而这个ftp地址并不存在，所以你不能运行这个例子。它仅用于演示Java DSL构造。关于可运行的FTP示例，请参阅第6章。

如你所见，这个清单包括一些环境设置和配置。真正用来解决问题的只有其中的一串简单Java语句❶。from方法告诉Camel使用来自FTP端点的消息，to方法告诉Camel将消息发送到JMS端点。

这个简单路由中的消息流可以看作是一个基本管道，消费者的输出作为生产者的输出。如图2.8所示。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114518)

图2.8 从文件到JMS消息的载体转换是自动完成的

你可能已经注意到，我们没有做任何从FTP文件类型到JMS消息类型的数据转换——这一步是通过Camel的类型转换工具自动完成的。Caml允许你在路由执行过程中的任何节点上做强制类型转换，但是大部分情况下你不需要自己做转换。数据转换和类型转换将在第3章中详细介绍。

你可能会想，虽然这个路由很好，也很简单，但是如果能看看路由中间发生了什么就更好了。幸运的是，**Camel提供了流式钩子和将行为特性注入**，让开发人员对其进行控制。有一种使用Processor访问消息的简单方法，我们将在下面讨论。

**添加一个Processor**

Camel中的Processor接口是复杂路由的重要组成部分。它是一个简单的接口，只有一个方法:

```java
public void process(Exchange exchange) throws Exception;
```

这个方法为你提供了对消息交换的完全访问权，让你可以对消息负载或消息头执行几乎任何你想要的操作。Camel中的所有EIP都是作为Processor实现的。你甚至可以添加一个简单的内联Processor到你的路由，像这样：

```java
from("ftp://rider.com/orders?username=rider&password=secret")
    .process(new Processor() {
    public void process(Exchange exchange) throws Exception {
        System.out.println("We just downloaded: "
        + exchange.getIn().getHeader("CamelFileName"));
     }
})
.to("jms:incomingOrders");
```

此路由现在将在控制台中打印已下载的订单的文件名，然后再将其发送到JMS队列。

通过将这个Processor添加到路由的中间，就可以将它添加到前面提到的路由的管道中，如图2.9所示。FTP消费者的输出作为输入，输入到Processor，Processor不修改消息负载或消息头，Exchange将这个Processor的输出到作为输入的JMS生成器。

> 注意，许多组件（如FileComponent和FtpComponent）会在消费到数据的时候设置一些消息头，这些消息头对传入消息负载进行了描述。在前面的示例中，你使用了一个消息头CamelFileName来判断通过FTP下载的文件的文件名。Camel的官方文档中包含了每个组件设置的消息头信息。你可以在[http://camel.apache.org/ftp.html](https://links.jianshu.com/go?to=http%3A%2F%2Fcamel.apache.org%2Fftp.html)上找到关于FTP组件的信息。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114527.)

图2.9 通过混入一个Processor，现在将FTP消费者的输出将输入到Processor中，然后将Processor的输出输入到JMS生产者中。

Camel创建路由的主要方法之一是通过Java DSL。毕竟，它是内置在Camel-core模块中的。不过，还有其他创建路由的方法，其中一些可能更适合你的业务场景。例如，Camel为在XML中编写路由提供了扩展，我们将在下面讨论。

## 2.4 在XML中定义路由

对于有经验的Java开发人员来说，Java DSL无疑是一种更强大的选择，它可以生成更简洁的路由定义。但是如果你可以在XML中定义这些东西，将会带来更多的可能性。也许有些用户在编写Camel路由的时候不太习惯Java。例如，我们知道许多系统管理员很容易地编写Camel路由来解决集成问题，但他们一生中从未使用过Java。有了XML DSL，你才可能使用各种简单易用的图形化工具❶来读写路由定义；你可以通过拖拉拽的形式来编辑路由，并且将其部署到camel的运行时环境中。当然，基于Java DSL去写这种读取和编辑工具也是可能的，但非常困难，到目前为止还没有这类工具可用。

------

❶ 可以在第19章了解更多有关Camel工具的信息。

------

在撰写本文时，你可以在两个控制反转（IoC） Java容器中编写XML路由：Spring DM和OSGi Blueprint。IoC框架允许你将各种bean“连接”在一起以形成应用程序。这种连接通常通过XML配置文件完成。本节将简要介绍如何使用Spring DM来创建应用程序，从而使IoC概念更加清晰。然后，我们将向您展示Camel如何使用Spring DM形成Java DSL的替代和补充解决方案。

> 注，如果想了解更多关于Sprign的相关信息，我们推荐你阅读Craig Walls的《Spring in action》 (Manning, 2014)。OSGi Blueprint则在另一本由Richard S. Hall等人的《OSGi in Action》(Manning, 2011) 进行了详细的讲解。

Spring DM和OSGi Blueprint之间的设置当然是不一样的，但它们都是在做相同的路由定义，因此我们在本章只讨论基于Spring DM的示例。在本书的其余部分中，我们仅将Spring或Blueprint中的路由称为XML DSL。

### 2.4.1 Bean注入和Spring

使用Spring以Bean为基础创建应用非常简单，你所需要的只是几个Java bean（类）、一个Spring XML配置文件和ApplicationContext。ApplicationContext类似于CamelContext，因为它是Spring的运行时容器，我们来看一个简单的例子。

有这样一个应用，它通过拼接字符串对用户进行问候。在这个应用程序中，你不希望greeting是硬编码的，因此可以使用接口来打破这种依赖关系。

所以我们采用接口：

```java
public interface Greeter {
    public String sayHello();
}
```

这个接口被下面两个类实现：

```java
public class EnglishGreeter implements Greeter {
    public String sayHello() {
        return "Hello " + System.getProperty("user.name");
    }
} 

public class DanishGreeter implements Greeter {
    public String sayHello() {
        return "Davs " + System.getProperty("user.name");
    }
}
```

你现在就可以创建一个问候语应用，如下：

```java
public class GreetMeBean {
    private Greeter greeter;
    public void setGreeter(Greeter greeter) {
        this.greeter = greeter;
    }
    public void execute() {
        System.out.println(greeter.sayHello());
    }
}
```

该应用程序将根据配置输出不同的语言的问候语，要使用Spring XML配置这个应用程序，你可以这样做：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/springbeans.
xsd">
<bean id="myGreeter" class="camelinaction.EnglishGreeter"/>
    <bean id="greetMeBean" class="camelinaction.GreetMeBean">
        <property name="greeter" ref="myGreeter"/>
    </bean>
</beans>
```

这个XML文件告诉Spring应该执行以下操作：

1. 创建一个`EnglishGreeter`类的实例，名为`myGreeter`。
2. 创建一个`GreetMeBean`类的实例，名为`greetMeBean`。
3. 将`GreetMeBean`的`greeter`属性的引用设置为名为`myGreeter`的bean。

这种对于bean的配置，我们称之为wire，即连接。

要将这个XML文件加载到Spring中，可以使用ClassPathXmlApplicationContext，它是Spring框架提供的ApplicationContext的具体实现。它可以从类路径中指定的位置加载Spring XML文件。

这里是`GreetMeBean`的最终版本：

```java
public class GreetMeBean {
    public static void main(String[] args) {
        ApplicationContext context =
            new ClassPathXmlApplicationContext("beans.xml");
        GreetMeBean bean = (GreetMeBean)
        context.getBean("greetMeBean");
        bean.execute();
    }
}
```

这段代码中new出来的ClassPathXmlApplicationContext将加载你在前面的bean.xml文件中定义的bean。然后在调用Context的getBean方法从Spring注册表中以greetMeBean为ID来查找bean，bean.xml文件中定义的所有bean都可以通过这种方式访问。

要运行这个例子，可以访问本书源代码中的chapter2/spring目录并运行以下Maven命令：

```sh
mvn compile exec:java -Dexec.mainClass=camelinaction.GreetMeBean
```

执行后，会在控制台上打印一行类似下面的英语问候语：

```undefined
Hello janstey
```

如果你使用了DanishGreeter进行wire（也就是说，使用camelinaction.DanishGreeter类来实例化myGreeterbean），你就会在控制台看到丹麦语问候语：

```undefined
Davs janstey
```

这个示例看起来可能很简单，但可以让你简单了解Spring以及其他的IoC容器的真正的作用。你可能会问，这和Camel有什么关系呢？Camel如果是另一个类的时候，它可以使用完全不同的配置方式。回顾一下在2.2.2节中您是如何通过Java代码配置JMS组件连接到ActiveMQ代理的：

```java
ConnectionFactory connectionFactory =
    new ActiveMQConnectionFactory("vm://localhost");
CamelContext context = new DefaultCamelContext();
    context.addComponent("jms",
        JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
```

你可以在Spring XML中使用bean标签来完成此操作，如下所示：

```xml
<bean id="jms"
      class="org.apache.camel.component.jms.JmsComponent">
  <property name="connectionFactory">
    <bean class="org.apache.activemq.ActiveMQConnectionFactory">
      <property name="brokerURL" value="vm://localhost"/>
    </bean>
  </property>
</bean>
```

在这种情况下，如果你要发送数据到一个端点，比如“jms:incomingOrders”，Camel将查找JMS bean，如果它是`org.apache.camel.Component`的实例，就会使用它。因此你不必手动向camelcontext添加组件——在Java DSL的2.2.2节中手动完成了这项任务。

但是Spring里面的CamelContext是怎么定义的呢？为了让事情看起来更简单，Camel利用了Spring的扩展机制，Spring XML文件中使用Camel时，使用了自定义的XML语法。要在Spring中加载CamelContext，可以执行以下操作:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/springbeans.
xsd
http://camel.apache.org/schema/spring
http://camel.apache.org/schema/spring/camel-spring.xsd">
  ...
  <camelContext xmlns="http://camel.apache.org/schema/spring"/>
</beans>
```

这个XML加载时将自动启动SpringCamelContext，这是之前用过的DefaultCamelContext的子类。还要注意一点，这个XML文件必须声明[http://camel.apache.org/schema/spring/camel-spring.xsd](https://links.jianshu.com/go?to=http%3A%2F%2Fcamel.apache.org%2Fschema%2Fspring%2Fcamel-spring.xsd)这个XML schema，这个xsd是导入自定义XML元素所需要的。

这个代码片段并没有什么实际的用途，要发挥作用，你还需要告诉Camel怎么构建路由，就像在使用Java DSL时所做的那样。下面这段代码使用了Spring XML，它定义的路由生成与清单2.1中定义的路由功能完全相同，两者可以认为是等价的。

***清单2.2用Spring来定义一个与清单2.1完全一样的路由\***

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/springbeans.
xsd
http://camel.apache.org/schema/spring
http://camel.apache.org/schema/spring/camel-spring.xsd">
    <bean id="jms"
          class="org.apache.camel.component.jms.JmsComponent">
        <property name="connectionFactory">
            <bean
                    class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL" value="vm://localhost" />
            </bean>
        </property>
    </bean>
    <bean id="ftpToJmsRoute" class="camelinaction.FtpToJMSRoute"/>
    <camelContext xmlns="http://camel.apache.org/schema/spring">
        <routeBuilder ref="ftpToJmsRoute"/>
    </camelContext>
</beans>
```

你可能已经发现，这段代码使用了`camelinaction.FtpToJMSRoute`作为`RouteBuilder`。为了复刻清单2.1中使用Java DSL定义的路由，你需要将2.1中定义的匿名内部类独立出来，写一个新的类`FtpToJMSRoute`，如下：

```java
public class FtpToJMSRoute extends RouteBuilder {
    public void configure() {
        from("ftp://rider.com/orders?username=rider&password=secret")
                .to("jms:incomingOrders");
    }
}
```

现在你已经了解了Spring的基础知识以及如何使用Spring来构建Camel路由，下面我们可以进一步了解如何只用xml来写Camel路由——不需要Java DSL。

### 2.4.2 XML DSL

我们目前使用Spring来集成Camel的用法已经足够了，但是这种写法并没有完全利用Spring零代码配置应用的特性。为了实现Spring XML完全的控制反转，Camel为我们提供了我们称为XML DSL的自定义XML扩展。用XML DSL你可以做几乎所有在Java DSL中能做的工作。

我们继续改进清单2.2中的“骑手摩配”业务场景，但这一次我们将只在RouteBuilder中，纯粹使用XML来定义路由，如下所示的Spring XML实现了这一点。

***清单2.3用XML DSL来实现一个与2.1完全一样的路由\***

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://camel.apache.org/schema/spring
       http://camel.apache.org/schema/spring/camel-spring.xsd">

    <bean id="jms" class="org.apache.camel.component.jms.JmsComponent">
        <property name="connectionFactory">
            <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL" value="vm://localhost"/>
            </bean>
        </property>
    </bean>
    <camelContext xmlns="http://camel.apache.org/schema/spring">
        <route>
            <from uri="ftp://rider.com/orders?username=rider&amp;password=secret"/>
            <to uri="jms:incomingOrders"/>
        </route>
    </camelContext>
</beans>
```

在这个清单中，在camelContext元素下，使用route元素替换routeBuilder元素。在route元素中，通过使用与Java DSL的RouteBuilder中使用的名称类似的元素来指定路由。请注意，我们必须修改FTP端点URI，以确保它是有效的XML。URI中QueryString用于连接多个选项之间的字符`&`是XML中的保留字符，因此必须使用`&`对其进行转义。仅此一个小小的改动，这个清单在功能上与于清单2.1中的Java DSL版本和清单2.2中的Spring + Java DSL组合版本的路由定义完全相同。

为了简化学习过程，我们使用本地文件目录代替FTP服务器。在本书的源代码中，我们将from方法改为使用本地文件目录中的消息。新路由是这样的：

```xml
<route>
    <from uri="file:src/data?noop=true"/>
    <to uri="jms:incomingOrders"/>
</route>
```

文件端点`FileEndpoint`将从相对路径`src/data`中加载订单文件。noop属性代表端点在文件被消费之后将不做任何改动，这个选项对于测试非常有用。在第6章中，你将看到Camel如何配置为在处理完成后删除或移动文件。

这段路由目前还没有什么有趣的东西，你需要为测试添加一个处理步骤。

**添加一个`Processor`**

要在路由中间添加处理步骤很简单，就像在Java DSL中一样。将一个自定义的Processor添加进去，就像在2.3.2节中所做的那样。

因为在Spring XML中不能引用匿名类，所以需要将匿名处理器独立出来，创建一个以下的类：

```java
public class DownloadLogger implements Processor {
    public void process(Exchange exchange) throws Exception {
        System.out.println("We just downloaded: "
                + exchange.getIn().getHeader("CamelFileName"));
    }
}
```

有了这个类的定义，你就可以在XML DSL中使用了，像这样：

```xml
<bean id="downloadLogger" class="camelinaction.DownloadLogger"/>

<camelContext xmlns="http://camel.apache.org/schema/spring">
    <route>
        <from uri="file:src/data?noop=true"/>
        <process ref="downloadLogger"/>
        <to uri="jms:incomingOrders"/>
    </route>
</camelContext>
```

现在可以运行示例了，跳转到本书源代码中的chapter2/spring目录，运行以下Maven命令：

```sh
mvn clean compile camel:run
```

因为在src/data目录中只有一个名为message1.xml的消息文件，它会在命令行中输出如下内容：

```css
We just downloaded: message1.xml
```

如果你想在使用incomingOrders队列后打印该消息，该怎么办？要实现这一点，你需要创建另一个路由。

**创建多个路由**

你可能还记得，在Java DSL中，每个以from开头的Java语句都会创建一个新路由。同样，你可以使用XML DSL创建多条路由。要这样做，你只需要在camelContext元素中添加一个新的route元素。

例如，在订单被发送到incomingOrders队列之后，用另一个路由来将数据传输到`DownloadLogger`这个Processor：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <route>
        <from uri="file:src/data?noop=true"/>
        <to uri="jms:incomingOrders"/>
    </route>
    <route>
        <from uri="jms:incomingOrders"/>
        <process ref="downloadLogger"/>
    </route>
</camelContext>
```

现在，你将在第二个路由中消费来自incomingOrders队列的消息，因此在订单通过队列发送后，下载的消息内容将打印出来。

**选择一种DSL进行使用**

对于Camel用户来说，在特定场景中最好使用哪种DSL是一个常见的问题，这个问题大部分情况下取决于个人偏好。如果你喜欢使用Spring或喜欢用XML定义东西，那么你可能更喜欢纯XML的方法。如果你能写Java代码，那么也许纯Java DSL方法更适合你。

无论哪种DSL，你都可以使用Camel几乎所有的功能。相对来说，Java DSL功能稍微丰富一些，因为你可以随时使用Java语言的其他功能。另外，一些Java DSL特性，比如值构建器`value builder`(用于构建表达式`expression`和判断`predicate`)，在XML DSL中是没有的。不过从另一个角度来讲，使用Spring XML可以利用它出色的对象构造功能，以及常用的数据库连接、JMS集成等等Spring开箱即用的功能。XML DSL还可以拿来制作图形化路由编辑工具，你可以在一个图形界面下编辑路由，然后转为XML，并可以将XML DSL的路由进行发布和读取❷。有一种常见的折衷方法是同时使用Spring XML和Java DSL，这是我们将要讨论的主题之一。

------

❷ 请参阅Bilgin Ibryam关于使用哪种Camel DSL的博客文章:[http://www.ofbizian.com/2017/12/which-camel-dsl-to-choose-and-why.html](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ofbizian.com%2F2017%2F12%2Fwhich-camel-dsl-to-choose-and-why.html)。

------

### 2.4.3 使用Camel和Spring

无论你是用Java还是XML DSL编写路由，在Spring容器中运行Camel都会给您带来许多其他好处。例如，如果你正在使用XML DSL，想要更改路由规则时，你不必重新编译任何代码。此外，你还可以访问Spring的数据库连接器、事务支持等组件。

让我们详细看看Camel提供的其他Spring集成。

**路由构建器发现**

Camel无论路由在构建时还是运行时，写Java DSL时使用Spring的CamelContext都是一个好主意。之前的清单2.2中可以看到，你可以显式地告诉Spring的CamelContext要加载什么路由构建器。你可以使用routeBuilder元素来做到这一点：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <routeBuilder ref="ftpToJmsRoute"/>
</camelContext>
```

通过这种明确的方式，可以清晰而简洁地定义Camel要装载什么东西。

不过，有时你可能需要更动态一些，这时，你需要`packageScan`和`contextScan`来进行动态扫描：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <packageScan>
        <package>camelinaction.routes</package>
    </packageScan>
</camelContext>
```

`packageScan`元素会从`camelinaction.routes`包下面加载所有的类，包括子包。

你也可以定义的更加精确一点，添加几个名称过滤器：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <packageScan>
        <package>camelinaction.routes</package>
        <excludes>**.*Test*</excludes>
        <includes>**.*</includes>
    </packageScan>
</camelContext>
```

在这个例子里，你将在camelinaction.routes包中加载所有路由构建器。但是会排除名字里含有Test的类。这种匹配语法类似于Apache Ant的文件模式匹配器中使用的语法。
 `contextScan`元素利用Spring的组件扫描特性来加载任何用`org.springframework.stereotype.@Component`注解标记的Camel路由构建器。让我们修改以下FtpToJMSRoute类，添加一个注解：

```java
@Component
public class FtpToJMSRoute extends RouteBuilder {
    public void configure() {        
    from("ftp://rider.com" +
        "/orders?username=rider&password=secret")
        .to("jms:incomingOrders");
    }
}
```

现在就可以改一下Spring XML文件，让它启用组件扫描：

```xml
<context:component-scan base-package="camelinaction.routes"/>
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <contextScan/>
</camelContext>
```

这将加载camelinaction.routes包中的任何打了@Component注解的Camel路由构建器。

在底层，Camel的一些组件（如JMS组件）构建在Spring的抽象库之上。这就是为什么在Spring中配置这些组件很容易。

**配置组件和端点**

在2.4.1节中可以看到，组件可以在Spring XML中定义，并且Camel可以自动取用。例如，我们再看下JMS组件：

```xml
<bean id="jms" class="org.apache.camel.component.jms.JmsComponent">
    <property name="connectionFactory">
        <bean class="org.apache.activemq.ActiveMQConnectionFactory">
            <property name="brokerURL" value="vm://localhost"/>
        </bean>
    </property>
</bean>
```

Bean id是这个组件被调用的唯一标识，这为我们提供了根据用例为组件赋予不同命名的灵活性。例如，你的应用可能需要集成两个JMS代理。一个可能是Apache ActiveMQ和另一个可能是WebSphere MQ：

```xml
<bean id="activemq" class="org.apache.camel.component.jms.JmsComponent">
  ...
</bean>
<bean id="wmq" class="org.apache.camel.component.jms.JmsComponent">
  ...
</bean>
```

然后你可以使用诸如activemq:myActiveMQQueue或wmq:myWebSphereQueue这样的uri。端点同样可以通过使用Camel的Spring XML扩展来定义。例如，你可以将连接到“骑手摩配”老接口中获取订单的FTP端点拆分为元素，如下所示的endpoint元素部分：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <endpoint id="ridersFtp"
              uri="ftp://rider.com/orders?username=rider&amp;password=secret"/>
    <route>
        <from ref="ridersFtp"/>
        <to uri="jms:incomingOrders"/>
    </route>
</camelContext>
```

> 注意，你可能会注意到，各种用户名密码是直接添加到端点URI中的被，这并不总是最好的解决方案。更好的方法是引用在其他地方定义并充分保护凭据。在第14章的14.1节中，你可以看到Camel的Properties组件或使用Spring属性占位符是如何替换明文密码的。

对于较长的端点uri，如果将它们分成几行会更加易于阅读。请参阅前面的路由，其中端点URI选项可以被分解为单独的行：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <endpoint id="ridersFtp" uri="ftp://rider.com/orders?
                                           username=rider&amp;
                                           password=secret"/>
    <route>
        <from ref="ridersFtp"/>
        <to uri="jms:incomingOrders"/>
    </route>
</camelContext
```

**引入配置和路由**

Spring开发中的一种常见做法是将应用的配置分到几个XML文件中，这样做主要是为了使XML更具可读性，你可能不希望在一个文件中去看数千行没有任何分隔的XML。

将应用分离到几个XML文件的另一个原因是使其可以重用。例如，一个JMS配置可能需要在另一个应用里面使用，这种场景下直接复制一个Spring XML文件，名为jms-setup.xml，其中包含以下内容：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/springbeans.
xsd">
    <bean id="jms"
          class="org.apache.camel.component.jms.JmsComponent">
        <property name="connectionFactory">
            <bean
                    class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL" value="vm://localhost" />
            </bean>
        </property>
    </bean>
</beans>
```

这个文件可以被其他文件的CamelContext通过import标签来引入，例如下面这个XML：

```cpp
<import resource="jms-setup.xml"/>
```

现在这个CamelContext可以使用另一个文件内定义的JMS组件定义。

在单独的文件中定义的另一个好处是可以引入路由本身，为此因为我们需要引入一个新的概念，即routeContext。首先我们可以定义出一个routeContext，如下所示：

```xml
<routeContext id="ftpToJms"
              xmlns="http://camel.apache.org/schema/spring">
    <route>
        <from uri="ftp://rider.com/orders?
username=rider&password=secret"/>
        <to uri="jms:incomingOrders"/>
    </route>
</routeContext>
```

routeContext 和camelContext可以在同一个文件中进行定义，也可以在不同文件中进行定义。在camelContext中，我们可以使用routeContextRef元素来引入routeContext中定义的路由，如下所示:

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <routeContextRef ref="ftpToJms"/>
</camelContext>
```

注意：如果将routeContext导入到多个CamelContext中进行使用，会在每个context中创建一个路由的新实例。在前面的例子中，两个具有相同端点uri的相同路由将导致它们争抢同一个资源。在这种情况下，每次将只有一个路由从FTP接受到文件。通常，在多个camelContext中重用路由的时候需要很小心，避免资源争抢带来的副作用。

**配置高级Spring选项**

在使用Spring CamelContext时，还有许多其他配置选项可用：

- 可插拔式的bean注册表将在第4章中讨论。
- 拦截器的配置在第9章中介绍。
- 第15章提到了流缓存、故障处理和重启。
- 跟踪机制在第16章中介绍。

## 2.5 重新审视端点

在2.2节中，我们介绍了Camel中端点的基础知识，我们看到了Java和XML路由的实际应用，现在就可以引入更高级的端点配置了。

### 2.5.1 发送数据到动态端点

端点URI，例如2.2.2节中讲的JMS。在Camel启动时只会初始化一次，因此它们是Camel中的静态实体，这种设计在大多数场景下都很好，因为多数情况下，我们提前知道将把数据传递到哪个位置。但是如果我们需要在运行时根据情况来确定数据传递的目的地呢？这种情况下，提供给to方法的静态端点URI就无能为力了，因为这些URI只在启动时初始化一次。Camel为此提供了一个额外的DSL方法：toD。

例如，假设你想把数据传递的目标的一部分信息放到消息头里面，那么下面这段代码可以实现：

```java
.toD("jms:queue:${header.myDest}");
```

同样拿XML写是这样的：

```xml
<toD uri="jms:queue:${header.myDest}"/>
```

这个端点URI在${}占位符中使用一个简单的表达式来引用传入消息中myDest头的值。如果myDest是incomingOrders，那么端点URI就将是jms:queue:incomingOrders，在这种情况下，这个端点的作用与之前使用静态配置相同。这个表达式使用了一种构建在Camel核心中的一种轻量级表达式语言，名为Simple，如果想了解更多关于Simple语言的内容，本章的2.6.1节中对Simple进行了更详细的讨论，附录A提供了一个完整的参考。

要自己运行这个示例，请转到本书源代码中的chapter2/ftp-jms目录并运行以下Maven命令：

```bash
mvn test -Dtest=FtpToJMSWithDynamicToTest
```

### 2.5.2 在端点uri中使用properties占位符

与硬编码的端点URI不同，Camel允许在uri中使用properties占位符来替换需要动态指定的部分。这里所谓的动态，与toD的动态还略有不同，toD的动态是在运行时动态，这里的动态是根据properties文件中定义的内容实现动态。

在测试场景下， 使用properties文件来配置环境的做法是非常常见的。Camel路由通常要在不同的环境中进行测试，开发在本地笔记本上，然后还需要经过一轮专有环境的测试，最后再部署到生产环境上等等。你肯定不希望在每次迁移到新环境的时候时都改代码、重写测试，这就是我们需要用properties文件来动态配置路由的原因。

**使用properties组件**

Camel拥有一个properties组件，这个组件允许你从外部的properties文件（或其他数据源）中读取配置。Properties组件的运行原理与Spring properties占位符相同，但是它拥有一些值得注意的改进点：

- Properties组件是内建在camel-core这个jar包中的，也就是说，你可以不需要Spring或者其他第三方框架引入。
- 这个组件可以在任何DSL中使用，例如Java DSL。Camel并没有限制必须在XML中使用。
- 支持通过引入第三方加密插件来实现对敏感数据的隐藏

更多关于Porperties组件的信息，可以参考Camel的官方文档：[https://camel.apache.org/components/latest/properties-component.html](https://links.jianshu.com/go?to=https%3A%2F%2Fcamel.apache.org%2Fcomponents%2Flatest%2Fproperties-component.html)

> 注意：可以使用Jasypt组件来加密Properties文件中的敏感信息。例如，你不希望在Properties文件中使用明文形式的密码。你可以在第14章中了解更多关于Jasypt组件的信息。

要在启动时使用Properties占位符，你必须在创建CamelContext的时候，把PropertiesComponent配置好：

```java
CamelContext context = new DefaultCamelContext();
        PropertiesComponent prop = camelContext.getComponent(
        "properties", PropertiesComponent.class);
        prop.setLocation("classpath:rider-test.properties");
```

在rider-test.properties文件，你可以定义一个外部的键值对属性：

```properties
myDest=incomingOrders
```

RouteBuilder现在就可以直接把外部定义的属性值用于URI的定义了，如下：

```java
   return new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("file:src/data?noop=true")
                        .to("jms:{{myDest}}");
                from("jms:incomingOrders")
                        .to("mock:incomingOrders");
            }
        };
```

请注意，Properties占位符在Camel中的语法与Spring的语法略有不同。Camel的Properties组件使用`{{key}}`语法，而Spring使用的是`${key}`。

你可以尝试一下这个示例，进入`chapter2/ftp-jms`目录，然后运行下面的maven命令：

```bash
mvn test -Dtest=FtpToJMSWithPropertyPlaceholderTest
```

这一套玩法在XML中的用法略有不同，下一节我们详细讲解。

**在XML DSL中使用Properties组件**

要在Spring XML中使用Camel的Properties组件，必须将其声明为一个有ID的Spring bean，如下所示：

```xml
<bean id="properties"
      class="org.apache.camel.component.properties.PropertiesComponent">
    <property name="location" value="classpath:ridertest.
properties"/>
</bean>
```

同样在rider-test.properties文件，定义一个外部的键值对属性：

```properties
myDest=incomingOrders
```

同样在camelContext元素里，你可以这样使用这个键值对：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <route>
        <from uri="file:src/data?noop=true"/>
        <to uri="jms:{{myDest}}" />
    </route>
    <route>
        <from uri="jms:incomingOrders"/>
        <to uri="mock:incomingOrders"/>
    </route>
</camelContext>
```

除了直接使用一个Spring bean来定义Camel的Properties组件外，还可以更简单的使用一个`<propertyPlaceholder>`元素来定义，如下：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <propertyPlaceholder id="properties"
                         location="classpath:rider-test.properties"/>
    <route>
        <from uri="file:src/data?noop=true"/>
        <to uri="jms:{{myDest}}"/>
    </route>
    <route>
        <from uri="jms:incomingOrders"/>
        <to uri="mock:incomingOrders"/>
    </route>
</camelContext>
```

这个示例，可以进入`chapter2/spring`目录，然后运行下面的maven命令：

```bash
mvn test -Dtest=SpringFtpToJMSWithPropertyPlaceholderTest
```

下面，我们将使用Spring的property替换Camel的Properties文件来实现相同的例子。

**使用Spring的Properties占位符**

Spring框架使用Properties占位符这种特性来支持将Spring XML文件中定义的部分属性外部化。如上节所讲，我们同样可以使用Spring的Properties占位符来替代Camel的Properties组件。

首先，我们要写一个URI依赖外部属性的路由。你像下面这段代码这样写，注意，Spring使用的是`${key}`的语法：

```xml
    <context:property-placeholder properties-ref="properties"/> 
    <util:properties id="properties"
                     location="classpath:rider-test.properties"/> ❶
    <camelContext xmlns="http://camel.apache.org/schema/spring">
        <endpoint id="myDest" uri="jms:${myDest}"/> ❷
        <route>
            <from uri="file:src/data?noop=true"/>
            <to uri="jms:${myDest}"/> ❸
        </route>
        <route>
            <from uri="jms:incomingOrders"/>
            <to uri="mock:incomingOrders"/>
        </route>
    </camelContext>
```

如上，非常不幸的是，Spring不支持直接在端点URI中直接使用占位符，你必须使用endpoint注解先把组件定义出来，这就需要三步：

1. 从外部properties文件中载入属性键值对

```xml
<context:property-placeholder properties-ref="properties"/>
<util:properties id="properties" location="classpath:rider-test.properties"/> ❶
```

1. 使用外部属性键值对定义一个endpoint

```xml
<endpoint id="myDest" uri="jms:${myDest}"/> ❷
```

1. 在路由中引用

```xml
<to ref="myDest"/> ❸
```

所以，要使用Spring的Properties占位符，你就要定义`<context:property-placeholder>`来指定你要使用哪个Properties文件。上面的代码中的❶代表Spring将会从classpath中读取rider-test.properties文件。然后在camelContext中，你可以定义一个endpoint，如❷所示，这个endpoint使用了properties的键作为id，JMS组件的目标则使用了`${myDest}`这种Spring占位符的写法来替换为properties中定义的动态值。

最后在路由中，你必须引用刚刚定义的endpoint，如❸所示，它展示了使用在`<to>`标签中进行引用。

同样在rider-test.properties文件，定义一个外部的键值对属性：

```properties
myDest=incomingOrders
```

这个示例，可以进入`chapter2/spring`目录，然后运行下面的maven命令：

```bash
mvn test -Dtest=SpringFtpToJMSWithSpringPropertyPlaceholderTest
```

> **Camel的Properties占位符与Spring的Properties占位符**
>  Camel的Properties组件比Spring的Properties占位符机制更强大，后者仅在使用Spring XML定义路由时有效，并且必须在专用的标记中声明端点，Properties占位符才能工作。
>  Camel属性组件是开箱即用的，这意味着完全可以不依赖于Spring。而且Camel的占位符还支持各种DSL语言来定义路由，比如Java、Spring XML和Blueprint OSGi XML。最重要的是，你可以在路由定义中的任何位置声明占位符。

**在端点URI中使用原始值**

有时，你会发现你在定义路由时，URI中写的部分字符并不是它原本的意思。例如，你有一个FTP，密码为++%%w?rd。如果直接写进去，因为这个密码中有保留字符，所以这个组件并不能正常运行。你可以对这些保留字符进行编码，但是这样会降低代码的可读性（并不是说你希望你的密码更容易读取，这只是个示例）。Camel对于这种情况的解决方案是使用一个RAW标记。例如，让我们使用这个原始密码连接到rider.com的FTP服务器：

```java
from("ftp://rider.com/orders?username=rider&password=RAW(++%%w?
rd)")
```

如你所见，密码被`RAW()`标签包裹起来了，这样可以将这部分数据作为原始数据来对待。

### 2.5.4 在端点uri中引用注册过的bean

本文已经提到过Camel注册表好几次了，但是没有真正的使用过它。我们不会在这里深入讨论注册表的细节（第四章会详细讲），这里我们将展示与注册表相关的端点uri中的通用语法。每当Camel端点需要一个对象实例作为一个参数值时，可以使用#语法在注册表中引用一个对象实例。例如，假设你只想从FTP站点获取CSV订单文件。你可以这样定义一个过滤器：

```java
public class OrderFileFilter<T> implements GenericFileFilter<T> {
    public boolean accept(GenericFile<T> file) {
        return file.getFileName().endsWith("csv");
    }
}
```

然后将其加到注册表中：

```java
registry.bind("myFilter", new OrderFileFilter<Object>());
```

然后你就可以在路由中引用它了：

```java
from("ftp://rider.com/orders?username=rider&password=secret&filter=#myFilter")
```

掌握了这些端点配置的战术之后，你就可以使用Camel的EIP来完成更高级的操作了。

## 2.6 路由和EIP

到目前为止，我们还没有是怎么涉及使用Camel来实现EIP。这种学习步骤是特意设置的，在继续学习更复杂的示例之前，你应当先对Camel在最简单的情况下所做的工作有一个很好的理解。

至于EIP，我们将马上介绍基于消息的路由、消息过滤器、消息广播、收件人列表和窃听器。书中还介绍了其他模式，第5章介绍了很多复杂的EIP。Camel支持的EIP的完整列表可以从Camel网站([http://camel.apache.org/eip.html](https://links.jianshu.com/go?to=http%3A%2F%2Fcamel.apache.org%2Feip.html))获得。

现在，让我们从最著名的EIP开始：基于消息的路由。

### 2.6.1 基于消息内容的路由

基于消息内容的路由，即content-based router，简称CBR。顾名思义，它是根据上游传递过来的消息内容来确定消息的传输目的地。判断标准可以是消息头、消息的数据类型甚至消息的一部分，任何从Exchange中传过来的数据都可以作为判断标准。

为了演示方便，我们回到骑手摩配的场景：有一部分客户已经开始使用较新的XML格式来替代CSV格式，这些新的格式描述的订单将上传到FTP服务器。两种类型的消息都将会进入incomingOrders队列。我们在此之前没有深入探讨过这个问题，现在，你需要将两种订单都转换为内部统一的POJO，这就意味着你需要把两种不同类型的传单执行不同的转换程序。

在实际应用的过程中，你可能会想到一个可行的方案：根据文件名来判断文件类型。以csv结尾的文件丢到csv的处理队列、xml结尾的文件丢带xml的处理队列，如下图：

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114551)

图2.10 CBR路由最终的目的地取决于消息内容。在这个场景下，文件的后缀名（会存储在header中）会被用来确定路由的目标

在之前的例子中，我们可以看到Camel在路由文件的时候，会把文件名放到key为CamelFileName的header中一并进行传输。

为了执行CBR所需的条件路由，Camel在DSL中引入了几个关键字。choice方法用于创建一个CBR处理器，并通过使用when方法和判断条件，在choice后面添加一条执行路径。

Camel的作者本可以使用contentBasedRouter作为方法名，来匹配EIP，但是他们坚持使用现有的关键字，因为它读起来更自然。如下：

```java
from("jms:incomingOrders")
        .choice()
        .when(predicate)
        .to("jms:xmlOrders")
        .when(predicate)
        .to("jms:csvOrders");
```

你可能已经注意到，我们没有填充每个when方法所需的predicate。Camel中的predicate是一个简单的接口，它只有一个matches方法：

```java
public interface Predicate {
    boolean matches(Exchange exchange);
}
```

这样你就可以把这个predicate当作java语法中，if后括号中的内容来看待。

在很多情况下，你可能不需要直接使用Exchange来做判断。好在predicate可以使用很多表达式来构建，表达式则可以很直观的从Exchange中提取结果。Camel支持很多种表达式，包括Simple、SpEL、JXPath、MVEL、OGNL、JavaScript、Groovy、XPath和XQuery等等。你甚至可以使用bean方法调用作为Camel中的表达式，这个将在后面的第四章中详细讲。在本例中，我们将使用表达式构建方法来在Java DSL中构建表达式。

表达式构建器是Java DSL一部分。在RouteBuilder中，我们可以使用header方法来获得一个表达式，这个表达式可以取出header中的值。例如，`header("CamelFileName")`创建出一个表达式，解析后将取得Exchange传入消息头中CamelFileName头的值。基于这个表达式，你可以调用方法来构建predicate。要检查文件名扩展名是否等于.xml，可以使用以下predicate：

```java
header("CamelFileName").endsWith(".xml")
```

一个完整的CBR路由的例子如下：

> 清单2.4 一个使用Java DSL构建的完整的CBR路由例子

```java
    return new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                // 从文件夹src/data中读取订单文件到JMS队列
                from("file:src/data?noop=true").to("jms:incomingOrders");
                //❶ CBR路由定义
                from("jms:incomingOrders")

                        .choice()
                        .when(header("CamelFileName").endsWith(".xml"))
                        .to("jms:xmlOrders")
                        .when(header("CamelFileName").endsWith(".csv"))
                        .to("jms:csvOrders");
                //❷ 测试路由，把接收到的消息打出来
                from("jms:xmlOrders")
                        .log("Received XML order: ${header.CamelFileName}")
                        .to("mock:xml");
                from("jms:csvOrders")
                        .log("Received CSV order: ${header.CamelFileName} ")
                        .to("mock:csv");
            }
        };
```

这个示例，可以进入`chapter2/cbr`目录，然后运行下面的maven命令：

```bash
mvn test -Dtest=OrderRouterTest
```

chapter2/cbr/src/data文件夹下实际是有两个文件的，一个xml一个cxv，所以你将看到如下的输出结果：

```css
Received CSV order: message2.csv
Received XML order: message1.xml
```

这两个输出来自于上面代码中注释❷下面的两个路由，这两个路由监听不同的队列，并通过log将文件名打印出来。

> **SIMPLE语言**
>
> 你可能已经注意到，传递给log方法的字符串有一些看起来像是为读取特定属性而定义的写法。`${header.CamelFileName}`这个写法来自Simple语言，这是一个包含在Camel核心模块中的表达式语言。Simle语言包含了很多游泳的变量、函数和操作符，通过这些可以对传入的Exchange进行操作。Simple语言中的各种表达式都需要写在`${}`占位符中，如清单2.4写的那样。让我们来看这个例子：
> `${header.CamelFileName}`
> 这里，`header`意思是Exchange中传入消息的Header。在点之后，你可以接任何你想从Header中取的数据。在运行时，Camel会将`CamelFileName`的值从header中取出。同样你可以将清单2.4中的CBR路由的判断条件改为Simple语言的写法：

```java
from("jms:incomingOrders")
    .choice()
    .when(simple("${header.CamelFileName} ends with 'xml'"))
    .to("jms:xmlOrders")
    .when(simple("${header.CamelFileName} ends with 'csv'"))
    .to("jms:csvOrders");
```

> 这里我们用到了`ends with`操作去检查从${header.CamelFileName}取出的动态字符串是否符合条件。Simple语言对于Camel应用而言非常有用，在附件A中我们完整的介绍了它。

在上面的例子里，你使用了一个测试路由来检查路由是否能够按照预期执行。关于Camel的路由测试，还可以使用模拟组件来完成，关于模拟组件，在第9章中有详细说明。

你还可以通过使用XML DSL来写一个等效的CBR，如清单2.5所示。在XML中的写法唯一不同的时，在Java中传入的是Predicate，而在XML里面，写的是一串包裹在simple标签内的表达式。Simple表达式语言是替换Java DSL中的Predicate的很好的选项。

> 清单2.5 使用XML DSL些一个完整的基于内容的路由

```xml
<route>
    <from uri="file:src/data?noop=true"/>
    <to uri="jms:incomingOrders"/>
</route>
<route>
    <from uri="jms:incomingOrders"/>
    <choice>
        <when>
            <!-- 使用Simple表达式代替Java表达式写法 -->
            <simple>${header.CamelFileName} ends with 'xml'</simple>
            <to uri="jms:xmlOrders"/>
        </when>
        <when>
            <!-- 使用Simple表达式代替Java表达式写法 -->
            <simple>${header.CamelFileName} ends with 'csv'</simple>
            <to uri="jms:csvOrders"/>
        </when>
    </choice>
</route>

<!-- 以下两个为测试路由，将消息内容打印出来 -->
<route>
    <from uri="jms:xmlOrders"/>
    <log message="Received XML order: ${header.CamelFileName}"/>
    <to uri="mock:xml"/>
</route>
<route>
    <from uri="jms:csvOrders"/>
    <log message="Received CSV order: ${header.CamelFileName}"/>
    <to uri="mock:csv"/>
</route>
```

要运行这个例子，可以访问本书源代码中的chapter2/cbr目录并运行以下Maven命令：

```sh
mvn test -Dtest=SpringOrderRouterTest
```

你将会看到与JavaDSL类似的输出。

**使用OtherWise语句**

一个汽车摩配的客户发送了一份以.csl结尾的CSV订单文件，但是你当前的路由只处理.csv和.xml文件，并将删除所有带有其他扩展名的订单。这并不是一个优雅的解决方案，所以还需要进行一些改进。

让一个订单处理系统支持多种扩展名的一种方法是使用正则表达式作为条件，而不是使用endsWith。下面的路由可以处理多个文件扩展名：

```java
from("jms:incomingOrders")
    .choice()
        .when(header("CamelFileName").endsWith(".xml"))
            .to("jms:xmlOrders")
        .when(header("CamelFileName").regex("^.*(csv|csl)$"))
            .to("jms:csvOrders");
```

但是，这个解决方案仍然会面临同样的问题，只要文件扩展无法匹配预设的几种，订单文件都将直接删除。正常情况下这些无法处理的订单都应当接收并存储到一个位置，由人工来进行处理解决。为此，你可以使用otherwise子句：

```java
from("jms:incomingOrders")
    .choice()
        .when(header("CamelFileName").endsWith(".xml"))
            .to("jms:xmlOrders")
        .when(header("CamelFileName").regex("^.*(csv|csl)$"))
            .to("jms:csvOrders")
        .otherwise()
            .to("jms:badOrders");
```

现在，所有扩展名不为.csv、.csl或.xml的订单都被发送到异常订单队列进行处理。

等效的XML DSL路由定义如下：

```xml
<route>
    <from uri="jms:incomingOrders"/>
    <choice>
        <when>
            <simple>${header.CamelFileName} ends with '.xml'</simple>
            <to uri="jms:xmlOrders"/>
        </when>
        <when>
            <simple>${header.CamelFileName} regex '^.*(csv|csl)$'
            </simple>
            <to uri="jms:csvOrders"/>
        </when>
        <otherwise>
            <to uri="jms:badOrders"/>
        </otherwise>
    </choice>
</route>
```

要运行这个例子，可以访问本书源代码中的chapter2/cbr目录并运行以下Maven命令：

```sh
mvn test -Dtest=OrderRouterOtherwiseTest
mvn test -Dtest=SpringOrderRouterOtherwiseTest
```

执行上述命令，chapter2/cbr/src/data_full目录中的四个order文件将会被消费，并输出如下内容：

```css
Received CSV order: message2.csv
Received XML order: message1.xml
Received bad order: message4.bad
Received CSV order: message3.csl
```

如上，可以看到接收到了一个异常订单。

**在CBR之后继续进行路由**

前面的CBR路由写起来很像是路由的重点：消息被路由到了几个目的地之一，仅此而已。我们是否可以接着CBR的逻辑继续写一些路由逻辑呢？

答案是，可以，有几种方法可以在CBR之后继续写。一种是使用另一种路由，如清单2.4那样，将测试消息打印到控制台。当然，也可以把choice代码块“关掉”，然后在其后添加需要的路由逻辑。

你可以通过在choice后面添加一个end来关掉choice逻辑块：

```java
from("jms:incomingOrders")
        .choice()
        .when(header("CamelFileName").endsWith(".xml"))
            .to("jms:xmlOrders")
        .when(header("CamelFileName").regex("^.*(csv|csl)$"))
            .to("jms:csvOrders")
        .otherwise()
            .to("jms:badOrders")
        .end()// 关闭choice块
        .to("jms:continuedProcessing");
```

在路由流程执行到end位置时，choice就已经被关闭了，在每一个消息选择了一个订单处理队列传递进去之后，消息同样会被被路由到continuedProcessing队列一次。如图2.11所示。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114602)

图2.11 通过使用end方法，你可以将消息路由到CBR之后的目的地

你还可以在choice块中标记哪些目的地是最终目的地，例如，我们可能不希望异常订单在其余的路径中继续存在。那么这些异常订单只要发送到异常订单队列就停止。在这种情况下，你可以在DSL中使用stop方法：

```java
from("jms:incomingOrders")
        .choice()
        .when(header("CamelFileName").endsWith(".xml"))
            .to("jms:xmlOrders")
        .when(header("CamelFileName").regex("^.*(csv|csl)$"))
            .to("jms:csvOrders")
        .otherwise()
            .to("jms:badOrders").stop() // 使用stop标记这是一个最终目的地
        .end()
        .to("jms:continuedProcessing");
```

现在，进入otherwise块的任何订单将只发送到badOrders队列，而不会继续发送到continuedProcessing队列。

使用XML DSL的时候，这个路由会看起来有一点不一样：

```xml
<route>
    <from uri="jms:incomingOrders"/>
    <choice>
        <when>
            <simple>${header.CamelFileName} ends with '.xml'</simple>
            <to uri="jms:xmlOrders"/>
        </when>
        <when>
            <simple>${header.CamelFileName} regex '^.*(csv|csl)$'</simple>
            <to uri="jms:csvOrders"/>
        </when>
        <otherwise>
            <to uri="jms:badOrders"/>
            <stop/>
        </otherwise>
    </choice>
    <to uri="jms:continuedProcessing"/>
</route>
```

注意，你不需要单独写一个end来结束选择块，因为XML每一个元素都需要一个显式的结束块，</choice>一定程度上代替了end的功能。

### 2.6.2 使用消息过滤器

骑手摩配现在有了一个新问题：公司的QA部门需要能够将测试订单发送到订单系统的web前端。当前的解决方案将接受所有订单，并将它们发送到内部系统进行处理。你曾经建议过QA应该建一个测试环境来模拟生产环境，但管理层否决了这个想法，理由是预算有限。因此目前需要做一个能够丢弃这些测试消息的解决方案，于此同时，实际订单仍然可以被正确传递。

图2.12所示，消息过滤器EIP提供了一种很好的方法来处理这类问题。传入消息只有在满足特定条件时才能通过过滤器，未能满足条件的消息将被删除。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211109114610)

图2.12 消息过滤器允许你根据特定条件过滤掉不需要的消息，使用过滤器，测试消息将被过滤掉。

我们看看如何使用Camel来实现它。回想一下，骑手摩配使用的web前端只以XML格式发送订单，因此我们可以将这个过滤器放在xmlOrders队列之后，这里所有的订单都是XML格式的。测试消息有一个额外的测试属性，因此我们可以使用它来进行筛选，测试消息是这样的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<order name="motor" amount="1" customer="foo" test="true"/>
```

整个解决方案是在OrderRouterWithFilterTest.java中实现的，它在本书源代码的chapter2/filter项目中，过滤器是这样的：

```java
from("jms:xmlOrders")
        .filter(xpath("/order[not(@test)]"))
        .log("Received XML order: ${header.CamelFileName}")
        .to("mock:xml");
```

要运行这个例子，可以访问本书源代码中的chapter2/cbr目录并运行以下Maven命令：

```sh
mvn test -Dtest=OrderRouterWithFilterTest
```

执行时，将会看到如下输出：

```css
Received XML order: message1.xml
```

在这里，我们只会接收到一条消息，因为测试消息被过滤掉了。
