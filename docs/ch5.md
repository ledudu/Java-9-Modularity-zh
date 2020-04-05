# 第 5 章 模块化模式

> Chapter 5. Modularity Patterns

Mastering a new technology or language feature in some ways feels like acquiring a new superpower. You immediately see the potential, and you want to change the world by applying it everywhere. If anything, superheroes from comics have shown us the world isn’t this black-and-white—even when you believe you have a superpower. Rarely is good and bad use of new powers immediately apparent; with great superpower comes great responsibility.

> 从某些方面讲，掌握一门新技术或一项新语言功能感觉就像是掌握了一项新的超能力一样。此时，非常渴望立即看到超能力的潜力，并希望由此来改变世界。如果有什么区别的话，就是漫画中的超级英雄已经向我们展现了世界不是黑白分明的——即使你相信自己已经拥有了超能力。能力运用的好坏不会马上显现出来；往往能力越大，责任也就越大。

Learning the Java module system is no different. Knowing the module system does nothing for you if you do not also know about modular design principles to wield its power. Modularity is more than just an implementation concern, eased by the introduction of new language features. It’s a matter of design and architecture as well. Applying modular design also is a long-term investment. With it, you can hedge against changes in requirements, environments, teams, and other unforeseen events.

> 学习 Java 模块系统也是一样的。如果不通过了解模块化设计原则来掌握其功能，那么仅仅了解模块系统对你来说没有任何意义。模块化不仅仅是一个实现问题（该问题可通过引入新的语言特性来缓解），也是一个设计和架构的问题。应用模块化设计是一项长期的投资。通过模块化，可以应对需求、环境、团队以及其他不可预见事件所带来的变化。

In this chapter, we discuss common patterns to improve the maintainability, flexibility, and reusability of systems built using modules. It’s essential to remember that many of these patterns and design practices are technology agnostic. Although there is code in this chapter, it serves to illustrate these sometimes abstract patterns in the context of the Java module system. The focus is on effectively modularizing a system by applying established patterns of modularity.

> 本章将讨论通用模式，以提高使用模块所构建系统的可维护性、灵活性和可重用性。请记住，这些模式和设计实践中的大部分与技术无关。虽然本章提供了代码，但它们主要用于在 Java 模块系统上下文中说明这些抽象的模式，关注的重点应是如何通过应用已建立的模块化模式有效地模块化系统。

If these patterns seem obvious to you, congratulations! You’ve been doing modular development all along. Still, the Java module system offers more support for writing modular code than ever before in the language. And not just for you, but across your whole team and even the Java ecosystem at large. If, on the other hand, these patterns are new to you, congratulations as well. By learning and applying these patterns and design practices, your applications become easier to maintain and extend.

> 如果你对这些模式非常熟悉，那么恭喜了！你一直在从事模块化开发。尽管如此，Java 模块系统比以前的语言提供了更多的编写模块化代码的支持。这样做不仅仅是为了你，更是为了整个团队，甚至整个 Java 生态系统。而如果你是首次接触这些模式，那么也要恭喜你。通过学习和应用这些模式和设计实践，你开发的应用程序会变得更容易维护和扩展。

This chapter discusses basic modularity patterns that you will encounter frequently in application development. We start with some general module design guidelines, followed by more concrete module patterns. In Chapter 6, we discuss more advanced patterns that may appeal only to developers of extremely flexible applications, such as generic application containers or plug-in-based systems.

> 本章将讨论应用程序开发中经常遇到的基本模块化模式。首先，从一些通用模块设计指南开始，然后再学习更具体的模块模式。第 6 章将会讨论更高级的模式，那些需要开发具有较高灵活性的应用程序的开发人员会对这些模式感兴趣，比如通用的应用程序容器或基于插件的系统。

## 5.1 Determining Module Boundaries 确定模块边界

What makes a good module? It may surprise you that this question is in fact age-old. Dividing systems into small, manageable modules has been identified as a winning strategy since the inception of our profession. Take, for example, this quote from a 1972 paper:

> 什么样的模块是一个好的模块？你会惊奇地发现，这个问题实际上由来已久。长久以来，将系统划分为小型的、可管理的模块已被认为是一项成功的策略。举个例子，下面的一段话引自 1972 年的一篇论文（http://bit.ly/parnas-on-the）：

The effectiveness of a “modularization” is dependent upon the criteria used in dividing the system into modules.

D.L. Parnas, “On the Criteria To Be Used in Decomposing Systems into Modules”

> “模块化”的有效性取决于将系统划分成模块所使用的标准。
>
> —D.L.Parnas, 《On the Criteria To Be Used in Decomposing Systems into Modules》

One of the main points this paper makes is that modularization starts before any code is written. Module boundaries should be derived from the system’s design and intent. In the following sidebar, you can read how Parnas approached this challenge. Like the quote says, the criteria used to draw boundaries determine the success of a modularization effort. So what are those criteria? As always, it depends.

> 该文提出的要点之一是在编写任何代码之前进行模块化。模块边界应该源自系统的设计和意图。在下面的栏目中，可以了解一下 Parnas 是如何应对这一挑战的。就像上面引文所说的那样，确定边界的标准决定了模块化工作的成功。那么这些标准是什么呢？通常，视情况而定。

### PARNAS PARTITIONING Parnas 分区

Based on his 1972 paper, D.L. Parnas devised an approach to modularization called Parnas partitioning. It’s always a good idea to take possible change into account when thinking about module boundaries. With Parnas partitioning, you construct a hiding assumption list. The list contains areas from the system that are expected to change or have contentious design decisions, along with their estimated probability of changing later. Functionality is divided into modules using this list to minimize the impact of change. High-probability items on the hiding assumption list are prime candidates to be encapsulated in a module. Creating such a list is not a hard science. It’s something you can do together with technical and nontechnical stakeholders.

> 根据 D.L.Parnas 在 1972 年的一篇论文中所述，他设计了一种称为 Parnas 分区（Parnaspartitioning）的模块化方法。在思考模块边界时，考虑到可能的变化总是一件好事。通过使用 Parnas 分区，可以构建一个隐藏假设列表（hiding assumption list）。该列表包含了系统中预期会发生变化或存在设计决策争议的地方，以及日后变化的概率。根据该列表将功能划分为模块，可以最小化更改所带来的影响。隐藏假设列表中的高概率项是要封装在模块中的主要候选项。创建这样的一个列表并不是什么难事，技术人员和非技术人员可以一起完成。

Read more about constructing a hiding assumption list in this primer on Parnas partitioning.

> 通过该引文（http://www.jodypaul.com/SWE/HAL/hal.html），可以了解更多关于构建隐藏假设列表的内容。

There’s a big difference between creating a modular library designed for reuse versus building a large enterprise application where understandability and maintainability are chief concerns. In general, we can distinguish several axes you can align with when designing modules:

> 创建用于重用的模块化库与创建以可理解性和可维护性为主要关注点的大型企业应用程序之间存在着很大的区别。通常，在设计模块时可以划分几个参考标准：

#### Comprehension 可理解性

Modules and their relations reflect the overall structure and intent of the system. Whenever someone looks at the codebase without prior knowledge, the high-level structure and functionality is immediately apparent. The module structure guides developers through the system when looking for specific functionality.

> 模块及其相互关系反映了系统的整体结构和意图。每当有人在没有先验知识的情况下查看代码库时，首先看到的是高级结构和功能。模块结构可以引导开发人员查找系统中特定的功能。

#### Changeability 可变性

Requirements change constantly. By using modules to encapsulate decisions that are likely to change, the impact of change decreases. Two systems with similar functionality but different anticipated areas of change may have different optimal module boundaries.

> 需求总是在不断地变化。通过使用模块来封装可能发生变化的决策，就会让变化所产生的影响降到最小。具有相似功能但具有不同预期变化范围的两个系统可具有不同的最佳模块边界。

#### Reuse 可重用性

Modules are an ideal unit of reuse. To increase reusability, modules should be narrowly focused and as independent as possible. Reusable modules can be composed in many ways across different applications.

> 模块是理想的重用单元。为了提高可重用性，模块应该尽可能地集中并尽可能独立。可重复使用的模块可以在不同的应用程序中以多种方式组合。

#### Teamwork 团队合作

Sometimes you want to use module boundaries to clearly divide work across multiple teams. Rather than using technical considerations, you align module boundaries with organizational boundaries.

> 有时，可以使用模块边界来明确划分多个团队的工作。可以将模块边界与组织边界对齐，而不需要考虑技术方面的问题。

Some tension exists between these axes. There’s no one-size-fits-all answer here. By designing for change in a particular area, abstractions can be introduced that decrease comprehension at first sight. Say, for example, a step from a larger process is put in a separate module because it’s expected to change more frequently than the surrounding parts. Logically, the step still belongs with the main process, but it’s now in a separate module. This makes the whole a bit less comprehensible in a way, while changeability increases.

> 这些标准之间存在一些冲突，这里无法给出一个适合所有人的答案。通过设计特定区域的变化，可以引入一些抽象概念，从而便于初次理解相关内容。比方说，将一个过程中的一个步骤放在一个单独的模块中，因为该步骤预计会比其他步骤更频繁地变化。从逻辑上讲，该步骤仍然属于主要过程，但它现在在一个单独的模块中。虽然从某种程度上讲这样做增加了整体的理解难度，但却增加了可变性。

Another important trade-off comes from the tension between reuse and use. Generic components or reusable libraries may become complex because they need to be adaptable to different usage scenarios. Nonreusable application components, not directly burdened by the needs of many consumers, can be more straightforward and specific.

> 另一个重要的权衡来自于重复使用和使用之间的冲突。通用组件或可重用库可能会变得非常复杂，因为它们需要适应不同的使用场景，而那些不需要重复使用的应用程序组件则可以更直接和更具体。

When designing for reuse, you have two main drivers:

> 当设计重用时，可以考虑以下两点：

- Adhering to the Unix philosophy of doing only one thing and doing it well.
- Minimizing the number of dependencies the module has itself. Otherwise, you’re burdening all reusing consumers with those transitive dependencies.

---

> - 坚持 UNIX 哲学理念，只做一件事情，并且将其做好。
> - 将模块本身的依赖关系数量减到最少。否则，所有重用的使用者都要承担可传递依赖关系所带来的负担。

These drivers do not necessarily work for application modules, where ease of use, comprehension, and speed of development are more important. If you can use a library module to speed up the development of your application module, and make the application module simpler in the process, you do so. On the other hand, if you want to create a reusable library module, your future consumers will thank you for not bringing in a multitude of transitive dependencies. There is no right or wrong here, only deliberate trade-offs. The module system makes these choices explicit.

> 以上两点并不一定适用于应用程序模块，对于应用程序模块来说，易用性、可理解性以及开发速度更为重要。如果使用一个库模块可加快应用程序模块的开发速度，并使应用程序模块更加简单，那么就可以这样做。另一方面，如果你创建了一个可重用的库模块，那么未来模块的使用者会感谢你没有引入过多的传递性依赖关系。这里没有绝对的对与错，只有慎重的权衡。模块系统非常明确地做出了选择。

Typically, reusable modules will be smaller and more focused than single-use application modules. On the other hand, orchestrating many small modules brings about its own complexity. When reuse is of no immediate concern, having larger modules can make sense.

> 可重用模块通常比一次性使用的应用程序模块更小、更集中。另一方面，协调使用多个小模块也带来了一定的复杂性。当重复使用不是主要的关注点时，创建更大的模块可能更合理。

In the next section, we’ll explore the notion of module size.

> 在下一节，将会探讨模块大小的概念。

## 5.2 Lean Modules 精益化模块

How big should a module be? Although it sounds like a natural question, it’s like asking how big your application should be. Just big enough to serve its purpose, but no bigger—which isn’t terribly useful advice.

> 模块应该有多大？这听起来像是一个很自然的问题，就好像是在问应用程序有多大一样。只要大到可以达到目标就可以了，但也不能太大——显然，这并不是一个非常有用的建议。

There are more important concerns to address when thinking about module design than just size. When thinking about the measure of a module, take into account two metrics: the size of its public surface area and of its internal implementation.

> 在考虑模块设计时除了大小之外，还有更重要的问题要解决。而在考虑模块的度量时，需要考虑两个指标：模块公共区域面积（surface area）的大小以及内部实现的大小。

Simplifying and minimizing the publicly exported part of your module is beneficial for two reasons. First, a simple and small API is easier to use than a large and convoluted one. Users of the module are not burdened with unnecessary details. The whole point of modularity is to break down concerns into manageable chunks.

> 出于两点理由，简化和最小化模块的公开导出部分是有益的。首先，一个简单而小巧的 API 比一个大而复杂的 API 更容易使用，模块用户不必考虑任何不必要的细节。模块化的主旨是将问题分解成可管理的块。

Second, minimizing the public part of a module reduces the liability of the module’s maintainer. You don’t have to support what others can’t access, leaving the module authors free to change internal details without grave consequences. The less is revealed, the less is relied upon by consumers, and the more stable an API can be. Whatever is put in the exported part of a module becomes a contract between the module producer and its consumers. This is not something to be taken lightly if you want to evolve the module in a backward-compatible manner (which should be the default position). Ergo, minimizing the public surface area of a module is recommended.

> 其次，最大限度地减少模块的公共部分可以减轻模块维护者的责任。维护者不需要考虑其他人无法访问的内容，模块作者可以自由地更改内部细节而不会产生严重的后果。公开的越少，消费者所依赖的也就越少，API 就越稳定。模块导出部分所放置的任何内容都将成为模块生产者和消费者之间的协议。如果要以向后兼容的方式（应该是默认方式）来发展模块，那么该问题是不容忽视的。建议尽量减少模块的公共区域面积。

That leaves the other metric: the measure of the nonexported part of a module. Here it makes less sense to talk about raw size as with the public part of a module. Again, a module’s private implementation should be as big as it needs to be to fulfill its API contract. More interesting is the question: how many other modules does it need to accomplish its goals? A lean module is as independent as possible, avoiding dependencies on other modules where possible. Nothing is more discouraging than to see a load of (transitive) dependencies being added to your system because you want to use a specific module. As discussed previously, when wide reuse is not a main concern for the module, this becomes less of an issue.

> 还需要考虑另一个指标：模块非导出部分的大小。与模块的公共部分一样，在此谈论原始大小是没有多大意义的。模块的私有实现应该与实现其 API 约定所需的内容一样大。可更有趣的问题是：需要多少其他模块来完成目标呢？精益化模块要尽可能独立，在可能的情况下避免依赖于其他模块。如果只是想使用特定的模块，那么没有什么比看到系统中添加了大量的（可传递的）依赖项更让人沮丧的了。如前所述，当广泛的重复使用不是模块的主要关注点时，这就不再是一个问题了。

You may have noticed parallels between developing lean modules and the prevailing best practices around reusable microservices. Strive to be as small as possible. Have a well-defined contract to the outside world while at the same time being as independent as possible. Indeed, these are similar concerns at different levels in your system architecture. Modules with their public APIs and module descriptors facilitate intraprocess reuse and composition. Microservices function at a higher level in the architecture, through (networked) interprocess communication. Modules and microservices are therefore complementary concepts. A microservice may very well be implemented using Java modules on the inside. One important difference is that with modules and their descriptors, we can explicitly describe not only what they offer (export), but also what they require. Modules in the Java module system can therefore be resolved and linked reliably, something that cannot be said of most microservices environments.

> 你可能注意到，开发精益化模块与围绕可重用微服务的最佳实践之间存在许多相似的地方：尽可能的小，与外界有定义良好的协议，同时尽可能保持独立。事实上，在系统架构中的不同层次上也存在类似问题。具有公共 API 和模块描述符的模块可以方便进程内重用和组合。通过（网络）进程间通信，微服务在架构的较高级别上起到了非常大的作用。因此，模块和微服务是互补的概念。微服务很可能在内部使用 Java 模块来实现。但两者之间的一个重要区别是，使用模块及其描述符不仅可以明确描述所提供（导出）的内容，还可以明确地描述所需要的内容，因此可以非常安全可靠地解析和链接 Java 模块系统中的模块，但在大多数微服务环境中却不是这样的。

## 5.3 API Modules API 模块

So far, you’ve seen that being deliberate about the API of a module is a must. API design becomes a first-class citizen when building proper modules. It may sound like this is a concern for only library authors, but that’s not at all the case. When you modularize an application, crafting the public API of application modules is every bit as important. The API of a module is by definition the sum of its exported packages. Applications consisting of modules exporting everything they contain are typically a warning sign, leaving you no better off than before you used modules. A modular application hides implementation details from other parts of the application just as a good library hides its internals from applications. Whenever your module is used in different parts of your application, or by different teams in your organization, having a well-defined and stable API is paramount.

> 到目前为止已经看到，仔细研究模块的 API 是很有必要的，API 设计已经成为构建合适模块的基本元素。这似乎听起来只有库的作者才会关注此问题，但事实并非如此。当对应用程序进行模块化时，构建应用程序模块的公共 API 是非常重要的。根据定义，模块的 API 指的是其导出包的总和。由模块组成的应用程序（这些模块导出了其所包含的所有内容）通常是一个警告信号，表明没有比使用模块更好的方法了。模块化应用程序隐藏了应用程序其他部分的实现细节，就像一个设计良好的库对应用程序隐藏了其内部实现细节。无论何时模块用在应用程序的哪个部分，或者由不同的团队使用，拥有一个定义良好且稳定的 API 是非常重要的。

### 5.3.1 What Should Be in an API Module? API 模块中应该包含什么

If you expect to have only a single implementation of an interface, you can combine the API and implementation in a single module. In that case, the exported part is visible to the consumers of the module, and your implementation packages are concealed. The implementation can be exposed as a service through the ServiceLoader. Even if you expect multiple implementations, in some cases it makes sense to bundle a default implementation into the API module. For the remainder of this discussion, we assume the API module does not contain such a default implementation, but stands on its own. In “API Module with a Default Implementation”, we discuss some caveats when a module contains both an exported API and an implementation of that API.

> 如果希望一个接口只有一个实现，那么可以将 API 和实现组合到一个模块中。在这种情况下，导出的部分对模块的使用者是可见的，而实现包是隐藏的。可以通过使用 ServiceLoader 将这个实现作为一个服务公开。即使希望有多个实现，在某些情况下将默认实现捆绑到 API 模块中也是很有意义的。在本节的后部分假设 API 模块不包含这样的默认实现，而是独立存在。在 5.3.3 节中，将讨论模块包含导出的 API 和该 API 实现时的一些注意事项。

We already established that the public part of a module should be as lean as possible. But what do you export from an API module? Interfaces have been mentioned a lot, as they form the backbone of most APIs. Of course, there’s more.

> 前面已经讲过，模块的公共部分应该尽可能地精益化。但是，应该从 API 模块中导出什么内容？前面已经多次提到过接口，因为它们构成了大多数 API 的骨架。当然，还有其他内容。

An interface contains methods with parameters and result types. In its most basic form, the interface stands on its own, using only types from java.base:

> 接口包含了带有参数和结果类型的方法。在最基本的形式中，接口是独立的，只使用 java.base 中的类型：

```java
public interface SimpleTextRepository {
  String findText(String id);
}
```

Having such a simple interface is rare in practice. In an application such as EasyText, you’d expect a repository implementation to return a domain-specific type from getText (see Example 5-1).

> 实际上，拥有一个如此简单的接口是非常少见的。在诸如 EasyText 之类的应用程序中，希望完成一个存储库实现，通过 get Text 返回一个特定域的类型（参见示例 5-1）。

Example 5-1. An interface with nonprimitive types

> 示例 5-1：带有非原始类型的接口

```java
public interface TextRepository {
  Text findText(String id);
}
```

In this case, the Text class is placed in the API module as well. It could be a typical JavaBean-style class, describing the data the caller can expect to get out of a service. As such, it’s part of the public API. Exceptions declared in methods on an interface are part of the API, too. They should be colocated in the API module and exported alongside the methods that (declare to) throw them.

> 此时，Text 类被放置在 API 模块中。它可能是一个典型的 JavaBean 风格的类，描述了调用者期望从服务中获得的数据。因此，它是公共 API 的一部分。在接口的方法中声明的异常也是 API 的一部分。它们应该放在 API 模块中，并与（声明）抛出它们的方法一起导出。

Interfaces are the primary means of achieving decoupling between the API provider and consumer. Of course, API modules can contain much more than interfaces—for example, (abstract) base classes you expect the API consumer to extend, enums, annotations, and so on. Whatever you put into an API module, remember: minimalism goes a long way.

> 接口是实现 API 提供者和消费者之间解耦的主要手段。当然，API 模块可以包含比接口更多的内容。例如，希望 API 消费者扩展的（抽象）基类、枚举、注释等。无论在 API 模块中放入什么内容，都要记住：简约大有裨益。

When another module requires the API module, you want it to be usable as is. You don’t want to require additional modules just to get readability of all the types used in the interface. Making the API module fully self-contained, as discussed so far, is one way to do this. Still, that’s not always feasible. Often, an interface method returns or accepts a parameter of a type that resides in another module. To streamline this scenario, the module system offers implied readability.

> 当另一个模块需要 API 模块时，你希望该模块是可用的，但并不希望为了读取接口中所使用的所有类型而使用额外的模块。如上所述，使 API 模块完全自包含是实现这一目标的一种方法。不过，该方法并不总是可行的。通常，接口方法返回或接收另一个模块中的类型参数。为了简化这种情况，模块系统提供了隐式可读性。

### 5.3.2 Implied Readability 隐式可读性

“Implied Readability” provided an introduction to implied readability based on platform modules. Let’s look at an example from the EasyText domain to see how implied readability helps create self-contained and fully self-describing API modules. You’re going to look at an example consisting of three modules, shown in Figure 5-1.

> 2.5 节介绍了基于平台模块的隐式可读性。接下来看一个来自 EasyText 域的示例，了解一下隐式可读性如何帮助创建自包含和完全自描述的 API 模块。图 5-1 所示的示例由三个模块组成。

Three modules, without implied readability yet.
Figure 5-1. Three modules, without implied readability yet

In this example, the TextRepository interface in Example 5-1 lives in the module easytext.repository.api, and the Text class that its findText method returns lives in another module, easytext.domain.api. The module-info.java of easytext.client (which calls the TextRepository) starts out like Example 5-2.

> 在本示例中，TextRepository 接口位于模块 easytext.repository.api 模块中，而 findText 方法返回的 Text 类位于另一个模块 easytext.domain.api 中。easytext.client（调用 TextRepository）的 module-info.java 如示例 5-2 所示。

Example 5-2. Module descriptor of a module using TextRepository (➥ chapter5/implied_readability)

> 示例 5-2：使用了 TextRepository 的模块的模块描述符（chapter5/implied_readability）

```java
module easytext.client {
  requires easytext.repository.api; 1

  uses easytext.repository.api.TextRepository; 2
}
```

1. Requires the API module because we need to access TextRepository in it
2. Indicates this client wants to use a service implementing TextRepository

---

> 1. 需要 API 模块，因为需要访问该模块中的 TextRepository。
> 2. 表明客户端想要使用一个实现了 TextRepository 的服务。

The easytext.repository.api in turn depends on the easytext.domain.api, since it uses Text as the return type in the TextRepository interface:

> easytext.repository.api 又依赖于 easytext.domain.api，因为它使用了 Text 作为 TextRepository 接口中的返回类型：

```java
module easytext.repository.api {
  exports easytext.repository.api; 1
  requires easytext.domain.api; 2
}
```

1. Exposes the API package containing TextRepository
2. Requires the domain API module because it contains Text, which is referenced in the TextRepository interface

---

> 1. 公开包含了 TextRepository 的 API 包。
> 2. 需要域 API 模块，因为它包含了 TextRepository 接口所引用的 Text。

Last, the easytext.domain.api module contains the Text class:

> 最后，easytext.domain.api 模块包含了 Text 类：

```java
public class Text {

   private String theText;

   public String getTheText() {
      return this.theText;
   }

   public void setTheText(String theText) {
      this.theText = theText;
   }

   public int wordcount() {
      return 42; // Why not
   }

}
```

Note that Text has a wordcount method, which we’ll be using later in the client code. The easytext.domain.api module exports the package containing this Text class:

> 请注意，Text 拥有一个 wordcount 方法，在稍后的客户端代码中将会使用该方法。Easytext.domain.api 模块导出了包含 Text 类的包：

```java
module easytext.domain.api {
   exports easytext.domain.api;
}
```

The client module contains the following invocation of the repository:

> 客户端模块包含了以下对该存储库的调用：

```java
TextRepository repository = ServiceLoader.load(TextRepository.class)
   .iterator().next();

repository.findText("HHGTTG").wordcount();
```

If we compile this, the following error is produced by the compiler:

> 如果进行编译，编译器会产生如下所示错误：

```log
./src/easytext.client/easytext/client/Client.java:13: error: wordcount() in
Text is defined in an inaccessible class or interface
      repository.findText("HHGTTG").wordcount();
                                 ^
```

Even though we’re not mentioning the Text type directly in easytext.client, we’re trying to call a method on this type as it is returned from the repository. Therefore, the client module needs to read the easytext.domain.api module, which exports Text. One way to solve this compilation error is to add a requires easytext.domain.api clause to the client’s module descriptor. That’s not a great solution, though; why should the client module have to handle the transitive dependencies of the repository module? A better solution is to improve the repository’s module descriptor:

> 尽管没有直接在 easytext.client 中提到 Text 类型，但却试图调用这个类型的方法，因为它是从存储库返回的。因此，客户端模块需要读取导出了 Text 的 easytext.domain.api 模块。解决这个编译错误的一种方法是在客户端的模块描述符中添加一个 requires easytext.domain.api 子句，但这并不是一个好的解决方案。为什么客户端模块必须处理库模块的传递依赖关系呢？更好的解决方案是改进存储库的模块描述符：

```java
module easytext.repository.api {
  exports easytext.repository.api;
  requires transitive easytext.domain.api; 1
}
```

1. Sets up implied readability by adding the transitive keyword

> 1. 通过添加 transitive 关键字设置隐式可读性。

Note the additional transitive keyword in the exports clause. It effectively says that the repository module reads easytext.domain.api, and every module requiring easytext.repository.api also automatically reads easytext.domain.api, as illustrated in Figure 5-2.

> 请注意 exports 子句中额外的关键字 transitive。实际上是在说，存储库模块读取 easytext.domain.api，并且每个需要 easytext.repository.api 的模块也会自动读取 easytext.domain.api，如图 5-2 所示。

Implied readability edge
Figure 5-2. Implied readability relation set up by requires transitive, shown as a bold edge

Now, the example compiles without problems. The client module can read the Text class through the requires transitive clause in the repository’s module descriptor. Implied readability allows the repository module to express that its own exported packages are not enough to use the module.

> 现在，示例可以顺利编译。通过存储库的模块描述符中的 requires transitive 子句，客户端模块可以读取 Text 类。隐式可读性允许存储库模块表达其自己导出的包不足以使用该模块。

You’ve seen an example where the return type comes from a different module, necessitating the use of requires transitive. Whenever a public, exported type refers to types from another module, use implied readability. Besides return types, this applies to argument types and thrown exceptions from different modules as well.

> 上面所示的是一个返回类型来自不同模块的例子（需要使用 requires transitive）。只要一个公共导出类型引用了来自另一个模块的类型，就会使用隐式可读性。除了返回类型之外，也适用于参数类型以及从不同模块抛出的异常。

A compiler flag helps you spot dependencies in an API module that should be marked as transitive. If you compile with -Xlint:exports, any types that are part of exported types which are not transitively required (but should be) result in a warning. This way, you can find the problem while compiling the API module itself. Otherwise, the error surfaces only when compiling a consuming module that is missing said dependency because no implied readability is set up. For example, when we leave out the transitive modifier in the previous easytext.repository.api module descriptor and compile it with -Xlint:exports, the following warning is produced:

> 一个编译器标志有助于发现 API 模块中应该被标记为 transitive 的依赖项。如果使用-Xlint:exports 进行编译，那么导出类型中应该以传递方式依赖但却没有依赖的类型就会产生警告信息。这样一来，就可以在编译 API 模块时发现问题。否则，就只有在编译缺少所需依赖项的消费模块时才会发现错误，因为没有建立隐式可读性。例如，如果在前面的 easytext.repository.api 模块描述符中省略修饰符 transitive 并使用-Xlint:exports 进行编译，就会产生以下警告信息：

```sh
$ javac -Xlint:exports --module-source-path src -d out -m easytext.repository.api
src/easytext.repository.api/easytext/repository/api/TextRepository.java:6:
warning: [exports] class Text in module easytext.domain.api is not indirectly
 exported using requires transitive
  Text findText(String id);
  ^
1 warning
```

When designing an API module, implied readability is a powerful technique to make modules easier to consume. There is a subtle danger to relying on implied readability in consuming modules. In our example, the client module can access the Text class by virtue of the transitive nature of implied readability. This works fine when the type is used through the interface, as in the example repository.findText("HHGTTG").wordcount(). However, what if inside the client module we start using the Text class directly, without getting it as a return value through the interface’s method—say, by instantiating Text directly and storing it in a field? That continues to compile and run correctly. But does the client’s module descriptor now truly reflect our intentions? You can argue that the client module must take an explicit, direct dependency on the easytext.domain.api module in this case. That way, should the client module stop using the repository module (and thus lose the implied readability on the domain API), the client code continues to compile.

> 在设计 API 模块时，隐式可读性是使模块更易于使用的强大技术。但依靠消费模块中的隐式可读性存在一个微妙的风险。在示例中，客户端模块可以通过隐式可读性的传递性来访问 Text 类。当通过接口使用类型时，这种方式也可以正常工作，如示例中 repository.findText("HHGTTG").wordcount()所示。但是，如果在客户端模块中直接使用 Text 类，而不是通过接口的方法将其作为返回值，如直接实例化 Text 并将其存储在字段中，那么会发生什么事情呢？此时，可以正常地编译和运行。但客户端的模块描述符是否真正反映了我们的意图？你可以争辩说，在这种情况下，客户端模块必须显式地依赖 easytext.domain.api 模块。这样一来，即使客户端模块停止使用存储库模块（从而失去域 API 的隐式可读性），客户端代码也会继续编译。

This may seem like an inconsequential issue. It compiles and works, so what’s the big deal? However, as a module author, you are responsible for declaring the correct dependencies based on your code. Implied readability is meant only to prevent surprises such as the compiler error you saw previously. It is not a carte blanche to be lazy about real dependencies your module has.

> 这似乎是一个无关紧要的问题。代码仍然可以编译和工作，那么有什么大不了的呢？然而，作为模块的作者，你有责任根据代码声明正确的依赖关系。隐式可读性仅用于防止以前所看到的编译器错误等意外情况，将模块所拥有的真正依赖关系全权委托给隐式可读性是一种“懒惰的做法”。

### 5.3.3 API Module with a Default Implementation 带有默认实现的 API 模块

Whether the implementation of the API should live in the same module or in a separate module is an interesting question. Separating the API and implementation is a useful pattern when you expect there to be multiple implementations of the API. On the other hand, when only a single implementation exists, bundling the API and implementation in the same module makes the most sense. It also makes sense when you want to offer a default implementation as a convenience.

> API 的实现应该放在同一个模块还是单独的模块中？这是一个有趣的问题。当希望有多个 API 的实现时，分离 API 和实现是一个有用的模式。而当只有一个实现时，将 API 和实现捆绑在同一个模块中则更有意义的。此外，提供默认实现作为一种便利手段也是有意义的。

A combined API/implementation module doesn’t preclude alternative implementations in separate modules, at a later point in time. However, those alternative implementation modules then need a dependency on the combined module for the API. This combined module also contains an implementation. That may lead to superfluous transitive dependencies, because of the combined module’s implementation dependencies that are of no use to the alternative implementation but still need to be resolved.

> 组合的 API /实现模块并不排除日后在单独模块中的替代实现。但是，这些替代实现模块则需要依赖于 API 的组合模块。同时，这个组合模块也包含一个实现。这样一来可能会产生多余的传递依赖关系，因为组合模块的实现依赖关系对替代实现没有任何用处，但仍需要被解析。

Figure 5-3 illustrates this problem. Modules easytext.analysis.coleman and easytext.analysis provide an implementation of the Analyzer interface as service. The latter module exports the API in addition to providing an implementation. However, the implementation inside easytext.analysis (but not the API) requires the syllablecounter module. Now, the alternative implementation of the API in easytext.analysis.coleman cannot run without syllablecounter being present on the module path, even though the alternative implementation doesn’t need it. Separating the API into its own module avoids such problems, as shown in Figure 5-4.

> 图 5-3 说明了这个问题。模块 easytext.analysis.coleman 和 easytext. analysis 提供了一个 Analyzer 接口实现作为服务，后者除了提供一个实现之外，还导出 API。但是，easytext.analysis（而不是 API）中的实现需要 syllablecounter 模块。此时如果 syllablecounter 不在模块路径上，即使 easytext.analysis.coleman 中的 API 替换实现不需要 syllablecounter 模块也无法运行。将 API 分离到自己的模块可以避免此类问题的出现，如图 5-4 所示。

When the API is in a module with an implementation (like in the `easytext.analysis` module here), the dependencies of the implementation (in this case `syllablecounter`) transitively burden alternative implementations well.
Figure 5-3. When the API is in a module with an implementation (as in the easytext.analysis module here), the dependencies of the implementation (in this case syllablecounter) transitively burden alternative implementations as well

Separate API module
Figure 5-4. A separate API module works better when there are multiple implementations

Towards the end of Chapter 3 and in Chapter 4, we introduced a separate API module for the EasyText Analyzer interface. Whenever you have (or expect to have) multiple implementations of the API, it makes sense to extract the public API into its own module. This is certainly the case here: the whole idea was to make EasyText extensible with new analysis functionality. When the implementations are provided as a service, this leads to the pattern in Figure 5-4 when extracting an API module.

> 在第 3 章和第 4 章的末尾，为 EasyText Analyzer 接口引入了一个单独的 API 模块。如果有（或期望有）多个 API 的实现，那么将公共 API 提取到自己的模块中是很有意义的。此时就是这种情况：主要想法是通过新的分析功能使 EasyText 更具有可扩展性。当实现作为服务提供时，在提取 API 模块时就会产生如图 5-4 所示的模式。

In this case, you end up with multiple implementation modules that do not export anything, which depend on the API module and publish their implementation as a service. The API module, in turn, only exports packages and does not contain any encapsulated implementation code. With this setup, the implementation dependency of the easytext.analysis.kincaid module on syllablecounter is not imposed on the alternative easytext.analysis.coleman module. You can use easytext.analysis.coleman without having easytext.analysis.kincaid or syllablecounter on your module path.

> 最终会有多个不导出任何内容的实现模块，这些模块依赖于 API 模块并将其实现作为服务发布。反之，API 模块仅导出包，不包含任何封装的实现代码。通过这样的设置，easytext.analysis.kincaid 模块对 syllablecounter 的实现依赖关系就不会强加于 easytext.analysis.coleman 模块。而使用 easytext.analysis.coleman 时，则无须在模块路径上放置 easytext.analysis.kincaid 或 syllablecounter。

## 5.4 Aggregator Modules 聚合器模块

With implied readability fresh in your mind, you’re ready for a new module pattern: the aggregator module. Imagine you have a library consisting of several loosely related modules. A user of your imaginary library can use either one or several modules, depending on their needs. So far, nothing new.

> 在了解隐式可读性之后，接下来学习一个新的模块模式：聚合器模块。假设有一个由几个关联度不高的模块组成的库，根据不同的需求，该虚拟库的用户可以使用一个或多个模块。到目前为止，还没有什么新的内容。

### 5.4.1 Building a Facade over Modules 在模块上构建一个外观

Sometimes you don’t want to burden the user of the library with the question of which exact modules to use. Maybe the modules of the library are split up in a way that benefits the library maintainer but might confuse the user. Or maybe you just want to have a way for people to quickly get started by having to depend on only a single module representing the whole library. One way to do this would be to build both the individual modules and a “super-module” combining all content from the individual modules. That works but is not a particularly nice solution.

> 有时候，不希望让库的使用者面临具体使用哪个模块的问题。也许对库模块进行分割有益于库的维护者，但却可能让用户感到困惑。或者你希望有一种方法，可以让人们只需依赖一个代表整个库的单个模块就可以快速开发。一种方法是首先构建各个模块，然后再构建一个“超级模块”，将各个模块的所有内容组合在一起。该方法虽然有效，但不是一个特别好的解决方案。

Another way to achieve a similar result is to use implied readability to construct an aggregator module. Essentially, you are building a facade over the existing library modules. An aggregator module contains no code; it has only a module descriptor setting up implied readability to all other modules:

> 实现类似结果的另一种方法是使用隐式可读性来构建聚合器模块，其本质上是通过现有的库模块构建一个外观（facade）。聚合器模块不包含代码，它只有一个模块描述符，为所有其他模块设置了隐式可读性：

```java
module library {
  requires transitive library.one;
  requires transitive library.two;
  requires transitive library.three;
}
```

Now, if the library user adds a dependency on library, all three library modules are transitively resolved, and their exported types are readable for the application. Figure 5-5 shows the new situation. Of course, it’s still possible to depend on a single specific module if so desired.

> 现在，如果库用户添加了对库的依赖关系，那么所有三个库模块都将以传递的方式解析，并且它们的导出类型对应用程序是可读的。图 5-5 显示了新的情况。当然，如果需要的话，依然可以依赖于一个特定的模块。

Implied readability edge
Figure 5-5. The application module can use all exported types from the three library modules through the aggregator module library. Implied readability edges are shown with bold edges.

Because aggregator modules are lightweight, it’s perfectly possible to create several different aggregator modules. For example, you could provide several profiles or distributions of a library, targeted towards specific users.

> 因为聚合器模块是轻量级的，因此完全可以创建多个不同的聚合器模块。例如，可以针对特定用户，提供库的多个配置文件或者分布。

The JDK contains a good example of this approach, as we saw before in Chapter 2. It has several aggregator modules, listed in Table 5-1.

> 就像第 2 章所看到的那样，JDK 就是该方法的一个很好的示例。它包含了多个聚合器模块，如表 5-1 所示。

Table 5-1. Aggregator modules in the JDK

> 表 5-1：JDK 中的聚合器模块

| Module     | Aggregates                                                                      |
| ---------- | ------------------------------------------------------------------------------- |
| java.se    | All modules officially belonging to the Java SE specification                   |
| java.se.ee | The java.se modules, plus all Java EE modules bundled with the Java SE platform |

On the one hand, creating an aggregator module is convenient for consumers. Consumer modules can just require the aggregator module, without thinking too hard about the underlying structure. On the other hand, it poses a risk as well. The consuming module can now transitively access all exported types of the aggregated modules through the aggregator module. That may include types from modules you don’t want to depend on as a consumer. Because of implied readability, you will not be warned by the module system anymore when you do use those types. Specifying precise dependencies on the exact underlying modules you need protects you from these pitfalls.

> 一方面，对于消费者来说，创建一个聚合器模块非常方便。消费者模块只需要聚合器模块即可，而不必过多考虑底层结构。另一方面，使用聚合器模块也存在风险。现在，消费者模块可以通过聚合器模块以可传递的方式访问聚合器模块中的所有导出类型，其中有些类型可能包括来自不希望依赖的模块。由于隐式可读性，当使用这些类型时，将不会收到来自模块系统的警告。准确地指定对底层模块的依赖可以避开这些缺陷。

Having aggregator modules available is a convenience. From the point of view of the JDK developers, it’s more than just a convenience. Aggregator modules are a great way to compose the platform into manageable chunks without having to repeat yourself. Let’s take a look at one of the platform aggregator modules, java.se.ee, to see what this means. As you’ve seen before, you can use java `--describe-module <modulename>` to view the module descriptor content of a module:

> 使用聚合器模块是一种便利。而从 JDK 开发人员的角度来看，这又不仅仅是一种便利。聚合器模块是一种可以将平台组合成可管理且不重复的块的好方法。接下来，看看 JDK 中一个平台聚合器模块 java.se.ee。正如前所见，可以使用`java --describe-module <modulename>`来查看模块的模块描述符内容：

```sh
$ java --describe-modules java.se.ee
java.se.ee@9
requires java.se transitive
requires java.xml.bind transitive
requires java.corba transitive
...
```

The aggregator module java.se.ee transitively requires relevant EE modules. It also transitively requires another aggregator module, java.se. Implied readability works transitively, so when a module reads java.se.ee, it also reads everything that is required transitively by java.se, and so on. This pattern of hierarchical aggregation is a clean way to organize a large number of modules.

> 聚合器模块 java.se.ee 以传递的方式请求相关的 EE 模块。此外，还需要另一个聚合器模块 java.se。隐式可读性是可以传递的，所以当一个模块读取 java.se.ee 时，它也会读取 java.se 所需的所有内容，以此类推。这种分层聚合模式是组织大量模块的一种好方式。

### 5.4.2 Safely Splitting a Module 安全拆分模块

There is another useful application of the aggregator module pattern. It’s when you have a single, monolithic module that you need to split up after it has been released. It may have grown too large to maintain, for example, or you might want to separate out unrelated functionality for improved reusability.

> 聚合器模块模式还有另一个有用的应用，即当一个整体模块发布后需要对其进行拆分时可以使用该模式。比如，模块变得太大而无法维护，或者需要分离出不相关的功能以提高可重用性。

Suppose that a module largelibrary (shown in Figure 5-6) is in need of further modularization. However, there are already users of largelibrary in the wild, using its public API. The split needs to happen in a backward-compatible manner. That is, we can’t rely on existing users of largelibrary to switch to the new, smaller modules right away.

> 假设需要对模块 largelibrary（如图 5-6 所示）进行进一步的模块化，但是 largelibrary 的用户正在使用其公共 API，拆分需要以向后兼容的方式进行。也就是说，不能指望 largelibrary 的现有用户立即切换使用新的、更小的模块。

The solution, shown in Figure 5-7, is to replace largelibrary with an aggregator module of the same name. In turn, this aggregator module arranges implied readability to the new, smaller modules.

> 如图 5-7 所示的解决方案使用了一个同名的聚合器模块替换了 largelibrary。该聚合器模块为新的、更小的模块设置了隐式可读性。

Module `largelibrary` before the split.
Figure 5-6. Module largelibrary before the split

Module `largelibrary` after the split.
Figure 5-7. Module largelibrary after the split

The existing packages, both exported and encapsulated, are distributed over the newly introduced modules. New users of the library now have a choice of using either one of the individual modules, or largelibrary for readability on the whole API. Existing users of the library do not have to change their code or module descriptors when upgrading to this new version of largelibrary.

> 现有的包（包括导出的和封装的包）分布在新引入的模块中。现在，该库的新用户既可以选择使用其中一个单独的模块，也可以选择使用 largelibrary 来提高整个 API 的可读性。升级到 largelibrary 的这个新版本时，该库的现有用户不必更改其代码或模块描述符。

It’s not necessary to always create a pure aggregator module, containing only a module descriptor and no code of its own. Often a library consists of core functionality that is independently useful. Going with the largelibrary example, package largelibrary.part2 might build on top of largelibrary.part1.

> 没有必要总是创建一个纯粹的聚合器模块，即只包含一个模块描述符而没有自己的代码。通常一个库由独立有用的核心功能所组成。以库 largelibrary 为例，包 largelibrary.part2 可能会建立在 largelibrary.part1 之上。

In that case, it makes sense to create two modules, as shown in Figure 5-8.

> 此时，创建两个模块是有意义的，如图 5-8 所示。

Alternative approach to splitting `largelibrary`.
Figure 5-8. Alternative approach to splitting largelibrary

Consumers of largelibrary can keep using it as is, or require only the largelibrary.core module to use a subset of the functionality. This approach is implemented with the following module descriptor for largelibrary:

> largelibrary 的消费者可以继续使用它，或者只需 largelibrary.core 模块来使用功能的一个子集。这个方法是使用 largelibrary 的模块描述符实现的：

```java
module largelibrary {
  exports largelibrary.part2;

  requires transitive largelibrary.core;
}
```

As you have seen, implied readability offers a safe way to split modules. Consumers of the new aggregator modules won’t notice any difference to the situation where all code was in a single module.

> 如你所见，隐式可读性提供了一种拆分模块的安全方法。新聚合器模块的消费者不会注意到其与所有代码在单个模块中之间有什么不同。

## 5.5 Avoiding Cyclic Dependencies 避免循环依赖

In practice, safely splitting an existing module will be hard. The previous example assumed that the original package boundaries allow for cleanly splitting the module. What if different classes from the same package need to end up in different modules? You can’t export the same package from two different modules for your users. Or, what if types across two packages have a mutual dependency? Putting each package into a separate module won’t work in that case, because module dependencies cannot be circular.

> 实际上，安全地拆分现有的模块是非常困难的。前面的示例假设原来的包边界允许对模块进行彻底拆分。如果要将同一个包中的不同类放置到不同的模块，那么应该怎么做呢？不能从两个不同的模块导出相同的包。或者，如果两个包中的类型存在相互依赖关系，又该怎么做呢？在这种情况下，将每个包放入单独的模块将不起任何作用，因为模块依赖关系是不能循环的。

We’ll look at the problem of split packages first. Then we’ll investigate how to refactor circular dependencies between modules.

接下来，首先看一下拆分包所存在的问题，然后探讨一下如何重构模块之间的循环依赖。

### 5.5.1 Split Packages 拆分包

One scenario you can run into when splitting modules is the introduction of split packages. A split package is a single package that spans multiple modules, as shown in Figure 5-9. It occurs when partitioning a module doesn’t nicely align with existing package boundaries.

> 在拆分模块时，需要考虑的一种情况是拆分包的引入。拆分包（split package）是跨多个模块的单个包，如图 5-9 所示。当模块划分与现有的包边界不能保持一致时就会产生拆分包。

Two modules containing the same packages, but different classes.
Figure 5-9. Two modules containing the same packages but different classes

In this example, module.one and module.two both contain classes from the same packages called splitpackage and splitpackage.internal.

> 在本示例中，module.one 和 module.two 都包含了来自相同包（splitpackage 和 splitpackage.internal）的类。

TIP

Remember, packages in Java are nonhierarchical. Despite their appearance, splitpackage and splitpackage.internal are two unrelated packages that happen to share the same prefix.

> 请记住，Java 中的包是非层次结构的。不管外表如何，splitpackage 和 splitpackage.internal 是两个不相关却共享相同前缀的包。

Putting both module.one and module.two on the module path results in an error when starting the JVM. The Java module system doesn’t allow split packages. Only one module may export a given package to another module. Because exports are declared in terms of package names, having two modules export the same package leads to inconsistencies. If this were allowed, a class with the exact same fully qualified name might be exported from both modules. When another module depends on both these modules and wants to use this class, conflicts arise as to which module the class should come from.

> 如果将 module.one 和 module.two 放在模块路径上，那么在启动 JVM 时会产生错误。Java 模块系统不允许拆分包。只允许一个模块可以将给定的包导出到另一个模块。由于导出是根据包名称声明的，因此如果两个模块导出相同的包，那么就会产生不一致。如果允许这样做，那么两个模块就可能会导出具有相同的完全限定名称的类。而当另外一个模块依赖于这两个模块并且想要使用这个类时，就会出现该类应该来自哪个模块的冲突。

Even if the split package is not exported from the modules, the module system won’t allow it. Theoretically, having a nonexported split package (such as splitpackage.internal in Figure 5-9) isn’t a problem. It’s all encapsulated, after all. In practice, the way the module system loads modules from the module path prohibits this arrangement. All modules from the module path are loaded within the same classloader. A classloader can have only a single definition of a package, and whether it’s exported or encapsulated doesn’t matter. In “Container Application Patterns”, you’ll see how more advanced uses of the module system allow for multiple modules with the same encapsulated packages.

> 即使拆分包不是从模块中导出的，模块系统也是不允许的。从理论上讲，有一个非导出的拆分包（如图 5-9 所示的 splitpackage.internal）并不是一个问题，毕竟拆分包进行了很好的封装。而实际上，模块系统从模块路径加载模块的方式禁止使用拆分包。模块路径中的所有模块都是在相同的类加载器中加载的。一个类加载器只能有一个包定义，是导出的还是封装的无关紧要。在 6.3 节中，你将看到模块系统的更多高级应用如何允许具有相同封装包的多个模块。

The obvious way to avoid split packages is to not create them in the first place. That works when you create modules from scratch, but is harder when you’re transitioning existing JARs to modules.

> 首先，避免使用拆分包的方法就是不要创建它们。当从头开始创建模块时，这种方法是很有效的，但如果是将现有的 JAR 转换为模块，那就比较困难了。

The example in Figure 5-9 illustrates a cleanly split package, meaning no types with the same fully qualified name appear in the different modules. When converting existing JARs to modules, encountering unclean splits (where multiple JARs have the same types) is not uncommon. Of course, on the classpath these JARs might have worked together (but only accidentally). Not in the module system, though. In that case, merging the JARs and their overlapping packages into a single module is the solution.

> 图 5-9 所示的示例演示了一个“干净”的拆分包，这意味着在不同模块中将不会出现具有相同完全限定名的类型。在将现有的 JAR 转换为模块时，遇到“不干净”拆分（多个 JAR 具有相同的类型）的情况比较少见。当然，在类路径上这些 JAR 可能会一起工作（但只是偶然情况），但在模块系统中是不可能的。此时，解决方案是将 JAR 及其重叠的包合并成一个模块。

Keep in mind that the module system checks for package overlap with all modules. That includes platform modules. Several JARs in the wild try to add classes to packages owned by modules in the JDK. Modularizing these JARs and putting them on the module path will not work, because they overlap with these platform modules.

> 请记住，模块系统会检查在所有模块中是否存在重叠包，其中包括平台模块。许多 JAR 尝试向 JDK 中模块所拥有的包添加类。对这些 JAR 进行模块化并将其放在模块路径上是不起任何作用的，因为它们与这些平台模块重叠。

WARNING

When JARs containing packages that overlap with JDK packages are placed on the classpath, their types will be ignored and will not be loaded.

> 当 JAR 所包含的包与 JDK 包重叠时，如果将这些 JAR 放到类路径上，那么它们的类型将被忽略，并且不会被加载。

### 5.5.2 Breaking Cycles 打破循环

Now that we’ve addressed the problem of split packages, we are still left with the issue of cyclic dependencies between packages. When splitting up a module with mutually dependent packages, this would give rise to cyclic module dependencies. You can create these modules, but they won’t compile.

> 现在已经解决了拆分包的问题，但是仍然存在包之间循环依赖的问题。当所拆分的模块包含相互依赖的包时，就会产生循环的模块依赖关系。虽然可以创建这些模块，但是它们不会被编译。

In “Module Resolution and the Module Path”, you learned that readability relations between modules must be acyclic at compile-time. Two modules cannot require each other from their module descriptors. Then, in “Creating a GUI Module”, you learned that cyclic readability relations can arise at run-time. Also note that services can use each other, forming a cyclic call-graph at run-time.

> 在 2.7 节中已经讲过，模块之间的可读性关系在编译时必须是非循环的。两个模块不能通过各自的模块描述符互相需求。随后，在 3.3.2 节中谈到了循环可读性关系可能会在运行时出现。另外请注意，服务可以互相使用，从而在运行时形成一个循环调用图。

So, why this strict enforcement against cycles at compile-time? The JVM can load classes lazily at run-time, allowing for a multistage resolution strategy in the case of cycles. However, the compiler can compile a type only if all the other types it uses are either already compiled, or being compiled in the same compilation run.

> 那么，为什么在编译时严格禁止循环呢？JVM 可以在运行时延迟加载类，在出现循环的情况下允许多级解析策略。但是，只有在编译器所使用的所有其他类型已经被编译或在同一编译运行中正在被编译时，编译器才能编译该类型。

The easiest way to achieve this would be to always compile mutually dependent modules together. Although not impossible, this leads to hard-to-manage builds and codebases. Indeed, that is a subjective statement. Disallowing cyclic module dependencies at compile-time is an opinionated choice made by the Java module system based on the premise that cyclic dependencies are generally bad news for modularity.

> 满足上述要求的最简单方法是始终将相互依赖的模块编译在一起。虽然这种做法并不是不可能，但会导致难以管理的版本和代码库。当然，这只是一个主观的说法。在编译时禁止循环模块依赖关系是 Java 模块系统所做的一个“固执”的选择，因为它认为循环依赖关系不利于进行模块化。

It’s not controversial to say code containing cycles is harder to understand—especially because cycles can hide behind many levels of indirection. It is not always simply two classes in two different packages in two JARs that depend on each other. Cyclic dependencies significantly muddy the waters, and have no place in applications modularized with the Java module system.

> 如果说包含循环的代码很难理解，这是没有任何争议的。尤其是因为循环可能隐藏在许多间接层次的后面，而不仅仅是简单的两个相互依赖的 JAR 中两个不同包的两个类。循环依赖使情况更加混乱，在由 Java 模块系统模块化的应用程序中毫无作用。

What to do when you need to modularize an application that does have cyclic dependencies between existing JARs? Or, when splitting packages from a JAR would result in such cycles? You cannot just convert them to two modules that require each other, because the compiler disallows such a configuration.

> 当需要对现有 JAR 之间具有循环依赖关系的应用程序进行模块化时，该怎么办？或者说，如果从 JAR 中拆分包时会产生循环依赖，那么应该怎么办？此时不能将它们转换成两个彼此需要的模块，因为编译器不允许这样的配置。

One obvious solution is to merge these JARs into a single module. When there is such a tight (cyclic) relation between two components, it’s not much of a stretch to conclude they’re effectively one module. Of course, this solution assumes the cyclic relationship is benign to start with. It also breaks down when the cycle is indirect and involves several components, not just two, unless you want to merge all components participating in the cycle, which is unlikely.

> 一个显而易见的解决方案是将这些 JAR 合并成一个模块。当两个组件之间存在如此紧密（循环）的关系时，就可以断定它们是一个模块的。当然，该解决方案的前提是循环关系是良性的。当循环是间接的并且涉及多个组件（而不仅仅是两个组件）时，该方案就行不通了，除非想要合并参与循环的所有组件，但这不太可能。

Often, a cycle indicates questionable design. That means breaking the cycle involves a bit of redesign. As the saying goes, all problems in computer science can be solved by introducing another level of indirection (except the problem of too many indirections, of course). Let’s see how adding an indirection can help break cycles.

> 通常，存在循环说明设计是有问题的，这意味着需要进行一些重新设计来打破这个循环。常言道，计算机科学的一切问题都可以通过引入另一个间接层来解决（当然太多的间接层也会出现问题）。接下来看看如何使用一个间接的方法来打破循环。

We start out with two JAR files: authors.jar and books.jar. Each JAR contains a single class (respectively Author and Book), referencing each other. By naively turning the existing JARs into modular JARs, the cyclic dependency becomes apparent, as shown in Figure 5-10.

> 首先，从两个 JAR 文件开始：authors.jar 和 books.jar。每个 JAR 包含一个类（分别是 Author 和 Book），并且相互引用。如果只是将现有的 JAR 转换为模块化的 JAR，那么循环依赖就变得更加明显，如图 5-10 所示。

These modules won't compile or resolve because of their cyclic dependency.
Figure 5-10. These modules won’t compile or resolve because of their cyclic dependency

The first question we should answer is: what is the correct relation between those modules? There’s no one-size-fits-all approach to be taken here. We need to zoom in on the code, and see what it’s trying to accomplish. Only then can the question of what’s appropriate be answered. We’ll explore this question based on Example 5-3.

> 应该回答的第一个问题是：这些模块之间的正确关系是什么？这里没有一个通用的方法。需要仔细观察一下代码，看看它试图完成什么。只有这样才能回答上述问题。接下来将根据示例 5-3 来探讨这个问题。

Example 5-3. Author.java (➥ chapter5/cyclic_dependencies/cycle)

> 示例 5-3:Author.java（chapter5/cyclic_dependencies/cycle）

```java
public class Author {
  private String name;
  private List<Book> books = new ArrayList<>();

  public Author(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public void writeBook(String title, String text) {
    this.books.add(new Book(this, title, text));
  }

}
```

An Author has a name and can write a book, which is added to the list of books:

> 一名 Author 拥有一个姓名，并且可以写书（所写的书将被添加到书籍列表中）：

```java
public class Book {
  private Author author;
  private String title;
  private String text;

  public Book(Author author, String title, String text) {
    this.author = author;
    this.text = text;
    this.title = title;
  }

  public void printBook() {
    System.out.printf("%s, by %s\n\n%s", title, author.getName(), text);
  }
}
```

A book can be created given an Author, a title, and some text. After creation, a Book can be printed using printBook. Looking at that method’s code, we can see that its functionality is causing the dependency from Book to Author. In the end, the Author is just there to get a name for printing. This points to a new abstraction. The Book cares only about getting a name. Why should it be coupled to Author? Maybe there are even other ways of creating books than with authors (I heard deep learning is taking over our jobs any day now…).

> 一本书可以由一名 Author 创作，并且包含一个标题和一些文本，创建后可以使用 printBook 打印图书。仔细看看上述代码，会发现它生成了 Book 对 Author 的依赖。此时之所以需要 Author，是因为在打印时需要获取一个姓名。这样就形成了一个新的抽象。Book 只关心得到一个名字。为什么要把它和 Author 耦合起来呢？也许除了作者之外，还有其他创作书籍的方式（我曾经听说过，深度学习正在取代我们完成编书的任务）。

All of these deliberations point to a new abstraction: the indirection we are looking for. Because the book’s module is interested only in the name of things, let’s introduce an interface Named, as shown in Example 5-4.

> 所有的这些讨论都指向了一个新的抽象：即正在寻找的间接层。因为图书模块只对事物名称感兴趣，所以可以引入一个名为 Named 的接口，如示例 5-4 所示。

Example 5-4. Named.java (➥ chapter5/cyclic_dependencies/without_cycle)

> 示例 5-4:Named.java（chapter5/cyclic_dependencies/without_cycle）

```java
public interface Named {
  String getName();
}
```

Now Author can implement this interface. It already has the required getName implementation. The Book implementation needs to switch out usages of Author for Named. In the end, this results in the module structure shown in Figure 5-11. As an added bonus, the javamodularity.authors package need not be exported anymore.

> 现在，Author 可以实现这个接口，且它已经有了所需的 getName 实现。Book 实现需要使用的是 Named，而不是 Author。最终，模块结构如图 5-11 所示。这样做还有一个额外的好处，即不再需要导出 javamodularity.authors 包。

The `Named` interface breaks the cyclic dependency.
Figure 5-11. The Named interface breaks the cyclic dependency

This is far the from the only solution. In a larger system where multiple other components use Author similarly, the Named interface can also be lifted into its own module. For this example, that would lead to a triangular dependency graph with Named in the top module, and both the books and authors modules pointing to it.

> 以上并不是唯一的解决方案。在一些大型系统（多个其他组件使用了 Author）中，也可以将 Named 接口放置在自己的模块中。在本示例中，这样将产生一个三角依赖关系图，Named 为顶部模块，书籍和作者模块都指向它。

In general, interfaces play a big role in breaking cyclic dependencies. An interface is one of the most powerful means of abstraction in Java. They enable the pattern of dependency inversion by which cycles can be broken.

> 一般来说，接口在打破循环依赖方面起着重要的作用。接口是 Java 中最强大的抽象手段之一，它启用了可以打破循环的依赖反转模式。

Until now, we’ve assumed that the exact nature of the cyclic dependency is known. When cycles are indirect and arise through many steps, they can be hard to spot, though. Tools (for example SonarQube) can help detect cyclic dependencies in existing code. Use them to your advantage.

> 到目前为止，都是假定循环依赖的确切性质是已知的。当循环是间接的，并且通过很多步骤产生时，它们可能很难被发现，此时可以使用工具（如 SonarQube）帮助检测现有代码中的循环依赖。使用这些工具是很有帮助的。

## 5.6 Optional Dependencies 可选的依赖关系

One of the hallmarks of a modular application is its explicit dependency graph. So far, you’ve seen how to construct such a graph through requires statements in module descriptors. But what if a module isn’t strictly necessary at run-time, but nice to have?

> 模块化应用程序的标志之一是其显式的依赖关系图。到目前为止，已经学习了如何通过模块描述符中的 requires 语句来构造这样一个图。但是，如果一个模块在运行时不是绝对需要的，那么又该怎么办呢？

Many frameworks currently work this way: by adding a JAR file (let’s say fastjsonlib.jar) to the classpath, you get additional functionality. Whenever fastjsonlib.jar is not available, the framework uses a fallback mechanism or just doesn’t offer the enhanced functionality. For the given example, parsing JSON might be a bit slower but it still works. The framework effectively has an optional dependency on fastjsonlib. If your application already makes use of fastjsonlib, the framework also uses it; otherwise, it won’t.

> 目前许多框架都是按照以下方式工作的：向类路径中添加一个 JAR 文件（比如 fastjsonlib.jar），从而获得更多的功能。每当 fastjsonlib.jar 不可用时，框架就会使用回退机制或者不提供增强的功能。对于特定示例，解析 JSON 可能会慢一点，但它仍然有效。框架对 fastjsonlib 存在可选的依赖关系。如果应用程序已经使用了 fastjsonlib，那么框架也使用它，否则就不会使用。

NOTE

The Spring Framework is a famous example of a framework that has many optional dependencies. For instance, it has a Base64Utils helper class that delegates to either Java 8’s Base64 class or to an Apache Commons Codec Base64 class if it’s on the classpath. Regardless of the run-time environment, Spring itself must be compiled against both implementations.

> Spring Framework 是一个存在很多可选依赖关系的框架示例。例如，它有一个 Base64Utils 辅助类，代表了 Java 8 的 Base64 类，或者 Apache Commons Codec Base64 类（如果它在类路径中的话）。无论运行时环境如何，Spring 本身都必须针对两种实现进行编译。

Such optional dependencies cannot be expressed with our current knowledge of module descriptors. You can express a single module dependency graph for compile-time and run-time only through requires statements.

> 这些可选的依赖关系无法使用前面所介绍的模块描述符来表达，可通过 requires 语句表达编译时和运行时的单一模块依赖关系图。

Services offer this flexibility and are a great way to address optional dependencies in your application, as you will see later in “Implementing Optional Dependencies with Services”. However, services are bound to the ServiceLoader API, which can be an intrusive change for existing code. Since optional dependencies are often used by frameworks and libraries, they might not want to force the use of the services API on its users. When adopting services is not an option, another feature of the module system can be used to model optional dependencies: compile-time dependencies.

> 服务提供了极大的灵活性，并且是解决应用程序中可选依赖关系的好方法，稍后将在 5.6.2 节中看到。但是，将服务绑定到 ServiceLoader API 可能是对现有代码一种侵入式更改。由于框架和库经常使用可选依赖关系，因此它们可能不希望强制用户使用服务 API。当不能选择使用服务时，可以使用模块系统的另一个特性来建立可选依赖关系：编译时依赖关系。

### 5.6.1 Compile-Time Dependencies 编译时依赖关系

As the name already implies, compile-time dependencies are dependencies that are only required during compilation. You can express a compile-time dependency on a module by adding the static modifier to a requires clause, as shown in Example 5-5.

> 顾名思义，编译时依赖关系指的是仅在编译时需要的依赖关系。通过将修饰符 static 添加到 requires 子句中，可以在模块上表达编译时依赖关系，如示例 5-5 所示。

Example 5-5. Declaring a compile-time dependency (➥ chapter5/optional_dependencies)

> 示例 5-5：声明一个编译时依赖关系（chapter5/optional_dependencies）

```java
module framework {
  requires static fastjsonlib;
}
```

By adding static, the fastjsonlib module needs to be present when compiling, but not when running framework. This effectively makes fastjsonlib an optional dependency of framework from the perspective of the module consumer.

> 通过添加 static, fastjsonlib 模块需要在编译时出现，而不是在运行框架时出现。从模块消费者的角度来看，这样做可以有效地让 fastjsonlib 成为框架的可选依赖关系。

NOTE

Compile-time dependencies have other applications as well. For example, a module can export Java annotations that are used only during compilation. Requiring such a module should not lead to a run-time dependency, so requires static suits this scenario perfectly.

> 编译时依赖关系还有其他应用。例如，模块可以导出仅在编译期间使用的 Java 注释。请求此类模块不应导致运行时依赖关系，所以 requires static 适用这种情况。

The question of course is, what happens when framework is run without fastjsonlib? Let’s explore the scenario where framework uses the class FastJson exported from fastjsonlib directly:

> 当然，此时的问题是在没有 fastjsonlib 的情况下运行 framework 时会发生什么事情呢？接下来看一下 framework 使用直接从 fastjsonlib 导出的 FastJson 类：

```java
package javamodularity.framework;

import javamodularity.fastjsonlib.FastJson;

public class MainBad {

  public static void main(String... args) {
    FastJson fastJson = new FastJson();
  }

}
```

Running framework without fastjsonlib in this case leads to a NoClassDefFound​Error. The resolver doesn’t complain about the missing fastjsonlib module, because it is a compile-time only dependency. Still, we get an error at run-time because Fast​Json clearly is necessary for framework in this case.

> 如果在没有 fastjsonlib 的情况下运行 framework，会产生 NoClassDefFound Error。此时编译器并不会“抱怨”所缺少的 fastjsonlib 模块，因为它只是一个编译时依赖关系。但在运行时会产生错误，因为 FastJson 是运行 framework 所必需的。

The onus is on the module expressing a compile-time dependency to guard against these problems at run-time. That means framework needs to code defensively around the usage of classes from compile-time dependencies. A direct reference to FastJson as in MainBad is problematic, because the VM will always try to load the class and instantiate it, leading to the NoClassDefFoundError.

> 在模块上表达编译时依赖关系的作用是防止在运行时出现上述问题。这意味着 framework 需要围绕编译时依赖关系对类的使用进行防御性编码。在 MainBad 中直接引用 FastJson 是有问题的，因为 VM 将总是尝试加载类并将其实例化，这样会导致 NoClassDefFoundError。

Fortunately, Java has lazy loading semantics for classes: they are loaded at the last possible time. If we can somehow tentatively try to use the FastJson class and gracefully recover if it’s not available, we achieve our goal. By using reflection with appropriate try-catch blocks, framework can prevent run-time errors:

> 幸运的是，Java 对类具有延迟加载语义：即在最后可能的时间里加载类。如果可以以某种方式试探性地尝试使用 FastJson 类，并在该类不可用的情况下恢复正常，那么就可以实现我们的目标。通过使用反射以及适当的 try-catch 块，framework 可以防止运行时错误：

```java
package javamodularity.framework;

import javamodularity.fastjsonlib.FastJson;

public class Main {

  public static void main(String... args) {
    try {
      Class<?> clazz = Class.forName("javamodularity.fastjsonlib.FastJson");
      FastJson instance =
        (FastJson) clazz.getConstructor().newInstance();
      System.out.println("Using FastJson");
    } catch (ReflectiveOperationException e) {
      System.out.println("Oops, we need a fallback!");
    }
  }

}
```

When the FastJson class isn’t found, the catch block can be used as a fallback. On the other hand, if fastjsonlib is there, we can use FastJson without any problems after the reflective instantiation.

> 当找不到 FastJson 类时，可以使用 catch 块作为一个后备。另一方面，如果 fastjsonlib 存在，则可以在反射实例化之后使用 FastJson 类。

WARNING

The requires static fastjsonlib clause does not cause fast​jsonlib to be resolved at run-time, even if fast​jsonlib is on the module path! There needs to be a direct requires clause on fast​jsonlib, or you can add it as root module through --add-modules fastjsonlib for it to be resolved. In both cases, fastjsonlib is then resolved, and framework reads and uses it.

> 即使 fastjsonlib 在模块路径上，requires static fastjsonlib 子句也不会导致 fastjsonlib 在运行时被解析！在 fastjsonlib 上需要有一个直接的 requires 子句，或者可以通过--add-modulesfastjsonlib 将其添加为根模块，以便进行解析。在这两种情况下，fastjsonlib 都会被解析，并被 framework 读取和使用。

Guarding each and every use of classes from a compile-time dependency this way can be quite painful. Lazy loading also means that the moment a class is loaded can be surprising. A notorious example is the static initializer block in a class. That should already be a warning sign that requires static may not be the best way to create optional coupling between modules.

> 通过编译时依赖关系监视每个类及其使用可能是非常痛苦的。同时，延迟加载也意味着一个类的加载时间可能是出人意料的，一个臭名昭著的例子是类中静态初始化块。由此可见，requiresstatic 可能并不是在模块之间创建可选耦合的最佳方式。

Whenever your module uses compile-time dependencies, try to concentrate the use of these optional types in one part of the module. Put the guard on the instantiation of a single top-level class, that in turn has all the (direct) references to types in the optional dependency. This way, the module won’t get littered with defensive guard code.

> 无论模块何时使用编译时依赖关系，都要尝试将这些可选类型的使用集中在模块的一部分中。首先保护好单个顶级类的实例化，而该类反过来（直接）引用了可选依赖关系中的类型。这样一来，模块就不会被防御性代码所破坏。

There are other legitimate uses for requires static—for example, to reference compile-time-only annotations from another module. A requires clause (without static) would cause the dependency to be required at run-time as well.

> 当然，在某些情况下也是需要使用 requires static 的。例如，引用另一个模块的编译时注释。一个 requires 子句（不带 static）也会产生在运行时需要的依赖关系。

Sometimes annotations on a class are used only at compile-time—for example, to perform static analysis (such as checking @Nullable or @NonNull), or to mark a type as input for code generation. When you retrieve annotations for an element at run-time and the annotation class is not present, the JVM gracefully degrades and won’t throw any classloading exceptions. Still, you need access to the annotation types when compiling the code.

> 有时，类上的注释仅在编译时使用。例如，执行静态分析（如检查@Nullable 或@NonNull），或将类型标记为代码生成的输入。当在运行时检索元素的注释并且注释类不存在时，JVM 会正常降级，并且不会抛出任何类加载异常。不过，编译代码时仍然需要访问注释类型。

Let’s look at a fictional @GenerateSchema annotation that can be put on entities. At build-time, these annotations are used to find classes to generate a database schema based on their signature. The annotation is not used at run-time. Therefore, we want code annotated with @GenerateSchema not to require the schemagenerator module (which exports the annotation) at run-time. Say we have the class in Example 5-6 in module application.

> 接下来看一个可以放在实体上的虚构的@GenerateSchema 注释。在构建时，这些注释用于查找类，根据其签名生成数据库模式。注释在运行时不使用。因此，希望使用@GenerateSchema 注释的代码在运行时不要求使用 schemagenerator 模块（该模块导出了注释）。假设在模块 application 中有示例 5-6 中的类。

Example 5-6. An annotated class (➥ chapter5/optional_dependencies_annotations)

> 示例 5-6：注释类（chapter5/optional_dependencies_annotations）

```java
package javamodularity.application;

import javamodularity.schemagenerator.GenerateSchema;

@GenerateSchema
public class BookEntity {

  public String title;
  public String[] authors;

}
```

The module descriptor for application should have the right compile-time dependency:

> application 的模块描述符应该包含正确的编译时依赖关系：

```java
module application {
  requires static schemagenerator;
}
```

In application, there’s also a main class that instantiates the BookEntity and tries to obtain the annotations on that class:

> 在 application 中，还有一个实例化 BookEntity 并尝试获取该类上注释的主类：

```java
package javamodularity.application;

public class Main {
  public static void main(String... args) {
    BookEntity b = new BookEntity();
    assert BookEntity.class.getAnnotations().length == 0;
    System.out.println("Running without annotation @GenerateSchema present.");
  }
}
```

When you run the application without the schemagenerator module, everything works out fine:

> 当运行不带 schemagenerator 模块的应用程序时，一切正常：

```sh
$ java -ea --module-path out/application \
       -m application/javamodularity.application.Main
Running without annotation @GenerateSchema present.
```

(The -ea flag enables run-time assertions in the JVM.)

> （-ea 标志启用 JVM 中的运行时断言。）

There are no classloading or module resolution problems due to the missing schemagenerator module at run-time. Leaving out the module at run-time is possible because it’s a compile-time dependency. Subsequently, the JVM gracefully handles the absence of annotation classes at run-time, as it always has. The call to getAnnotations returns an empty array.

> 由于运行时缺少 schemagenerator 模块，因此不存在类加载或模块解析问题。在运行时缺少模块是允许的，因为它是一个编译时依赖关系。随后，JVM 像往常一样在运行时处理注释类的缺失。调用 getAnnotations 将返回一个空数组。

However, if we add the schemagenerator module explicitly, the @GenerateSchema annotation is found and returned:

> 然而，如果显式地添加了 schemagenerator 模块，那么就会找到并返回注释@GenerateSchema：

```sh
$ java -ea --module-path out/application:out/schemagenerator \
       -m application/javamodularity.application.Main
Exception in thread "main" java.lang.AssertionError
        at application/javamodularity.application.Main.main(Main.java:6)
```

An AssertionError is thrown because now the @GenerateSchema annotation is returned. No longer is the annotations array empty.

> 此时，抛出了一个 AssertionError，因为现在返回了@GenerateSchema 注释，注释数组不再为空了。

There’s no need for guard code to cope with missing annotation types at run-time, unlike the previous examples of compile-time dependencies we’ve seen. The JVM already takes care of this during classloading and reflective access to annotations.

> 与前面所看到的编译时依赖关系示例不同的是，在运行时不需要使用防护代码来处理缺少的注释类型。JVM 已经在类加载和反射访问注释过程中处理了这个问题。

It is also possible to combine the static modifier on requires with transitive:

> 此外，还可以将 requires 上的修饰符 static 与 transitive 结合起来使用：

```java
module questionable {
  exports questionable.api;
  requires transitive static fastjsonlib;
}
```

You need to do this when a type from the optional dependency is referenced in an exported package. The fact that it’s possible to combine static and transitive doesn’t make it a good idea. It puts the responsibility of creating proper guards on the consumer of the API, which certainly doesn’t adhere to the principle of the least surprise. In fact, the only reason this combination of modifiers is possible is to enable modularization of legacy code by using this pattern.

> 当在导出包中引用可选依赖关系的一个类型时，可能需要使用上面的代码。虽然 static 可以与 transitive 结合使用，但这样做并不是一个好主意。此时需要在 API 消费者上创建适当的防护代码，这显然不符合最小惊讶原则（the principle of least surprise）。事实上，使用这种组合修饰符的唯一原因是通过这种模式来完成遗留代码的模块化。

Optional dependencies can be approximated through requires static, but with services in the module system we can do better!

> 虽然通过 requires static，可选依赖关系可以完成很多功能，但在模块系统中的服务可以做得更好！

### 5.6.2 Implementing Optional Dependencies with Services 使用服务实现可选依赖关系

Using compile-time dependencies to model optional dependencies is possible but requires diligent use of reflection to safeguard classloading. Services are a better fit for this usage. You already saw in Chapter 4 that a service consumer can obtain zero or more implementations of a service type from service provider modules. Getting zero or one service implementation is just a special case of this general mechanism.

> 虽然使用编译时依赖关系来建立可选依赖关系是可能的，但需要经常使用反射来保护类的加载。此时，使用服务更加合适。在第 4 章已经看到，服务消费者可以从服务提供者模块中获得零个或多个服务类型的实现。获取零个或一个服务实现只是这种通用机制的一个特例。

Let’s apply this to the previous example with framework by using the optional fast​jsonlib. Fair warning: we start with a naive refactoring, refining it into a real solution in several steps.

> 接下来使用可选的 fastjsonlib 将该机制应用于前面的框架示例。再次说明一下：首先从一个不成熟的重构开始，然后通过几个步骤将其精炼成一个真正的解决方案。

In Example 5-7, framework becomes a consumer of an optional service provided by fastjsonlib.

> 在示例 5-7 中，framework 变成了由 fastjsonlib 所提供的可选服务的消费者。

Example 5-7. Consuming a service, which may or may not be available at run-time (➥ chapter5/optional_dependencies_service)

> 示例 5-7：消费一个在运行时可能可用，也可能不能用的服务（chapter5/optional_dependencies_service）

```java
module framework {
  requires static fastjsonlib;
  uses javamodularity.fastjsonlib.FastJson;
}
```

By virtue of the uses clause, we can now load FastJson with ServiceLoader in the framework code:

> 由于使用了 uses 子句，因此可以在框架代码中使用 ServiceLoader 加载 FastJson：

```java
FastJson fastJson =
  ServiceLoader.load(FastJson.class)
               .findFirst()
               .orElse(getFallBack());
```

We no longer need reflection in the framework code to get FastJson when it’s available. If there’s no service found by ServiceLoader (meaning findFirst returns an empty Optional), we assume we can get a fallback implementation with getFallBack.

> 当 FastJson 可用时，不再需要在框架代码中使用反射来获取 FastJson。如果 ServiceLoader 没有找到任何服务（意味着 findFirst 返回了一个空 Optional），那么假设可以通过使用 getFallBack 来实现回退操作。

Of course, fastjsonlib must provide the class we’re interested in as a service:

> 当然，fastjsonlib 必须提供我们感兴趣的类作为服务：

```java
module fastjsonlib {
  exports javamodularity.fastjsonlib;
  provides javamodularity.fastjsonlib.FastJson
      with javamodularity.fastjsonlib.FastJson;
}
```

With this setup, the resolver even resolves fastjsonlib without having to add it explicitly, based on the uses and provides clauses.

> 有了上面的设置，解析器甚至可以在不显式添加 fastjsonlib 的情况下，根据 uses 和 provides 子句来解析 fastjsonlib。

There are several awkward problems with this refactoring from a pure compile-time dependency to a service, though. First of all, it’s a bit strange to expose a class directly as a service, rather than exposing an interface and hiding the implementation. Splitting FastJson into an exported interface and encapsulated implementation solves this. This refactoring also enables the framework to implement a fallback class implementing the same interface.

> 不过，从纯粹的编译时依赖关系到服务的重构还有一些问题需要解决。首先，将一个类直接公开为一个服务而不是公开一个接口并隐藏实现细节，这种做法本身就有点奇怪。将 FastJson 拆分成一个导出接口和封装的实现就可以解决这个问题。此外，该重构也使框架能够实现一个实现了相同接口的回退（fallback）类。

A bigger problem surfaces when trying to run framework without fastjsonlib. After all, fastjsonlib is supposed to be optional so this should be possible. When framework is started without fastjsonlib on the module path, the following error occurs:

> 当尝试在没有 fastjsonlib 的情况下运行 framework 时会出现更大的问题。毕竟，fastjsonlib 是可选的，所以缺少 fastjsonlib 是完全可能的。当在模块路径上没有 fastjsonlib 的情况下启动 framework 时，会发生以下错误：

```log
Error occurred during initialization of VM
java.lang.module.ResolutionException: Module framework does not read a module
  that exports javamodularity.fastjsonlib
```

You cannot declare a service dependency with uses on a type that you can’t read at run-time, regardless of whether there is a provider module. The obvious solution is to change the compile-time dependency on fastjsonlib into a normal dependency (requires without static). However, that’s not what we want: the dependency on the library should be optional.

> 无论是否存在提供者模块，都不能在运行时无法读取的类型上使用 uses 声明服务依赖关系。明显的解决方案是将对 fastjsonlib 的编译时依赖关系改为正常的依赖关系（使用不带 static 的 requires）。然而，这并不是一个理想的解决方案：对库的依赖应该是可选的。

At this point, a more intrusive refactoring is warranted. It becomes clear that not everything can be optional in the relation between framework and fastjsonlib for this service setup to work. Why does the FastJson interface (assuming we refactored it into an interface) live in fastjsonlib? In the end, it’s the framework which dictates that functionality it wants to use. The functionality is optionally provided by a library or by fallback code in the framework itself. Through this realization, it makes sense to put the interface in framework or in a separate API module that is shared between the framework and library.

> 从这一点上讲，更具有侵入性的重构是非常必要的。很显然，为了让上面的服务设置正常工作，在 framework 和 fastjsonlib 之间的关系中并不是所有的关系都是可选的。为什么 FastJson 接口（假设将其重构成一个接口）要位于 fastjsonlib 中呢？最终，将由框架决定所想要使用的功能。功能可由库或框架本身的回退代码有选择地提供。鉴于这个现实，将接口放在 framework 中或者框架和库之间共享的独立 API 模块中会更有意义。

This is an intrusive redesign. It almost inverts the relationship between the framework and the library. Instead of the framework optionally requiring the library, the library must require the framework (or its API module) to implement an interface and offer the implementation as a service. However, when such a redesign can be pulled off, it results in an elegant decoupled interaction between the framework and library.

> 这是一个侵入性重新设计，它几乎颠倒了框架和库之间的关系。此时，框架不再选择性地使用库，而是库必须使用框架（或其 API 模块）来实现一个接口，并将该实现作为一个服务提供。然而，当取消这种重新设计时，框架和库之间就会实现完美的解耦。

## 5.7 Versioned Modules 版本化模块

Talking about modules for frameworks and libraries inevitably leads to questions around versioning. Modules are independently deployable and reusable units. Applications are built by combining the right deployment units (modules). Having just a module name is not sufficient to select the right modules that work together. You need versions as well.

> 当谈论到框架和库的模块时，不可避免地会谈到版本化问题。模块是可独立部署和可重复使用的单元，可通过组合正确的部署单元（模块）来构建应用程序。仅使用模块名称来选择在一起工作的正确模块是不够的，此时还需要版本信息。

Modules in the Java module system cannot declare a version in module-info.java. Still, it is possible to attach version information when creating a modular JAR. You can set a version by using the `--module-version=<V>` flag of the jar tool. The version V is set as an attribute on the compiled module-info.class and is available at run-time. Adding a version to your modular JARs is a good practice, especially the for API modules discussed earlier in this chapter.

> Java 模块系统中的模块不能在 module-info.java 中声明一个版本。不过，在创建模块化 JAR 时可以附加版本信息。可以使用 jar 工具的`--module-version = <V>`标志来设置版本。版本 V 被设置为已编译的 module-info.class 上的一个属性，并且在运行时可用。为模块化 JAR 添加版本是一个很好的做法，特别是本章前面讨论的 API 模块。

#### SEMANTIC VERSIONING 语义版本控制

There are many different ways to version modules. The most important goal in versioning is to communicate the impact of changes to consumers of a module. Is a newer version a safe drop-in replacement for the previous version? Are there any changes to the API of the module? Semantic versioning formalizes a versioning scheme that is widely used and understood:

> 版本化模块有很多不同的方法。版本控制中最重要的目标是将更改的影响传达给模块的消费者。新版本是否是以前版本的安全替代品，模块的 API 是否有任何更改？语义版本控制规范了被广泛使用和理解的版本控制方案：

MAJOR.MINOR.PATCH

Breaking changes, such as changing a method signature in an interface, lead to a bump in the MAJOR version part. Changes to a public API that are backward compatible, such as adding a method on a public class, bump the MINOR part of the version string. Last, the PATCH part is incremented when implementation details change. This can be a bug fix, or optimizations to the code. In any case, a PATCH increment should always be a safe drop-in replacement for the previous version.

> 破坏性的更改（例如，更改了接口中的方法签名）会导致 MAJOR 版本部分发生变化。对向后兼容的公共 API 的更改（例如，在公共类上添加一个方法）则会改变版本字符串的 MINOR 部分。最后，当改变实现细节时 PATCH 部分会增加，所改变的实现细节可以是一个 Bug 修复或代码优化。在任何情况下，PATCH 增量都应该是以前版本的安全替代品。

Note that it is not always straightforward to determine whether something is a major or minor change. For example, adding a method to an interface is a major change only if consumers of the interface are supposed to implement it. If, on the other hand, the interface is only called by consumers (but not implemented), consumers won’t break when a method is added. To muddy the waters further, default methods on interfaces (introduced in Java 8) can turn the addition of a method on an interface into a minor change even in the first scenario. The important thing is to always reason from the perspective of the consumer of the module. The module’s next version should reflect the impact of the upcoming changes for the consumers of a module.

> 请注意，判断某些事情是主要变化还是次要变化并不总是那么简单的。例如，只有当接口的消费者应该实现该接口时，向接口添加方法才属于重大变化。而如果接口仅由消费者调用（而不是实现），那么在添加了方法之后并不会破坏消费者的使用体验。更复杂的是，即使在第一种情况下，接口上的默认方法（在 Java 8 中引入）也可以将接口上的方法添加变为次要变化。不管是哪种情况，重要的是始终从模块消费者的角度来推理：模块的下一个版本应该反映即将发生的变化对模块用户的影响。

### Module Resolution and Versioning 模块解析和版本控制

Even though there’s support for adding a version to modular JARs, it is not used in any way by the module system (yet). Modules are resolved purely by name. This may seem strange, because we already established that versions play an important role in deciding which modules play nice together. Ignoring versions during module resolution is not an oversight, but a deliberate design choice in the Java module system.

> 即使支持向模块化 JAR 添加版本，模块系统也不会以任何方式使用它。模块完全是由名称进行解析的。这听起来可能很奇怪，因为前面已经讲过，版本在决定哪些模块可以一起很好地工作上扮演着重要的角色。在模块解析期间忽略版本并不是一个疏忽，而是 Java 模块系统中一个慎重的设计选择。

How you indicate which versions of deployment units work together is a rather controversial topic. What are the syntax and semantics of the version string? Do you specify a version range for dependencies? Or only exact versions? What happens if you end up requiring two versions at the same time, in the latter case? Such conflicts must be addressed.

> 如何指出哪个版本的部署单元可以一起工作是一个颇有争议的话题。版本字符串的语法和语义是什么？是否指定了依赖关系的版本范围？或者只有确切的版本？如果最终同时需要两个版本，会发生什么情况呢？这些冲突必须解决。

Tools and frameworks (such as Maven and OSGi) have opinionated answers to each of those questions. It turns out these version-selection algorithms and associated conflict-resolution heuristics are both complex and (sometimes subtly) different. That’s why the Java module system, for now, eschews the notion of version selection during the module-resolution process. Whatever strategy adopted would end up deeply ingrained in the module system and, hence, in the compiler, language specification, and JVM. The cost of not getting it right was simply too high. Therefore, the requires clause in module descriptors takes only a module name, not a module version (or range).

> 对于上述问题，不同的工具和框架（例如，Maven 和 OSGi）都给出了自己的解决方案。事实证明，这些版本选择算法以及相关的冲突解决试探法既复杂又不同（有时仅存在微小的区别）。这就是为什么现在的 Java 模块系统在模块解析过程中避开了版本选择的概念。无论采用什么策略，都会在模块系统中变得根深蒂固，因此也会深入到编译器、语言规范和 JVM 中。策略的错误选择所造成的代价太高了，因此，模块描述符中的 requires 子句只使用模块名称，而不是模块版本（或范围）。

That still leaves us developers with a challenge. How do we select the right module versions to put on the module path? The answer is surprisingly straightforward, if a bit unsatisfactory: just as we did before with the classpath. Selection and retrieval of the right versions of dependencies are delegated to existing build tools. Maven and Gradle et al. handle this by externalizing the dependency version information into a POM file. Other tools may use other means, but the fact remains this information must be stored outside the module.

> 此时，开发人员仍然面临一项挑战。如何选择正确的模块版本放在模块路径上？答案非常简单，虽然有点不尽如人意：就像前面使用类路径那样，即将正确版本的依赖关系的选择和检索委托给现有的构建工具来完成。Maven 和 Gradle 等通过将依赖关系版本信息外部化到一个 POM 文件来处理这个问题，而其他工具可能会使用其他方法，但不管用什么方法，这些信息必须存储在模块之外。

Figure 5-12 shows the steps involved with building a source module application that depends on two other modular JARs, lib and foo. At build-time, build tools download the right versions of the dependencies from a repository, using information from POM files. Any version conflicts arising during this process must be resolved by the build tool’s conflict-resolution algorithm. The downloaded modular JARs are put on the module path for compilation. Then, the Java compiler resolves the module graph guided by information from module-info descriptors on the module path and application itself. More details on how existing build tools handle modules can be found in Chapter 11.

> 图 5-12 显示了构建一个依赖于两个模块化 JAR（lib 和 foo）的源模块 application 所涉及的步骤。在构建时，构建工具使用来自 POM 文件的信息从存储库下载正确版本的依赖项。在此过程中出现的任何版本冲突必须由构建工具的冲突解决算法来解决。下载的模块化 JAR 被放在模块路径上进行编译。然后，Java 编译器解析由模块路径上的模块信息描述符以及 application 所引导的模块图。有关现有构建工具如何处理模块的更多详细信息请参阅第 11 章。

Build tools for version selection
Figure 5-12. Build tools select the right version of a dependency to put on the module path

1. A build tool such as Maven or Gradle downloads dependencies from a repository such as Maven Central. Version information in a build descriptor (for example, pom.xml) controls which versions are downloaded.
2. The build tool sets up the module path with the downloaded versions.
3. Then, the Java compiler or runtime resolves the module graph from the module path, which contains the correct versions without duplicates for the application to work. Duplicate versions of a module result in an error.

---

> 1. 诸如 Maven 或 Gradle 之类的构建工具从存储库（如 Maven Central）下载依赖项。由构建描述符中的版本信息控制下载哪个版本。
> 2. 构建工具使用下载的版本设置模块路径。
> 3. 然后，Java 编译器或运行时从模块路径中解析模块图，为了保证应用程序正常运行，不能包含重复的版本。模块的重复版本会导致错误。

The Java module system ensures that all necessary modules are present when compiling and running application through the module-resolution process. However, the module system is oblivious to which version of a module is resolved. As long as there is a module with the correct name, it will be resolved. In addition, if multiple modules with the same name (but possibly a different version) are found in a directory on the module path, this leads to an error.

> Java 模块系统确保在通过模块解析过程编译和运行 application 时，所有必需的模块都存在。但是，模块系统不知道模块的哪个版本被解析。只要存在一个正确名称的模块，它就会被解析。另外，如果在模块路径的目录中找到多个名称相同（但可能不同版本）的模块，则会导致错误。

WARNING

If two modules with the same name are in different directories on the module path, the resolver uses the first one it encounters and ignores the second. No errors are produced in this case.

> 如果两个具有相同名称的模块位于模块路径上的不同目录中，那么解析器将使用所遇到的第一个模块，而忽略第二个模块。此时，不会产生任何错误。

There are genuine situations where having multiple versions of the same module at the same time is expedient. We’ve seen that by default, this scenario is not supported when launching an application from the module path. This is similar to the situation before the module system. On the classpath, two JARs of a different version can lead to undefined run-time behavior, which is arguably worse.

> 真实的情况是，同时使用相同模块的多个版本只是权宜之计。前面已经看到，在默认情况下，当通过模块路径启动应用程序时，上述情况是不支持的。这与模块系统之前的情况类似。在类路径上，两个不同版本的 JAR 可能导致未定义的运行时行为，这可能更糟。

In application development, it is highly advisable to find a way to unify dependencies on a single module version. Often, the desire for running multiple concurrent versions of the same module stems from laziness rather than a pressing need. When developing generic, container-like applications, this may not always be possible. In Chapter 6, you will see that there is a way to resolve multiple versions of a module by using a more sophisticated module system API to construct module graphs at run-time. Another option in these cases is to adopt an existing module system such as OSGi, which offers the possibility to run multiple versions concurrently out of the box.

> 在应用程序开发中，强烈建议找到一种统一依赖于单个模块版本的方法。通常，运行同一模块的多个并发版本的原因是因为懒惰，而不是迫切需要。在开发类似于容器的通用应用程序时，这种懒惰的做法往往并不可行。在第 6 章中，将会看到有一种方法可以通过使用更复杂的模块系统 API 在运行时构建模块图来解析模块的多个版本。而另一种选择是采用现有的模块系统，如 OSGi，它则提供了可以同时运行多个版本的可能性。

## 5.8 Resource Encapsulation 资源封装

We’ve spent a considerable amount of time talking about strongly encapsulating code in modules. Although that’s the most obvious use of strong encapsulation, applications typically have more resources than just code. Think of files containing translations (localized resource bundles), configuration files, images used in user interfaces, and so on. It’s worthwhile to encapsulate such resources in a module as well, colocating them with the code using the resources. Having modules nose into another module’s resources is just as bad as depending on private implementation classes.

> 前面已经花了相当多的时间来讨论模块中的强封装代码。尽管这是强封装的最显著的用法，但除了代码，应用程序通常拥有更多的资源，比如包含翻译（本地化资源包）的文件、配置文件、用户界面中使用的图像等。将这些资源封装在一个模块中是非常值得的，可以将它们与使用资源的代码进行整合。一个模块使用另一个模块的资源就像依赖私有实现类一样糟糕。

Historically, resources on the classpath were even more free-for-all than code, because no access modifiers can be applied to resources. Any class can read any resource on the classpath. With modules, that changes. By default, resources within packages in a module are strongly encapsulated. Those resources can be used only from within the module, just as with classes in nonexported packages.

> 从历史上看，代码可以“免费地”使用类路径上的资源，因为没有在资源上应用访问修饰符。任何类都可以读取类路径上的任何资源。但随着模块的出现，情况也发生了改变。在默认情况下，模块中包内的资源被强制封装，这些资源只能在模块中使用，就像非导出包中的类一样。

However, many tools and frameworks depend on finding resources regardless of where they come from. Frameworks might scan for certain configuration files (for example, persistence.xml or beans.xml in Java EE) or otherwise depend on resources from application code. This requires resources within a module to be accessible from other modules. To handle these cases gracefully and maintain backward compatibility, many exceptions have been added to the default of encapsulating resources in modules.

> 然而，许多工具和框架依赖于查找资源，而不管它们来自哪里。框架可能会扫描某些配置文件（例如，Java EE 中的 persistence.xml 或 beans.xml），或者依赖于应用程序代码中的资源，这就要求模块内的资源可以从其他模块访问。为了妥善处理这些情况并保持向后兼容性，在模块中封装资源的默认设置中增加了许多例外情况。

First, we are going to look at loading resources from within the same module. Second, we are going to look at how modules can share resources and which exceptions to the default of strong encapsulation are in place. Last, we turn our attention to a special case of resource loading: ResourceBundles. This mechanism is mainly used for localization and has been updated for the module system.

> 首先，看看如何从同一个模块中加载资源。其次，了解一下模块如何共享资源，以及默认强封装的例外情况。最后，将注意力转向资源加载的一个特例：ResourceBundles。这个机制主要用于本地化，并且已经针对模块系统进行了更新。

### 5.8.1 Loading Resources from a Module 从模块加载资源

Here’s an example of a compiled module firstresourcemodule containing both code and resources:

> 下面是一个已编译模块 firstresourcemodule 的示例，包含了代码和资源：

```
firstresourcemodule
├── javamodularity
│   └── firstresourcemodule
│       ├── ResourcesInModule.class
│       ├── ResourcesOtherModule.class
│       └── resource_in_package.txt
├── module-info.class
└── top_level_resource.txt
```

There are two resources in this example alongside two classes and the module descriptor: resource_in_package.txt and top_level_resource.txt. We assume the resources are put into the module during the build process.

> 在本示例中，除了两个类以及模块描述符之外，还有两个资源：resource_in_package. txt 和 top_level_resource.txt。假设在构建过程中将资源放入模块中。

Resources are typically loaded through resource loading methods provided by the Class API. This still works within a module, as shown in Example 5-8.

> 资源通常通过 Class API 提供的资源加载方法进行加载。在模块中的加载方法也是一样的，如示例 5-8 所示。

Example 5-8. Various ways of loading resources in a module (➥ chapter5/resource_encapsulation)

> 示例 5-8：在模块中加载资源的各种方法（chapter5/resource_encapsulation）

```java
   public static void main(String... args) throws Exception {
      Class clazz = ResourcesInModule.class;
      InputStream cz_pkg = clazz.getResourceAsStream("resource_in_package.txt"); 1
      URL cz_tl = clazz.getResource("/top_level_resource.txt"); 2

      Module m = clazz.getModule(); 3
      InputStream m_pkg = m.getResourceAsStream(
        "javamodularity/firstresourcemodule/resource_in_package.txt"); 4
      InputStream m_tl = m.getResourceAsStream("top_level_resource.txt"); 5

      assert Stream.of(cz_pkg, cz_tl, m_pkg, m_tl)
                   .noneMatch(Objects::isNull);
   }

}
```

1. Reading a resource with Class::getResource resolves the name relative to the package the class is in (javamodularity.firstresourcemodule in this case).
2. When reading a top-level resource, a slash must be prefixed to avoid relative resolution of the resource name.
3. You can obtain a java.lang.Module instance from Class, representing the module the class is from.
4. The Module API introduces new ways to obtain resources from a module.
5. This getResourceAsStream method also works for top-level resources. The Module API always assumes absolute names, so no leading slash is necessary for a top-level resource.

---

> 1. 使用 Class:getResource 读取一个资源，并解析相对于类所在包的名称（此时为 javamodularity.firstresourcemodule）。
> 2. 当读取顶级资源时，必须以斜杠为前缀，以避免资源名称的相对解析。
> 3. 可以通过 Class 获取 java.lang.Module 实例（表示类来自哪个模块）。
> 4. Module API 引入了从模块获取资源的新方法。
> 5. getResourceAsStream 方法也适用于顶级资源。Module API 总是使用绝对名称，因此顶级资源不需要前导斜杠。

All of these methods work for loading resources within the same module. Existing code does not have to be changed for loading resources. As long as the Class instance you’re calling getResource{AsStream} on belongs to the current module containing the resource, you get back a proper InputStream or URL. On the other hand, when the Class instance comes from another module, null will be returned because of resource encapsulation.

> 上述方法用于加载同一个模块中的资源，无须为了加载资源而修改现有的代码。只要调用 getResource{AsStream}的 Class 实例属于包含资源的当前模块，就会返回正确的 InputStream 或 URL。另一方面，当 Class 实例来自另一个模块时，由于资源封装，将返回 null。

WARNING

You can also load resources through ClassLoader::getResource\* methods. In a module context, it is better to use the methods on Class and Module instead. The ClassLoader methods do not take into account the current module context like the methods Class and Module do, leading to potentially confusing results.

> 也可以通过 ClassLoader::getResource\*方法加载资源。在模块上下文中，最好使用 Class 和 Module 上的方法。ClassLoader 方法不会像 Class 和 Module 方法那样考虑当前模块上下文，从而导致潜在的混乱结果。

There’s a new way to load resources as well. It’s through the new Module API, which is discussed in more detail in “Reflection on Modules”. The Module API exposes getResourceAsStream to load a resource from that module. Resources in a package can be referred to in an absolute way by replacing the dots in the package name with slashes and appending the filename. For example, javamodularity.firstresourcemodule becomes javamodularity/firstresourcemodule. After adding the filename, the argument to load a resource in a package becomes javamodularity/firstresourcemodule/resource_in_package.txt.

> 还可以使用一种新的方法来加载资源。它是通过新的 Module API 来实现的，在“对模块的反射”一节（6.2 节）将对此进行更详细的讨论。Module API 公开了 getResourceAsStream，以便从模块加载资源。包中的资源可以以绝对方式引用，用斜杠替换包名称中的点并附加文件名即可。例如，javamodularity.firstresourcemodule 变成 javamodularity/firstresourcemodule。添加文件名后，加载包中资源的参数变为 javamodularity/firstresourcemodule/resource_in_package.txt。

Any resource in the same module, whether it is in a package or at the top level, can be loaded by the methods discussed so far.

> 同一模块中的任何资源，无论是在包中或是在顶层，都可以通过前面所介绍的方法加载。

### 5.8.2 Loading Resources Across Modules 跨模块加载资源

What happens if you obtain a Module instance representing a module other than the current module? It might seem you can call getResourceAsStream on this other Module instance and get access to all resources in that module. Thanks to strong encapsulation of resources, this is not (always) the case. Several exceptions to this rule exist, so let’s add a module secondresourcemodule to our example to explore the different scenarios:

> 如果获取了表示当前模块以外的模块的 Module 实例，那么又会发生什么情况？看起来似乎可以在这个模块实例上调用 getResourceAsStream，并访问该模块中的所有资源。但由于资源的强封装性，情况并非如此。前面所介绍的资源访问规则存在几个例外情况，所以接下来向示例添加一个模块 secondresourcemodule，从而研究一下不同的场景：

```
secondresourcemodule
├── META-INF
│   └── resource_in_metainf.txt
├── javamodularity
│   └── secondresourcemodule
│       ├── A.class
│       └── resource_in_package2.txt
├── module-info.class
└── top_level_resource2.txt
```

We assume that both module descriptors for firstresourcemodule and secondresourcemodule have an empty body, meaning no package is exported. There’s a package containing class A and a resource, there’s a top-level resource, and a resource inside the META-INF directory. While looking at the following code, keep in mind resource encapsulation applies only to resources inside packages in a module.

> 假设 firstresourcemodule 和 secondresourcemodule 的模块描述符都是空的，这意味着没有包被导出。此时，分别有一个包含类 A 和资源的包、一个顶级资源以及 META-INF 目录下的一个资源。在查看下面的代码时，请记住，资源封装只适用于模块中包内的资源。

We’re going to try to access those resources in secondresourcemodule from a class in firstresourcemodule:

> 接下来尝试从 firstresourcemodule 的一个类中访问 secondresourcemodule 中的资源：

```java
Optional<Module> otherModule =
   ModuleLayer.boot().findModule("secondresourcemodule"); 1

otherModule.ifPresent(other -> {
   try {
      InputStream m_tl = other.getResourceAsStream("top_level_resource2.txt"); 2
      InputStream m_pkg = other.getResourceAsStream(
          "javamodularity/secondresourcemodule/resource_in_package2.txt"); 3
      InputStream m_class = other.getResourceAsStream(
          "javamodularity/secondresourcemodule/A.class"); 4
      InputStream m_meta =
          other.getResourceAsStream("META-INF/resource_in_metainf.txt"); 5
      InputStream cz_pkg =
        Class.forName("javamodularity.secondresourcemodule.A")
             .getResourceAsStream("resource_in_package2.txt"); 6

      assert Stream.of(m_tl, m_class, m_meta)
                   .noneMatch(Objects::isNull);
      assert Stream.of(m_pkg, cz_pkg)
                   .allMatch(Objects::isNull);

   } catch (Exception e) {
      throw new RuntimeException(e);
```

1. A Module can be obtained through the boot layer. The corresponding ModuleLayer API is introduced in the next chapter.
2. Top-level resources from other modules can always be loaded.
3. A resource from a package in another module is encapsulated by default, so this returns null.
4. An exception is made for .class files; these can always be loaded from another module.
5. Because META-INF is never a valid package name, resources from that directory can be accessed as well.
6. While we can get a `Class<A>` instance by using Class::forName, loading the encapsulated resource through it will return null, just as in (3).

---

> 1. 可以通过引导层获取一个 Module。下一章将会介绍相应的 Module Layer API。
> 2. 可以随时加载来自其他模块的顶级资源。
> 3. 在默认情况下，来自其他模块的包中的资源是被封装的，所以此时返回 null。
> 4. .class 文件存在例外；这些文件可以始终从另一个模块加载的。
> 5. 因为 META-INF 不是一个有效的包名称，所以可以访问该目录中的资源。
> 6. 虽然通过使用 Class::forName 可以获取一个`Class<A>`实例，但就像第 3 步那样，通过该实例加载封装资源时将返回 null。

Resource encapsulation applies only to resources in packages. Class file resources (ending with .class) are the exception; they’re not encapsulated even if they’re inside a package. All other resources can be freely used by other modules. Even though you can doesn’t mean you should do this. Relying on resources from another module is not quite modular. It’s best to load resources only from within the same module. When you do need resources from another module, consider exposing the content of the resources through a method on an exported class or even as a service. This way, the dependency becomes explicit in the module descriptor.

> 资源封装仅适用于包中的资源。但类文件资源（以．class 结尾）是例外；即使它们在包内，也不会被封装。所有其他资源可以被其他模块自由使用，但并不意味着应该这么做。依赖来自另一个模块的资源并不是完全的模块化，最好只从同一个模块中加载资源。如果确实需要来自其他模块的资源，可以考虑通过使用导出类中的方法甚至作为服务来公开资源的内容。这样一来，依赖关系在模块描述符中就变得更加明确了。

#### EXPOSING RESOURCES IN PACKAGES 公开包中资源

You can expose encapsulated resources in packages to other modules by using open modules or open packages. These concepts are introduced in “Open Modules and Packages”. A resource that is in an open module or package can be loaded as if no resource encapsulation is applied.

> 通过使用开放式模块（open module）或开放式包（open package），可以向其他模块公开包中封装的资源。这些概念将在“开放式模块和包”一节（6.1.2 节）中介绍。加载位于开放式模块或包中的资源就好像没有对资源进行封装一样。

### 5.8.3 Working with ResourceBundles 使用 ResourceBundle 类

An example where the JDK itself heeds the advice of exporting resources through services is with ResourceBundles. ResourceBundles provide a mechanism for localization as part of the JDK. They’re essentially lists of key-value pairs, specific to a locale. You can implement the ResourceBundle interface yourself, or use, for example, a properties file. The latter is convenient, because there is default support for loading properties files following a predefined format, as shown in Example 5-9.

> JDK 自身通过服务公开了资源的一个示例就是 ResourceBundle, ResourceBundle 提供了一种机制将本地化作为 JDK 的一部分。从本质上讲，它们就是特定于语言环境的键值对列表。可以自己实现 ResourceBundle 接口或使用一个属性文件，后一种方法更为方便，因为默认情况下支持按照预定义的格式加载属性文件，如示例 5-9 所示。

Example 5-9. Properties files to be loaded by the ResourceBundle mechanism, where Translations is a user-defined basename

> 示例 5-9:ResourceBundle 机制加载的属性文件，其中 Translations 是用户定义的基础名称

```
Translations_en.properties
Translations_en_US.properties
Translations_nl.properties
```

You then load a bundle for a specific locale, and get a translation for a key:

> 然后加载针对特定语言环境的资源包，并对密钥进行转换：

```java
Locale locale = Locale.ENGLISH;
ResourceBundle translations =
   ResourceBundle.getBundle("javamodularity.resourcebundle.Translations",
                            locale);
String translation = translations.getString("modularity_key");
```

The translation properties files live in a package in a module. Traditionally, the getBundle implementation could scan the classpath for files to load. Then, the most appropriate properties file would be selected based on the locale, regardless of which JAR it comes from.

> 转换属性文件位于模块的一个包中。按照惯例，getBundle 实现可以扫描类路径以加载文件。然后，根据语言环境选择最合适的属性文件，而不管它来自哪个 JAR。

TIP

Explaining how ResourceBundle::getBundle selects the right bundle given a basename and locale is beyond the scope of this book. If you’re unfamiliar with this process, the ResourceBundle JavaDoc contains extensive information about how it loads the most specific file with fallback mechanisms. You will also find that an additional class-based format is supported besides properties files.

> 解释 ResourceBundle::getBundle 如何根据给定的基础名称和语言环境选择正确的资源包已经超出了本书的范围。如果不熟悉此过程，ResourceBundle JavaDoc 包含了大量有关如何使用回退机制加载特定文件的信息。此外，除了属性文件，还支持其他基于类的格式。

With modules, there is no classpath to scan. And you have already seen that resources in modules are encapsulated. Only files within the module calling getBundle are considered.

> 如果使用了模块，那么就不存在类路径扫描。前面已经讲过，模块中的资源是被封装的。只考虑调用 getBundle 的模块中的文件。

It’s desirable to put translations for different locales into separate modules. At the same time, opening these modules or packages (see “Exposing Resources in Packages”) has more consequences than just exposing resources. This is why a services-based mechanism for ResourceBundles is introduced in Java 9. It’s based on a new interface called ResourceBundleProvider, containing a single method with the signature ResourceBundle getBundle(String basename, Locale locale). Whenever a module wants to offer additional translations, it can create an implementation of this interface and register it as a service. The implementation then locates the correct resource inside the module and returns it, or returns null if no suitable translation for the given locale is in the module.

> 将不同语言环境的转换结果放入单独的模块是可取的。相比于仅公开资源，开放这些模块或包（请参阅“公开包中资源”内容）会取得更好的结果。这就是在 Java 9 中引入基于服务的 ResourceBundle 机制的原因。它基于一个名为 ResourceBundleProvider 的新接口，包含一个带有签名 ResourceBundle getBundle(String basename, Locale locale)的单一方法。每当一个模块想要提供额外的转换时，可以创建该接口的实现并将其注册为服务。然后，实现找到模块中正确的资源并返回它，如果模块中没有给定语言环境的合适转换，则返回 null。

Using this pattern, you can now extend the supported locales in an application by adding a module. As long as it registers a ResourceBundleProvider implementation, it’s automatically picked up by the module requesting the translations through ResourceBundle::getBundle. A complete example can be found in the code accompanying this chapter (➥ chapter5/resourcebundles).

> 在这种模式下，只需添加一个模块就可以扩展应用程序中支持的语言环境。只要注册了一个 ResourceBundleProvider 实现，请求语言环境转换的模块就可以通过 ResourceBundle::getBundle 自动获取该实现。在本章所附的代码中可以找到完整的示例（chapter5/resourcebundles）。
