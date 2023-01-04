## 领域驱动六角形

​    该项目的重点会避免推荐任何设计应用软件。在这里展示了一些技术、工具、最佳实践、架构模式和指导，这些内容从不同源收集而来。

​    **以下所有内容都应当被视为是建议**。请记住不同的项目有不同的需求，所以以下任何建议在具体的项目里面都可以被替换或跳过。

​    代码示例使用如下技术栈：[NodeJS](https://nodejs.org/en/), [TypeScript](https://www.typescriptlang.org/)语言, [NestJS](https://docs.nestjs.com/)框架,   [Typeorm](https://www.npmjs.com/package/typeorm)数据库连接件。

​    但是本文列举的这些模式和原则是**框架/语言无关的**，所以以上技术栈可以被替换为其他替代技术。无论使用什么语言或框架，一个易用都能从以下原则上面获益。

## 目录

- [架构](#架构)

- [架构图](#架构图)

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
  - [数据仓库](#数据仓库)
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

​    该节主要参考以下链接:

- [Domain-Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design)
- [Hexagonal (Ports and Adapters) Architecture ](https://blog.octo.com/en/hexagonal-architecture-three-principles-and-an-implementation-example/)
- [Secure by Design](https://www.manning.com/books/secure-by-design)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Onion Architecture](https://herbertograca.com/2017/09/21/onion-architecture/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Software Design Patterns](https://refactoring.guru/design-patterns/what-is-pattern)

​    以下分为正反两面进行讲解。

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

# 架构图

![Domain-Driven Hexagon](./assets/images/DomainDrivenHexagon.png)
    这张架构图主要参考了[GitHub - hgraca/explicit-architecture-php](https://github.com/hgraca/explicit-architecture-php#explicit-architecture-1)

​    简而言之，数据流呈现如下（从左到右）：

- 网络请求/CLI命令/事件 通过简单DTO送到控制层；
- 控制层中对应的控制器处理DTO，将它映射到内部命令/查询对象，之后传递到应用服务层。
- 服务层处理这些内部命令/查询。处理器通过领域特定服务或领域特定实体来执行业务逻辑，之后通过端口来调用基础设施层。
- 基础设施层使用映射器将数据转化为目标格式，使用数据仓储来拉取或持久化数据，使用适配器来发送事件或者做其他的IO通信。该层还会将数据映射会领域特定的格式，并将数据返回给服务层。
- 在服务层完成相应的任务之后，它返回数据或认证结果给控制层。
- 控制层返回数据给用户。（如果应用有呈现层/视图层，则数据返回给它们进行渲染。）  

​    每一层只负责自身逻辑的实现，并且各基本构建块应当尽量遵循[Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)原则。（例如，只针对数据库存取使用数据仓储，而对于业务逻辑则使用实体）。

​    记住，不同的项目和此处列举的典型项目存在不同的步骤、层次、构建块，这是很常见的。所以当项目需要的时候，你就应该添加需要的部分；当项目不需要那么复杂或者不需要那么多的抽象层的时候，就应该果断地移除掉多余的部分。

​    对任意项目的普适的建议如下：首先预估该项目将会变得多么庞大、多么复杂，再去寻找平衡点来对你的项目划分适当的层次，并避免那些可能导致你的项目过分复杂的多余的层次。

# 模块

​    该项目代码示例通过模块进行分离（这些模块也被称为是组件）。每一个模块存放在单独的文件夹下，作为专有的代码库。模块当中的每一个用例也有自己的文件夹（这也被称为是垂直切分）。

​    面对一组关联密切的组件，这样就会很轻易去同时修改它们。尽量不要在模块或用例之间建立关联。当产生关联的时候，你应当抽取共享的逻辑片段成为一个单独的文件，然后使所有相关的模块都依赖于它，而不是使模块之间产生相互的依赖。

​    尽量使不同的模块之间独立，使模块之间的交互尽可能的小。**你应当将每个模块都视作是一个迷你的应用，它们之间被单一的上下文分隔开。**尽量避免在模块之间直接导入（而是应该像从其他不同的领域中导入某个服务那样），直接导入将导致模块之间的[紧耦合](https://en.wikipedia.org/wiki/Coupling_(computer_programming))。

​    模块之间的通信与交互可以通过以下几种方式来实现：通过发起和捕获事件；通过公共的接口；通过特定的端口（也称为“适配器”）。

​    适配器的方式确保了模块之间的[松耦合](https://en.wikipedia.org/wiki/Loose_coupling)。如果定义边界的上下文被设计得很好，各个模块能够轻易地分离成为一个微型服务，这个分离的过程并不会修改各领域内的任何业务逻辑。

​    更多有关模块化编程的好处：

- [Modular programming: Beyond the spaghetti mess](https://www.tiny.cloud/blog/modular-programming-principle/).

# 核心层

​    核心层是系统核心，采用了以下的构建块（[DDD building blocks](https://dzone.com/articles/ddd-part-ii-ddd-building-blocks)）：

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

​    应用服务也被称作是“工作流服务”，“使用案例”，“交互器”等。这些服务编排了许多步骤，来实现用户施加的命令。

- 应用服务通常用于协调外部世界如何与你的应用程序交互，并执行端用户所需的任务。

- 应用服务不需要包含领域特定的业务逻辑；

- 对简单类型的数据进行操作时，应该将其转换为相应的领域特定的类型。一个简单类型可以被认为是领域模型所不知道的类型，这包括基本类型和域外类型。

- 应用服务声明了对执行领域特定逻辑所需的基础设施的依赖(通过使用端口)。

- 用于通过端口从数据库/外部获取领域特定实体(或其他任何数据)；

- 通过端口执行其他进程外的通信（如事件触发、发送邮件等)；

- 当与一个实体或聚集体交互时，直接执行其方法；

- 在使用多个实体/聚集体的情况下，使用领域服务来编排它们；

- 应用服务基本上是一个执行命令或查询命令的处理器；

- 应用服务不应该依赖其他应用服务，因为它可能会导致问题（如循环依赖）。

​    为每个使用案例提供一个服务是一种好的实践。

<details>
<summary>什么是"使用案例"（“用例”）</summary>  
  [wiki](https://en.wikipedia.org/wiki/Use_case):
  在软件和系统工程中，用例是一系列动作或事件步骤，通常定义角色（在UML中称为参与者）和系统之间的交互，以实现一个目标。
  简单地说，用例是应用程序所需要完成的一系列行为。
</details>

---

​    示例代码： [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts)

​    关于应用服务更多参考：

- [Domain-Application-Infrastructure Services pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-application-infrastructure-services-pattern.html)
- [Services in DDD finally explained](https://developer20.com/services-in-ddd-finally-explained/)

### 是否应该为每个用例都设计接口？

​    有些人喜欢为每个用例设计一个接口（驱动端口），`应用服务`实现了这个接口，而`控制器`依赖于它。这是一个可行的选择，但这个项目为了简单应该不为每个用例都设计接口：当存在同一个工作流的多个实现时，使用接口才有意义。但当用例过于具体，我们就不应该为同一个工作流设计多种实现。(此时应当采取之前提到的原则：“为每个使用案例提供一个服务”)。`控制器`很自然地依赖于具体的实现，因此使得接口变得冗余。更多关于这个话题的信息[在这里](https://stackoverflow.com/questions/62818105/interface-for-use-cases-application-services)。

### 局部的DTO（数据传输对象）

​    在一些项目中可以看到的另一些实体是局部的DTO。有些人不喜欢在核心之外（比如在“控制器”中）使用领域对象（比如实体），而是使用DTO。这个项目没有使用这种技术来避免额外的接口设计和数据映射。是否使用局部DTO取决于个人喜好。

​    [这里](https://martinfowler.com/bliki/LocalDTO.html)是Martin Fowler对局部DTO的想法，简而言之:

> ​    有些人认为它们（DTO）是服务层API的一部分，因为它们确保服务层的客户不依赖于底层的领域模型。虽然这可能很方便，但我认为不值得为所有这些数据映射付出代价。

## 命令和查询命令

​    有一个原则被称为[命令查询分离(CQS)](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)。在可能的情况下，方法应该被分为“命令”（状态改变操作|写操作）和“查询命令”(数据检索操作|读操作)。为了明确区分这两种类型的操作，输入对象可以表示为“命令”和“查询”。在DTO到达领域层之前，它应该被转换为“命令”或“查询”对象。

### 命令对象

​    `命令`是一个表示用户意图的对象，例如`CreateUserCommand `。它描述单个操作（但不执行该操作）。

​    `命令`用于状态更改操作，比如创建新用户并将其保存到数据库中。创建、更新和删除操作（`Create, Update and Delete`）被认为是状态更改操作。数据检索是“查询”的职责，因此“命令”操作不应该返回业务数据。

​    一些CQS纯粹主义者可能会说，“命令”根本不应该返回任何东西。但是您至少需要一个已创建表项的ID才能在以后访问它。要实现这一点，您可以让客户端生成[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)（更多信息请参见：[CQS与服务器生成的ID](https://blog.ploeh.dk/2014/08/11/cqs-versus-server-generated-ids/)）。然而，违反这一规则并返回一些元数据，如**创建项的ID、重定向链接、确认消息、状态或其他元数据**是比遵循教条更实用的实现方式。

​    通过命令对象（或通过事件或其他东西）跨越多个聚集体完成的所有更改应该保存在单体数据库的事务中（如果您使用单体数据库）。这意味着在单个进程中，应用程序的一个命令/请求通常应该只执行一个**[事务性操作](https://en.wikipedia.org/wiki/Database_transaction)**，以保存所有更改（或在发生故障时取消该命令/请求的所有更改）。为了保持数据一致性，应该这样做。为了达到这个目的，可以使用[Unit of Work](https://www.c-sharpcorner.com/UploadFile/b1df45/unit-of-work-in-repository-pattern/)或类似的模式。示例代码：[create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts)——注意它如何从`this.unitOfWork`中获取事务存储库。

​    **注意**:`Command`与设计模式中描述的`命令模式`类似但不相同：[Command Pattern](https://refactoring.guru/design-patterns/command)。互联网上有多种定义，它们有着相似但稍有差异的实现。

​    要执行一项命令，可以使用`命令总线`的模式，而不是直接导入一项服务。命令总线能让你分离命令的请求者和命令的接收者，因此您可以从任何地方发送命令，而不创建耦合。

​    示例代码：

- [create-user.command.ts](src/modules/user/commands/create-user/create-user.command.ts) - 一个命令对象实例
- [create-user.message.controller.ts](src/modules/user/commands/create-user/create-user.message.controller.ts) - 控制器通过总线方式执行一个命令，这将命令同其处理器解耦了。
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 一个命令处理器实例
- [command-handler.base.ts](src/libs/ddd/domain/base-classes/command-handler.base.ts) - 一个命令处理器类的基类，将命令的执行过程封装在`Unit of Work`里面。

​    更多参考：

- [What is a command bus and why should you use it?](https://barryvanveen.nl/blog/49-what-is-a-command-bus-and-why-should-you-use-it)

### 查询对象

​    `查询对象`类似于`命令对象`。它表示用户想要找到某样东西并描述如何去做。

​    `查询对象`用于检索数据，所以不应该进行任何状态更改（包括写入数据库、文件等）。

​    查询通常只是一个数据检索操作，不涉及业务逻辑。因此，如果需要，可以完全绕过应用程序层和领域层。不过，如果在返回查询响应结果之前，施加了一些额外的不改变数据库状态的操作（比如计算某些内容），那么可以在应用层或领域层中执行这些操作。

​    与命令的执行类似，查询可以使用“查询总线”来执行。

​    示例代码：

- [find-users.query.ts](src/modules/user/queries/find-users/find-users.query.ts) - 一个查询对象实例
- [find-users.query-handler.ts](src/modules/user/queries/find-users/find-users.query-handler.ts) - 一个完全绕开应用层和领域层的查询对象实例

---

​    通过使“命令”和“查询”分离，代码变得更容易理解。前者改变一些状态，后者只是检索数据。

​    此外，从一开始就遵循CQS将有助于将经常写数据的模型和经常读数据的模型分离到不同的数据库（CQRS）中（这被称为**数据库的读写分离设计**）。

​    **注意**：这个代码仓库使用[NestJS CQRS](https://docs.nestjs.com/recipes/cqrs)包提供一个命令/查询总线。

​    阅读更多有关CQS和CQRS的内容：

- [Command Query Segregation](https://khalilstemmler.com/articles/oop-design-principles/command-query-segregation/).
- [Exposing CQRS Through a RESTful API](https://www.infoq.com/articles/rest-api-on-cqrs/)
- [What is the CQRS pattern?](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [CQRS and REST: the perfect match](https://lostechies.com/jimmybogard/2016/06/01/cqrs-and-rest-the-perfect-match/)

---

## 端口

​    端口（对于驱动适配器）是定义契约的接口，这些契约必须由基础设施适配器实现，以便执行一些与业务逻辑无关、与技术细节相关的操作。端口就像是一种针对技术细节的抽象，同时业务逻辑不关心这些技术细节。

​    在应用核心层，**依赖指向内侧**。外侧可以依赖于内侧，但内侧从不依赖于外侧。应用核心不应该依赖于框架，也不应该直接访问外部资源。任何对核心进程外资源的外部调用，或从核心外进程取回数据，都应该通过“端口”来完成。具体的方式为，将**资源实现类**创建在基础设施层的某处，并注入到应用核心层(这被称为[依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)和[控制反转](https://en.wikipedia.org/wiki/Dependency_inversion_principle))。这使得业务逻辑独立于应用的核心逻辑。这样便于测试，也允许插入/拔出/交换任何外部资源，轻松地使应用程序模块化和[松散耦合](https://en.wikipedia.org/wiki/Loose_coupling)。

- 端口基本上只是接口，定义了什么必须做，而不关心是如何做的。

- 可以创建端口来抽象I/O操作、技术细节、具有侵入性的库、被复用的遗留代码等。

- 端口应该被创建来适应具体领域的需要，而不是简单地模仿工具API。

- 模拟的资源实现可以在测试时传递到端口，所以端口能使您的测试更快并且独立于运行环境。

- 在设计端口时，请记住[接口隔离原则](https://en.wikipedia.org/wiki/Interface_segregation_principle)。如果有必要，可以将大的接口拆分成小的接口，但也要记住，在不必要的时候不要做得太过。

- 端口也有助于延迟决策。领域层甚至可以在决定使用什么具体的技术（框架、数据库等）之前实现。

​    **注意**：由于大多数端口的实现方式都是在应用服务中注入和执行的，所以应用层可以是保存这些端口的好地方。但是有时候领域层的业务逻辑依赖于消耗一些外部资源，在这种情况下，这些端口可以放在领域层中。

​    **注意**：在较小的应用程序/API 中创建端口可能会添加不必要的抽象，从而使此类解决方案过于复杂。在此类应用程序中，直接使用具体实现而不是端口可能就足够了。在使用“端口”之前，请考虑所有的优缺点。

​    示例代码：

- [repository.ports.ts](src/core/ports/repository.ports.ts)
- [logger.port.ts](src/core/ports/logger.port.ts)

# 领域层

​    领域（或简称“域”），是应用程序所涉及的具体领域的业务规则的集合。领域应该只使用域对象进行操作，下面描述了最重要的域对象。

## 实体

​    实体是领域层的核心，它们封装企业范围内的业务规则和属性。实体可以是具有属性和方法的对象，也可以是一组数据结构和函数的集合。

​    实体描述了业务模型，并表示特定模型具有什么属性、它可以做什么、何时以及在什么条件下可以做这些事情。业务模型的一个实例可能是用户、产品、预订、机票、钱包等。

​    实体必须始终保持其[不变字段](https://en.wikipedia.org/wiki/Class_invariant)：

> ​    域实体应该始终是有效的实体。对于一个对象，应当保证其总是持有一定数量的不变字段。例如，一个订单项对象始终必须有一个购买数量字段，该字段必须是正整数，再加上商品的名称和价格构成一个完整的对象。因此，保证不变字段的有效性是域实体（尤其是聚合根）的责任，实体对象不应该在无效的情况下存在。

​    实体的设计应当遵循以下规则：

- 实体需要完全包含领域特定的业务逻辑。尽可能避免在服务层中包含业务逻辑，否则会导致[Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html)（域服务是业务逻辑的例外，故不能放在单独的实体中）。

- 实体需要拥有一个唯一的标识号，并使其与其他标识号区分开来。且其标识号在它的生命周期中是不变的。

- 两个实体之间的相等，可以通过比较它们的标识符（通常是“id”字段）来确定。

- 实体可以包含其他对象，如其他实体或值对象。

- 实体需要负责收集所有对状态的理解，以及确定某个状态在同一处如何变化。

- 实体需要负责对其所拥有的对象进行协调的操作。

- 实体需要对上层（服务层、控制器层等）的内容一无所知。

- 应该对域实体数据建立合适的模型，以适应具体的业务逻辑，而不是适应数据库的某些固定模式。

- 实体必须保护它们的不变量，尽量避开暴露修改权。好的做法是使用实体暴露给外界的方法更新状态，并在需要时对每次更新执行对“不变字段有效性”的验证（这可以是一个简单的`validate()`方法，用于检查更新是否违反了预先设定的业务规则）。

- 实体在创建时就必须有效。在创建时验证实体和其他域对象，并在第一次失败时抛出错误。[快速失败](https://en.wikipedia.org/wiki/Fail-fast)。

- 针对实体应当避免无参数构造函数或缺省构造函数。这方便通过构造函数接受和验证所有必需的不变量字段。

- 在设计实体时，针对一些需要复杂设置的可选字段，可以使用[Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)和[Builder Pattern](https://refactoring.guru/design-patterns/builder)。

- 实体的部分字段应该不可修改。确定哪些字段在创建后不可修改，并将它们设置为“只读”(例如“id”或“createdAt”字段)。

​    **注意**：很多人倾向于为每个实体创建一个模块，但这种方法并不提倡。一个模块中可以有多个实体。但需要记住的一点是，将多个实体放在单个模块中需要这些实体具有相关的业务逻辑，不要将不相关的实体分组在一个模块中。

​    示例代码：

- [user.entity.ts](src/modules/user/domain/entities/user.entity.ts)

​    更多参阅：

- [Domain Entity pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-entity-pattern.html)
- [Secure by design: Chapter 6 Ensuring integrity of state](https://livebook.manning.com/book/secure-by-design/chapter-6/)

---

## 聚集体

​    [聚集体](https://martinfowler.com/bliki/DDD_Aggregate.html)是一个域对象的群体，可以将其视为单个对象。它封装了概念上聚集在一起的实体对象和值对象。它还包含了一组可以对这些域对象进行操作的操作。

​    聚集体的设计应当遵循以下规则：

- 聚集体通过将多个域对象聚集到单个抽象对象中来帮助简化领域模型。

- 聚集体不应受数据模型的影响。域对象之间的关联与数据库的关系模型存在差异。

- `聚合根（聚集体的根对象）`是一个包含其他实体/值对象和所有操作它们的逻辑的实体。

- 聚合根具有全局的唯一标识。相比之下，边界内的实体仅具有局部的唯一标识，仅在聚集体内唯一。

- 聚合根是访问聚集体内实体的关口。任何来自聚集体外部的引用应该**仅仅**到达聚合根。

- 一个聚集体上的任何操作都必须是[事务性操作](https://en.wikipedia.org/wiki/Database_transaction)。要么保存/更新/删除所有内容，要么什么都不做。

- 只有聚合根可以直接通过数据库查询获得。其他一切聚集体内实体都必须通过内部的遍历来完成。

- 与实体类似，聚集体必须在其整个生命周期中保护其不变量。当提交对聚集体边界内任何对象的更改时，必须满足整个聚集体内的所有不变量。简单地说，一个聚集体中的所有对象都必须是一致的、自洽的，这意味着如果聚集体中的某个对象改变了状态，不应该与该聚集体中的其他域对象发生冲突（这被称为_Consistency bordery）。

- 聚集体中的对象可以包含对其他聚合根的引用。只需要通过全局唯一的标识来引用外部聚集体，而不是通过持有直接对象引用。

- 尽量避免聚集体的体量太过庞大，否则会导致性能和维护问题。

- 聚集体上可以发布“域事件”（更多内容在下面）。

​    所有这些规则都来自于围绕聚集体创建边界的想法。边界简化了业务模型，因为它迫使我们在一组定义良好的规则中非常仔细地考虑每个关系。

​    总之，如果您将多个相关实体和值对象组合在一个根“实体”中，这个根“实体”将成为一个“聚合根”，而这个相关实体和值对象的群体将成为一个“聚集体”。

示例代码：

- [aggregate-root.base.ts](src/core/base-classes/aggregate-root.base.ts) - 聚合根的抽象基类
- [user.entity.ts](src/modules/user/domain/entities/user.entity.ts) - 聚合只是必须遵循上面描述的一组特定规则的实体

​    更多参阅：

- [Understanding Aggregates in Domain-Driven Design](https://dzone.com/articles/domain-driven-design-aggregate)
- [What Are Aggregates In Domain-Driven Design?](https://www.jamesmichaelhickey.com/domain-driven-design-aggregates/)  这是一个系列的多篇文章，不要忘记点击“下一篇文章”在最后。
- [Effective Aggregate Design Part I: Modeling a Single Aggregate](https://www.dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_1.pdf)
- [Effective Aggregate Design Part II: Making Aggregates Work Together](https://www.dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_2.pdf)

---

## 领域事件

​    域事件表示在同一域（同一过程内）的其他部分，知道域中发生的事情。域事件只是推送给位于内存内的域事件分发器的消息。

​    例如，如果用户购买了什么东西，你可能会想：

- 更新购物车;

- 从钱包里取出钱;

- 创建新的发货订单;

- 执行其他与包含“buy”操作的聚集体不相关的域操作。

​    典型的方式包括在所有执行“buy”操作的服务中执行所有这些相关的逻辑，但是这会导致不同子域之间产生耦合。

​    另一种方法是发布“域事件”（也就是采用“事件驱动的隐式调用方式”）。如果执行与一个聚集体实例相关的命令，需要在一个或多个附加聚集体上运行额外的域规则，则可以设计和实现由域事件触发的这些响应。通过订阅一个具体的“域事件”，并根据需求创建尽可能多的事件处理器，可以在同一领域模型中的多个聚集体之间传播状态的变化。这避免了聚集体之间的耦合。

​    域事件可以用于创建[审计日志](https://en.wikipedia.org/wiki/Audit_trail)，通过将每个事件保存到数据库来跟踪对重要实体的所有更改。阅读更多关于为什么审计日志可能是有用的：[为什么软删除是邪恶的，应该做什么代替](https://jameshalsall.co.uk/posts/why-soft-deletes-are-evil-and-what-to-do-instead)。

​    域事件（或其他任何东西）在单个进程中引起的、跨多个聚集体的所有更改，都应该被保存在单个数据库事务中，以保持一致性。像[Unit of Work](https://www.c-sharpcorner.com/UploadFile/b1df45/unit-of-work-in-repository-pattern/)或类似的模式可以帮助解决这个问题。

​    **注意**：该项目使用自定义的实现来发布域事件。不使用[节点事件发射器](https://nodejs.org/api/events.html)，或提供一个事件总线的包（比如[NestJS CQRS](https://docs.nestjs.com/recipes/cqrs)）的理由如下：它们不提供一个选项，来为异步等待`await`所有事件完成，这使得所有事件时是同一个事务的一部分。在单个流程中，要么保存事件所做的所有更改，要么不保存任何更改，以防其中一个事件失败。

​    有多种方法可以实现域事件的事件总线，例如使用[Mediator](https://refactoring.guru/design-patterns/mediator)或[Observer](https://refactoring.guru/design-patterns/observer)等模式。

​    示例代码：

- [domain-events.ts](src/libs/ddd/domain/domain-events/domain-events.ts) - 该类负责为需要发出或捕获事件的任何人提供发布/订阅功能。请记住，这只是一个概念证明示例，可能不是生产应用程序的最佳解决方案。
- [user-created.domain-event.ts](src/modules/user/domain/events/user-created.domain-event.ts) - 保存与已发布事件相关的数据的简单对象。
- [create-wallet-when-user-is-created.domain-event-handler.ts](src/modules/wallet/application/event-handlers/create-wallet-when-user-is-created.domain-event-handler.ts) - 这是一个域事件处理程序的示例，它在引发域事件时执行一些操作（在本例中，当创建用户时，它还为该用户创建钱包）。
- [typeorm.repository.base.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm.repository.base.ts) - 使用存储库发布所有域事件，以便在将更改持久化到聚集体时执行。
- [typeorm-unit-of-work.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm-unit-of-work.ts) - 这保证了所有更改操作被写在了一个事务里面。
- [unit-of-work.ts](src/infrastructure/database/unit-of-work/unit-of-work.ts) - 为事务中使用的领域特定的存储库创建工厂。
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 在这里，我们从`UnitOfWork`获得一个用户存储库并执行一个事务。

​     **注意**：`UnitOfWork`不适用于一部分操作：例如查询操作，或者任何对其他聚集体不会引起副作用的操作。所以你不需要对这些操作使用一个`UnitOfWork`。你应该通过构造函数将依赖注入一个常规的仓库，而不是通过UnitOfWork完成依赖注入。

​    要更好地理解域事件及其实现，请阅读以下内容:

- [域事件模式](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-event-pattern.html)

- [领域事件:设计和实现](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)

​    **注意**：记住，如果你只使用事件的复杂工作流与许多步骤，将很难跟踪所有正在发生的应用程序。一个事件可能触发另一个事件，然后是另一个，以此类推。要跟踪整个工作流，您必须去追踪多个地方，并为每个难以维护的步骤搜索相应的事件处理器。在这种情况下，使用服务/编排器/中介可能比只使用事件更可取，因为您将在一个地方拥有整个工作流。这可能会产生一些耦合，但更容易维护。不要只依赖于事件，要选择最合适的方式。

​    **注意**：记住，如果你使用[事件来源模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)与单聚集体单事件流，你很可能无法保证所有事件跨多个聚集体写在一个事务里。跨多个聚集体在这种情况下保存事件应该区别对待（例如通过使用[Sagas](https://microservices.io/patterns/data/saga.html)的补偿事件机制，或[过程管理器](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)，或类似的机制）。

## 集成事件

​    进程外通信(调用微服务，外部API)被称为“集成事件”。如果需要向外部进程发送域事件，则域事件处理程序应该发送一个“集成事件”。

​    只有在所有域事件完成执行，并保存所有对数据库的更改后，才应该发布集成事件。

​    要处理微服务中的集成事件，你可能需要一个外部消息代理/事件总线，比如第三方的[RabbitMQ](https://www.rabbitmq.com/)或[Kafka](https://kafka.apache.org/)，也可能需要实现 [Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html), [Change Data Capture](https://en.wikipedia.org/wiki/Change_data_capture), [Sagas](https://microservices.io/patterns/data/saga.html)或者[Process Manager](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)等模式，来保证事件的最终一致性（[eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency)）。

​    针对分布式系统中的集成事件，下列模式可能很有用：

- [Saga distributed transactions](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- [Saga vs. Process Manager](https://blog.devarchive.net/2015/11/saga-vs-process-manager.html)
- [The Outbox Pattern](https://www.kamilgrzybek.com/design/the-outbox-pattern/)
- [Event Sourcing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)

---

## 域服务

​    摘自Eric Evans的《领域驱动设计》:

> 域服务用于“域中的重要过程或转换，而不是**实体或值对象**的自然责任”。

- 域服务是一种特定类型的领域层的类，用于执行依赖于两个或更多实体的域逻辑。

- 如果将逻辑放到一个特定的实体上会打破封装，并要求这个实体知道它实际上不应该关心的特定逻辑。这种情况下，就应该使用域服务。

- 域服务是非常细粒度的，而应用服务是旨在暴露API给外界的表观模式的实现。

- 域服务只对属于该域的特定类型进行操作。它们包含能在通用语言中找到的、有实际意义的概念。它们封装了不适合值对象或实体的操作。

## 值对象

​    一些属性和行为可以移出实体本身，放入“值对象”当中。

​    值对象的特征：

- 不具有标识符。值对象之间是否相等是由其结构性质决定的。

- 是不可变的。

- 可以用作'实体'和其他'值对象'的属性。

- 显式定义和强制的有效性约束（不变量的约束）。

​    值对象不应该只是一个便捷的属性分组，而是应该在领域模型中形成一个定义良好的概念。即使它只包含一个属性，也是如此。当需要将一个值对象建模为一个概念整体时，它在被传递时就具备意义，并且能够维护其约束。

​    举个例子，假设你有一个“用户”实体，它需要一个用户的“地址”。通常，地址只是一个复杂的值，在域中没有标识，由多个其他值组成，如“国家”，“街道”，“邮编”等。因此，“地址”可以基于自己的业务逻辑进行建模，并作为一个“值对象”对待。

​    “值对象”不仅仅是一个保存值的数据结构。它还可以封装与它所代表的概念相关联的行为逻辑。

​    示例代码：

- [address.value-object.ts](src/modules/user/domain/value-objects/address.value-object.ts)

​    更多参阅：

- [Martin Fowler blog](https://martinfowler.com/bliki/ValueObject.html)
- [Value Objects to the rescue](https://medium.com/swlh/value-objects-to-the-rescue-28c563ad97c6).
- [Value Object pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/value-object-pattern.html)

## 域对象的强制不变性

### 使用值对象取代基本类型变量

​    大多数代码的计算是基于基本类型——“字符串”、“数字”等的。在领域模型中，这个抽象级别可能太低了。

​    重要的业务概念可以使用特定的类型和类来表示。“值对象”可以代替基本变量，以避免[基本变量引起的困扰](https://refactoring.guru/smells/primitive-obsession)。

​    例如，类型为'string'的'email'：

> email: string;

​    可以用“值对象”来代替：

> email: Email;

​    现在创建email的唯一方法是首先创建一个email类的新实例，这样可以确保它在创建时被验证，错误的值不会被写入实体。

​    此外，将领域相关的基本变量的重复行为集中封装在一个地方。通过让域原语拥有并控制域操作，您可以减少由于缺乏对操作中涉及的概念的详细领域知识而导致的错误。

​    为基本类型变量创建对象可能很麻烦，但它在某种程度上**迫使开发人员更详细地研究该特定领域**，而不是仅仅抛出一个基本类型，甚至不考虑该类型在该领域中具体表示什么。

​    对基本类型使用“值对象”也称为“域基本类型”。这个概念和命名在[《设计安全》](https://www.manning.com/books/secure-by-design)一书中提出。

​    使用“值对象”代替基本变量:

- 通过使用[泛在语言](https://martinfowler.com/bliki/UbiquitousLanguage.html)而不是仅仅使用`string`使代码更容易理解。

- 通过确保每个属性的不变量来提高安全性。

- 封装与值关联的特定业务规则。

​    另外，创建对象的一种替代方法是[类型别名](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases)，目的是给这个域基本类型赋予语义意义。

​    示例代码：

- [email.value-object.ts](src/modules/user/domain/value-objects/email.value-object.ts)

​    更多参阅：

- [Primitive Obsession — A Code Smell that Hurts People the Most](https://medium.com/the-sixt-india-blog/primitive-obsession-code-smell-that-hurt-people-the-most-5cbdd70496e9)
- [Value Objects Like a Pro](https://medium.com/@nicolopigna/value-objects-like-a-pro-f1bfc1548c72)
- [Developing the ubiquitous language](https://medium.com/@felipefreitasbatista/developing-the-ubiquitous-language-1382b720bb8c)

​    **使用值对象/域基本类型和类型系统，能使非法的状态在您的程序中不可表示、不可产生。**

​    有些人甚至推荐为**每个值**都使用一个对象来表示。摘自 [John A De Goes](https://twitter.com/jdegoes)：

> 使非法状态不可表示，就是静态地证明所有运行时值(没有例外)都对应于业务域中的有效对象。这种技术在消除无意义运行时状态方面的效果是惊人的。

​    以下，让我们为“**类型系统**”区分两种针对非法状态进行保护的场景：编译期和运行时。

### 编译期

​    类型为开发人员提供了有用的语义信息。好的代码应该**易于正确地使用，而难于错误地使用**。类型系统是一个很好的帮助。它可以在编译时防止一些严重的错误，因此IDE或编译器将立即显示类型错误。

​    最简单的例子可能是**使用枚举而不是常量**，并使用这些枚举作为某些输入类型。当传递任何非预期的内容时，IDE将显示类型错误。

​    或者，想象一下，业务逻辑要求通过“电子邮件”或“电话”，或两者都有，来获取一个人的联系信息。' email '和' phone '都可以表示为可选的，例如：

```typescript
interface ContactInfo {
  email?: Email;
  phone?: Phone;
}
```

​    但是，如果程序员没有提供这两种类型，会发生什么呢？业务规则将被违反，非法状态将被允许。

​    解决方案：可以被呈现为一个[联合类型](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#union-types)：

```typescript
type ContactInfo = Email | Phone | [Email, Phone];
```

​    现在只需要输入“Email”或“Phone”，或者两者都要输入。如果什么也不提供，IDE将立即显示一个类型错误。现在，业务规则验证从运行时转移到编译时，这使应用程序更安全，并在该类型没有按预期使用时提供更快的反馈。

​    这被称为*typestate pattern*。（这也是为什么TypeScript要在JavaScript上增加类型系统的原因。）

> typestate pattern是一种API设计模式：将一个对象的运行时状态信息编码在它的编译期类型中。

​    参阅更多有关typestates的内容：

- [Typestates Would Have Saved the Roman Republic](https://blog.yoavlavi.com/state-machines-would-have-saved-the-roman-republic/)
- [The Typestate Pattern](https://cliffle.com/blog/rust-typestate/)

### 运行时

​    运行时与编译期的区别在于，程序对不能在编译期验证的值（比如用户的输入）在运行时验证。

​    域对象必须保护它们的不变量，维持其合法性。在此设置一些验证有效性的规则将避免这些状态陷入非法状态。

​    “值对象”可以表示域中具备类型的值（域基本类型）。这里的目标是，封装仅与所表示字段相关的验证和业务逻辑，并通过强制首先创建有效的“值对象”，从而不可能传递原始值。这些被创建的值对象只接受在其上下文中有意义的值。

​    如果方法的每个参数和返回值从定义上来说都是有效的，那么无需任何额外的工作，就可以在代码库中的每个方法中进行输入和输出验证。这将使应用程序对输入错误更有容忍性，并将保护程序不受无效输入数据引起的一系列错误和安全漏洞的影响。

​    数据不应该被信任。在许多情况下，无效数据可能最终出现在域中。例如，如果数据来自外部API、数据库，或者仅仅是程序员的错误，都可能导致无效数据的产生。

​    强制自我验证将在数据损坏时立即通知。不验证域对象的有效性，而允许它们处于不正确的状态，这可能会导致问题。

> 如果没有域原语，剩下的代码需要处理验证、格式化、比较和许多其他细节。实体代表着具有显著标识的长生命周期对象，例如新闻提要中的文章、酒店中的房间和在线销售中的购物车。系统的功能通常以改变这些对象的状态为重点：酒店的房间被预定，购物车的内容被预定付费，等等。控制流迟早会转到表示这些实体的代码。如果所有数据都以Int或String等无限定的类型传输，那么验证、比较和格式化数据的责任就落在了实体部分的代码上。实体代码将被大量的细节任务，而不是关注它所建模的核心业务的状态更改。因此，使用域原语可以削弱实体部分代码变得过于复杂的趋势。

引自：[Secure by design: Chapter 5.3 Standing on the shoulders of domain primitives](https://livebook.manning.com/book/secure-by-design/chapter-5/96)

**注意**：虽然_primitive obsession_ 是一种代码坏味，一些人认为为每个域原语创建一个类/对象可能是过度的。对于不太复杂和较小的项目，这个观点可能是正确的。但对于更大的项目，有人支持也有人反对这种方法。如果不喜欢为每个域原语创建类，那么只为那些具有特定规则或特定行为的域原语创建类，或者只在域外使用某些验证框架进行验证。以下是关于这个话题的一些想法：

- [From Primitive Obsession to Domain Modelling - Over-engineering?](https://blog.ploeh.dk/2015/01/19/from-primitive-obsession-to-domain-modelling/#7172fd9ca69c467e8123a20f43ea76c2)

- [Making illegal states unrepresentable](https://v5.chriskrycho.com/journal/making-illegal-states-unrepresentable-in-ts/)

- [Domain Primitives: what they are and how you can use them to make more secure software](https://freecontent.manning.com/domain-primitives-what-they-are-and-how-you-can-use-them-to-make-more-secure-software/)

- ["Secure by Design" Chapter 5: Domain Primitives](https://livebook.manning.com/book/secure-by-design/chapter-5/) (a full chapter of the article above)

### 如何做简单的验证

​    对于简单的验证，比如检查空值、空数组、输入长度等，可以创建一个[guards](https://en.wikipedia.org/wiki/Guard_(computer_science)) 函数库。

​    示例代码：[guard.ts](src/libs/ddd/domain/guard.ts)

​    更多参阅：[Refactoring: Guard Clauses](https://medium.com/better-programming/refactoring-guard-clauses-2ceeaa1a9da)

​    另一个解决方案是使用外部验证库，但是将域绑定到外部库并不是一个好的实践，通常也不推荐。

​    尽管在需要时也会有例外，例如对于只验证某个对象的特定字段（如特定的ID、比特币钱包地址）。只将一个或几个“值对象”绑定到这样一个特定的库不会造成任何伤害。通用验证库可以被绑定到任何域，如果改变与之相关的任意“值对象”，就会存在以下麻烦：旧的库可能不再维护、旧的库可能包含致命的漏洞。

​    不过，如果你是要完成完整性的检查，则可以使用**域外**的验证框架或库（例如针对DTOs的装饰器：[class-validator](https://www.npmjs.com/package/class-validator)），也可以使用只做一些基本内部检查的值对象（不涉及业务规则），例如检查“零值”或“未定义值”，检查长度，匹配简单的正则表达式等，检查值是否有意义以及额外的安全性。

<details>
<summary>使用正则表达式（regexp）的注意事项</summary>
​    对于像验证email一类的数据，要小心地自定义regexp验证，只对一些非常简单的规则使用自定义regexp。如果可能，让验证库在更困难的规则上完成它的工作，以避免在你的regexp不够好时出现问题。
    另外，请记住，如果自定义regexp执行的验证类型与域外验证库已经执行的验证类型相同，则可能会在您的regexp和验证库使用的regexp之间产生冲突。
    例如，value可以被验证库接受为有效，但value Object可能抛出错误，因为自定义regexp不够好(验证email比复制粘贴在谷歌中的正则表达式要复杂得多。不过，它可以通过一个简单的规则来验证，这个规则一直都是正确的，不会引起任何冲突，比如每一封“电子邮件”必须包含一个“@”)。尝试只查找和验证不会引起冲突的模式。

---

</details>

​    虽然在域内进行验证还有其他策略，比如在创建新的“值对象”时将验证模式作为依赖进行传递，但这增加了额外的复杂性。

​    是否使用外部库/框架进行域内验证是一种权衡，分析所有的利弊，并选择更适合当前应用程序的方法。对于一些项目，特别是小型项目，使用验证库/框架可能更容易也更合适。

​    请记住，不是所有的验证都可以在一个“值对象”中完成，它应该只验证所有上下文共享的规则。在某些情况下，根据上下文的不同，验证可能会有所不同，或者一个字段可能涉及另一个字段，甚至可能涉及不同的实体。应当分别处理这些具体情况。

### 验证的种类

​    本节对验证顺序提出一些一般性的建议。像检查null/undefined和检查数据长度这样的低代价操作列在列表前部，而需要调用数据库的更高代价的操作列在后面。

- *数据源*：数据是否来自合法的发送方？在可能的情况下，根据情况只接受授权用户/白名单IP等的数据源。

- *存在性*：数据是否为空？如果数据为空，进一步的验证没有意义。待检查的空值：空值变量、未定义变量、空对象和空数组。

- *大小*：数据的规模是否过大？在下一步之前，检查输入数据的长度/大小，无论它是什么类型。这将防止验证可能会完全阻塞线程的大规模数据（发送太大规模的数据可能是一次[DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)攻击）。

- *词法内容*：数据是否包含非法的字符或编码？例如，如果我们期望数据只包含数字，我们扫描它看看是否有其他字符。如果我们发现了其他任何非数字的字符，我们就会得出结论：数据要么是被错误地破坏了，要么是被恶意设计来欺骗我们的系统。

- *语法*：数据的格式是否正确？检查数据格式是否正确。有时检查语法会像使用正则表达式检查基础字符串一样简单，但也存在复杂情况，比如检查XML或JSON格式的数据。

- *语义*：数据有意义吗？检查与系统其他部分（如数据库，其他进程等）相互连接的数据。例如，查询数据库以验证ID是否存在。

更多参阅：

- ["Secure by Design" Chapter 4.3: Validation](https://livebook.manning.com/book/secure-by-design/chapter-4/109).

## 领域错误

​    异常是针对异常情况的。复杂的领域上通常有很多错误，这些错误并不是异常的，而是业务逻辑的一部分（比如座位已经预订，请选择另一个）。这些错误可能需要特殊处理。在这些情况下，返回显式错误类型可能是比抛出运行时异常更好的做法。

​    返回错误而不是显式地抛出，显示方法可以返回的每个异常的类型，以便您可以相应地处理它。这可以使错误处理和跟踪更容易。

​    为了帮助使用某种成功或失败的结果对象类型（例如来自函数式编程语言的 [monad](&lt;https://en.wikipedia.org/wiki/Monad_(functional_programming)&gt;) 类型，如Haskell)。与抛出异常不同，这种方式允许为每个错误定义类型，并将迫使你显式地处理这些情况，而不是懒惰地使用“try/catch”结构。例如:

```typescript
if (await userRepo.exists(command.email)) {
  return Result.err(new UserAlreadyExistsError()); // returning an Error
}
// else
const user = await this.userRepo.create(user);
return Result.ok(user);
```

​    [@badrap/result](https://www.npmjs.com/package/@badrap/result) 如果你想使用Result类型的话，这会是一个很好的npm包。

​    返回错误而不是抛出错误会增加一些额外的样板性的代码，但会使应用程序更加健壮和安全。

​    **注意**：区分领域错误和异常。异常通常会被抛出而不返回。如果您返回技术上的异常（例如连接失败，进程内存不足等），它可能会导致一些安全问题，且违背[Fail-fast](https://en.wikipedia.org/wiki/Fail-fast)原则。返回异常不是终止程序流，而是继续程序执行，并允许它在不正确的状态下运行，这可能会导致更多意想不到的错误，所以在这些情况下，通常最好抛出一个异常，而不是返回它。

示例代码：

- [user.errors.ts](src/modules/user/errors/user.errors.ts) - 用户错误
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 注意到` Result.err(new UserAlreadyExistsError()) `将错误结果返回而不是抛出
- [create-user.http.controller.ts](src/modules/user/commands/create-user/create-user.http.controller.ts) - 在用户的HTTP控制器中，我们解构一个错误结果并决定如何处理之。如果错误是`UserAlreadyExistsError`我们将抛出一个`Conflict Exception` 异常，如此用户会收到消息：`409 - Conflict`。如果某个错误是未知的，我们直接将其抛出，如此NestJS会返回给用户消息：`500 - Internal Server Error`。
- [create-user.cli.controller.ts](src/modules/user/commands/create-user/create-user.cli.controller.ts) - 一个CLI控制器，我们不关心返回一个恰当的状态码，所以我们直接使用`.unwrap()`方法来解构这个结果即可，这个方法针对错误的结果会将其抛出。

更多参阅：

- ["Secure by Design" Chapter 9.2: Handling failures without exceptions](https://livebook.manning.com/book/secure-by-design/chapter-9/51)
- [Flexible Error Handling w/ the Result Class](https://khalilstemmler.com/articles/enterprise-typescript-nodejs/handling-errors-result-class/)

## 在应用核心中使用库

​    是否在应用程序核心中，特别是领域层中使用库是一个有很多争议的话题。在现实世界中，注入每个库而不是直接导入它并不总是可行的，因此对于一些帮助实现领域特定逻辑的单一职责库（如处理数字的函数库）可能会出现异常。

​    需要牢记的主要建议是，在应用程序的核心中导入的库**不应该**暴露以下功能：

- 访问任何进程外资源的功能（如HTTP调用，数据库访问等）

- 与领域无关的功能（包括各种框架和技术细节，如ORM, Logger等）

- 带来随机性的功能（生成随机ID，时间戳等），因为这会让测试变得不可预测（尽管在TypeScript世界里，这并不是什么大不了的事，因为这可以通过测试工具进行模拟，而不是通过依赖注入）

- 如果一个库经常变化或者有很多自己的依赖，它很可能不应该在领域层中使用。
  
  要使用这样的库，可以考虑使用[adapter](https://refactoring.guru/design-patterns/adapter)或[facade](https://refactoring.guru/design-patterns/facade)模式创建一个“反腐败层"。

​    我们有时允许库位于中心，但要小心通用的库可能分散在许多域对象中。在需要时很难替换这些库。只将一个或几个域对象绑定到某个单一职责库应该没问题。与随处可见的通用库相比，替换绑定到一个或几个对象的特定库要容易得多。

​    除了不同的库之外，还有框架需要考虑。框架可能是一个真正的麻烦，因为根据框架的定义，它们想要控制整个程序，当你的整个应用程序都粘在框架上时，就很难取代框架。在外部层（如基础设施）中使用框架是可行的，但是在可能的情况下，不要在你的领域层中使用框架。您应该能够分离您的领域层，并使用任何其他框架围绕它构建一个新的基础设施，从而会破坏您的业务逻辑。

​    在这一点上，NestJS做得很好，因为它使用的装饰器不具备重度的侵入性，所以你可以使用`@Inject()`之类的装饰器，而不会影响你的业务逻辑，且在需要时删除或替换这些装饰器也相对容易。不要完全放弃框架，但要将它们限制在一定范围内，不要让它们影响您的业务逻辑。

​    从核心层中，尤其是从领域层中，尽可能多地卸下无关的责任。此外，要尽量减少依赖关系的使用。软件的依赖性越大，意味着潜在的错误和安全漏洞就越多。使软件更健壮的一个诀窍就是，最小化您的软件所依赖的事物——出错的概率越小，出错的频率就越小。另一方面，删除所有依赖项将会适得其反，因为重现这些功能将会需要大量的工作，而且比仅仅使用一个应用广泛的依赖项的可靠性更低。找到一个好的平衡很重要，这个技巧需要经验。

更多参阅：

- [Referencing external libs](https://khorikov.org/posts/2019-08-07-referencing-external-libs/).
- [Anti-corruption Layer — An effective Shield](https://medium.com/@malotor/anticorruption-layer-a-effective-shield-caa4d5ba548c)

# 接口适配层

​    接口适配器（也称为驱动适配器/首要适配器，或称为API层）是面向用户的接口，它们从用户处获取输入数据，并将其重新打包为便于用例（服务/命令处理程序）的形式和实体。然后，它们从这些用例和实体中获取输出，并将其重新打包成便于向用户显示的形式。用户可以是使用应用程序的人，也可以是其他下游服务器。

​    这些接口适配器可以包含“控制器”和“请求”/“响应”DTO（也可以包含“视图”，比如后端生成的HTML模板）。

## 控制器

- 控制器是面向用户的API，能够处理服务的请求，触发特定的业务逻辑，也可以将服务的结果表示返回给客户端。
- 最好是针对每一个用例设计一个控制器。
- 在使用[NextJS](https://docs.nestjs.com/)框架时，控制器可以选用[OpenAPI/Swagger decorators](https://docs.nestjs.com/openapi/operations)装饰器来生成文档。

针对每个触发类型设计一个独立的控制器会更加条理清晰。实例：

- [create-user.http.controller.ts](src/modules/user/commands/create-user/create-user.http.controller.ts) 针对HTTP请求 ([NestJS Controllers](https://docs.nestjs.com/controllers)),
- [create-user.cli.controller.ts](src/modules/user/commands/create-user/create-user.cli.controller.ts) 针对CLI调用([NestJS Console](https://www.npmjs.com/package/nestjs-console))
- [create-user.message.controller.ts](src/modules/user/commands/create-user/create-user.message.controller.ts) 针对外部消息 ([NetJS Microservices](https://docs.nestjs.com/microservices/basics)).
- 等等

### 解析器

​    如果您使用[GraphQL](https://graphql.org/)而不是控制器，您将使用[Resolvers](https://docs.nestjs.com/graphql/resolvers)解析器。

​    分层架构的主要好处之一是**关注点分离**。正如您所看到的，使用[REST](https://en.wikipedia.org/wiki/Representational_state_transfer)或GraphQL并不重要，唯一的重点是面向用户的API层（接口适配层）。所有的应用程序的核心逻辑保持不变，因为它不依赖于你所使用的技术。

​    示例代码：

- [create-user.graphql-resolver.ts](src/modules/user/commands/create-user/create-user.graphql-resolver.ts)

---

## 数据传输对象

​    来自应用程序外围的数据应当被**表示**为特定的数据类型——数据传输对象，简写为[DTO](https://en.wikipedia.org/wiki/Data_transfer_object)（Data Transfer Object）。DTO是在过程之间携带数据的对象，它定义了API层和客户端之间的一种合约。

### 请求DTO

​    请求DTO表示的是来自用户的输入数据。

- 使用请求DTO将给出一种合约，它指定了客户端如何向API层提出正确的请求。

​    示例代码：

- [create-user.request.dto.ts](src/modules/user/commands/create-user/create-user.request.dto.ts)
- [create.user.interface.ts](src/interface-adapters/interfaces/user/create.user.interface.ts)

### 响应DTO

​    响应DTO表示的是返回给用户的输出数据。

- 使用响应DTO保证了客户端只会收到此类DTO合约描述的数据，而不是程序拥有的任何模型或实体（输出这些实体甚至会导致数据泄露）。

​    示例代码：

- [user.response.dto.ts](src/modules/user/dtos/user.response.dto.ts)
- [user.interface.ts](src/interface-adapters/interfaces/user/user.interface.ts)

---

​    使用DTO可以保护客户端免受API层中可能发生的内部数据结构的变更的影响。当内部数据模型发生变化（如重命名变量或分割表）时，仍然可以将它们映射为相应的DTO，以保持对任何API使用者的兼容性。

​    当更新DTO接口时，可以通过在端点前加上版本号，来创建API的新版本，例如:' v2/users '。这将使得用户缓慢地更新他们的应用程序中的API，从而减少对兼容性的破坏，并使API的过渡过程变得轻松。

​    你可能已经注意到我们的[create-user.command.ts](src/modules/user/commands/create-user/create-user.command.ts)包含与[create-user.request.dto.ts](src/modules/user/commands/create-user/create-user.request.dto.ts)相同的属性。那么，如果我们已经有带有属性的Command对象，为什么还需要DTO呢？我们不应该只设计一个类来避免重复吗？

>  因为Command类和DTO类是不同的，它们处理不同方面的问题。命令是可序列化的方法调用——领域模型中方法的调用。而DTO是数据合约。引入这个描述数据合约的独立层的主要原因，是为依赖API的客户端程序提供向后兼容性。如果没有DTO，每次修改领域模型时，API都会发生重大变化，这会给用户带来困扰。

​    更多参阅：[CQRS命令是领域模型的一部分吗？](https://enterprisecraftsmanship.com/posts/cqrs-commands-part-domain-model/)(阅读“_Commands vs DTOs_”部分)。

### 额外建议

- DTO应该是面向数据的，而不是面向对象的。它的属性应该是基本的。我们这里没有建模，只是发送纯粹的数据。

- 当返回一个“响应”，_whitelisting_属性比_blacklisting_属性更好。因为这确保了不会有敏感数据的泄露，以防程序员忘记将新添加的不应该返回给用户的属性列入黑名单。

- “请求”/“响应”对象的接口应该保存在共享目录的某个地方，而不是模块目录，因为它们可能被不同的应用程序（如前端页面、移动端应用或微服务）复用。可以创建一个单独的Git子模块或者一个单独的包来共享接口。

- “请求”/“响应”DTO类中可以嵌入验证器和卫生处理装饰器，例如[class-validator](https://www.npmjs.com/package/class-validator)和[class-sanitizer](https://www.npmjs.com/package/class-sanitizer)（确保收集完全部的验证错误，之后再返回给用户，这被称为[通知模式](https://martinfowler.com/eaaDev/Notification.html)。类验证器默认是这样处理的）。

- “请求/响应”DTO类也可以使用[NestJS](https://docs.nestjs.com/openapi/types-and-parameters)提供的Swagger/OpenAPI库装饰器。

- 如果DTO不使用验证或文档相关的装饰器，DTO可以只是一个接口，而不是类和接口的组合。

- 数据可以使用单独的映射器转换为DTO格式。如果使用DTO类，可以正好在构造函数中进行格式转换。

---

# 基础设施层

​    基础设施层对技术的保持负有严格的责任。可以在基础设施层找到业务实体、消息代理、I/O组件、依赖项注入、框架和其他任何描述架构细节的数据库和存储库的实现，同时基础设施层还囊括了框架依赖、外部依赖等项目依赖。

​    基础设施层也是最易于变化的一层。由于这一层中的代码很可能会发生变化，所以这些代码应当尽可能地远离更稳定的软件层。同时，因为基础设施层中的组件之间是隔离的，所以较为容易进行组件的重构，或将一个组件替换为另一个组件。

​    基础架构层可以包含“适配器”，也可以包含数据库相关代码，如“数据仓库”、“ORM实体和模式”，以及框架相关的代码等等。

## 适配器

- 基础设施层的适配器（也称为驱动适配器或二级适配器）使软件系统能够在被请求时与外部系统进行交互，主要是通过接收、存储和提供数据，包括数据持久化、消息代理、发送电子邮件、请求第三方API等方式。

- 适配器还可以用于与单个过程中的不同域交互，以避免这些域之间的紧耦合。

- 适配器本质上是端口的实现。它们不应该在代码中的任何地方直接被调用，只能通过端口被调用。

- 适配器可以用作遗留代码的反腐败层（ACL，anti-corruption layer）。

​    关于ACL更多参阅： [Anti-Corruption Layer: How to Keep Legacy Support from Breaking New Systems](https://www.cloudbees.com/blog/anti-corruption-layer-how-keep-legacy-support-breaking-new-systems)

​    一个特定的适配器应当包含以下实现：

- 一个端口，实现在应用层或领域层的某处；
- 一个映射器，从领域映射出数据，或将数据映射到领域上（如果必要的话）；
- 一个DTO/接口，在接收数据的时候发挥作用；
- 一个验证器，来确保输入的数据未被破坏（验证工作可以放在DTO类中，交由装饰器来完成，也可以通过值对象来验证）。

## 数据仓库

​    数据仓库集中管理公共数据的访问功能。仓库封装了访问这些公共数据的特定逻辑。实体或聚合体可以放在数据仓库中，以便后期的检索。检索过程不依赖于特定的域，甚至不需要知道数据保存在数据库、文件还是其他数据源当中。

​    我们使用数据仓库将用于访问数据库的基础设施和基本过程，从领域模型层中解耦分离出来。

​    Martin Fowler对数据仓库的描述如下:

> 数据仓库在领域层和数据映射之间执行中介任务，其作用方式类似于内存中的**一组**域对象。客户端对象以声明的形式构建查询，并将查询发给数据仓库以获取结果。从概念上讲，数据仓库封装了一组存储在数据库中的对象和可以对其执行的增删改查操作，提供了一种更接近于持久层的方式。数据仓库的另一个优点是，在一个方向上清晰地分离了工作域与数据分配或数据映射之间的依赖关系。

​    这里的数据流看起来像这样：数据仓库从应用服务接收到一个域实体，将它映射成为数据库模式或ORM格式，执行所需的增删改查操作，并将操作结果映射回域实体，最后返回给服务。

​    **注意**，应用程序的核心层不允许直接依赖数据仓库，而应当依赖于更进一步的封装或抽象形式（如端口、接口的形式）。这使得数据的检索技术是不可知的。

### 示例

​    这个项目  [typeorm.repository.base.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm.repository.base.ts) 包含了一个抽象的数据仓库类，它允许进行基本的CRUD操作。然后，这个基类被一个特定的数据仓库扩展，实体可能需要的所有特定操作都在这段特定的代码中实现：[user.repository.ts](src/modules/user/database/user.repository.ts)。

## 持久化模型
