## 领域驱动六边形

​	该项目的重点会避免推荐任何设计应用软件。在这里展示了一些技术、工具、最佳实践、架构模式和指导，这些内容从不同源收集而来。

​	**以下所有内容都应当被视为是建议。**请记住不同的项目有不同的需求，所以以下任何建议在具体的项目里面都可以被替换或跳过。

​	代码示例使用如下技术栈：[NodeJS](https://nodejs.org/en/), [TypeScript](https://www.typescriptlang.org/)语言, [NestJS](https://docs.nestjs.com/)框架,   [Typeorm](https://www.npmjs.com/package/typeorm)数据库连接件。

​	但是本文列举的这些模式和原则是**框架/语言无关的**，所以以上技术栈可以被替换为其他替代技术。无论使用什么语言或框架，一个易用都能从以下原则上面获益。



## 目录

- [架构](#架构)

  - [图表](#图表)
  - [模块](#模块)
  - [应用核心层](#应用核心层)
  - [应用表示层](#应用表示层)
    - [应用服务](#应用服务)
    - [命令和查询](#命令和查询)
    - [端口](#端口)
  - [领域层](#领域层)
    - [实体](#实体)
    - [聚集体](#聚集体)
    - [领域事件](#领域事件)
    - [集成事件](#集成事件)
    - [领域服务](#领域服务)
    - [值对象](#值对象)
    - [针对领域对象的强制不变量](#针对领域对象的强制不变量)
    - [领域错误](#领域错误)
    - [在应用核心中使用库](#在应用核心中使用库)
  - [接口适配层](#接口适配层)
    - [控制器](#控制器)
    - [数据传输对象](#数据传输对象)
  - [基础设施层](#基础设施层)
    - [适配器](#适配器)
    - [数据仓储](#数据仓储)
    - [持久化模型](#持久化模型)
    - [基础设施层的其他内容](#基础设施层的其他内容)
  - [针对更细微的API的建议](#针对更细微的API的建议)
  - [普适的建议](#普适的建议)

- [其他建议和最佳实践](#其他建议和最佳实践)

  - [异常捕获](#异常捕获)
  - [测试](#测试)
    - [负载测试](#负载测试)
    - [模糊测试](#模糊测试)
  - [配置](#配置)
  - [日志](#日志)
  - [健康监测](#健康监测)
  - [目录和文件结构](#目录和文件结构)
  - [文件命名](#文件命名)
  - [静态代码分析](#静态代码分析)
  - [代码格式化](#代码格式化)
  - [文档编写](#文档编写)
  - [使应用更容易启动](#使应用更容易启动)
  - [种子](#种子)
  - [迁移](#迁移)
  - [规模限制](#规模限制)
  - [代码生成](#代码生成)
  - [自定义实用的类型](#自定义实用的类型)
  - [在上传/在提交之前的操作](#在上传/在提交之前的操作)
  - [当前巨大的继承链](#当前巨大的继承链)
  - [习惯化的提交](#习惯化的提交)

- [其他资源](#其他资源)
  - [文章](#文章)
  - [仓库](#仓库)
  - [文档](#文档)
  - [博客](#博客)
  - [视频](#视频)
  - [书籍](#书籍)

# 架构

​	该节主要参考以下链接:

- [Domain-Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design)
- [Hexagonal (Ports and Adapters) Architecture ](https://blog.octo.com/en/hexagonal-architecture-three-principles-and-an-implementation-example/)
- [Secure by Design](https://www.manning.com/books/secure-by-design)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Onion Architecture](https://herbertograca.com/2017/09/21/onion-architecture/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Software Design Patterns](https://refactoring.guru/design-patterns/what-is-pattern)

​	以下分为正反两面进行讲解。

## 正面

- 架构应当独立于外部框架、技术、数据库等。外部框架和资源能被轻易整合进架构，或从项目中分离。
- 架构能方便测试和评估。
- 架构应当更加安全。一些安全原则应当被整合进架构的设计过程。
- 架构可以被不同的团队维护，而避免出现重大冲突。
- 架构能轻易地添加新的特性。随着系统的增长，向系统增加新的特性的难度应当被维持在恒定水平，并尽量低。
- 如果架构触碰到[有界上下文](https://martinfowler.com/bliki/BoundedContext.html)并且分裂，该架构应当易于转变为若干更微小的子架构。

## 反面

- 面对更加复杂的架构，需要更加深入理解“软件质量原则”，比如SOLID原则、Clean/Hexagonal架构、领域驱动设计等。任何需要实践类似架构的团队都需要一位专家来领导，以避免误入歧途甚至欠下技术债。
- **这些建议或许并不适合中小规模的应用软件，因为它们没有大量的业务逻辑**。对这些不同的、异构的构建体、构建层、引用代码、抽象结构、数据映射进行支持存在预先的复杂性。所以单纯的[CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)应用并不适用于类似的架构，并且将它靠向CRUD可能会引起额外的复杂性。以下零散的原则能适用于小规模的软件，但是必须先分析和理解正反两面才能展开实现。

# 图表

![Domain-Driven Hexagon](assets/images/DomainDrivenHexagon.png)
<sup>Diagram is mostly based on [this one](https://github.com/hgraca/explicit-architecture-php#explicit-architecture-1) + others found online</sup>

​	简而言之，数据流呈现如下（从左到右）：

- 网络请求/CLI命令/事件 通过简单DTO送到控制层；
- 控制层中对应的控制器处理DTO，将它映射到内部命令/查询对象，之后传递到应用服务层。
- 服务层处理这些内部命令/查询。处理器通过领域特定服务或领域特定实体来执行业务逻辑，之后通过端口来调用基础设施层。
- 基础设施层使用映射器将数据转化为目标格式，使用数据仓储来拉取或持久化数据，使用适配器来发送事件或者做其他的IO通信。该层还会将数据映射会领域特定的格式，并将数据返回给服务层。
- 在服务层完成相应的任务之后，它返回数据或认证结果给控制层。
- 控制层返回数据给用户。（如果应用有呈现层/视图层，则数据返回给它们进行渲染。）  

​	每一层只负责自身逻辑的实现，并且各基本构建块应当尽量遵循[Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)原则。（例如，只针对数据库存取使用数据仓储，而对于业务逻辑则使用实体）。

​	记住，不同的项目和此处列举的典型项目存在不同的步骤、层次、构建块，这是很常见的。所以当项目需要的时候，你就应该添加需要的部分；当项目不需要那么复杂或者不需要那么多的抽象层的时候，就应该果断地移除掉多余的部分。

​	对任意项目的普适的建议如下：首先预估该项目将会变得多么庞大、多么复杂，再去寻找平衡点来对你的项目划分适当的层次，并避免那些可能导致你的项目过分复杂的多余的层次。

# 模块

​	该项目代码示例通过模块进行分离（这些模块也被称为是组件）。每一个模块存放在单独的文件夹下，作为专有的代码库。模块当中的每一个用例也有自己的文件夹（这也被称为是垂直切分）。

​	面对一组关联密切的组件，这样就会很轻易去同时修改它们。尽量不要在模块或用例之间建立关联。当产生关联的时候，你应当抽取共享的逻辑片段成为一个单独的文件，然后使所有相关的模块都依赖于它，而不是使模块之间产生相互的依赖。

​	尽量使不同的模块之间独立，使模块之间的交互尽可能的小。**你应当将每个模块都视作是一个迷你的应用，它们之间被单一的上下文分隔开。**尽量避免在模块之间直接导入（而是应该像从其他不同的领域中导入某个服务那样），直接导入将导致模块之间的[紧耦合](https://en.wikipedia.org/wiki/Coupling_(computer_programming))。

​	模块之间的通信与交互可以通过以下几种方式来实现：通过发起和捕获事件；通过公共的接口；通过特定的端口（也称为“适配器”）。

​	适配器的方式确保了模块之间的[松耦合](https://en.wikipedia.org/wiki/Loose_coupling)。如果定义边界的上下文被设计得很好，各个模块能够轻易地分离成为一个微型服务，这个分离的过程并不会修改各领域内的任何业务逻辑。

​	更多有关模块化编程的好处：

- [Modular programming: Beyond the spaghetti mess](https://www.tiny.cloud/blog/modular-programming-principle/).

# 核心层

​	核心层是系统核心，采用了以下的构建块（[DDD building blocks](https://dzone.com/articles/ddd-part-ii-ddd-building-blocks)）：

### 领域层

- 实体
- 聚集体
- 领域服务
- 值对象
- 领域错误

### 应用层

- 应用服务
- 命令和查询
- 端口
- 还有更多

# 应用层

## 应用服务

​	应用服务也被称作是“工作流服务”，“使用案例”，“交互器”等。这些服务编排了许多步骤，来实现用户施加的命令。

- 应用服务通常用于协调外部世界如何与你的应用程序交互，并执行端用户所需的任务。

- 应用服务不需要包含领域特定的业务逻辑；

- 对简单类型的数据进行操作时，应该将其转换为相应的领域特定的类型。一个简单类型可以被认为是领域模型所不知道的类型，这包括基本类型和域外类型。

- 应用服务声明了对执行领域特定逻辑所需的基础设施的依赖(通过使用端口)。

- 用于通过端口从数据库/外部获取领域特定实体(或其他任何数据)；

- 通过端口执行其他进程外的通信（如事件触发、发送邮件等)；

- 当与一个实体或聚合体交互时，直接执行其方法；

- 在使用多个实体/聚合体的情况下，使用领域服务来编排它们；

- 应用服务基本上是一个执行命令或查询命令的处理器；

- 应用服务不应该依赖其他应用服务，因为它可能会导致问题（如循环依赖）。

​	为每个使用案例提供一个服务是一种好的实践。

<details>
<summary>什么是"使用案例"（“用例”）</summary>

[wiki](https://en.wikipedia.org/wiki/Use_case):

> ​	在软件和系统工程中，用例是一系列动作或事件步骤，通常定义角色（在UML中称为参与者）和系统之间的交互，以实现一个目标。

​	简单地说，用例是应用程序所需要完成的一系列行为。

---

​	示例代码： [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts)

​	关于应用服务更多参考：

- [Domain-Application-Infrastructure Services pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-application-infrastructure-services-pattern.html)
- [Services in DDD finally explained](https://developer20.com/services-in-ddd-finally-explained/)




### 是否应该为每个用例都设计接口？

​	有些人喜欢为每个用例设计一个接口（驱动端口），`应用服务`实现了这个接口，而`控制器`依赖于它。这是一个可行的选择，但这个项目为了简单应该不为每个用例都设计接口：当存在同一个工作流的多个实现时，使用接口才有意义。但当用例过于具体，我们就不应该为同一个工作流设计多种实现。(此时应当采取之前提到的原则：“为每个使用案例提供一个服务”)。`控制器`很自然地依赖于具体的实现，因此使得接口变得冗余。更多关于这个话题的信息[在这里](https://stackoverflow.com/questions/62818105/interface-for-use-cases-application-services)。

### 局部的DTO（数据传输对象）

​	在一些项目中可以看到的另一些实体是局部的DTO。有些人不喜欢在核心之外（比如在“控制器”中）使用领域对象（比如实体），而是使用DTO。这个项目没有使用这种技术来避免额外的接口设计和数据映射。是否使用局部DTO取决于个人喜好。

​	[这里](https://martinfowler.com/bliki/LocalDTO.html)是Martin Fowler对局部DTO的想法，简而言之:

> ​	有些人认为它们（DTO）是服务层API的一部分，因为它们确保服务层的客户不依赖于底层的领域模型。虽然这可能很方便，但我认为不值得为所有这些数据映射付出代价。

## 命令和查询命令

​	有一个原则被称为[命令查询分离(CQS)](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)。在可能的情况下，方法应该被分为“命令”（状态改变操作|写操作）和“查询命令”(数据检索操作|读操作)。为了明确区分这两种类型的操作，输入对象可以表示为“命令”和“查询”。在DTO到达领域层之前，它应该被转换为“命令”或“查询”对象。

### 命令对象

​	`命令`是一个表示用户意图的对象，例如`CreateUserCommand `。它描述单个操作（但不执行该操作）。

​	`命令`用于状态更改操作，比如创建新用户并将其保存到数据库中。创建、更新和删除操作（`Create, Update and Delete`）被认为是状态更改操作。数据检索是“查询”的职责，因此“命令”操作不应该返回业务数据。

​	一些CQS纯粹主义者可能会说，“命令”根本不应该返回任何东西。但是您至少需要一个已创建表项的ID才能在以后访问它。要实现这一点，您可以让客户端生成[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)（更多信息请参见：[CQS与服务器生成的id](https://blog.ploeh.dk/2014/08/11/cqs-versus-server-generated-ids/)）。然而，违反这一规则并返回一些元数据，如**创建项的ID、重定向链接、确认消息、状态或其他元数据**是比遵循教条更实用的实现方式。

​	通过命令对象（或通过事件或其他东西）跨越多个聚合体完成的所有更改应该保存在单体数据库的事务中（如果您使用单体数据库）。这意味着在单个进程中，应用程序的一个命令/请求通常应该只执行一个**[事务性操作](https://en.wikipedia.org/wiki/Database_transaction)**，以保存所有更改（或在发生故障时取消该命令/请求的所有更改）。为了保持数据一致性，应该这样做。为了达到这个目的，可以使用[Unit of Work](https://www.c-sharpcorner.com/UploadFile/b1df45/unit-of-work-in-repository-pattern/)或类似的模式。示例代码：[create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts)——注意它如何从`this.unitOfWork`中获取事务存储库。

​	**注意**:`Command`与设计模式中描述的`命令模式`类似但不相同：[Command Pattern](https://refactoring.guru/design-patterns/command)。互联网上有多种定义，它们有着相似但稍有差异的实现。

​	要执行一项命令，可以使用`命令总线`的模式，而不是直接导入一项服务。命令总线能让你分离命令的请求者和命令的接收者，因此您可以从任何地方发送命令，而不创建耦合。

​	示例代码：

- [create-user.command.ts](src/modules/user/commands/create-user/create-user.command.ts) - 一个命令对象实例
- [create-user.message.controller.ts](src/modules/user/commands/create-user/create-user.message.controller.ts) - 控制器通过总线方式执行一个命令，这将命令同其处理器解耦了。
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 一个命令处理器实例
- [command-handler.base.ts](src/libs/ddd/domain/base-classes/command-handler.base.ts) - 一个命令处理器类的基类，将命令的执行过程封装在`Unit of Work`里面。

​	更多参考：

- [What is a command bus and why should you use it?](https://barryvanveen.nl/blog/49-what-is-a-command-bus-and-why-should-you-use-it)

### 查询对象

​	`查询对象`类似于`命令对象`。它表示用户想要找到某样东西并描述如何去做。

​	`查询对象`用于检索数据，所以不应该进行任何状态更改（包括写入数据库、文件等）。

​	查询通常只是一个数据检索操作，不涉及业务逻辑。因此，如果需要，可以完全绕过应用程序层和领域层。不过，如果在返回查询响应结果之前，施加了一些额外的不改变数据库状态的操作（比如计算某些内容），那么可以在应用层或领域层中执行这些操作。

​	与命令的执行类似，查询可以使用“查询总线”来执行。

​	示例代码：

- [find-users.query.ts](src/modules/user/queries/find-users/find-users.query.ts) - 一个查询对象实例
- [find-users.query-handler.ts](src/modules/user/queries/find-users/find-users.query-handler.ts) - 一个完全绕开应用层和领域层的查询对象实例

---

​	通过使“命令”和“查询”分离，代码变得更容易理解。前者改变一些状态，后者只是检索数据。

​	此外，从一开始就遵循CQS将有助于将经常写数据的模型和经常读数据的模型分离到不同的数据库（CQRS）中（这被称为**数据库的读写分离设计**）。

​	**注意**：这个代码仓库使用[NestJS CQRS](https://docs.nestjs.com/recipes/cqrs)包提供一个命令/查询总线。

​	阅读更多有关CQS和CQRS的内容：

- [Command Query Segregation](https://khalilstemmler.com/articles/oop-design-principles/command-query-segregation/).
- [Exposing CQRS Through a RESTful API](https://www.infoq.com/articles/rest-api-on-cqrs/)
- [What is the CQRS pattern?](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [CQRS and REST: the perfect match](https://lostechies.com/jimmybogard/2016/06/01/cqrs-and-rest-the-perfect-match/)

---

## 端口

​	端口（对于驱动适配器）是定义契约的接口，这些契约必须由基础设施适配器实现，以便执行一些与业务逻辑无关、与技术细节相关的操作。端口就像是一种针对技术细节的抽象，同时业务逻辑不关心这些技术细节。

​	在应用核心层，**依赖指向内侧**。外侧可以依赖于内侧，但内侧从不依赖于外侧。应用核心不应该依赖于框架，也不应该直接访问外部资源。任何对核心进程外资源的外部调用，或从核心外进程取回数据，都应该通过“端口”来完成。具体的方式为，将**资源实现类**创建在基础设施层的某处，并注入到应用核心层(这被称为[依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)和[控制反转](https://en.wikipedia.org/wiki/Dependency_inversion_principle))。这使得业务逻辑独立于应用的核心逻辑。这样便于测试，也允许插入/拔出/交换任何外部资源，轻松地使应用程序模块化和[松散耦合](https://en.wikipedia.org/wiki/Loose_coupling)。

- 端口基本上只是接口，定义了什么必须做，而不关心是如何做的。

- 可以创建端口来抽象I/O操作、技术细节、具有侵入性的库、被复用的遗留代码等。

- 端口应该被创建来适应具体领域的需要，而不是简单地模仿工具API。

- 模拟的资源实现可以在测试时传递到端口，所以端口能使您的测试更快并且独立于运行环境。

- 在设计端口时，请记住[接口隔离原则](https://en.wikipedia.org/wiki/Interface_segregation_principle)。如果有必要，可以将大的接口拆分成小的接口，但也要记住，在不必要的时候不要做得太过。

- 端口也有助于延迟决策。领域层甚至可以在决定使用什么具体的技术（框架、数据库等）之前实现。

​	**注意**：由于大多数端口的实现方式都是在应用服务中注入和执行的，所以应用层可以是保存这些端口的好地方。但是有时候领域层的业务逻辑依赖于消耗一些外部资源，在这种情况下，这些端口可以放在领域层中。

​	**注意**：在较小的应用程序/API 中创建端口可能会添加不必要的抽象，从而使此类解决方案过于复杂。在此类应用程序中，直接使用具体实现而不是端口可能就足够了。在使用“端口”之前，请考虑所有的优缺点。

​	示例代码：

- [repository.ports.ts](src/core/ports/repository.ports.ts)
- [logger.port.ts](src/core/ports/logger.port.ts)

# 领域层

