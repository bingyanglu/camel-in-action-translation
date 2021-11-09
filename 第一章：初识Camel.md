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

消息`Message`都有一个ID，数据类型是字符串，即java.lang.String对象。消息创建者可以根据各自的协议不同，创建一个消息ID并且保证它的唯一性。Camel并没有规定消息ID的格式，对于没有ID生成方案的协议，Camel会使用自己的ID生成器生成一个ID。

**消息头和附件**

*Header*，即消息头`Header`，是与消息`Message`的参数，消息头可以包含发送方ID、内容编码、身份验证信息等等。消息头是键值对存储，它的键是一个唯一的、不区分大小写的字符串，它的值是一个对象，即“java.lang.Object”。也就是说Camel对消息头的类型没有任何约束，对于消息头的大小或消息头的数量也没有任何限制，消息头在消息中，是一个Map<String,Object>。消息`Message`还可以附带附件，这些附件通常用于web服务和电子邮件组件。

**消息体**

*Body*，即消息体，类型为“java.lang.Object”，因此消息可以存储任何内容、任何大小，应用设计人员要确保接收者能够解析消息内容。当发送方和接收方使用不同的主体格式时，Camel提供了将数据转换为可接受格式的机制，这种格式转换在幕后通过类型转换器自动进行。第3章全面介绍类型转换。

**错误标志**

消息还包含错误标志，在某些协议和规范下，如SOAP Web服务，错误标识区分了*输出*和*错误*消息。它们都是调用操作的有效响应，但后者表示不成功的结果。通常，在集成框架内不会处理这些错误。它是服务接口提供者和调用者之间契约的一部分，应当在各自应用中进行处理。

在路由过程中，消息`Message`包含在交换`Exchange`中。



### 1.3.2 交换 `Exchange`

Camel中的交换`Exchange`是消息在路由期间的容器，交换还定义了系统之间的交互模式，也称为*message exchange patterns* (*MEP*)，MEP用于区分单向消息传递和请求-响应消息传递，Camel的数据交换拥有一个MEP属性，它可以是以下任意一种:

- “InOnly”—单向消息(也称为事件消息)。例如，JMS消息传递通常是单向消息传递。
- “InOut”—请求-响应消息。例如，基于http的传输通常是请求-应答：客户端提交web请求，等待服务器的应答。

图1.7说明了Camel中Exchange的内容。

![img](assets/webp-20211108183146204)

让我们更详细地看看图1.7中的元素：

- *Exchange ID* - 交换的唯一ID。Camel自动生成唯一ID。
- *MEP* - MEP模式，表示你是使用“InOnly”还是“InOut”进行消息传递。当模式为“InOnly”时，交换只包含in消息。对于“InOut”，交换还包含out消息，即返回给调用者的应答消息。
- *Exception* - 如果在路由过程中任何时候发生异常，将在Exception字段中设置一个异常。
- *Properties* - 即属性，类似于消息头，但它们在整个路由数据交换期间都有效。属性用于包含全局级别的信息，而消息头是消息特有的。Camel本身在路由过程中也会向交换中添加各种树形。作为开发人员，你可以在路由数据交换期间的任何时候存取这些属性。
- *In message* - 输入消息，一次交换必定有in消息，in消息包含请求消息。
- *Out message* - 可选，只有当MEP是'InOut'时才存在。out消息即回复消息。

同一个数据路由从数据开始传递到数据传递结束的整个生命周期里，交换对象保持不变，消息对象则可能随着路由路线的行进而改变，例如，在把消息从一种格式转换为另一种格式的时候。

我们在Camel架构之前讨论了Camel的消息模型，我们希望你对Camel中的消息能够深入理解。毕竟Camel最主要的功能就是传递消息。现在，你已经准备好学习更多关于Camel及其架构的知识了。



## 1.4 Camel的架构设计

我们首先将介绍顶层架构，然后深入讲解一些概念。阅读了本节之后，你会理解大部分集成术语，并为第2章做好了准备，在第2章中，将会讲解Camel的路由功能。



### 1.4.1 从一万英尺的高度看看整体架构

要理解Camel的架构，首先要看看Camel的顶层架构。图1.8显示了构成Camel体系结构的主要概念架构图。

![img](assets/webp-20211108183631347)

图1.8 Camel的顶层架构，Camel由路由、处理器和组件组成。所有这些都包含在CamelContext中

路由引擎运行路由`Route`，路由规定了消息从数据源传递到目的地的行进路线，路由通过Camel的几个DSL之一进行定义。处理器`Processor`是路由行进路线上对数据进行操作和转换的处理单元，EIP同样是处理器实现的，所有的EIP在Camel的DSL中都具有相应的名称。组件`Component`是Camel中的扩展点，用于接入更多的技术和系统，组件体系提供了标准化的接口来接入更多的系统。

理解了这个架构图之后，让我们进一步理解图1.8中的各个概念。



### 1.4.2 Camel的概念

图1.8里面包含许多新概念，所以让我们花一些时间逐一介绍它们。我们将从Camel上下文环境`CamelContext`开始，它是Camel的运行时环境。

**Camel上下文环境`CamelContext`**

从图1.8可以看出，你可能已经猜到Camel上下文环境是某种容器，你可以将它看作Camel的运行时系统，它将Camel的所有功能组合在一起。

图1.9显示了Camel上下文环境提供的最主要的几个功能。

![img](assets/webp-20211108183729588)

图1.9 CamelContext提供了对许多有用功能的访问，其中最值得关注的是组件、类型转换器、注册表、端点、路由、数据格式和语言。

可以看到，Camel上下文环境提供了很多功能。表1.1对此进行了描述。

***表1.1 Camel上下文环境提供的服务\***

| **服务**        | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| 组件`Component` | 上下文中包含需要使用的组件。Camel可以通过Class Path上的自动发现或在OSGi容器中激活一个新的组件包来动态加载组件。第6章更详细地介绍了组件。 |
| 端点`Endpoint`  | 上下文中包含了所有已使用的端点。                             |
| 路由`Route`     | 上下文中包含已添加的路由。第2章介绍了路由。                  |
| 类型转换器      | 上下文中包含加载的类型转换器。Camel有一种机制，允许你手动或自动地将一种类型转换为另一种类型。第3章讨论了类型转换器。 |
| 数据格式        | 上下文中包含加载的数据格式。第3章讨论了数据格式。            |
| 注册表          | 上下文中包含一个注册表，允许你找到特定的bean。我们将在第4章讨论注册表。 |
| 语言            | 上下文中包含加载的语言。Camel允许你使用多种语言创建表达式。稍后你将简单了解XPath语言。有关Camel自己的Simple表达式语言的完整参考资料可在附录A.中找到 |

每一项功能的细节都会在全书中进行了讨论，首先，我们看看路由和Camel的路由引擎。

**路由引擎**

Camel的路由引擎默默执行着路由`Route`，这个引擎对开发人员而言是透明的，你只需要知道，有个引擎在那里，默默的运行着路由，确保数据被正确的送达目的地。

**路由`Route`**

路由，显而易见，是Camel最核心的抽象概念，简言之，路由的定义就是将组件串起来。在数据交换应用中使用路由，可以将客户端与服务器、生产者和消费者相互分离，路由可以实现如下功能:

- 动态确认客户端调用哪个服务器
- 提供灵活的方法进行数据处理
- 客户端和服务端分离
- 不同的系统完成不同的业务，通过路由进行通讯，促进更好的设计实践
- 增强某些系统(如消息代理和esb)的特性和功能
- 允许对路由上的某个节点进行模拟测试

Camel中的每个路由都有一个惟一的标识符，用于记录、调试、监视和启动和停止路由。路由最终只会有一个消息输入源，也就是说，在一个路由中配置多个输入的语法最终都会被拆解成单个输入源，例如:

```java
from("jms:queue:A", "jms:queue:B", "jms:queue:C").to("jms:queue:D");
```

在Camel引擎运行这段路由的时候，Camel将路由复制为三个独立的路由。最终与下面这三个路由的功能相同：



```java
from("jms:queue:A").to("jms:queue:D");
from("jms:queue:B").to("jms:queue:D");
from("jms:queue:C").to("jms:queue:D");
```

尽管多个输入源的语法在Camel 2.x里面是完全合法的。但是我们不建议任何路由使用多数据源输入，这种写法有可能在下一个大版本更新中被移除。

要定义路由，我们需要使用DSL。

**领域特定语言`DSL`**

为了将数据处理器和端点连接在一起形成路由，Camel定义了DSL，DSL的概念在Camel里比较宽泛。在Camel中，DSL可以是方便使用的Java API，它包含了以EIP术语命名的一系列方法。

例如下面这个例子:

```java
from("file:data/inbox")
    .filter().xpath("/order[not(@test)]")
    .to("jms:queue:order");
```

这段代码在一个Java语句中，定义了一条使用文件为起始端点的的路由，这个路由将消息传递到过滤器EIP，过滤器EIP将使用XPath来检查消息的order节点是不是test。如果判断通过，则将其转发到JMS端点，不符合条件的消息将被删除。

Camel提供多种DSL语言，因此可以使用XML DSL定义相同的路由，如下所示:

```xml
<route>
  <from uri="file:data/inbox"/>
  <filter>
    <xpath>/order[not(@test)]</xpath>
    <to uri="jms:queue:order"/>
  </filter>
</route>
```

DSL为Camel用户构建应用提供了一个很好的抽象。在底层，路由是由各种处理器组成的。让我们花点时间看看处理器是什么。

**处理器`Processor`**

处理器`Processor`是Camel的核心概念，它表示在路由中，够创建、使用、修改传入交换`Exchange`的处理节点。在路由过程中，交换`Exchange`从一个处理器流向另一个处理器。因此，你可以将路由看作一个路线图，路线图中包含名为处理器的节点和将数据从一个处理器传递到另一个处理器的线。处理器可以是EIP的实现、特定的组件，也可以是你自定义的处理器。图1.10显示了处理器之间的数据流动。

![img](assets/webp-20211108184656757)

图1.10 通过路由的交换`Exchange`流动方向。注：MEP将确定是否将回复发送回路由的调用者。

路由首先从创建交换的节点开始(DSL中的“from”)，在每个处理步骤中，上一个步骤输出的消息是下一个步骤的输入消息，在许多情况下，处理器不需要out消息，在这种情况下，in消息被重用，即in消息既是输入消息，也是输出消息。在路由结束时，Exchange的MEP确定是否需要将应答发送回路由的调用者。如果是InOnly，则没有回复。如果是InOut，Camel将从最后一步获取out消息并返回给路由的调用者。

> 备注
> Camel抽象概念下的生产者和消费者一开始似乎有点违反直觉。毕竟，生产者不应该是第一个节点，消费者不应该在路由结束时消费消息吗？别担心--你不是第一个这么想的人。你只需要从Camel传递消息的角度来理解这些概念，消费者通过消费外部系统的消息，将外部系统的消息带入路由，到达路由的另一边时，生产者才向外部系统发送(生产)消息。

交换`Exchange`是如何开始和结束它的路由之旅的？想知道答案，你需要了解组件和端点。

**组件`Component`**

组件`Component`是Camel中的主要扩展点，到目前为止，Camel生态里已经有280多个组件，其功能范围从数据传输、DSL到数据格式等等。你甚至可以为Camel创建自己的组件—我们将在第8章中对此进行讨论。

从编程的角度来看，组件其实非常简单：它们与URI中使用的名称相关联，并且充当端点的数据工厂。例如，根据文件组件`FileComponent`的URI定义，就可以创建出一个文件端点`FileEndpoint`。在Camel中，端点可能是一个更基本的概念。

**端点`Endpoint`**

端点`Endpoint`是Camel对最终数据通道的一个抽象，系统可以通过该通道发送或接收消息。如图1.11所示。

![img](assets/webp-20211108184904866)

图1.11 端点是把系统集成在一起、传递数据的最终接口。

在Camel中，通过使用uri配置端点，比如`file:data/inbox?delay=5000`。在运行路由的时候，Camel根据URI符号查找端点。图1.12显示了这是如何工作的。

![img](assets/webp-20211108184922605)

图1.12 端点uri分为三个部分:Scheme、上下文路径和参数。

Camel的Scheme，即图中的❶，表示端点对应的处理组件，在图中的这种情况下，File Scheme会选择FileComponent。然后FileComponent作为数据工厂，根据URI的其余部分创建FileEndpoint。上下文路径data/inbox，即图中的❷，告诉FileComponent文件位置是data/inbox。参数，delay= 5000，即图中的❸，会告诉文件组件，文件扫描的间隔是5秒。

这些只是端点的冰山一角，图1.13显示了端点如何与交换器、生产者和消费者一起工作。乍一看，图1.13似乎有点过于庞大，但细看你就会明白，说白了，端点就是创建消费者和生产者的工厂，这些消费者和生产者能够向特定端点接收和发送消息。在图1.8的Camel顶层架构图中，我们没有提到生产者或消费者，但是它们是非常重要的概念。我们下一个讨论。

**生产者**

这里的生产者是指Camel抽象概念下的生产者，它指的是能够向端点发送消息的实体。图1.13说明了生产者与其他Camel概念的契合之处。

当消息发送到某个端点时，生产者读取这些消息并且将其处理成对应端点相兼容的数据。例如，文件生产者`FileProducer`会把消息体转成二进制，写入文件。再例如，JmsProducer将会在Camel消息发送的JMS目的地之前，首先将消息按照javax.jms进行构建。这是Camel中的一个重要特性，它隐藏了与特定传输交互的复杂性。你所需要做的就是将消息路由到端点，然后由生产者来完成繁重的工作。

> 备注：
>
> 数据从A系统传递到B系统，A系统提供了α，B系统需要β，在系统交互的角度看，A系统是生产者，B系统是消费者，但是对于Camel而言，A系统将数据交给Camel的α组件，α组件就是数据的消费者，传递到B系统时，Camel交给β，β组件就是数据的生产者。

![img](assets/webp-20211108185218646)

图1.13 端点如何与生产者、消费者和交换器一起工作

**消费者**

消费者本身也是一项服务，它接收一些外部系统生成的消息，将它们封装在交换器中，并将它们发送给要处理的消息。消费者是在Camel中路由的交换的来源。

回顾图1.13，你可以看到消费者与其他Camel概念的契合度。要创建一个新的交换，消费者端点将数据装载成需要的数据载体，然后使用处理器通过路由引擎在Camel中启动交换的路由。

Camel有两种消费者：事件驱动消费者和轮询消费者。这些消费者之间的差异很重要，因为它们有助于解决不同的问题。

**事件驱动消费者**

最常见的消费者可能就是事件驱动消费者，如图1.14所示。

![img](assets/webp-20211108185231899)

图1.14 事件驱动的使用者平时是空闲的状态，消息到达的时候，Camel会将唤醒它开始干活。

这种消费者与CS或者BS系统都很相似，在EIP世界中，也被称为异步接收器。事件驱动的消费者侦听特定的消息通道，如TCP/IP端口、JMS队列、Twitter句柄、Amazon SQS队列、WebSocket等等。然后，它等待客户端向它发送消息。当消息到达时，客户端醒来并接受消息进行处理。

**轮询消费者**

另一种类型的消费者是轮询消费者，如图1.15所示。

![img](assets/webp-20211108185254180)

图1.15 轮询使用者主动检查新的消息。

与事件驱动消费者相反，轮询消费者主动地从特定的源(如FTP服务器)获取消息。轮询使用者在EIP术语中也称为同步接收方，因为它在处理完当前消息之前不会轮询更多的消息。轮询使用者的一个常见特性是定期轮询使用者，它定期进行轮询。文件、FTP和电子邮件组件都使用定期轮询消费者。

现在我们已经了解了Camel的所有核心概念。有了这些新知识，你可以回头看看你的第一次骑骆驼的经历，看看那段代码背后发生了什么。

## 1.5 重温你的第一次骑骆驼的过程

回想一下，在你的第一次骑骆驼(1.2.2节)中，你从一个目录(data/inbox)读取文件，并将结果写到另一个目录(data/outbox)。现在你已经了解了Camel的核心概念，现在你就可以准确的理解这个示例。

在下面的清单中再看一下Camel应用程序。

***清单1.4 使用Camel将文件从一个文件夹路由到另一个文件夹\***

```java
import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class FileCopierWithCamel {

    public static void main(String args[]) throws Exception {
        CamelContext context = new DefaultCamelContext();
        context.addRoutes(new RouteBuilder() {
            public void configure() {
                from("file:data/inbox?noop=true")     //Java DSL route
                    .to("file:data/outbox");     
            }
        });
        context.start();                         
        Thread.sleep(10000);
        context.stop();
    }
}
```

在本例中，首先创建CamelContext，它是Camel运行时。然后添加路由逻辑使用DSL❶RouteBuilder和Java。通过使用DSL，你可以简洁明了地让Camel实例化组件、端点、消费者、生产者等等。你所要关注的是定义对集成项目极为重要的路由。而此时，在Camel的引擎盖下，Camel正在访问FileComponent，并将其用作工厂来创建端点及其生产者。同样的FileComponent也用于创建消费者端。

> 注意
>
> 你可能想知道写Camel的代码是不是一定要写那句恶心的Thread.sleep。答案是不需要，以这种方式创建这个示例是为了演示Camel API的底层机制。如果你正在将Camel路由部署到另一个容器或运行时(你将在第7章和第15章中看到)，或者作为单元测试(在第9章中详细讨论，但也在第2章中使用)，则不需要显式地等待一段时间。即使对于没有部署到任何容器的独立路由，也有更好的方法。Camel提供org.apache.camel.main。启动你选择的路由直到JVM终止，我们将在第7章讨论这个问题。

## 1.6 本章小结

在本章中，你遇到了Camel。你已经看到了Camel如何使用企业集成模式(EIP)来简化集成过程。你还看到了Camel的DSL，它的目标是使Camel代码一看就知道在干啥，使开发人员专注于做把系统拼在一起，而不需要考虑怎么拼。

我们介绍了Camel的主要特性，什么是Camel，什么不是Camel，以及在哪里可以使用Camel。我们展示了Camel如何提供抽象和API，这些API可以在大量协议和数据格式上工作。

此时，你应该对Camel的功能及其基本概念有一个很好的理解。很快你就可以充满信息的去翻阅Camel应用，并对它们的功能有一个很好的了解。

在本书的其余部分，你将探索Camel的特性，并学习可以应用于日常集成场景的实用解决方案。我们还将解释在这匹骆驼坚硬的皮肤下发生了什么。为了确保你能从每一章中获得主要概念，后续章节将在摘要中向你展示最佳实践和关键点。

在下一章中，我们将开始研究路由，这是Camel最基本的特性，也是最值得学习的有趣特性。

