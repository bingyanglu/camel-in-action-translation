# 第一章：初识Camel

从零开始构建一个复杂的系统代价非常高昂，这种从轮子开始造起的做法几乎从未成功过。风险更低、更有效的方法是利用那些已有的、经过验证的组件，像玩拼图一样将他们组装起来，形成一个大的系统。我们每天都在使用着用拼图一样拼出来的各种综合系统，使一切成为可能，从电信、金融银行、医疗到旅游、娱乐，各行各业，方方面面。

要完成一个大拼图，需要将小拼图块通过简单的方法无缝、牢固的拼合在一起，最终形成一个整体。做系统集成也是一样，不同的是，拼图块做出来就是为了相互拼接，而那些小系统很少会在一开始就留出能拼接成一个大系统的接口。集成框架旨在解决这一痛点，作为一名开发人员，不需要关心需要集成的系统内部是如何工作的，只需要关注系统外部如何进行交互。一个好的集成框架可以为复杂系统之间的集成提供简单、可管理的抽象，成为能将这些小系统无缝地拼在一起的“粘合剂”。

Apache Camel就是这样一个集成框架。在本书中，我们将帮助你理解Camel是什么、如何使用它、以及为什么我们认为它是最好的集成框架之一。本章首先介绍了Camel，讲解了它的一些核心特性。然后我们将讲解如何获取Camel并解释如何在书中运行Camel示例。我们本章的最后部分将Camel的核心概念展开，这样你就可以理解Camel的体系架构。

你准备好了吗?让我们一起见识一下，什么是Camel。

## 1.1 来 看看什么是Camel

Camel是一个集成框架，旨在使项目集成更加高效。Camel项目于2007年初启动，现在已经成长为了一个成熟的开放源码项目，Camel使用Apache 2.0许可，具有健壮的社区。

Camel解决的问题是简化集成。我们相信，当你读完本章时，你会喜欢Camel并将其添加到你的必备工具列表中。

这个Apache项目被命名为Camel，名字很短，很容易记住。据说，起这个名字的灵感来自创始人之一曾经抽过的骆驼牌香烟。在Camel网站上，有[一篇问答](https://links.jianshu.com/go?to=http%3A%2F%2Fcamel.apache.org%2Fwhy-the-name-camel.html)列出使用这个名字的一些原因。

### 1.1.1 什么是Camel

Camel框架的核心是路由引擎，或者更准确地说，是路由引擎构建器。它允许你定义自己的路由规则，从哪个数据源接收消息、如何处理这些消息并将这些消息发送到目的地。使用Camel的集成语言，让你能够基于业务流程定义复杂的路由规则。如图1.1所示，Camel是不同系统之间的粘合剂。

![img](https://gitee.com/bingqilinpeishenme/imgschuang/raw/master/img/20211108101502)

Camel对系统集成的问题进行高度的抽象，你可以使用相同的API与不同的系统进行交互，即使这些系统使用的是不同协议、不同的数据类型。Camel的组件提供了针对不同协议和数据类型的API的接入，Camel支持超过280种协议和数据类型，可扩展、模块化的架构允许用户写出自己的实现来接入各类私有或共有协议。在这样的架构下，消除了不必要的转换，使Camel更快、更瘦。所以Camel非常适合嵌入任何需要的项目中。很多开源项目，如Apache ServiceMix、Karaf和ActiveMQ，都已经使用Camel作为集成工具。

Camel实现了很多功能，但是Camel并没有做所有的事情。Camel不是企业服务总线(Enterprise Service Bus,简称 ESB)，尽管有些人将Camel称为轻量级ESB，因为它支持路由、转换、编排、监控等。Camel没有容器或可靠的消息总线，Camel可以部署在容器中作为ESB的一个组成部分，比如前面提到的Apache ServiceMix。因此，我们更愿意将Camel称为集成框架，而不是ESB。

提到ESB，大家一般都会想到想到庞大而复杂的部署，不要害怕，Camel在微服务或物联网(IoT)网关等资源受限的环境下也可以运行的非常好。

为了理解Camel是什么，让我们来看看它的主要特性。

### 1.1.2 要用Camel的十来个理由

Camel的作者创建Camel的主要想法，是希望在集成领域引入一些新的思想。我们将会详细讲解Camel的特性，以下是Camel的一些主要思想：

- 路由和中介引擎
- 丰富的组件库
- 企业集成模式(EIP)
- 领域限定语言(DSL)
- 不限定内容的数据路由
- 模块化/插件化架构
- POJO模型路由
- 简易配置
- 自动类型转换器
- 为微服务量身打造的轻量级核心
- 云就绪
- 测试组件
- 充满活力的社区

让我们深入了解这些特性的细节：

**路由和中介引擎**

Camel的核心特性是它的路由和中介引擎。路由引擎根据路由配置灵活地传递消息。Camel使用企业集成模式（EIP）和领域限定语言（DSL）组合配置，形成路由，后面我们将介绍这两种语言。

**丰富的组件库**

Camel提供了280多个组件库。这些组件使Camel能够接入各种技术、使用各种api、理解各种数据格式。在图1.2中，你可以试着找找看有没有你使用过或者想去使用的一些技术。当然，在本书中我们不会详细的介绍所有的组件，我们会介绍最广泛使用的20个组件。如果你对某一个感兴趣，可以在官网找到对应的文档。

![img](assets/webp-20211108102257746)

**企业集成模式（EIP）**

系统集成所遇到的问题是多种多样的，格雷戈·霍普和鲍比·伍尔夫发现很多问题的解决方法是类似的。他们在他们的《企业集成模式》(Addison-Wesley, 2003)一书中对这些共性问题进行了分类，该书是任何系统集成专业人士的必读书籍(www.enterpriseintegrationpatterns.com)。如果你还没有读过，我们建议你去读一下。它将帮助你更快、更容易地理解Camel概念。

企业集成模式(Enterprise Integration Patterns，简称EIP)非常有用，这些集成模式不仅为各类问题提供了经过验证的解决方案，还可以帮助你来定义和沟通这些问题，EIP就像一套语言，具有明确的语义，这使得通信问题更加容易。Camel主要基于EIP，EIP描述了集成问题和解决方案，并提供了一个公共词汇表，但是这些词汇并没有规范化。Camel通过对EIP的实现来使其规范化，企业集成模式中描述的各种模式与Camel的领域限定语言之间几乎存在一对一的关系。

**领域限定语言（DSL）**

Camel最初引入的领域限定语言(DSL)的概念是Camel对集成领域的主要贡献之一。此后，其他几个集成框架也纷纷效仿。现在Camel以Java、XML或定制语言的DSL为特色。DSL的目的是让开发人员关注集成问题，而不是工具(编程语言)。以下是一些DSL使用不同格式并在功能上保持相同的例子：

Java DSL

```java
 from("file:data/inbox").to("jms:queue:order");
```

XML DSL

```xml
<route>
  <from uri="file:data/inbox"/>
  <to uri="jms:queue:order"/>
</route>
```

这些示例都是真实的代码，它们展示了如何轻松地将文件从文件夹路由到Java Message Service (JMS)队列。因为你使用的是一种真正的编程语言在写Camel的路由，所以你可以使用现有的工具支持，例如代码自动补全和错误检测，如图1.3所示。

![img](assets/webp-20211108102449475)

**不限定内容的数据路由**

Camel可以路由任何内容，不需要限定必须使用某种标准化的格式(如XML)。这种设定意味着你不需要为了方便做路由，而将数据转换为规范的格式。

**模块化和可插拔的架构**

Camel的模块化体系结构，允许你将任何组件加载到Camel中：自带的、第三方提供的、还是你自己定制的，统统可以。你可以在Camel中配置几乎所有的东西，许多特性都是可插入、可配置的——ID生成、线程管理、关闭顺序、流缓存等等。

**POJO模型**

Java bean(即POJO)在Camel中被认为是一等公民，并且Camel努力让你在集成项目中随时随地使用bean。在许多地方，你可以使用自己的定制代码扩展Camel的内置功能。第4章完整地讨论了在Camel中使用java bean。

**简单的配置**

如果可能，尽量遵循*约定优于配置*，做最少的配置。为了在路由中直接配置路由的节点，Camel使用一种简单直观的URI配置。

例如，你可以配置一个文件节点开始的Camel路由，从指定的文件夹中递归扫描txt文件，如下所示:

```java
from("file:data/inbox?recursive=true&include=.*txt$")...
```

**自动类型转换器**
Camel有一个内置的类型转换器机制，可以装载350多个转换器。例如，你不再需要配置类型转换规则来将字节数组转换为字符串。如果需要转换Camel不支持的类型，可以创建自己的类型转换器。这些类型转换都是Camel自己帮你做的，不需要人工干预。

同时Camel组件也提供了对大量数据类型的支持，它们可以接受大多数类型的数据，并将数据转换为需要类型，这个特性是Camel社区中最受欢迎的特性之一。你有时候甚至会想，为什么Java语言没有提供这种转换机制！第三章将会详细介绍类型转换器。

**为微服务量身打造的轻量级核心**

Camel的核心可以被认为是轻量级的，整个库大约有4.9 MB，只有1.3 MB的运行时依赖关系。这使得Camel可以很容易地嵌入或部署到任何你想要的地方，例如独立应用程序、微服务、web应用程序、Spring应用程序、Java EE应用程序、OSGi、Spring Boot、WildFly，以及AWS、Kubernetes和cloud Foundry等云平台中。Camel的设计初衷不是成为服务器或ESB，而是嵌入到你选择的任何运行环境中。Camel的运行只需要Java。

**云就绪**

除了Camel本身是云原生的(在第18章中介绍)，Camel还提供了许多用于连接SaaS提供商的组件。例如，使用Camel可以接入如下内容:

- Amazon的DynamoDB、EC2、Kinesis、SimpleDB、SES、SNS、SQS、SWF和S3

- Braintree的(PayPal, Apple, Android Pay，等等)、Dropbox

- Facebook

- GitHub

- Google Big Query、Calendar、Drive、Mail和Pub Sub

- HipChat

- LinkedIn

- Salesforce

- Twitter

  ……

**测试组件**

Camel提供了测试组件，可以更容易地测试自己的Camel应用程序。这些测试组件被广泛用于测试Camel本身，它包括18,000多个单元测试。其中还包含用于特定场景下的测试工具，例如，可以帮你模拟真实数据、配置响应预期、判断应用是否满足需求。第9章介绍了Camel的测试组件。

**充满活力的社区**

Camel有一个活跃、持久的开源社区。在撰写本文时，它已经活跃(并不断增长)了10多年。如果你打算在应用程序中使用任何开源项目，那么拥有一个强壮的社区是必不可少的。不活跃的项目几乎没有社区支持，所以如果遇到问题时只能靠自己。对于Camel，如果你遇到任何问题，用户和开发人员都会提供帮助。有关Camel社区的更多信息，请参见附录B。

现在你已经了解了构成Camel的主要特性，通过下载Camel、跑一个示例，你将获得更多的实践经验。

## 1.2 快速开始

本节将向你展示如何获取Camel， 并讲解其中的文件内容。然后使用Apache Maven运行一个示例。本书源代码中的任何示例都可以按照这种模式去运行。

首先我们看看怎么获取Camel。

### 1.2.1 获取Camel

Camel可以从[Apache Camel官方网站](https://links.jianshu.com/go?to=http%3A%2F%2Fcamel.apache.org%2Fdownload.html)获得。在官网上，你将看到所有Camel版本的列表以及最新版本的下载。

在本书中，我们将使用Camel 2.20.1。要获得此版本，请单击Camel 2.20.1发布链接。在页面底部附近，你将发现两个编译发布版：zip版本是针对Windows用户的，tar.gz版本是针对macOS/Linux用户的。下载了其中一个发行版后，将其解压缩到硬盘上的某个位置即可。

打开命令提示符并转到你解压的Camel的位置。目录结构如下:

```shell
$ ls
doc  examples  lib  LICENSE.txt  NOTICE.txt  README.txt
```

如你所见，整个camel的文件夹很简单，你可能已经猜到每个目录包含了什么。

详情如下:

- *doc* - 文件夹里有HTML格式的Camel手册。本手册里包含了Apache Camel网站上发布的手册的大部分内容。因此，对于那些无法访问Camel网站的人来说，这是一个不错的参考(或者你忘记你的Camel实战这本书丢到哪里了)。
- *examples* - 文件夹里有97个Camel示例。
- *lib* - 所有Camel库，在本章后面的部分中，你将学习如何使用Maven轻松下载核心组件之外的依赖项。
- *LICENSE.txt* - Camel的license。因为这是一个Apache项目，所以使用了Apache 2.0的license。
- *NOTICE.txt* - 有关Camel包含的第三方依赖项的版权信息。
- *README.txt* - 关于Camel的简介和一个帮助新用户启动和运行的链接列表。



### 1.2.2 第一次骑骆驼

到目前为止，你已经知道了如何下载Camel，并且已经窥见了Camel的一部分内容。Camel提供的例子都有说明，可以帮助你来理解它们。

在此后的章节，我们不会使用这种方式下载的Camel。本书源代码中的所有示例都使用Apache Maven，Maven将自动为你下载依赖包，并且不需要把下载下来的依赖包添加到classpath里  。

你可以从托管源代码的GitHub项目中获得该书的源代码([https://github.com/camelinaction/camelinaction2](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcamelinaction%2Fcamelinaction2))。

你将看到的第一个示例将是Camel的“hello world”路由：假设你需要从一个目录(data/inbox)读取文件，以某种方式处理它们，并将结果写入另一个目录(data/outbox)。为了简单起见，我们不对数据做任何处理，输出文件将与输入的文件保持一致。图1.4说明了这个过程。

![img](assets/webp-20211108102917409)



看起来很简单，对吧?这里有一个使用纯Java(不使用Camel)的可能解决方案：

**清单1.1用原生Java写法将文件从一个文件夹路由到另一个文件夹**

~~~java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;

public class FileCopier {

    public static void main(String args[]) throws Exception {
        File inboxDirectory = new File("data/inbox");
        File outboxDirectory = new File("data/outbox");
        outboxDirectory.mkdir();
        File[] files = inboxDirectory.listFiles();
        for (File source : files) {
            if (source.isFile()) {
               File dest = new File(
                    outboxDirectory.getPath()
                    + File.separator
                    + source.getName());
               copyFile(source, dest);
            }
        }
    }
 
    private static void copyFile(File source, File dest)
        throws IOException {
        OutputStream out = new FileOutputStream(dest);
        byte[] buffer = new byte[(int) source.length()];
        FileInputStream in = new FileInputStream(source);
        in.read(buffer);
        try {
            out.write(buffer);
        } finally {
            out.close();
            in.close();
        }
    }
}
~~~

这个FileCopier示例已经非常简单了，但是它仍然需要37行代码。你必须使用Java提供的底层文件api，并确保正确关闭资源——这是一项很容易出错的任务。此外，如果你想在data/inbox目录中轮询新文件，你需要设计一个计时器并跟踪已经复制的文件。就这样简单的一个问题，就会随着需求的增加而变得越来越复杂。

这样的集成任务以前我们已经做过几千次了，我们不应该手工编写这样的代码，重复造轮子。让我们看看如果使用Apache Camel之类的集成框架，是怎么解决这个问题的。

**清单1.2使用Apache Camel将文件从一个文件夹路由到另一个文件夹**

~~~java
import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        CamelContext context = new DefaultCamelContext();
        context.addRoutes(new RouteBuilder() {
            public void configure() {
                from("file:data/inbox?noop=true") 
                     .to("file:data/outbox");   // (1) 将文件从inbox路由到outbox
            }
        });
        context.start();                         
        Thread.sleep(10000);
        context.stop();
    }
~~~

这段代码是照着Camel的样板写的：构建一个CamelContext、添加路由、启动Context，然后CamelContext终止。其中的sleep方法会让Camel在固定的时间内完成文件复制。

Camel的路由定义的写法的顺序，就是Camel的执行顺序。这个路由可以这样理解：从(from)文件(file)  中读取数据，文件位置是data/inbox，参数是noop=true。noop参数是告诉Camel这个路由是在做文件复制，而不是文件移动。大部分从未见过Camel代码的，也可以大致理解这句话是什么意思。你应该注意到了，除去外面的样板代码，真正让路由起作用的，这有代码中注释为`(1)`的这短短两行代码。

要运行这个示例，需要从Maven站点[http://maven.apache.org/download.html](https://links.jianshu.com/go?to=http%3A%2F%2Fmaven.apache.org%2Fdownload.html)下载并安装Apache Maven。当Maven启动并运行时，打开终端并浏览到该书源代码的chapter1/file-copy目录。如果你把目录列在这里，你会看到以下几点:

- *data* - 包含inbox目录，该目录本身包含一个名为message1.xml的文件。
- *src* - 包含本章所示清单的源代码。
- *pom.xml* - 包含构建示例所需的信息。这是Maven项目对象模型(POM) XML文件。

> 注意，在本书的开发过程中，我们使用了Maven 3.5.0。Maven的不同版本可能无法正常工作或者运行结果与我们不一样

POM如下面的清单所示。

**清单1.3使用Camel的核心库所需的Maven POM**

~~~java
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <!-- (1) 父POM -->
    <groupId>com.camelinaction</groupId>
    <artifactId>chapter1</artifactId>          
    <version>2.0.0</version>                   
  </parent>

  <artifactId>chapter1-file-copy</artifactId>
  <name>Camel in Action 2 :: Chapter 1 :: File Copy Example</name>

  <dependencies>
    <!-- （2）Camel的核心依赖  --> 
    <dependency>
      <groupId>org.apache.camel</groupId>     
      <artifactId>camel-core</artifactId>     
    </dependency>
    <dependency>
      <!--Logging support--> 
      <groupId>org.slf4j</groupId>   
      <artifactId>slf4j-log4j12</artifactId>     
    </dependency>                              
  </dependencies>
</project>
~~~

Maven本身是一个复杂的主题，我们在这里不深入讨论。我们会给你足够的信息，使你在本书的例子富有成效。第8章还介绍了如何使用Maven开发Camel应用程序，因此其中也包含了大量信息。

清单1.3中的Maven POM可能是你所见过的最短的POM之一——几乎所有的POM都使用Maven提供的缺省值。除了这个POM中定义的内容外，一些通用配置放在了父POM中。这个POM中最重要的部分是注释中标注为`(2)`的Camel核心依赖。这个依赖项告诉Maven执行以下操作:

基于groupId、artifactId和version创建搜索路径。版本元素设置为camel-version属性,这是定义在父POM中（即注释标注为`(1)`的哪一行），这个属性的值为2.20.1。依赖项没有指定类型，因此默认是JAR。所以搜索路径是org/apache/camel/camel-core/2.20.1/camel-core-2.20.1.jar。

由于清单1.3没有为Maven定义查找Camel依赖项的下载位置，所以它在Maven的中央存储库中查找，该存储库位于[http://repo1.maven.org/maven2](https://links.jianshu.com/go?to=http%3A%2F%2Frepo1.maven.org%2Fmaven2)。

结合搜索路径和存储库URL, Maven尝试下载[http://repo1.maven.org/maven2/org/apache/camel/camel-core/2.20.1/camel-core-2.20.1.jar](https://links.jianshu.com/go?to=http%3A%2F%2Frepo1.maven.org%2Fmaven2%2Forg%2Fapache%2Fcamel%2Fcamel-core%2F2.20.1%2Fcamel-core-2.20.1.jar)。

这个JAR将会保存到Maven的本地下载缓存中，该缓存通常位于.m2/repository下的主目录中。在Linux/macOS默认在~ /.m2/repository，Windows默认在C:\Users\ <用户名> .m2\repository。

当清单1.2中的应用程序代码启动时，Camel JAR被添加到Class Path中。

要运行清单1.2中的示例，请cd到chapter1/file-copy目录，并使用以下命令:

```shell
mvn compile exec:java
```

这条命令指挥Maven在src目录中编译源代码，并在类路径上使用camel-core的JAR执行FileCopierWithCamel类。

注意:要运行本书中的任何示例，都需要互联网连接。Apache Maven将下载示例的许多JAR依赖项。整个示例集将下载几百M的库。

从chapter1/file-copy目录运行Maven命令，完成之后，检查一下data/outbox文件夹，看文件是否已经复制过去了。

祝贺你——你已经运行了你的第一个Camel示例！这是一个简单的例子，书中几乎所有的例子都可以使用相同的方式运行。

现在我们需要介绍Camel的基本概念，搭建好你的运行环境。下面我们将把注意力转向消息模型、体系结构和其他一些Camel概念。大多数的抽象都基于已知的EIP概念，并保留它们的原本的名称和语义。我们将从Camel的消息模型开始。

## 1.3 Camel的消息模型

Camel使用了两个抽象对消息进行建模，这两个抽象我们都将在本节中介绍:

- org.apache.camel.Message -- 即消息`Message`，包含在Camel中传输和路由的数据的基本实体。
- org.apache.camel.Exchange -- 即交换`Exchange`，消息交换传输的Camel抽象。Exchange对象

一般有一个in消息，如果有应答消息，则还有一个out消息。

我们将从消息开始，这样你就可以理解这些数据在Camel中的模型和传输方式。然后，我们将向你展示交换如何使用Camel模型进行系统间的“对话”。

### 1.3.1 消息message

*Message*，即消息，是系统在使用消息传递通道时用来相互通信的实体。消息的流动是单向的，从发送方到接收方，如图1.5所示。

![img](assets/webp-20211108103317291)

消息`Message`具有正文`Body`(消息载体`Payload`)、头`Header`和可选的附件`Attachment`，如[图1.6](https://www.jianshu.com/p/7237d8d8ddc3#图1.6)所示。

![img](assets/webp-20211108103330335)



## 1.4 Camel的架构设计





## 1.5 和Camel的第一次亲密接触







## 1.6 本章小结





