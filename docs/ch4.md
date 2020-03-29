# 第 4 章 服务

> Chapter 4. Services

In this chapter, you will learn how to use services, an important feature for creating modular codebases. After learning the basics of providing and consuming services, we will apply them to EasyText, making it more extensible.

> 本章将学习如何使用服务（service），其是用来创建模块化代码库的一项重要功能。在学习了提供和消费服务的基本知识之后，会将所学的知识运用到 EasyText 应用程序中，使其更具有可扩展性。

## 4.1 Factory Pattern 工厂模式

In the previous chapter, you saw that encapsulation alone doesn’t get us very far when we want to create truly decoupled modules. If we still write

> 在前一章中已经看到，当需要创建真正的解耦模块时，仅仅依靠封装是不够的。如果仍然编写以下代码：

```java
MyInterface i = new MyImpl();
```

every time we need to use an implementation class, it means the implementation class must be exported. Consequently, strong coupling still remains between the consumer and provider of the implementation: the consumer requires the provider module directly to use its exported implementation class. Changes in the implementation directly affect all consumers. As you will soon see, services are an excellent solution to this problem. But before diving into services, let’s see if we can fix this problem by using an existing pattern, building on our knowledge of the module system so far.

> 每次需要使用一个实现类，就意味着必须导出实现类。因此，实现类的消费者和提供者之间仍然存在强耦合：消费者要求提供者模块直接使用其导出的实现类。这样一来，实现过程中的任何更改都会直接影响到所有消费者。很快你就会看到，服务是解决这个问题的最佳方案。但在学习服务之前，先来看一下，是否可以根据目前对模块系统的了解使用现有的模式来解决这个问题。

The factory pattern is a well-known creational design pattern that seems to address the very problem we are dealing with. Its goal is to decouple a consumer of objects from the instantiation of specific classes. Many variations on the factory pattern have emerged since it was first described in the iconic Gang of Four Design Patterns book by Gamma et al. (Addison-Wesley). Let’s try to implement a simple variation of this pattern and see how far it gets us with decoupling modules.

> 工厂模式（factory pattern）是一个著名的创建型设计模式（creational design pattern），似乎解决了目前所面临的问题，其目标是将对象的消费者与特定类的实例化进行解耦。自从 Gamma 等人首次在《Gang of Four Design Patterns》（Addison-Wesley 出版社出版）一书中描述了工厂模式以来，该模式已经发生了许多变化。接下来尝试实现一下该模式的简单变体，并看看它如何实现模块解耦。

We will use the EasyText application again to illustrate the example, by implementing a factory for Analyzer instances. Getting an implementation for a given algorithm name is quite straightforward, as shown in Example 4-1.

> 下面再次使用 EasyText 应用程序作为演示示例，为 Analyzer 实例实现一个工厂类。获取给定算法名称的实现是非常简单的，如示例 4-1 所示。

Example 4-1. A factory class for Analyzer instances (➥ chapter4/easytext-factory)

> 示例 4-1:Analyzer 实例的工厂类（chapter4/easytext-factory）

```java
public class AnalyzerFactory {

   public static List<String> getSupportedAnalyses() {
     return List.of(FleschKincaid.NAME, Coleman.NAME);
   }

   public static Analyzer getAnalyzer(String name) {
      switch (name) {
         case FleschKincaid.NAME: return new FleschKincaid();
         case Coleman.NAME: return new Coleman();
         default: throw new IllegalArgumentException("No such analyzer!");
      }
   }

}
```

You can retrieve a list of supported algorithms from the factory, and request an Analyzer instance for an algorithm name. Callers of the AnalyzerFactory are now oblivious to any underlying implementation classes for the analyzers.

> 可以从工厂类获取所支持的算法列表，并请求算法名称对应的 Analyzer 实例。现在，AnalyzerFactory 的调用者完全不知道分析器的任何底层实现类。

But where do we place this factory? For one, the factory itself still needs access to multiple analysis modules with their implementation classes. Otherwise, the instantiation of the various implementation classes in getAnalyzer would not be possible. We could put the factory in the API module, but then the API module would have a compile-time dependency on all implementation modules, which is unsatisfactory. An API should not be tightly coupled to its implementations.

> 应该将该工厂模块放置在哪里呢？一方面，工厂模块本身仍然需要访问带有实现类的多个分析模块，否则，就无法在 getAnalyzer 中实例化各种实现类。可以将工厂模块放在 API 模块中，但这样一来，API 模块将与所有的实现模块之间存在编译时依赖关系，这不符合要求。API 不应与其实现紧密耦合。

Let’s put the factory in its own module for now, as shown in Figure 4-1.

> 接下来，将工厂模块暂时放置到自己的模块中，如图 4-1 所示。

Factory
Figure 4-1. The factory module decouples the frontends from the analysis implementation modules. There is no requires relation from the frontend modules to the analysis implementation modules.

Now, the frontend modules know about only the API and the factory:

> 现在，前端模块仅知道 API 和工厂模块：

```java
module easytext.cli {
   requires easytext.analysis.api;
   requires easytext.analysis.factory;
}
```

Getting an Analyzer instance becomes trivial:

> 获取 Analyzer 示例变得非常简单：

```java
Analyzer analyzer = AnalyzerFactory.getAnalyzer("Flesch-Kincaid");
```

Did we gain anything with this factory approach, besides increased complexity?

> 除了增加了复杂性之外，使用工厂方法可以获得什么好处吗？

On the one hand, yes, the frontend modules are now blissfully unaware of the analysis modules and implementation classes. There is no direct requires relation anymore between the consumer and providers of analyses. Frontend modules can be compiled independently from analysis implementation modules. When the factory offers additional analyses, the frontends will happily use them without any modification. (Remember, with AnalyzerFactory::getSupportedAnalyses, they can discover algorithm names to request instances.)

> 当然有好处。一方面，前端模块现在完全不需要了解分析模块以及实现类。消费者和分析提供者之间不存在直接的 requires 关系。前端模块可以独立于分析实现模块进行编译。当工厂提供了额外的分析时，前端可以非常容易地使用它们，而无须进行任何修改。（请记住，通过使用 AnalyzerFactory::getSupportedAnalyses，可以根据算法名称来请求实例。）

On the other hand, the same tight coupling issues are still present at the factory module level and below. Whenever a new analysis module comes along, the factory needs to get a dependency on it and expand the getAnalyzer implementation. And the analysis modules still need to export their implementation classes for the factory to use. They could do so through a qualified export (as discussed in “Qualified Exports”) toward the factory module to limit the scope of exposure. But that presumes the analysis modules know about the factory module, which is another form of unwanted coupling.

> 另一方面，在工厂模块以及以下级别，仍然存在相同的紧耦合问题。每当出现一个新的分析模块时，工厂就需要依赖它来扩展 getAnalyzer 实现。分析模块仍然需要导出其实现类，以便工厂使用。可以通过对工厂模块的限制导出（如 2.6 节中所述）来限制暴露范围，但是这样做的前提是分析模块“了解”工厂模块，而这是另一种不必要的耦合形式。

So the factory pattern provides only a partial solution. We are running into a fundamental limitation of what you can do with modules through requires and exports relations. Programming to interfaces is all well and good, but we have to sacrifice encapsulation to create instances. Fortunately, there is a solution in the Java module system. In the next section, we’ll explore how services provide a way out of this tough spot.

> 所以工厂模式只提供了部分解决方案。如果只是使用 requires 和 exports 关系，那么消费模块可以完成的事情是有限制的。虽然对接口进行编程非常好，但是我们必须牺牲封装来创建实例。幸运的是，Java 模块系统中提供了一个解决方案。在下一节中，将探讨服务是如何提供方法来解决这个难题的。

#### DEPENDENCY INJECTION 依赖注入

Many Java applications use a dependency injection (DI) framework to solve the issue of programming to interfaces without tight coupling to implementation classes. A DI framework takes care of creating implementation instances based on metadata such as annotations or (more traditionally) XML descriptors. The DI framework then injects instances into code depending on the defined interfaces. This principle is more generally known as Inversion of Control (IoC) because the framework is in control of instantiating classes rather than the application code itself.

> 许多 Java 应用程序使用了依赖注入（Dependency Injection, DI）框架来解决对接口进行编程，同时与实现类松耦合的问题。DI 框架负责根据诸如注释或（更传统的）XML 描述符之类的元数据创建实现实例。然后，DI 框架根据定义的接口将实例注入代码中。这个原则通常被称为反转控制（Inversion of Control, IoC），因为框架控制的是类的实例化，而不是应用程序代码本身。

DI is an excellent way to decouple code, but there are several caveats in the context of modules. This chapter focuses on the module system’s solution for decoupling in the form of services. Services provide IoC through other means than dependency injection. Later, in “Dependency Injection”, you will see how to use DI frameworks in combination with modules for another way to achieve the same level of decoupling.

> DI 是解耦代码的绝佳方法，但在模块的上下文中有几个需要注意的事项。本章重点介绍了以服务形式进行解耦的模块系统解决方案。服务通过其他方式提供了 IoC（而不是通过依赖注入）。稍后，在 6.1.3 节中，将会学习如何将 DI 框架与模块结合使用，以实现相同级别的解耦。

## 4.2 Services for Implementation Hiding 用于实现隐藏的服务

We tried hiding the implementation classes by using the factory pattern and succeeded only partially. The main problem is that the factory still has to know about all available implementations at compile-time, and the implementation classes must be exported. A solution similar to traditional classpath scanning to discover implementations is not going to solve this, because this would still require readability to all implementation classes in all modules. It still wouldn’t be possible to extend the application with another implementation (new algorithms in the case of EasyText) without changing code and recompiling. This doesn’t sound like seamless extensibility at all!

> 前面尝试使用工厂模式来隐藏实现类，但仅取得了部分成功。主要的问题是工厂模块在编译时仍然必须知道所有可用的实现，并且必须导出实现类。这种解决方案与传统的通过扫描发现实现的类路径方法类似，无法解决目前的问题，因为仍然需要读取所有模块中的所有实现类。在不更改代码和重新编译的情况下，想要使用另一个实现（类似 EasyText 的新算法）来扩展应用程序也是不可能的。这听起来根本不像是无缝扩展！

The decoupling story can be improved a lot by the services mechanism in the Java module system. Using services, we can truly just share public interfaces, and strongly encapsulate implementation code in packages that are not exported. Keep in mind, using services in the module system is completely optional, unlike strong encapsulation (with explicit exports) and explicit dependencies (with requires). You don’t have to use services, but they offer a compelling way of decoupling modules.

> 解耦过程可以通过 Java 模块系统中的服务机制进行改进。使用服务，可以真正地共享公共接口，并将实现代码强封装到未导出的包中。请记住，与强封装（带有显式导出）和显式依赖（根据需要）不同，是否使用模块系统中的服务是完全可选的。可以不必使用服务，但服务却提供了一种令人信服的解耦模块的方法。

Services are expressed both in module descriptors and in code by using the ServiceLoader API. In that sense, using services is intrusive: you need to design your application to use them. As discussed in “Dependency Injection”, there are alternative ways to achieve inversion of control besides using services. In the remainder of this chapter, you will learn how services bring better decoupling and extensibility.

> 通过使用 ServiceLoader API 可在模块描述符和代码中表示服务。从这个意义上讲，使用服务是具有侵扰性的：需要设计一下应用程序才能使用它们。如 6.1.3 节中所述，除了使用服务之外，还可以使用其他方法来实现控制反转。在本章的其余部分，将学习服务如何带来更好的解耦和可扩展性。

We will refactor the EasyText application to start using services. Our goal is to have several modules provide an analysis implementation. The frontend modules can consume those analysis implementations without knowing the provider modules at compile-time.

> 接下来重新构建 EasyText 应用程序，以便使用服务，目标是让几个模块提供分析实现。前端模块可以在编译时不知道提供者模块的情况下使用这些分析实现。

### 4.2.1 Providing Services 提供服务

Exposing service implementations to another module without exporting implementation classes is not possible without special support from the module system. The Java module system allows for a declarative description of providing and consuming services in module-info.java.

> 如果没有来自模块系统的特殊支持，想要将服务实现暴露给另一个模块而又不导出实现类是不可能的。Java 模块系统允许在 module-info.java 中添加提供和消费服务的声明性描述。

In the EasyText code, we already defined the Analyzer interface, which will be our service interface type. The interface is exported by the easytext.analysis.api module, which is strictly an API-only module.

> 在 EasyText 代码中，已经定义了 Analyzer 接口，这就是服务接口类型。该接口由 easytext.analysis.api 模块（从严格意义上讲，该模块仅是一个 API 模块）导出。

```java
package javamodularity.easytext.analysis.api;

import java.util.List;

public interface Analyzer {

   String getName();

   double analyze(List<List<String>> text);

}
```

Typically, the service type is an interface, as is the case here. However, it could also be an abstract or even concrete class; there is no inherent technical limitation. Also, the Analyzer type is meant to be used by service consumers directly. It’s also possible to expose a service type that acts like a factory or proxy. If, for example, Analyzer instances would be expensive to instantiate, or extra steps or arguments are required for initialization, the service type could be more akin to the AnalyzerFactory. This approach allows the consumer to be more in control of the instantiation.

> 通常，服务类型是一个接口（就像本示例所示的那样），但也可以是一个抽象甚至具体的类，对此没有固有的技术限制。此外，Analyzer 类型可以直接由服务消费者使用，也可以公开类似于工厂或代理的服务类型。例如，如果 Analyzer 的实例化成本非常昂贵，或者需要额外的步骤或参数进行初始化，那么服务类型可能更加类似于 AnalyzerFactory。这种方法允许消费者对实例化进行更多的控制。

Now let’s refactor our first new analyzer implementation, the Coleman-Liau algorithm (provided by the easytext.algorithm.coleman module) to a service provider. This requires only a change to module-info.java, as shown in Example 4-2.

> 现在，重新构建第一个新的分析器实现，将 Coleman-Liau 算法（由 easytext.algorithm.coleman 模块提供）应用于服务提供者。只需要更改 module-info.java 即可，如示例 4-2 所示。

Example 4-2. Module descriptor providing an Analyzer service (➥ chapter4/easytext-services)

> 示例 4-2：提供了 Analyzer 服务的模块描述符（chapter4/easytext-services）

```java
module easytext.analysis.coleman {

   requires easytext.analysis.api;

   provides javamodularity.easytext.analysis.api.Analyzer
       with javamodularity.easytext.analysis.coleman.ColemanAnalyzer;

}
```

The provides with syntax declares that this module provides an implementation of the Analyzer interface with the ColemanAnalyzer as an implementation class. Both the service type (after provides) and the implementation class (after with) must be fully qualified type names. Most important, the package containing the ColemanAnalyzer implementation class is not exported from this provider module.

> provides with 语法声明该模块提供了 Analyzer 接口的一个实现（使用 ColemanAnalyzer 作为实现类）。服务类型（provides 之后）和实现类（with 之后）必须是完全限定类型名称。最重要的是，包含 ColemanAnalyzer 实现类的包并不从此提供者模块中导出。

This construct works only when the module declaring the provides has access to both the service type and the implementation class. Usually this means that an interface, Analyzer in this example, is either part of the module or is exported by another module that is required. The implementation class is typically part of the provider module, in an encapsulated (nonexported) package.

> 只有当模块声明 provides 可以访问服务类型和实现类时，该结构才会起作用。通常这意味着一个接口（此示例为 Analyzer）要么是模块的一部分，要么是由所需的另一个模块导出。一般来说，实现类是提供者模块的一部分，位于一个封装（未导出）包中。

When you use nonexistent or inaccessible types in the provides clause, the module descriptor won’t compile, and a compiler error is generated. The implementation class used in the with part of the declaration is normally not exported. After all, the whole point of services is to hide implementation details.

> 当在 provides 子句中使用了不存在或者无法访问的类型时，就无法对模块描述符进行编译，从而产生编译器错误。声明的 with 部分中所使用的实现类通常不导出。毕竟，使用服务的目的是隐藏实现细节。

No code changes are necessary to the service type or implementation class to provide it as a service. Besides this module-info.java declaration, nothing needs to be done. Service implementations are plain Java classes. There are no special annotations to use, no APIs to implement.

> 不需要对服务类型或者实现类进行任何代码修改就可以将其作为服务提供。除了这个 module-info.java 声明，不需要做其他任何事情了。服务实现是普通的 Java 类，没有使用特殊的注释，也没有实现任何 API。

Services allow a module to provide implementations to other modules without exporting the concrete implementation class. The module system has special privileges to reach into the provider module to instantiate the nonexported implementation class on behalf of the consumer. This means consumers of the service can use instances of this implementation class, without having access to it directly. Also, a service consumer doesn’t know which module provided an implementation, nor does it need to. Because the only shared type between provider and consumer is the service type (most often an interface), there is true decoupling.

> 服务允许一个模块向其他模块提供实现，而不导出具体的实现类。模块系统具有访问提供者模块的特殊权限，从而代表消费者实例化非导出的实现类。这意味着服务的消费者可以使用此实现类的实例，而无须直接访问该实现类。此外，服务消费者不知道哪个模块提供了所需的实现，也不需要知道。因为提供者和消费者之间唯一的共享类型是服务类型（通常是一个接口），所以实现了真正的解耦。

Now that we’re done providing our first service, we can repeat this process for the other Analyzer implementations, and we’re halfway done. Again, note that these service-providing modules don’t export any packages. Having a module without exports may seem a bit counterintuitive at first. Nevertheless, these analysis implementation modules contribute useful functionality through the services mechanism at run-time, encapsulating their implementation details at compile-time.

> 现在已经完成了第一个服务，可以针对其他 Analyzer 实现重复这个过程，且目前完成了一半的工作。同样，请注意，这些提供服务的模块没有导出任何包。乍一看，没有任何导出的模块似乎有点违反常理。然而，这些分析实现模块通过运行时的服务机制提供了有用的功能，而在编译时封装了实现细节。

The other half of the refactoring is consuming the services. Let’s refactor the CLI module to use the Analyzer services.

> 剩下的另一半工作是消费服务。接下来重新构建 CLI 模块，以便使用 Analyzer 服务。

### 4.2.2 Consuming Services 消费服务

Providing services is useful if other modules can consume them. Consuming a service in the Java module system requires two steps. The first step is adding a uses clause to module-info.java in the CLI module:

> 如果其他模块可以消费服务，那么提供服务是非常有用的。在 Java 模块系统中，消费服务需要两个步骤。第一步是向 CLI 模块的 module-info.java 中添加 uses 子句：

```java
module easytext.cli {
   requires easytext.analysis.api;

   uses javamodularity.easytext.analysis.api.Analyzer;
}
```

The uses clause instructs the ServiceLoader, which you will see in a moment, that this module wants to use implementations of Analyzer. The ServiceLoader then makes Analyzer instances available to the module.

> uses 子句告知 ServiceLoader（稍后将介绍该 API）该模块想要消费 Analyzer 的实现，然后 ServiceLoader 使 Analyzer 实例可用于模块。

The uses clause does not call for an Analyzer implementation to be available during compile-time. After all, a service implementation could be provided by a module that we don’t have on the module path at compile-time. Services provide extensibility exactly because providers and consumers are bound only at run-time. Compilation will not fail when no service providers are found. The service type (Analyzer), on the other hand, must be accessible at compile-time—hence the requires easytext.analysis.api clause in the module descriptor.

> uses 子句并不要求 Analyzer 实现在编译期间可用，毕竟服务实现可以由在编译时并不在模块路径上的模块提供。服务之所以提供了可扩展性，正是因为提供者和消费者仅在运行时才被绑定在一起。即使没有找到服务提供者，编译也不会失败。另一方面，服务类型（Analyzer）必须在编译时可访问，因此在模块描述符中需要使用 requires easytext.analysis.api 子句。

A uses clause also doesn’t guarantee there will be providers during run-time. The application will start successfully without any providers of services. This means there can be zero or more providers available at run-time, and our code has to deal with this.

> uses 子句同样无法保证在运行时存在提供者。即使没有任何服务提供者，应用程序也会成功启动。这意味着在运行时可能会提供零个或多个提供者，因此代码必须能够处理相应的情况。

Now that the module has declared that it wants to use Analyzer implementations, we can start writing code that uses the service. Consuming services happens through the ServiceLoader API. The ServiceLoader API itself has been around since Java 6 already. Although it is widely used in the JDK, few Java developers know or use ServiceLoader. “ServiceLoader Before Java 9” provides more historical background.

> 现在，模块已经声明了要消费 Analyzer 实现，接下来可以开始编写使用该服务的代码。可以通过 ServiceLoader API 消费服务。从 Java 6 开始，ServiceLoader API 就已经存在了。虽然它在 JDK 中被广泛使用，但是很少有 Java 开发人员知道或使用 ServiceLoader，随后“Java 9 之前的 ServiceLoader”内容提供了更多的历史背景。

The ServiceLoader API is repurposed in the Java module system to work with modules, and is an important programming construct when working with the Java module system. Let’s look at an example; see Example 4-3.

> Java 模块系统对 ServiceLoader API 进行了略微的修改，以便使用模块，并且其是使用 Java 模块系统时一个非常重要的编程结构。接下来看一个示例（如示例 4-3 所示）。

Example 4-3. Main.java

> 示例 4-3:Main.java

```java
Iterable<Analyzer> analyzers = ServiceLoader.load(Analyzer.class); 1

for (Analyzer analyzer: analyzers) { 2
   System.out.println(analyzer.getName() + ": " + analyzer.analyze(sentences));
}
```

> 1 Initialize a ServiceLoader for services of type Analyzer.
> 2 Iterate over the instances and invoke the analyze method.

The ServiceLoader::load method returns a ServiceLoader instance that also conveniently implements Iterable. When you iterate over it as in the example, instances are created for all the provider types that have been discovered for the requested Analyzer interface. Note that we get only the actual instances here, with no additional information on which modules provided them.

> ServiceLoader::load 方法返回一个实现了 Iterable 的 ServiceLoader 实例。当在示例中进行遍历时，将为针对所请求的 Analyzer 接口所发现的所有提供者类型创建实例。请注意，此时仅获取实际的实例，而没有获取关于哪些模块提供了这些实例的其他信息。

After iterating over the services, we can use them like any other Java object. In fact, they are plain Java objects, except they are instantiated by ServiceLoader for us. Being just normal Java instances, there is zero overhead when invoking a service. Invoking a method on a service is just a direct method call; there are no proxies or other indirections that decrease performance.

> 遍历服务后，就可以像任何其他 Java 对象一样使用它们了。实际上，它们都是普通的 Java 对象，只不过是由 ServiceLoader 代我们实例化而已。作为普通的 Java 实例，调用服务的开销为零。调用服务上的方法只是一个直接的方法调用，没有代理或其他降低性能的间接方法。

With these changes, we have refactored our EasyText code from a partially decoupled factory structure to a fully modular and extensible setup as shown in Figure 4-2.

> 完成了上述更改之后，将 EasyText 代码从部分解耦的工厂结构重构为完全模块化和可扩展的结构，如图 4-2 所示。

Structure of Easytext using ServiceLoader for extensibilitys
Figure 4-2. Structure of EasyText using ServiceLoader for extensibility

The code is fully decoupled because the CLI module doesn’t need to know anything about modules providing Analyzer implementations. The application is easily extensible because we can add a new Analyzer implementation by simply adding a new provider module to the module path. Any services provided by these additional modules are picked up automatically through the ServiceLoader service discovery. No code changes or recompilation are necessary. Arguably the best part is that the code is clean. Programming with services is just as simple as writing plain Java code (that’s because it is plain Java code), but the impact on architecture and design is quite positive.

> 此时代码完全解耦，因为 CLI 模块不需要知道关于提供 Analyzer 实现的模块的任何信息。该应用程序易于扩展，因为只需向模块路径添加新的提供者模块就可以添加新的 Analyzer 实现。由这些附加模块所提供的任何服务都将通过 ServiceLoader 服务发现程序自动获取，无须更改代码或重新编译。可以说，示例中最妙的是代码非常干净。虽然使用服务进行编程就像编写普通的 Java 代码一样简单（这是因为服务本身也是普通的 Java 代码），但对架构和设计的影响却是非常积极的。

#### SERVICELOADER BEFORE JAVA 9 Java 9 之前的 ServiceLoader

ServiceLoader has been around since Java 6. It was designed to make Java more pluggable, and is used in the JDK in several places. It never received widespread use in application development, although several frameworks and libraries rely on the old ServiceLoader besides the JDK itself.

> 从 Java 6 开始，ServiceLoader 就已经存在了。其设计目的是让 Java 具有更好的可插拔性，并在 JDK 的多个地方使用。尽管除了 JDK 本身之外，还有几个框架和库依赖于旧的 ServiceLoader，但它在应用程序开发中从未得到广泛的应用。

In principle, ServiceLoader had the same goal as services in the Java module system, but the mechanics were different, and there was no true strong encapsulation possible before modules. To register a provider, a file following a specific naming scheme must be created in the META-INF folder of a JAR file. For example, to provide an Analyzer implementation, a file named META-INF/services/javamodularity.easytext.analysis.api.Analyzer must be created. The contents of the file should be a single line representing the fully qualified name of the implementation class—for example, javamodularity.easytext.analysis.coleman.ColemanAnalyzer. Because these files are just text files, it is easy to make mistakes that the compiler won’t catch.

> 从原则上，ServiceLoader 与 Java 模块系统中的服务具有相同的目标，但机制却是不同的，且在模块出现之前没有真正意义上的强封装。为了注册一个提供者，必须在 JAR 文件的 META-INF 文件夹中创建遵循特定命名方案的文件。例如，为了提供一个 Analyzer 实现，必须创建一个名为 META-INF/services/javamodularity.easytext.analysis.api.Analyzer 的文件。文件的内容应该是表示实现类的完全限定名称的单行，如 javamodularity.easytext.analysis.coleman.ColemanAnalyzer。因为这些文件只是文本文件，所以很容易造成编译器无法捕获的错误。

With the Java module system, it’s also possible to consume services provided the “old” way, as long as the service type is accessible to the consuming module.

> 即使使用了 Java 模块系统，也可以使用提供了“旧”方式的服务，只要消费模块可以访问服务类型即可。

You have seen that services provide an easy way to achieve decoupling. Think of services as the cornerstone of modular development. Although strong mechanisms for defining module boundaries are the first step toward modular design, services are required to create and use strictly decoupled modules.

> 如前所示，服务提供了一种简单的方法来实现解耦，可以将服务视为模块化开发的基石。尽管确定模块边界的强大机制是迈向模块化设计的第一步，但是仍需要通过服务来创建和使用严格解耦的模块。

### 4.2.3 Service Life Cycle 服务生命周期

If the ServiceLoader is responsible for creating instances of provided services, it’s important to know how this works exactly. In Example 4-3, the iteration caused the Analyzer implementation classes to be instantiated. ServiceLoader works lazily, meaning the ServiceLoader::load call doesn’t immediately instantiate all known provider implementation classes.

> 如果 ServiceLoader 负责创建所提供服务的实例，那么了解它的工作方式就显得非常重要了。在示例 4-3 中，遍历导致 Analyzer 实现类被实例化。ServiceLoader 工作“比较懒惰”，这意味着 ServiceLoader::load 调用不会立即实例化所有已知的提供者实现类。

A new ServiceLoader is instantiated every time you call ServiceLoader::load. Such a new ServiceLoader in turn reinstantiates provider classes when they are requested. Requesting services from an existing ServiceLoader instance returns cached instances of provider classes.

> 每次调用 ServiceLoader::load 时，都会实例化一个新的 ServiceLoader。当请求提供者类时，新的 ServiceLoader 会重新实例化它们。从现有的 ServiceLoader 实例请求服务将返回提供者类的缓存实例。

This is demonstrated by the following code:

> 如下面代码所示：

```java
ServiceLoader<Analyzer> first = ServiceLoader.load(Analyzer.class);
System.out.println("Using the first analyzers");
for (Analyzer analyzer: first) { 1
  System.out.println(analyzer.hashCode());
}

Iterable<Analyzer> second = ServiceLoader.load(Analyzer.class);
System.out.println("Using the second analyzers");
for (Analyzer analyzer: second) { 2
  System.out.println(analyzer.hashCode());
}

System.out.println("Using the first analyzers again, hashCode is the same");
for (Analyzer analyzer: first) { 3
  System.out.println(analyzer.hashCode());
}

first.reload(); 4
System.out.println("Reloading the first analyzers, hashCode is different");
for (Analyzer analyzer: first) {
  System.out.println(analyzer.hashCode());
}
```

1 Iterating over first, ServiceLoader instantiates Analyzer implementations.
2 A new ServiceLoader, second, will instantiate its own, fresh, Analyzer implementations. It returns different instances than first.
3 The originally instantiated services are returned from first when iterating again, as they are cached by the first ServiceLoader instance.
4 After reload, the original first ServiceLoader provides fresh instances.

---

> 1. 遍历 first, ServiceLoader 实例化 Analyzer 实现。
> 2. 新的 ServiceLoader——second 将实例化自己的 Analyzer 实现，并返回不同于 first 的实例。
> 3. 当再次遍历时，从 first 返回最初实例化的服务，因为第一个 ServiceLoader 实例缓存了这些服务。
> 4. reload 之后，最初的 ServiceLoader——first 提供了新的实例。

This code outputs something like the following (actual hashCodes will vary, of course):

> 上述代码输出了如下所示内容（当然，实际输出的散列码可能会有所不同）：

```log
Using the first analyzers
1379435698
Using the second analyzers
876563773
Using the first analyzers again, hashCode is the same
1379435698
Reloading the first analyzers, hashCode is different
87765719
```

Because every invocation of ServiceLoader::load leads to new service instances, different modules using the same service will all have their own instance. This is something to remember when working with services that contain state. The state is, without other provisions, not shared between usages across different ServiceLoaders for the same service type. There is no singleton service instance, unlike what is typically the case in dependency injection frameworks.

> 因为每次调用 ServiceLoader::load 都会生成新的服务实例，所以使用相同服务的不同模块都拥有自己的实例。在使用包含状态的服务时，需要铭记这一点。对于相同的服务类型，如何没有其他规定，那么状态在不同的 ServiceLoader 用法之间是不能共享的。与依赖注入框架中的典型情况不同，没有单独的服务实例。

### 4.2.4 Service Provider Methods 服务提供者方法

Service instances can be created in two ways. Either the service implementation class must have a public no-arg constructor, or a static provider method can be used. It’s not always desirable for a service implementation class to have a public no-arg constructor. In cases where more information needs to be passed to the constructor, a static provider method is the better option. Or, you may want to expose an existing class without a no-arg constructor as a service.

> 可以通过两种方式创建服务实例，或者服务实现类必须具有公共的无参数构造函数，或者使用静态提供者方法。一个服务实现类并不总是需要有一个公共的无参数构造函数。当需要向构造函数传递更多信息时，静态提供者方法是更好的选择。或者，也可以公开一个不带有无参数构造函数的现有类作为服务。

A provider method is a public static no-arg method called provider, where the return type is the service type. It must return a service instance of the correct type (or a subtype). How the service is instantiated in this method is completely up to the provider implementation. Possibly a singleton is cached and returned, or it just instantiates a new service instance for each call.

> 提供者方法是一个名为 provider 的 public static 无参数方法，其返回类型是服务类型。它必须返回正确类型（或子类型）的服务实例。在提供者方法中如何实例化服务完全取决于 provider 的实现，可能缓存并返回单一实例，或者为每个调用实例化一个新的服务实例。

When using the provider method approach, the provides .. with clause refers to the class containing the provider method after with. This can very well be the service implementation class itself, but it can also be another class. A class appearing after with must have either a provider method or a public no-arg constructor. If there is no static provider method, the class is assumed to be the service implementation itself and must have a public no-arg constructor. The compiler will complain when this is not the case.

> 当使用提供者方法时，provides…with 子句指的是包含提供者方法的类（with 之后）。这个类很可能是服务实现类本身，也可能是另一个类。出现在 with 后面的类必须拥有一个提供者方法或一个公共的无参数构造函数。如果没有提供静态提供者方法，则该类被假定为服务实现类本身，并且必须有一个公共的无参数构造函数。否则编译器就会产生“抱怨”。

Let’s look at a provider method example (Example 4-4). We’ll use another Analyzer implementation for this, just to highlight the use of a provider method.

> 接下来看一个提供者方法示例（见示例 4-4）。为了突出提供者方法的使用，将使用另一个 Analyzer 实现。

Example 4-4. ExampleProviderMethod.java (➥ chapter4/providers/provider.method.example)

> 示例 4-4:ExampleProviderMethod.java（chapter4/providers/provider.method.example）

```java
package javamodularity.providers.method;

import java.util.List;
import javamodularity.easytext.analysis.api.Analyzer;

public class ExampleProviderMethod implements Analyzer {

  private String name;

  ExampleProviderMethod(String name) {
    this.name = name;
  }

  @Override
  public String getName() {
    return name;
  }

  @Override
  public double analyze(List<List<String>> sentences) {
     return 0;
  }

  public static ExampleProviderMethod provider() {
    return new ExampleProviderMethod("Analyzer created by static method");
  }
}
```

The Analyzer implementation is fairly useless, but it does show the usage of a provider method. The module-info.java for this example would be exactly the same as we have seen so far; the Java module system will figure out the right way to instantiate the class. In this example, the provider method is part of the implementation class. Alternatively, we can place the provider method in another class, which then serves as a factory for the service implementation. Example 4-5 shows this approach.

> 虽然该 Analyzer 实现没有什么用处，但演示了提供者方法的用法。示例中的 module-info.java 与我们到目前为止所看到的完全一样；Java 模块系统将会找出正确的方法来实例化类。在本示例中，provider 方法是实现类的一部分。或者也可以将 provider 方法放在另一个类中，然后以该类作为服务实现的工厂类。示例 4-5 显示了这种方法。

Example 4-5. ExampleProviderFactory.java (➥ chapter4/providers/provider.factory.example)

> 示例 4-5:ExampleProviderFactory.java（chapter4/providers/provider.factory.example）

```java
package javamodularity.providers.factory;

public class ExampleProviderFactory {
  public static ExampleProvider provider() {
    return new ExampleProvider("Analyzer created by factory");
  }
}
```

Now we do have to change module-info.java to reflect this change. The provides .. with must now point to the class containing the static provider method as shown in Example 4-6.

> 现在，必须对 module-info.java 进行修改，从而反映上面所做的更改。provides…with 必须指向包含该静态提供者方法的类，如示例 4-6 所示。

Example 4-6. module-info.java (➥ chapter4/providers/provider.factory.example)

> 示例 4-6:module-info.java（chapter4/provides/provider.factory.examples）

```java
module provider.factory.example {
    requires easytext.analysis.api;

    provides javamodularity.easytext.analysis.api.Analyzer
        with javamodularity.providers.factory.ExampleProviderFactory;
}
```

TIP

ServiceLoader can instantiate a service only if the provider class is public. Only the provider class itself needs to be public; our second example shows that the implementation can be package-private as long as the provider class is public.

> 只有在提供者类是公共的情况下，ServiceLoader 才可以实例化一个服务。只有提供者类本身需要是公共的；第二个示例表明只要提供者类是公共的，实现类就可以是包私有的。

Note that in all cases the exposed service type Analyzer remains unchanged. From the perspective of the consumer, it makes no difference how the service is instantiated. A static provider method offers more flexibility on the provider side. In many cases, a public no-arg constructor on the service implementation class suffices.

> 请注意，在所有情况下，公开的服务类型 Analyzer 是保持不变的。从消费者的角度来看，服务的实例化并没有什么不同，但静态提供者方法提供了更大的灵活性。在大多数情况下，服务实现类上的公共的无参数构造函数已经足够了。

Services in the module system don’t offer a shutdown or service de-registration mechanism. The death of a service is implicit, through garbage collection. Garbage collection behaves the same for service instances as for any other objects in Java. Once there are no hard references to the object anymore, it can be garbage collected.

> 模块系统中的服务没有提供关闭或服务注销机制。一个服务的死亡是隐式的，是通过垃圾回收机制完成的。对于服务实例来说，对其实施垃圾回收的行为与 Java 中的任何其他对象一样。一旦对象没有被强引用（hard reference），那么它就会被作为垃圾回收。

## 4.3 Factory Pattern Revisited 工厂模式回顾

Consumer modules can obtain services through the ServiceLoader API. You can employ a useful pattern to avoid the use of this API in consumers, if desired. Instead, you can offer an API to consumers similar to the factory example in the beginning of this chapter. It’s based on the ability to have static methods in interfaces as of Java 8.

> 消费者模块可以通过 ServiceLoader API 获取服务。如果需要，也可以使用有用的模式来避免使用此 API。或者，可以像本章开头的工厂示例中那样向消费者提供一个 API，这基于从 Java 8 开始可以在接口中使用静态方法。

The service type itself is extended with a static method (factory method) that does the ServiceLoader lookup, as shown in Example 4-7.

> 服务类型本身也是使用执行 ServiceLoader 查找的静态方法（工厂方法）进行扩展的，如示例 4-7 所示。

Example 4-7. Provide a factory method on the service interface (➥ chapter4/easytext-services-factory)

> 示例 4-7：在服务接口上提供一个工厂方法（chapter4/easytext-services-factory）

```java
public interface Analyzer {

   String getName();

   double analyze(List<List<String>> text);

   static Iterable<Analyzer> getAnalyzers() {
     return ServiceLoader.load(Analyzer.class); 1
   }

}
```

1 Lookup is now done inside the service type itself.

> 1. 此时，查询在服务类型内部完成。

Because the ServiceLoader lookup is done in Analyzer in the API module, its module descriptor must express the uses constraint:

> 由于 ServiceLoader 查找是在 API 模块的 Analyzer 内完成的，因此其模块描述符必须使用 uses 约束：

```java
module easytext.analysis.api {
   exports javamodularity.easytext.analysis.api;

   uses javamodularity.easytext.analysis.api.Analyzer;
}
```

Now, the API module both exports the interface and uses implementations of the Analyzer interface. Consumer modules that want to obtain Analyzer implementations no longer need to use ServiceLoader (though they still can, of course). Instead, all a consumer module needs to do is require the API module and call Analyzer::getAnalyzers. No need for a uses constraint or the ServiceLoader API anymore from the perspective of the consumer.

> 现在，API 模块导出了接口并使用了 Analyzer 接口的实现。想要获得 Analyzer 实现的消费者模块不再需要使用 ServiceLoader 了（当然，仍然可以使用 Service-Loader），取而代之，所有消费者模块需要做的就是请求 API 模块，并调用 Analyzer::getAnalyzers。从消费者角度来看，不再需要 uses 约束或 Service-Loader API 了。

Through this mechanism, you can use the power of services unobtrusively. Users of an API are not forced to know about services or ServiceLoader but still get the benefits of decoupling and extensibility.

> 通过这种机制，可以悄无声息地使用服务的强大功能，不再强迫 API 用户去了解服务或 ServiceLoader，与此同时仍然可以获得解耦和可扩展性所带来的好处。

## 4.4 Default Service Implementations 默认服务实现

So far, we’ve worked from the assumption that there is an API module, and there are several distinct provider modules implementing this API. That’s not unreasonable, but it’s far from the only way to set things up. It’s perfectly possible to put an implementation into the same module exporting the service type. When a service type has an obvious default implementation, why not provide it from the same module directly?

> 到目前为止都是假设有一个 API 模块，并且有几个不同的提供者模块实现了这个 API。虽然该假设不无道理，但并不是唯一的方法，将实现放在导出服务类型的同一模块中是完全可能的。当一个服务类型具有默认实现时，为什么不直接从同一个模块中提供它呢？

You see this pattern a lot in the way the JDK itself uses services. Even though it is possible to provide your own implementations for javax.sound.sampled.spi.AudioFileWriter or javax.print.PrintServiceLookup, most of the time the default implementations provided by the java.desktop module are adequate. These service types are exported from java.desktop, and at the same time default implementations are provided.

> 在 JDK 自身使用服务的方式中会经常看到这种模式。即使可以为 javax.sound.sampled.spi.Audio FileWriter 或 javax.print.PrintServiceLookup 提供自己的实现，大多数情况下 java.desktop 模块所提供的默认实现也是足够的。这些服务类型从 java.desktop 导出，同时提供了默认的实现。

In fact, java.desktop itself even has uses constraints for those service types. This shows how a module can play the role of API owner, service provider, and consumer at the same time.

> 事实上，java.desktop 自身对这些服务类型也使用了 uses 约束。这表明一个模块可以同时扮演 API 所有者、服务提供者和消费者的角色。

Bundling a default service implementation with the service type guarantees that at least one implementation is always available. In that case, no defensive coding is necessary on the consumer’s part. Some service dependencies are intended to be optional. A default implementation in the same module as the service type precludes this scenario. Then, having a separate API module is necessary. In “Implementing Optional Dependencies with Services”, this pattern is explored in more detail.

> 将默认的服务实现与服务类型进行绑定可以确保至少有一个实现始终可用。在这种情况下，消费者不需要编写防御性代码，且某些服务依赖项是可选的。与服务类型处在同一模块中的默认实现排除了这种情况。此时需要一个单独的 API 模块。在 5.6.2 节会更详细地探讨该模式。

## 4.5 Service Implementation Selection 服务实现的选择

When there are multiple providers, you don’t necessarily want to use them all. Sometimes you want to filter and select an implementation based on certain characteristics.

> 当存在多个提供者时，不一定要全部使用。可以根据某些特性筛选一种实现。

NOTE

It’s always the consumer deciding which service to use based on properties of the providers. Because providers should be completely unaware of each other, there is no way to favor a certain implementation from a provider’s perspective. What would happen if, for example, two providers designate themselves as default or best implementation? The logic to select the right service(s) is application-dependent and belongs to the consumer.

> 通常是消费者根据提供者的无属性决定使用哪种服务。因为提供者之间无法完全“彼此了解”，所以从提供者的角度来看无法支持某种实现。例如，如果两个提供者将自己指定为默认或最佳实施，那么会出现什么情况呢？选择正确的服务要依赖于应用程序，是由消费者确定的。

You have seen that the ServiceLoader API itself is fairly limited. Until now, we have iterated only over all existing service implementations. What if we have multiple providers but are interested in only “the best” implementation? The Java module system can’t possibly know what the best implementation is for your needs. Each domain has its own requirements in that regard. Therefore, it’s up to you to equip your service type with methods to discover the capabilities of a service and make decisions based on these methods. This need not be complicated, and usually comes down to adding self-describing methods to a service interface.

> 已经看到，ServiceLoader API 自身是有局限性的。到目前为止，仅对所有现有的服务实现进行了循环遍历。如果存在多个提供者，但只对“最好的”实现感兴趣，该怎么做呢？Java 模块系统不可能知道适合你的最佳实现。每个领域都有自己的需求。因此，将由自己根据服务类型配备方法，从而发现服务的功能，并根据这些方法进行决策。这个过程并不复杂，通常是将自描述方法添加到服务接口中。

For example, the Analyzer service interface offers a getName method. ServiceLoader doesn’t know or care about this method, but we can use it in consumer modules to identify an implementation. Besides selecting an algorithm by name, you can also think of describing different characteristics, for example, with getAccuracy or getCost methods. This way, a consumer of the Analyzer service can make a well-informed choice between implementations. There’s no explicit support from the ServiceLoader API necessary: it all boils down to designing self-describing interfaces.

> 例如，Analyzer 服务接口提供了一个 getName 方法。虽然 ServiceLoader 不知道或不关心该方法，但可以在消费者模块中使用它来识别一个实现。除了根据名称选择算法外，还可以考虑描述不同的特征，如使用 getAccuracy 或 getCost 方法。这样一来，Analyzer 服务的消费者可以在实现之间做出良好的选择。ServiceLoader API 没有必要显式支持：这些都归结于设计自描述接口。

## Service Type Inspection and Lazy Instantiation 服务类型检查和延迟实例化

In some scenarios, the mechanism described previously is still not sufficient. What if there’s no method on the service interface to distinguish the right implementation? Or instantiation of the services is expensive? We would incur the cost of initialization for all service implementations just to find the right one, using a ServiceLoader iteration. In most scenarios, this is not an issue, but a solution exists for problematic cases.

> 在某些情况下，前面所描述的机制还远远不够。如果服务接口上没有方法来区分正确的实现，或者实例化服务的代价非常昂贵，那么应该怎么办呢？为了找到正确的服务实现（使用 ServiceLoader 循环遍历），必须承担所有服务实现的初始化成本。在大多数情况下，这并不是一个问题，但可以使用一种解决方案来解决该问题。

With Java 9, ServiceLoader is enhanced to support service implementation type inspection before instantiation. Besides iterating over all provided instances as we’ve done so far, it’s also possible to inspect a stream of ServiceLoader.Provider descriptions. The ServiceLoader.Provider class makes it possible to inspect a service provider before requesting an instance. The stream method on ServiceLoader returns a stream of ServiceLoader.Provider objects to inspect.

> 在 Java 9 中，ServiceLoader 的功能得到了增强，支持在实例化之前对服务实现类型进行检查。除了遍历目前所有的实例以外，还可以检查一连串的 ServiceLoader. Provider 描述。ServiceLoader.Provider 类使得在请求实例之前检查服务提供者成为可能。ServiceLoader 上的 stream 方法返回一串要检查的 ServiceLoader. Provider 对象。

Let’s look at an example based on EasyText again.

> 接下来，再次以 EasyText 为基础查看一个示例。

First we introduce our own annotation that can be used to select the right service implementation in Example 4-8. Such an annotation can be part of the API module that is shared between providers and consumers. The example annotation describes whether an Analyzer is fast.

> 首先，介绍一下在示例 4-8 中可用来选择正确的服务实现的注释。此注释可以是提供者和消费者之间共享 API 模块的一部分。示例注释描述了 Analyzer 是否快速。

Example 4-8. Define an annotation to annotate the service implementation class (➥ chapter4/easytext-filtering)

> 示例 4-8：定义用来注释服务实现类的注释（chapter4/easytext-filtering）

```java
package javamodularity.easytext.analysis.api;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Fast {

  public boolean value() default true;

}
```

We can now use this annotation to add metadata to a service implementation. Here, we add it to an example Analyzer:

> 现在，可以使用该注释向服务实现添加元数据。此时将其添加到示例 Analyzer 中：

```java
@Fast
public class ReallyFastAnalyzer implements Analyzer {
  // Implementation of the analyzer
}
```

Now we just need some code to filter the Analyzers:

> 现在，只需添加一些代码就可以过滤 Analyzer：

```java
public class Main {
  public static void main(String args[]) {
    ServiceLoader<Analyzer> analyzers =
      ServiceLoader.load(Analyzer.class);

    analyzers.stream()
      .filter(provider -> isFast(provider.type()))
      .map(ServiceLoader.Provider::get)
      .forEach(analyzer -> System.out.println(analyzer.getName()));
  }

  private static boolean isFast(Class<?> clazz) {
    return clazz.isAnnotationPresent(Fast.class)
      && clazz.getAnnotation(Fast.class).value() == true;

  }
}
```

Through the type method on Provider, we get access to the java.lang.Class representation of the service implementation, which we pass to the isFast method for filtering.

> 通过使用 Provider 上的 type 方法，可以获取服务实现的 java.lang.Class 表示，并将其传递给 isFast 方法，从而实现过滤。

#### ENCAPSULATION VERSUS JAVA.LANG.CLASS 封装和 java.lang.Class

It might seem strange that we can get a java.lang.Class representation of the implementation class. Doesn’t that violate strong encapsulation? The package the class comes from isn’t even exported!

> 可以获取实现类的 java.lang.Class 表示看似非常奇怪。这不是违反了强封装吗？该类所在的包甚至没有被导出！

This is a clear case of “the map isn’t the territory”—even though Class describes the implementation class, you can’t actually do anything with it. When you try to get an instance through reflection (using provider.type().newInstance()), you will get an IllegalAccessError if the class is indeed not exported. So, having a Class object doesn’t necessarily mean you can instantiate it in your module. All access checks of the module system still apply.

> 这是“地图不是领土”的一个明显的例子——即使 Class 描述了实现类，但无法使用它完成任何事情。当尝试通过反射（使用 provider.type(). newInstance()）获取一个实例时，如果该类没有导出，那么将得到一个 IllegalAccessError。所以，拥有一个 Class 对象并不一定意味着可以在自己的模块中实例化它。模块系统的所有访问检查仍然适用。

The isFast method checks for the presence of our @Fast annotation, and checks the value explicitly for true (which is the default value). Analyzer implementations that are not annotated as being fast are ignored, but services annotated @Fast or @Fast(true) are instantiated and invoked. If you remove the filter from the stream pipeline, all Analyzers will be invoked indiscriminately.

> isFast 方法检查@Fast 注释是否存在，并显式检查该值是否为 true（这是默认值）。未被注释为快速的 Analyzer 实现将被忽略，而注释为@Fast 或@Fast(true)的服务则被实例化和调用。如果从流管道中删除 filter，则可以任意地调用所有 Analyzer。

The examples in this chapter demonstrate that although the ServiceLoader API is basic, the services mechanism is powerful. Services are an important construct in the Java module system when modularizing code.

> 本章的示例表明，虽然 ServiceLoader API 是基础，但服务机制是强大的。当模块化代码时，服务是 Java 模块系统中重要的组成部分。

NOTE

Using services as a means to improve decoupling is not new. For example, OSGi offers a services-based programming model as well. To be successful in creating truly modular code in OSGi, you must use services. So we’re building on a proven concept.

> 使用服务作为提高解耦性能的手段并不是一件新鲜的事，如 OSGi 也提供了基于服务的编程模型。要想在 OSGi 中创建真正的模块化代码，必须使用服务。所以以上内容都建立在一个成熟的概念之上。

## 4.6 Module Resolution with Service Binding 具有服务绑定的模块解析

Remember learning in “Module Resolution and the Module Path” that modules are resolved based on requires clauses in module descriptors? By recursively following all requires relations starting from a root module, the set of resolved modules is built from modules on the module path. During this process, missing modules are detected, giving the benefit of reliable configuration. The application won’t start if a required module is missing.

> 是否还记得在学习 2.7 节时曾讲过，模块是根据模块描述符中的 requires 子句进行解析的？从根模块开始以递归的方式寻找所有的 requires 关系，解析的模块集由模块路径上的模块构建。在此过程中可以检测到丢失的模块，从而提供了可靠的配置。如果缺少必需的模块，应用程序将无法启动。

Service provides and uses clauses add another dimension to the resolution process. Whereas requires clauses indicate strict compile-time relations between modules, service binding happens at run-time. Because service provider and consumer modules both declaratively state their intentions in module descriptors, this information can be used during the module resolution process as well.

> 服务的 provides 和 uses 子句为解析过程添加了另一个维度。requires 子句表示模块之间严格的编译时关系，而服务绑定则发生在运行时。因为服务提供者模块和消费者模块都在模块描述符中声明了各自的意图，所以在模块解析过程中也可以使用这些信息。

In theory, an application can start without any of its services being bound at run-time. Calling ServiceLoader::load won’t result in any instances. That’s hardly useful, so the module system locates service provider modules on the module path at startup in addition to modules that are required.

> 从理论上讲，即使在运行时没有绑定任何服务，应用程序也可以启动。此时调用 ServiceLoader::load 不会产生任何实例。这样做几乎没有任何用途，所以除了所需的模块之外，模块系统还可以在启动时在模块路径上查找服务提供者模块。

When a module with a uses clause is resolved, the module system locates all provider modules for the given service type on the module path and adds them to the resolution process. These provider modules, and their dependencies, become part of the run-time module graph.

当解析一个带有 uses 子句的模块时，模块系统将在模块路径上找到给定服务类型的所有提供者模块，并将其添加到解析过程中。这些提供者模块及其依赖项成为运行时模块图的一部分。

The implications of this extension to module resolution become clearer by looking at an example. In Figure 4-3 we look at our EasyText example again from the perspective of module resolution.

> 通过一个示例，可以更清楚地了解这个扩展对模块解析所产生的影响。在图 4-3 中，从模块解析的角度再看一下 EasyText 示例。

Service binding influences module resolution.
Figure 4-3. Service binding influences module resolution

We assume there are five modules on the module path: cli (the root module), api, kincaid, coleman, and an imaginary module syllablecounter. Module resolution starts with cli. It has a requires relation to api, so this module is added to the set of resolved modules. So far, nothing new.

> 假设在模块路径上有五个模块：cli（根模块）、api、kincaid、coleman 以及一个虚构模块 syllablecounter。模块解析从 cli 开始。它与 api 存在需求关系，所以将 api 模块添加到已解析模块集中。到目前为止，没有什么新内容。

However, cli also has a uses clause for Analyzer. There are two provider modules on the module path providing implementations of this interface. Hence the provider modules kincaid and coleman are added to the set of resolved modules. Module resolution stops for cli because it doesn’t have any other requires or uses clauses.

> 但是，cli 还有一个针对 Analyzer 的 uses 子句。在模块路径上，有两个实现了该接口的提供者模块。因此，提供者模块 kincaid 和 coleman 都被添加到已解析模块集中。由于再没有其他 requires 或 uses 子句，因此 cli 的模块解析停止。

For kincaid, there’s nothing left to add to the resolved modules. The api module that it requires has already been resolved. With coleman, things are more interesting. Service binding caused coleman to be resolved. In this example, the coleman module requires another module: syllablecounter. Therefore, syllablecounter is resolved as well, and both modules are added to the run-time module graph.

> 对于 kincaid 模块来说，无须向已解析模块集添加任何内容。它所需要的 api 模块已经被解析。而对于 coleman 模块来说，事情更加有趣。服务绑定导致 coleman 被解析。在本示例中，coleman 模块需要另一个模块 syllablecounter。因此，syllablecounter 也被解析，并且添加到运行时模块图中。

If syllablecounter itself would have had requires (or even uses!) clauses, these would be subject to module resolution as well. Conversely, if syllablecounter is not found on the module path, resolution fails, and the application won’t start. Even though cli, the consumer module, doesn’t have any static knowledge of the coleman provider module, it is still resolved with all its dependencies through service binding.

> 如果 syllablecounter 自身具有 requires（或者 uses）子句，那么这些模块也会完成模块解析过程。相反，如果在模块路径上找不到 syllablecounter，则解析失败，应用程序将无法启动。即使消费者模块 cli 对提供者模块 coleman 没有任何了解，也可以通过服务绑定获取所有依赖项并完成解析。

There is no way for a consumer to specify that it needs at least one implementation. When no service provider modules are found, the application starts just as well. Code using ServiceLoader needs to account for this possibility. You’ve already seen that many JDK service types have default implementations. When there’s a default implementation in the module exposing the service type, you’re guaranteed to always have at least one service implementation available.

> 对于消费者来说，则无法明确提出至少需要一个实现。当没有找到服务提供者模块时，应用程序也会启动。使用 ServiceLoader 的代码需要考虑到这种可能性。前面已经看到许多 JDK 服务类型都具有默认实现。当模块中的默认实现公开了服务类型时，就可以保证至少有一个可用的服务实现。

In the example, module resolution also succeeds when coleman isn’t on the module path. At run-time, the ServiceLoader::load call finds the implementation only from kincaid in that case. However, as we’ve explained, if coleman is on the module path but syllablecounter isn’t, the application won’t start because of module-resolution failure. Silently ignoring this problem would be possible for the module system, but runs counter to the mantra of reliable configuration based on module descriptors.

> 在示例中，即使 coleman 不在模块路径上，模块解析也会成功。此时，在运行时，ServiceLoader::load 调用只能从 kincaid 中查找服务实现。但是，正如前面所解释的那样，如果 coleman 在模块路径上，但 syllablecounter 不在，那么应用程序将因为模块解析失败而无法启动。对于模块系统来说，这个问题是可以忽略的，但却与基于模块描述符的可靠配置的原则背道而驰。

## 4.7 Services and Linking 服务和链接

In “Linking Modules”, you learned how to use jlink to create custom runtime images. We can create an image for the EasyText implementation with services as well. Based on what you’ve learned in the previous chapter, we can come up with the following jlink command:

> 在 3.1.7 节，学习了如何使用 jlink 创建自定义运行时映像，可以使用服务为 EasyText 实现创建一个映像。根据前一章所学到的内容，可以使用以下 jlink 命令：

```sh
$ jlink --module-path mods/:$JAVA_HOME/jmods --add-modules easytext.cli \
        --output image
```

jlink creates a directory image, containing a bin directory. We can inspect the modules included in the image by using the following command:

> jlink 创建了一个目录 image，其中包含了一个 bin 目录。可以使用下面的命令检查映像中所包含的模块：

```sh
$ image/bin/java --list-modules

java.base@9
easytext.analysis.api
easytext.cli
```

The api and cli modules are part of the image as expected, but what about the two analysis provider modules? If we run the application this way, it starts correctly because service providers are optional. But it is pretty useless without any analyzers.

> api 和 cli 模块是映像的一部分，但是两个分析提供者模块如何呢？如果以这种方式运行应用程序，它将正确启动，因为服务提供者是可选的。但是如何没有任何分析模块，这个应用程序是没有任何用处的。

jlink performs module resolution starting from the root module easytext.cli. All resolved modules are included in the resulting image. However, the resolution process differs from the resolution done by the module system at startup, which we discussed in the previous section. No service binding is done by jlink during module resolution. That means service providers are not automatically included in the image based on uses clauses.

> jlink 从根模块 easytext.cli 开始执行模块解析，所有已解析的模块都包含在生成的映像中。但是，该解析过程与上一节所讨论的启动时由模块系统完成的解析过程是不同的。在模块解析期间，jlink 没有完成任何服务绑定，这意味着不会根据 uses 子句自动将服务提供者包含在映像中。

Although this will certainly cause unexpected results for users who are not aware of this, it is a deliberate choice. Services are often used for extensibility. The EasyText application is a good example of this; new types of algorithms can be added by adding new service provider modules to the module path. Services used this way are not necessarily required to run the application. Which service providers you want to combine is an application-dependent concern. At build-time, there is no dependency to the service providers, and at link-time, it’s really up to the creator of the desired image to choose which service providers should be available.

> 虽然对于那些不了解这一点的用户来说，这么做肯定会导致意想不到的结果，但这是一个经过深思熟虑的选择。服务通常用于提供可扩展性。EasyText 应用程序就是一个很好的示例，可以通过向模块路径添加新的服务提供者模块来添加新的算法类型。以这种方式使用的服务并不一定是运行应用程序所需的。要组合哪些服务提供者需要根据应用程序而定。在生成时，对服务提供者没有任何依赖，而在链接时，由所需映像的创建者决定应该使用哪些服务提供者。

NOTE

A more down-to-earth reason to not do automatic service binding in jlink is that java.base has an enormous number of uses clauses. The providers for all these service types are in various other platform modules. Binding all these services by default would result in a much larger minimum image size. Without automatic service binding in jlink, you can create an image containing just java.base and application modules, as in our example. In general, automatic service binding can lead to unexpectedly large module graphs.

> 在 jlink 中不执行自动服务绑定的一个更切合实际的原因是 java.base 具有大量的 uses 子句。所有这些服务类型的提供者位于其他各种平台模块中，默认绑定所有这些服务将会增大映像的大小。如果 jlink 中没有执行自动服务绑定，就可以创建一个仅包含 java.base 和应用程序模块的映像，如本节示例。一般来说，自动服务绑定可能会产生意料之外的大型模块图。

Let’s try to create a runtime image for the EasyText application, configured to run from the command line. To include analyzers, we use the --add-modules argument when executing jlink for each provider module we want to add:

> 接下来尝试为 EasyText 应用程序创建一个运行时映像，并通过命令行运行。为了包含分析程序，当为每个要添加的提供者执行 jlink 时，使用了参数--add-modules：

```sh
$ jlink --module-path mods/:$JAVA_HOME/jmods \
        --add-modules easytext.cli           \
        --add-modules easytext.analysis.coleman \
        --add-modules easytext.analysis.kincaid \
        --output image
$ image/bin/java --list-modules

java.base@9
easytext.analysis.api
easytext.analysis.coleman
easytext.analysis.kincaid
easytext.cli
```

This looks better, but we will still find a problem when starting the application:

> 虽然一切看起来没有什么问题，但是当启动应用程序时仍然会发现一个问题：

```sh
$ image/bin/java -m easytext.cli input.txt
```

The application exits with an exception: java.lang.IllegalStateException: SyllableCounter not found. The kincaid module uses another service of type SyllableCounter. This is a case where a service provider uses another service to implement its functionality. We already know that jlink doesn’t automatically include service providers, so the module containing the SyllableCounter example wasn’t included either. We use --add-modules once more to finally get a fully functional image:

> 此时应用程序异常退出：java.lang.IllegalStateException:SyllableCount er not found。kincaid 模块使用了另一种 SyllableCounter 类型的服务。出现这种情况的原因是服务提供者使用了其他服务来实现其功能。前面已经讲过，jlink 不会自动包含服务提供者，所以包含 SyllableCounter 示例的模块也不会包括在内。再次使用--add-modules，从而最终获得一个功能完整的映像：

```sh
$ jlink --module-path mods/:$JAVA_HOME/jmods \
        --add-modules easytext.cli           \
        --add-modules easytext.analysis.coleman \
        --add-modules easytext.analysis.kincaid \
        --add-modules easytext.analysis.naivesyllablecounter \
        --output image
```

The fact that jlink doesn’t include service providers by default requires some extra work at link-time, especially when services use other services transitively. In return, it does give a lot of flexibility for fine-tuning the contents of a runtime image. Different images can serve different types of users, just by reconfiguring which service providers are included. In “Finding the Right Service Provider Modules”, we’ll see that jlink offers additional options to discover and link relevant service provider modules.

> 默认情况下，jlink 不包括服务提供者，因此在链接时需要完成一些额外的工作，尤其是当服务以传递的方式使用其他服务时。这样做的好处是可以非常灵活地微调运行时映像的内容。不同的映像可以为不同类型的用户提供服务，只需重新配置需要包含的服务提供者即可。在 13.3 节，将会看到 jlink 提供了额外的选项来发现和链接相关联的服务提供者模块。

The previous chapters covered the basics of the Java module system. Modularity is a lot about design and architecture, and this is where it really gets interesting. In the next chapter, we are going to look at patterns that improve the maintainability, flexibility, and reusability of systems built by using modules.

> 前面几章介绍了 Java 模块系统的基础知识。模块化主要关注的是设计和架构，这是它真正有趣的地方。下一章将学习一些模式，从而改善由模块构建的系统的可维护性、灵活性和可重用性。
