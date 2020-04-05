# 第 1 章 模块化概述

> Chapter 1. Modularity Matters

Have you ever scratched your head in bewilderment, asking yourself, “Why is this code here? How does it relate to the rest of this gigantic codebase? Where do I even begin?” Or did your eyes glaze over after scanning the multitude of Java Archives (JARs) bundled with your application code? We certainly have.

> 你是否曾经因为困惑而不停地挠头，并问自己：“代码为什么在这个位置？它们如何与其他庞大的代码块相关联？我该从哪里开始呢？”或者在看完应用程序代码所捆绑的大量 Java 归档文件（Java Archive, JAR）后，目光是否会呆滞？答案是肯定的。

The art of structuring large codebases is an undervalued one. This is neither a new problem, nor is it specific to Java. However, Java is one of the mainstream languages in which very large applications are built all the time—often making heavy use of many libraries from the Java ecosystem. Under these circumstances, systems can outgrow our capacity for understanding and efficient development. A lack of structure is dearly paid for in the long run, experience shows.

> 构建大型代码库是一项被低估的技术。这既不是一个新问题，也不是 Java 所特有的。然而，Java 一直是构建大型应用程序的主流语言之一，且通常会大量使用 Java 生态系统中的许多库。在这种情况下，系统可能会超出我们理解和有效开发的能力范围。经验表明，从长期来看，缺乏结构性所付出的代价是非常高的。

Modularity is one of the techniques you can employ to manage and reduce this complexity. Java 9 introduces a new module system that makes modularization easier and more accessible. It builds on top of abstractions Java already has for modular development. In a sense, it promotes existing best practices on large-scale Java development to be part of the Java language.

> 模块化是用来管理和减少这种复杂性的技术之一。Java 9 引入了一个新的模块系统，从而可以更容易地创建和访问模块。该系统建立在 Java 已用于模块化开发的抽象基础之上。从某种意义上讲，它促使了现有的大型 Java 开发最佳实践成为 Java 语言的一部分。

The Java module system will have a profound impact on Java development. It represents a fundamental shift to modularity as a first-class citizen for the whole Java platform. Modularization is addressed from the ground up, with changes to the language, Java Virtual Machine (JVM), and standard libraries. While this represents a monumental effort, it’s not as flashy as, for example, the addition of streams and lambdas in Java 8. There’s another fundamental difference between a feature like lambdas and the Java module system. A module system is concerned with the large-scale structure of whole applications. Turning an inner class into a lambda is a fairly small and localized change within a single class. Modularizing an application affects design, compilation, packaging, deployment, and so on. Clearly, it’s much more than just another language feature.

> Java 模块系统对 Java 开发产生了深刻的影响。它代表了模块化成为整个 Java 平台高级功能这一根本转变，从根本上解决了模块化的问题，对语言、JVM（Java Virtual Machine, Java 虚拟机）以及标准库都进行了更改。虽然完成这些更改付出了巨大的努力，但却并不像 Java 8 中添加流和 Lambda 表达式那样“华而不实”。诸如 Lambda 表达式之类的功能与 Java 模块系统之间还存在另一个根本区别。模块系统关注的是整个应用程序的大型结构，而将内部类转换为一个 Lambda 表达式则是单个类中相当小的局部变化。对一个应用程序进行模块化会影响设计、编译、打包、部署等过程，显然，这不仅仅是另一种语言功能。

With every new Java release, it’s tempting to dive right in and start using the new features. To make the most out of the module system, we should first take a step back and focus on what modularity is. And, more important, why we should care.

> 随着新 Java 版本的发布，你可能会迫不及待地想使用这个新功能了。为了充分利用模块系统，首先后退一步，先了解一下什么是模块化，更重要的是为什么要关注模块化。

## 1.1 What Is Modularity? 什么是模块化

So far, we’ve touched upon the goal of modularity (managing and reducing complexity), but not what modularity entails. At its heart, modularization is the act of decomposing a system into self-contained but interconnected modules. Modules are identifiable artifacts containing code, with metadata describing the module and its relation to other modules. Ideally, these artifacts are recognizable from compile-time all the way through run-time. An application then consists of multiple modules working together.

> 到目前为止，所讨论的只是实现模块化的目标（管理和减少复杂性），却没有介绍实现模块化需要什么？从本质上讲，模块化（modularization）是指将系统分解成独立且相互连接的模块的行为。模块（module）是包含代码的可识别工件，使用了元数据来描述模块及其与其他模块的关系。在理想情况下，这些工件从编译时到运行时都是可识别的。一个应用程序由多个模块协作组成。

So, modules group related code, but there’s more to it than that. Modules must adhere to three core tenets:

> 因此，模块对代码进行了分组，但不仅于此。模块必须遵循以下三个核心原则：

1.Strong encapsulation 强封装性

A module must be able to conceal part of its code from other modules. By doing so, a clear line is drawn between code that is publicly usable and code that is deemed an internal implementation detail. This prevents accidental or unwanted coupling between modules: you simply cannot use what has been encapsulated. Consequently, encapsulated code may change freely without affecting users of the module.

> 一个模块必须能够对其他模块隐藏其部分代码。这样一来，就可以在可公开使用的代码和被视为内部实现细节的代码之间划定一条清晰的界限，从而防止模块之间发生意外或不必要的耦合，即无法使用被封装的内容。因此，可以在不影响模块用户的情况下自由地对封装代码进行更改。

2.Well-defined interfaces 定义良好的接口

Encapsulation is fine, but if modules are to work together, not everything can be encapsulated. Code that is not encapsulated is, by definition, part of the public API of a module. Since other modules can use this public code, it must be managed with great care. A breaking change in nonencapsulated code can break other modules that depend on it. Therefore, modules should expose well-defined and stable interfaces to other modules.

> 一个模块通常需要使用其他模块来完成自己的工作，这些依赖关系必须是模块定义的一部分，以便使模块能够独立运行。显式依赖会产生一个模块图：节点表示模块，而边缘表示模块之间的依赖关系。拥有模块图对于了解应用程序以及运行所有必要的模块是非常重要的。它为模块的可靠配置提供了基础。

3.Explicit dependencies 显式依赖

Modules often need other modules to fulfill their obligations. Such dependencies must be part of the module definition, in order for modules to be self-contained. Explicit dependencies give rise to a module graph: nodes represent modules, and edges represent dependencies between modules. Having a module graph is important for both understanding an application and running it with all necessary modules. It provides the basis for a reliable configuration of modules.

> 一个模块通常需要使用其他模块来完成自己的工作，这些依赖关系必须是模块定义的一部分，以便使模块能够独立运行。显式依赖会产生一个模块图：节点表示模块，而边缘表示模块之间的依赖关系。拥有模块图对于了解应用程序以及运行所有必要的模块是非常重要的。它为模块的可靠配置提供了基础。

Flexibility, understandability, and reusability all come together with modules. Modules can be flexibly composed into different configurations, making use of the explicit dependencies to ensure that everything works together. Encapsulation ensures that you never have to know implementation details and that you will never accidentally rely on them. To use a module, knowing its public API is enough. Also, a module exposing well-defined interfaces while encapsulating its implementation details can readily be swapped with alternative implementations conforming to the same API.

> 模块具有灵活性、可理解性和可重用性。模块可以灵活地组合成不同的配置，利用显式依赖来确保一切工作正常。封装确保不必知道实现细节，也不会造成无意识间对这些细节的依赖。如果想要使用一个模块，只需知道其公共 API 就可以了。此外，如果模块在封装了实现细节的同时公开了定义良好的接口，那么就可以很容易地使用符合相同 API 的实现过程来替换被封装的实现过程。

Modular applications have many advantages. Experienced developers know all too well what happens when codebases are nonmodular. Endearing terms like spaghetti architecture, messy monolith, or big ball of mud do not even begin to cover the associated pain. Modularity is not a silver bullet, though. It is an architectural principle that can prevent these problems to a high degree when applied correctly.

> 模块化应用程序拥有诸多优点。经验丰富的开发人员都知道使用非模块化的代码库会发生什么事情。诸如意大利面架构（spaghetti architecture）、凌乱的巨石（messy monolith）以及大泥球（big ball of mud）之类的术语都描述了由此所带来的痛苦。但模块化也不是一种万能的方法。它是一种架构原则，如果使用正确则可以在很大程度上防止上述问题的产生。

That being said, the definition of modularity provided in this section is deliberately abstract. It might make you think of component-based development (all the rage in the previous century), service-oriented architecture, or the current microservices hype. Indeed, these paradigms try to solve similar problems at various levels of abstraction.

> 也就是说，本节中所提供的模块化定义是刻意抽象化的，这可能会让你想到基于组件的开发（20 世纪曾经风靡一时）、面向服务的体系结构或当前的微服务架构。事实上，这些范例都试图在各种抽象层面上解决类似问题。

What would it take to realize modules in Java? It’s instructive to take a moment and think about how the core tenets of modularity are already present in Java as you know it (and where it is lacking).

> 在 Java 中实现模块需要什么呢？建议先花时间思考一下在 Java 中已经存在哪些模块化的核心原则以及缺少哪些原则。

Done? Then you’re ready to proceed to the next section.

> 思考完了吗？如果想好了，就可以进入下一节的学习。

## 1.2 Before Java 9 在 Java 9 之前

Java is used for development of all sorts and sizes. Applications comprising millions of lines of code are no exception. Evidently, Java has done something right when it comes to building large-scale systems—even before Java 9 arrived on the scene. Let’s examine the three core tenets of modularity again in the light of Java before the arrival of the Java 9 module system.

> Java 可用于开发各种类型和规模的应用程序，开发包含数百万行代码的应用程序也是很常见的。显然，在构建大规模系统方面，Java 已经做了一些正确的事情——即使在 Java 9 出现之前。让我们再来看一下 Java 9 模块系统出现之前 Java 模块化的三个核心原则。

Encapsulation of types can be achieved by using a combination of packages and access modifiers (such as private, protected, or public). By making a class protected, for example, you can prevent other classes from accessing it unless they reside in the same package. That raises an interesting question: what if you want to access that class from another package in your component, but still want to prevent others from using it? There’s no good way to do this. You can, of course, make the class public. But, public means public to every other type in the system, meaning no encapsulation. You can hint that using such a class is not smart by putting it in an .impl or .internal package. But really, who looks at that? People use it anyway, just because they can. There’s no way to hide such an implementation package.

> 通过组合使用包（package）和访问修饰符（比如 private、protected 或 public），可以实现类型封装。例如，如果将一个类设置为 protected，那么就可以防止其他类访问该类，除非这些类与该类位于相同的包中。但这样做会产生一个有趣的问题：如果想从组件的另一个包中访问该类，同时仍然防止其他类使用该类，那么应该怎么做呢？事实是无法做到。当然，可以让类公开，但公开意味着对系统中的所有类型都是公开的，也就意味着没有封装。虽然可以将类放置到．impl 或．internal 包中，从而暗示使用此类是不明智的，但谁会在意呢？只要类可用，人们就会使用它。因此没有办法隐藏这样的实现包。

In the well-defined interfaces department, Java has been doing great since its inception. You guessed it, we’re talking about Java’s very own interface keyword. Exposing a public interface, while hiding the implementation class behind a factory or through dependency injection, is a tried-and-true method. As you will see throughout this book, interfaces play a central role in modular systems.

> 在定义良好接口方面，Java 自诞生以来就一直做得很好。你可能已经猜到了，我们所谈论的是 Java 自己的 interface 关键字。公开公共接口是一种经过验证的方法。它同时将实现类隐藏在工厂类后面或通过依赖注入完成。正如在本书中所看到的，接口在模块系统中起到了核心作用。

Explicit dependencies are where things start to fall apart. Yes, Java does have explicit import statements. Unfortunately, those imports are strictly a compile-time construct. Once you package your code into a JAR, there’s no telling which other JARs contain the types your JAR needs to run. In fact, this problem is so bad, many external tools evolved alongside the Java language to solve this problem. The following sidebar provides more details.

> 显式依赖是事情开始出问题的地方。Java 确实使用了显式的 import 语句。但不幸的是，从严格意义上讲，这些导入是编译时结构，一旦将代码打包到 JAR 中，就无法确定哪些 JAR 包含当前 JAR 运行所需的类型。事实上，这个问题非常糟糕，许多外部工具与 Java 语言一起发展以解决这个问题。以下栏目提供了更多的细节。

### EXTERNAL TOOLING TO MANAGE DEPENDENCIES: MAVEN AND OSGI 用来管理依赖关系的外部工具：Maven 和 OSGi

#### Maven

One of the problems solved by the Maven build tool is compile-time dependency management. Dependencies between JARs are defined in an external Project Object Model (POM) file. Maven’s great success is not the build tool per se, but the fact that it spawned a canonical repository called Maven Central. Virtually all Java libraries are published along with their POMs to Maven Central. Various other build tools such as Gradle or Ant (with Ivy) use the same repository and metadata. They all automatically resolve (transitive) dependencies for you at compile-time.

> 使用 Maven 构建工具所解决的一个问题是实现编译时依赖关系管理。JAR 之间的依赖关系在一个外部的 POM（Project Object Model，项目对象模型）文件中定义。Maven 真正成功之处不在于构建工具本身，而是生成了一个名为 Maven Central 的规范存储库。几乎所有的 Java 库都与它们的 POM 一起发布到 Maven Central。各种其他构建工具，比如 Gradle 或 Ant（使用 Ivy）都使用相同的存储库和元数据，它们在编译时会自动解析（传递）依赖关系。

#### OSGi

What Maven does at compile-time, OSGi does at run-time. OSGi requires imported packages to be listed as metadata in JARs, which are then called bundles. You must also explicitly define which packages are exported, that is, visible to other bundles. At application start, all bundles are checked: can every importing bundle be wired to an exporting bundle? A clever setup of custom classloaders ensures that at run-time no types are loaded in a bundle besides what is allowed by the metadata. As with Maven, this requires the whole world to provide correct OSGi metadata in their JARs. However, where Maven has unequivocally succeeded with Maven Central and POMs, the proliferation of OSGi-capable JARs is less impressive.

> Maven 在编译时做了什么，OSGi 在运行时就会做什么。OSGi 要求将导入的包在 JAR 中列为元数据，称之为捆绑包（bundle）。此外，还必须显式定义导出哪些包，即对其他捆绑包可见的包。在应用程序开始运行时，会检查所有的捆绑包：每个导入的捆绑包都可以连接到一个导出的捆绑包吗？自定义类加载器的巧妙设置可以确保在运行时，除了元数据所允许的类型以外，没有任何其他类型加载到捆绑包中。与 Maven 一样，需要在 JAR 中提供正确的 OSGi 元数据。然而，通过使用 Maven Central 和 POM, Maven 取得了巨大的成功，但支持 OSGi 的 JAR 的出现却没有给人留下太深刻的印象。

Both Maven and OSGi are built on top of the JVM and Java language, which they do not control. Java 9 addresses some of the same problems in the core of the JVM and the language. The module system is not intended to completely replace those tools. Both Maven and OSGi (and similar tools) still have their place, only now they can build on a fully modular Java platform.

> Maven 和 OSGi 构建在 JVM 和 Java 语言之上，这些都是它们所无法控制的。Java 9 解决了 JVM 和 Java 语言的核心中存在的一些相同问题。模块系统并不打算完全取代这些工具，Maven 和 OSGi（及类似工具）仍然有自己的一席之地，只不过现在它们可以建立在一个完全模块化的 Java 平台之上。

As it stands, Java offers solid constructs for creating large-scale modular applications. It’s also clear there is definitely room for improvement.

> 就目前来看，Java 为创建大型模块化应用程序提供了坚实的结构。当然，它还存在很多需要改进的地方。

### 1.2.1 JARs as Modules? 将 JAR 作为模块？

JAR files seem to be the closest we can get to modules pre-Java 9. They have a name, group related code, and can offer well-defined public interfaces. Let’s look at an example of a typical Java application running on top of the JVM to explore the notion of JARs as modules; see Figure 1-1.

> 在 Java 9 出现之前，JAR 文件似乎是最接近模块的，它们拥有名称、对相关代码进行了分组并且提供了定义良好的公共接口。接下来看一个运行在 JVM 之上的典型 Java 应用程序示例，研究一下 JAR 作为模块的相关概念，如图 1-1 所示。

<Figures figure="1-1">MyApplication is a typical Java application, packaged as a JAR and using other libraries</Figures>

There’s an application JAR called MyApplication.jar containing custom application code. Two libraries are used by the application: Google Guava and Hibernate Validator. There are three additional JARs as well. Those are transitive dependencies of Hibernate Validator, possibly resolved for us by a build tool like Maven. MyApplication runs on a pre-Java 9 runtime which itself exposes Java platform classes through several bundled JARs. The pre-Java 9 runtime may be a Java Runtime Environment (JRE) or a Java Development Kit (JDK), but in both cases it includes rt.jar (runtime library), which contains the classes of the Java standard library.

> 在图 1-1 中，有一个名为 MyApplication.jar 的应用程序 JAR，其中包含了自定义的应用程序代码。该应用程序使用了两个库：Google Guava 和 Hibernate Validator。此外，还有三个额外的 JAR。这些都是 Hibernate Validator 的可传递依赖项，可能是由诸如 Maven 之类的构建工具所创建的。MyApplication 运行在 Java 9 之前的运行时上（该运行时通过几个捆绑的 JAR 公开了 Java 平台类）。虽然 Java 9 之前的运行时可能是一个 JRE（Java Runtime Environment, Java 运行时环境）或 JDK（Java Development Kit, Java 开发工具包），但无论如何，都包含了 rt.jar（运行时库），其中包含了 Java 标准库的类。

When you look closely at Figure 1-1, you can see that some of the JARs list classes in italic. These classes are supposed to be internal classes of the libraries. For example, com.google.common.base.internal.Finalizer is used in Guava itself, but is not part of the official API. It’s a public class, since other Guava packages use Finalizer. Unfortunately, this also means there’s no impediment for com.myapp.Main to use classes like Finalizer. In other words, there’s no strong encapsulation.

> 当仔细观察图 1-1 时，会发现一些 JAR 以斜体形式列出了相关类。这些类都是库的内部类。例如，虽然在 Guava 中使用了 com.google.common.base.internal. Finalizer，但它并不是官方 API 的一部分，而是一个公共类，其他 Guava 包也可以使用 Finalizer。不幸的是，这也意味着无法阻止 com.myapp.Main 使用诸如 Finalizer 之类的类。换句话说，没有实现强封装性。

The same holds for internal classes from the Java platform itself. Packages such as sun.misc have always been accessible to application code, even though documentation sternly warns they are unsupported APIs that should not be used. Despite this warning, utility classes such as sun.misc.BASE64Encoder are used in application code all the time. Technically, that code may break with any update of the Java runtime, since they are internal implementation classes. Lack of encapsulation essentially forced those classes to be considered semipublic APIs anyway, since Java highly values backward compatibility. This is an unfortunate situation, arising from the lack of true encapsulation.

> 对于 Java 平台的内部类来说也存在同样的情况。诸如 sun.misc 之类的包经常被应用程序代码所访问，虽然相关文档严重警告它们是不受支持的 API，不应该使用。尽管发出了警告，但诸如 sun.misc.BASE64Encoder 之类的实用工具类仍然经常在应用程序代码中使用。从技术上讲，使用了这些类的代码可能会破坏 Java 运行时的更新，因为它们都是内部实现类。缺乏封装性实际上迫使这些类被认为是“半公共”API，因为 Java 高度重视向后兼容性。这是由于缺乏真正的封装所造成的结果。

What about explicit dependencies? As you’ve already learned, there is no dependency information anymore when looking strictly at JARs. You run MyApplication as follows:

> 什么是显式依赖呢？你可能已经看到，从严格意义上讲，JAR 不包含任何依赖信息。按照下面的步骤运行 MyApplication：

```sh
java -classpath lib/guava-19.0.jar:\
                lib/hibernate-validator-5.3.1.jar:\
                lib/jboss-logging-3.3.0Final.jar:\
                lib/classmate-1.3.1.jar:\
                lib/validation-api-1.1.0.Final.jar \
     -jar MyApplication.jar
```

Setting up the correct classpath is up to the user. And, without explicit dependency information, it is not for the faint of heart.

> 正确的类路径都是由用户设置的。由于缺少明确的依赖关系信息，因此完成设置并非易事。

### 1.2.2 Classpath Hell 类路径地狱

The classpath is used by the Java runtime to locate classes. In our example, we run Main, and all classes that are directly or indirectly referenced from this class need to be loaded at some point. You can view the classpath as a list of all classes that may be loaded at runtime. While there is more to it behind the scenes, this view suffices to understand the issues with the classpath.

> Java 运行时使用类路径（classpath）来查找类。上面的示例运行了 Main，所有从该类直接或间接引用的类都需要加载。可以将类路径视为可能在运行时加载的所有类的列表。虽然在“幕后”还有更多的内容，但是以上观点足以说明类路径所存在的问题。

A condensed view of the resulting classpath for MyApplication looks like this:

> 为 MyApplication 所生成的类路径简图如下所示：

```java
java.lang.Object
java.lang.String
...
sun.misc.BASE64Encoder
sun.misc.Unsafe
...
javax.crypto.Cypher
javax.crypto.SecretKey
...
com.myapp.Main
...
com.google.common.base.Joiner
...
com.google.common.base.internal.Joiner
org.hibernate.validator.HibernateValidator
org.hibernate.validator.constraints.NotEmpty
...
org.hibernate.validator.internal.engine.ConfigurationImpl
...
javax.validation.Configuration
javax.validation.constraints.NotNull
```

There’s no notion of JARs or logical grouping anymore. All classes are sequenced into a flat list, in the order defined by the -classpath argument. When the JVM loads a class, it reads the classpath in sequential order to find the right one. As soon as the class is found, the search ends and the class is loaded.

> 此时，没有 JAR 或逻辑分组的概念了。所有类按照-classpath 参数定义的顺序排列成一个平面列表。当 JVM 加载一个类时，需要按照顺序读取类路径，从而找到所需的类。一旦找到了类，搜索就会结束并加载类。

What if a class cannot be found on the classpath? Then you will get a run-time exception. Because classes are loaded lazily, this could be triggered when some unlucky user clicks a button in your application for the first time. The JVM cannot efficiently verify the completeness of the classpath upon starting. There is no way to tell in advance whether the classpath is complete, or whether you should add another JAR. Obviously, that’s not good.

> 如果在类路径中没有找到所需的类又会发生什么情况呢？此时会得到一个运行时异常。由于类会延迟加载，因此一些不幸的用户在首次运行应用程序并点击一个按钮时会出现找不到类的情况。JVM 无法在应用程序启动时有效地验证类路径的完整性，即无法预先知道类路径是否是完整的，或者是否应该添加另一个 JAR。显然，这并不够好。

![](./jv9m_01in01.png)

More insidious problems arise when duplicate classes are on the classpath. Let’s say you try to circumvent the manual setup of the classpath. Instead, you let Maven construct the right set of JARs to put on the classpath, based on the explicit dependency information in POMs. Since Maven resolves dependencies transitively, it’s not uncommon for two versions of the same library (say, Guava 19 and Guava 18) to end up in this set, through no fault of your own. Now both library JARs are flattened into the classpath, in an undefined order. Whichever version of the library classes comes first is loaded. However, other classes may expect a class from the (possibly incompatible) other version. Again, this leads to run-time exceptions. In general, whenever the classpath contains two classes with the same (fully qualified) name, even if they are completely unrelated, only one “wins.”

> 当类路径上有重复类时，则会出现更为隐蔽的问题。假设尝试避免手动设置类路径，而是由 Maven 根据 POM 中的显式依赖信息构建将放到类路径中的 JAR 集合。由于 Maven 以传递的方式解决依赖关系问题，因此在该集合中出现相同库的两个版本（如 Guava 19 和 Guava 18）是非常常见的，虽然这并不是你的过错。现在，这两个库 JAR 以一种未定义的顺序压缩到类路径中。库类的任一版本都可能会被首先加载。此外，有些类还可能会使用来自（可能不兼容的）其他版本的类。此时就会导致运行时异常。一般来说，当类路径包含两个具有相同（完全限定）名称的类时，即使它们完全不相关，也只有一个会“获胜”。

It now becomes clear why the term classpath hell (also known as JAR hell) is so infamous in the Java world. Some people have perfected the art of tuning a classpath through trial-and-error—a rather sad occupation when you think about it. The fragile classpath remains a leading cause of problems and frustration. If only more information were available about the relations between JARs at run-time. It’s as if a dependency graph is hiding in the classpath and is just waiting to come out and be exploited. Enter Java 9 modules!

> 现在你应该明白为什么类路径地狱（classpath hell，也被称为 JAR 地狱）在 Java 世界如此臭名昭著了。一些人通过不断地摸索，逐步完善了调整类路径的方法——但该过程是相当艰苦的。脆弱的类路径仍然是导致问题和失败的主要原因。如果能够在运行时提供更多关于 JAR 之间关系的信息，那就最好了，就好像是一个隐藏在类路径中并等待被发现和利用的依赖关系图。接下来学习 Java 9 模块！

## 1.3 Java 9 Modules Java 9 模块

By now, you have a solid understanding of Java’s current strengths and limitations when it comes to modularity. With Java 9, we get a new ally in the quest for well-structured applications: the Java module system. While designing the Java Platform Module System to overcome current limitations, two main goals were defined:

> 到目前为止，我们已经全面了解了当前 Java 在模块化方面的优势和局限。随着 Java 9 的出现，可以使用一个新的工具——Java 模块系统来开发结构良好的应用程序。在设计 Java 平台模块系统以克服当前所存在的局限时，主要设定了两个目标：

- Modularize the JDK itself.
- Offer a module system for applications to use.

---

> - 对 JDK 本身进行模块化。
> - 提供一个应用程序可以使用的模块系统。

These goals are closely related. Modularizing the JDK is done by using the same module system that we, as application developers, can use in Java 9.

> 这两个目标是紧密相关的。JDK 的模块化可通过使用与应用程序开发人员在 Java 9 中使用的相同的模块系统来实现。

The module system introduces a native concept of modules into the Java language and runtime. Modules can either export or strongly encapsulate packages. Furthermore, they express dependencies on other modules explicitly. As you can see, all three tenets of modularity are addressed by the Java module system.

> 模块系统将模块的本质概念引入 Java 语言和运行时。模块既可以导出包，也可以强封装包。此外，它们显式地表达了与其他模块的依赖关系。可以看到，Java 模块系统遵循了模块的三个核心原则。

Let’s revisit the MyApplication example, now based on the Java 9 module system, in Figure 1-2.

> 接下来回到前面的 MyApplication 示例，此时示例建立在 Java 9 模块系统之上，如图 1-2 所示。

Each JAR becomes a module, containing explicit references to other modules. The fact that hibernate-validator uses jboss-logging, classmate, and validation-api is part of its module descriptor. A module has a publicly accessible part (on the top) and an encapsulated part (on the bottom, indicated with the padlock). That’s why MyApplication can no longer use Guava’s Finalizer class. Through this diagram, we discover that MyApplication uses validation-api, as well, to annotate some of its classes. What’s more, MyApplication has an explicit dependency on a module in the JDK called java.sql.

> 每个 JAR 都变成了一个模块，并包含了对其他模块的显式引用。hibernate-validator 的模块描述符表明了该模块使用了 jboss-logging、classmate 和 validation-api。一个模块拥有一个可公开访问的部分（位于顶部）以及一个封装部分（位于底部，并以一个挂锁表示）。这也就是为什么 MyApplication 不能再使用 Guava 的 Finalizer 类的原因。通过图 1-2，会发现 MyApplication 也使用了 validation-api 来注释它的类。此外，MyApplication 还显式依赖 JDK 中的一个名为 java.sql 的模块。

<Figures figure="1-2">MyApplication as a modular application on top of modular Java 9</Figures>

Figure 1-2 tells us much more about the application than in the classpath situation shown in Figure 1-1. All that could be said there is that MyApplication uses classes from rt.jar, like all Java applications—and that it runs with a bunch of JARs on the (possibly incorrect) classpath.

> 相比于图 1-1 所示的类路径图，图 1-2 告诉了我们关于应用程序的更多信息。可以这么说，就像所有 Java 应用程序一样，MyApplication 使用了来自 rt.jar 的类，并且运行了（可能不正确的）该类路径上的一堆 JAR。

That’s just the application layer. It’s modules all the way down. At the JDK layer, there are modules as well (Figure 1-2 shows a small subset). Like the modules in the application layer, they have explicit dependencies and expose some packages while concealing others. The most essential platform module in the modular JDK is java.base. It exposes packages such as java.lang and java.util, which no other module can do without. Because you cannot avoid using types from these packages, every module requires java.base implicitly. If the application modules require any functionality from platform modules other than what’s in java.base, these dependencies must be explicit as well, as is the case with MyApplication’s dependency on java.sql.

> 这只是在应用层。模块的概念一直向下延伸，在 JDK 层也使用了模块（图 1-2 显示了一个小的子集）。与应用层中的模块一样，JDK 层中的模块也具有显式依赖，并在隐藏了一些包的同时公开了另一些包。在模块化 JDK 中，最基本的平台模块是 java.base。它公开了诸如 java.lang 和 java.util 之类的包，如果没有这些包，其他模块什么也做不了。由于无法避免使用这些包中的类型，因此每个模块毫无疑问都需要 java. base。如果应用程序模块需要任何来自 java.base 之外的平台模块的功能，那么这些依赖关系也必须是显式的，就像 MyApplication 对 java.sql 的依赖一样。

Finally, there’s a way to express dependencies between separate parts of the code at a higher level of granularity in the Java language. Now imagine the advantages of having all this information available at compile-time and run-time. Accidental dependencies on code from other nonreferenced modules can be prevented. The toolchain knows which additional modules are necessary for running a module by inspecting its (transitive) dependencies, and optimizations can be applied using this knowledge.

> 最后，使用一种方法可以在 Java 语言的更高级别的粒度上表示代码不同部分之间的依赖关系。现在想象一下在编译时和运行时获得所有这些信息所带来的优势。这可以防止对来自其他非引用模块的代码的意外依赖。通过检查（传递）依赖关系，工具链可以知道运行模块需要哪些附加模块并进行优化。

Strong encapsulation, well-defined interfaces, and explicit dependencies are now part of the Java platform. In short, these are the most important benefits of the Java Platform Module System:

> 现在，强封装性、定义良好的接口以及显式依赖已经成为 Java 平台的一部分。总之，Java 平台模块系统带来了如下最重要的好处：

1.Reliable configuration 可靠的配置

The module system checks whether a given combination of modules satisfies all dependencies before compiling or running code. This leads to fewer run-time errors.

> 在编译或运行代码之前，模块系统会检查给定的模块组合是否满足所有依赖关系，从而导致更少的运行时错误。

2.Strong encapsulation 强封装型

Modules explicitly choose what to expose to other modules. Accidental dependencies on internal implementation details are prevented.

> 模块显式地选择了向其他模块公开的内容，从而防止对内部实现细节的意外依赖。

3.Scalable development 可扩展开发

Explicit boundaries enable teams to work in parallel while still creating maintainable codebases. Only explicitly exported public types are shared, creating boundaries that are automatically enforced by the module system.

> 显式边界能够让开发团队并行工作，同时可创建可维护的代码库。只有显式导出的公共类型是共享的，这创建了由模块系统自动执行的边界。

4.Security 安全性

Strong encapsulation is enforced at the deepest layers inside the JVM. This limits the attack surface of the Java runtime. Gaining reflective access to sensitive internal classes is not possible anymore.

> 在 JVM 的最深层次上执行强封装，从而减少 Java 运行时的攻击面，同时无法获得对敏感内部类的反射访问。

5.Optimization 优化

Because the module system knows which modules belong together, including platform modules, no other code needs to be considered during JVM startup. It also opens up the possibility to create a minimal configuration of modules for distribution. Furthermore, whole-program optimizations can be applied to such a set of modules. Before modules, this was much harder, because explicit dependency information was not available and a class could reference any other class from the classpath.

> 由于模块系统知道哪些模块是在一起的，包括平台模块，因此在 JVM 启动期间不需要考虑其他代码。同时，其也为创建模块分发的最小配置提供了可能性。此外，还可以在一组模块上应用整个程序的优化。在模块出现之前，这样做是非常困难的，因为没有可用的显式依赖信息，一个类可以引用类路径中任何其他类。

In the next chapter, we explore how modules are defined and what concepts govern their interactions. We do this by looking at modules in the JDK itself. There are many more platform modules than shown in Figure 1-2.

> 在下一章，将通过查看 JDK 中的模块，学习如何定义模块以及使用哪些概念管理模块之间的交互。JDK 包含了如图 1-2 所示更多的平台模块。

Exploring the modular JDK in Chapter 2 is a great way to get to know the module system concepts, while at the same time familiarizing yourself with the modules in the JDK. These are, after all, the modules you’ll be using first and foremost in your modular Java 9 applications. After that, you’ll be ready to start writing your own modules in Chapter 3.

> 在第 2 章研究模块化 JDK 是了解模块系统概念的一个非常好的方法，同时还可以熟悉 JDK 中的模块。毕竟我们将首先使用这些模块创建自己的模块化 Java 9 应用程序。随后，在第 3 章我们将准备开始编写自己的模块。
