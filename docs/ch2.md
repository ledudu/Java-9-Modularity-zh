# 第 2 章 模块化和模块化 JDK

> Chapter 2. Modules and the Modular JDK

Java is over 20 years old. As a language, it’s still popular, proving that Java has held up well. The platform’s long evolution becomes especially apparent when looking at the standard libraries. Prior to the Java module system, the runtime library of the JDK consisted of a hefty rt.jar (as shown previously in Figure 1-1), weighing in at more than 60 megabytes. It contains most of the runtime classes for Java: the ultimate monolith of the Java platform. In order to regain a flexible and future-proof platform, the JDK team set out to modularize the JDK—an ambitious goal, given the size and structure of the JDK. Over the course of the past 20 years, many APIs have been added. Virtually none have been removed.

> Java 有超过 20 年的发展历史。作为一种语言它仍然很受欢迎，这表明 Java 一直保持很好的状态。只要查看一下标准库，就会很明显地看到该平台长期的演变过程。在 Java 模块系统之前，JDK 的运行时库由一个庞大的 rt.jar 所组成（如前一章的图 1-1 所示），其大小超过 60MB，包含了 Java 大部分运行时类：即 Java 平台的最终载体。为了获得一个灵活且符合未来发展方向的平台，JDK 团队着手对 JDK 进行模块化——考虑到 JDK 的规模和结构，不得不说这是一个雄心勃勃的目标。在过去 20 年里，增加了许多 API，但几乎没有删除任何 API。

Take CORBA—once considered the future of enterprise computing, and now a mostly forgotten technology. (To those who are still using it: we feel for you.) The classes supporting CORBA in the JDK are still present in rt.jar to this day. Each and every distribution of Java, regardless of the applications it runs, includes those CORBA classes. No matter whether you use CORBA or not, the classes are there. Carrying this legacy in the JDK results in unnecessary use of disk space, memory, and CPU time. In the context of using resource-constrained devices, or creating small containers for the cloud, these resources are in short supply. Not to mention the cognitive overhead of obsolete classes showing up in IDE autocompletions and documentation during development.

> 以 CORBA 为例，它曾经被认为是企业计算的未来，而现在是一种被遗忘的技术（对于那些仍然在使用 CORBA 的人，我们深表同情）。如今，JDK 的 rt.jar 中仍然存在支持 CORBA 的类。无论运行什么应用程序，Java 的每次发布都会包含这些 CORBA 类。不管是否使用 CORBA，这些类都在那里。在 JDK 中包含这些遗留类会浪费不必要的磁盘空间、内存和 CPU 时间。在使用资源受限设备或者为云创建小容器时，这些资源都是供不应求的。更不用说开发过程中在 IDE 自动完成和文档中出现这些过时类所造成的认知超载（cognitive overhead）。

Simply removing these technologies from the JDK isn’t a viable option, though. Backward compatibility is one of the most important guiding principles for Java. Removal of APIs would break a long streak of backward compatibility. Although it may affect only a small percentage of users, plenty of people are still using technologies like CORBA. In a modular JDK, people who aren’t using CORBA can choose to ignore the module containing CORBA.

> 但是，从 JDK 中删除这些技术并不是一个可行的办法。向后兼容性是 Java 最重要的指导原则之一，而移除 API 会破坏长久以来形成的向后兼容性。虽然这样做只会影响一小部分用户，但仍然有许多人在使用 CORBA 之类的技术。而在模块化 JDK 中，不使用 CORBA 的人可以选择忽略包含 CORBA 的模块。

Alternatively, an aggressive deprecation schedule for truly obsolete technologies could work. Still, it would take several major releases before the JDK sheds the excess weight. Also, deciding what technology is truly obsolete would be at the discretion of the JDK team, which is a difficult position to be in.

> 另外，主动地弃用那些真正过时的技术也是可行的。只不过在 JDK 删除这些技术之前，可能还会再发布多个主要版本。此外，决定什么技术是真正过时的取决于 JDK 团队，这是一个非常难做的决定。

NOTE

In the specific case of CORBA, the module is marked as deprecated, meaning it will likely be removed in a subsequent major Java release.

> 具体到 CORBA，该模块被标记为已弃用，这意味着它将有可能在随后主要的 Java 版本中被删除。

But the desire to break up the monolithic JDK is not just about removing obsolete technology. A vast array of technologies are useful to certain types of applications, while useless for others. JavaFX is the latest user-interface technology in Java, after AWT and Swing. This is certainly not something to be removed, but clearly it’s not required in every application either. Web applications, for example, use none of the GUI toolkits in Java. Yet there is no way to deploy and run them without all three GUI toolkits being carried along.

> 但是，分解整体 JDK 的愿望并不仅仅是删除过时的技术。很多技术对于某些类型的应用程序来说是有用的，而对于其他应用程序来说是无用的。JavaFX 是继 AWT 和 Swing 之后 Java 中最新的用户界面技术。这些技术当然是不能删除的，但显然它们不是每个应用程序都需要的。例如，Web 应用程序不使用 Java 中任何 GUI 工具包。然而，如果没有这些 GUI 工具包，就无法完成部署和运行。

Aside from convenience and waste, consider the security perspective. Java has experienced a considerable number of security exploits in the past. Many of these exploits share a common trait: somehow attackers gain access to sensitive classes inside the JDK to bypass the JVM’s security sandbox. Strongly encapsulating dangerous internal classes within the JDK is a big improvement from a security standpoint. Also, decreasing the number of available classes in the runtime decreases the attack surface. Having tons of unused classes around in your application runtime only for them to be exploited later is an unfortunate trade-off. With a modular JDK, only those modules your application needs are resolved.

> 除了便利与避免浪费之外，还要从安全角度进行考虑。Java 在过去经历过相当多的安全漏洞。这些漏洞都有一个共同的特点：不知何故，攻击者可以绕过 JVM 的安全沙盒并访问 JDK 中的敏感类。从安全的角度来看，在 JDK 中对危险的内部类进行强封装是一个很大的改进。同时，减少运行时中可用类的数量会降低攻击面。在应用程序运行时中保留大量暂时不使用的类是一种不恰当的做法。而通过使用模块化 JDK，可以确定应用程序所需的模块。

By now, it’s abundantly clear that a modular approach for the JDK itself is sorely needed.

> 目前可以清楚地看到：极需要一种对 JDK 本身进行模块化的方法。

## 2.1 The Modular JDK 模块化 JDK

The first step toward a more modular JDK was taken in Java 8 with the introduction of compact profiles. A profile defines a subset of packages from the standard library available to applications targeting that profile. Three profiles are defined, imaginatively called compact1, compact2, and compact3. Each profile is a superset of the previous, adding more packages that can be used. The Java compiler and runtime were updated with knowledge of these predefined profiles. Java SE Embedded 8 (Linux only) offers low-footprint runtimes matching the compact profiles.

> 迈向模块化 JDK 的第一步是在 Java 8 中采用了紧凑型配置文件（compact profile）。配置文件定义了标准库中可用于针对该配置文件的应用程序的一个包子集。假设定义了三个配置文件，分别为 compact1、compact2 和 compact3。每个配置文件都是前一个配置文件的超集，添加了更多可用的包。使用这些预定义配置文件更新 Java 编译器和运行时。Java SE Embedded 8（仅针对 Linux）提供了与紧凑型配置文件相匹配的占用资源少的运行时。

If your application fits one of the profiles described in Table 2-1, this is a good way to target a smaller runtime. But if you require even so much as a single class outside of the predefined profiles, you’re out of luck. In that sense, compact profiles are far from flexible. They also don’t address strong encapsulation. As an intermediate solution, compact profiles fulfilled their purpose. Ultimately, a more flexible approach is needed.

> 如果你的应用程序符合表 2-1 中所描述的其中一个配置文件，那么就可以在一个较小的运行时上运行。但是，如果需要使用预定义配置文件之外的类，那么就比较麻烦了。从这个意义上讲，紧凑型配置文件的灵活性非常差，并且也无法解决强封装问题。作为一种中间解决方案，紧凑型配置文件实现了它的目的，但最终需要一种更灵活的办法。

Table 2-1. Profiles defined for Java 8

| Profile  | Description                                                            |
| -------- | ---------------------------------------------------------------------- |
| compact1 | Smallest profile with Java core classes and logging and scripting APIs |
| compact2 | Extends compact1 with XML, JDBC, and RMI APIs                          |
| compact3 | Extends compact2 with security and management APIs                     |

You already saw a glimpse of how JDK 9 is split into modules in Figure 1-2. The JDK now consists of about 90 platform modules, instead of a monolithic library. A platform module is part of the JDK, unlike application modules, which you can create yourself. There is no technical distinction between platform modules and application modules. Every platform module constitutes a well-defined piece of functionality of the JDK, ranging from logging to XML support. All modules explicitly define their dependencies on other modules.

> 从图 1-2 中可以看到 JDK 9 是如何被拆分为模块的。目前，JDK 由大约 90 个平台模块组成，而不是一个整体库。与可由自己创建的应用程序模块不同的是，平台模块是 JDK 的一部分。从技术上讲，平台模块和应用模块之间没有任何技术区别。每个平台模块都构成了 JDK 的一个定义良好的功能块，从日志记录到 XML 支持。所有模块都显式地定义了与其他模块的依赖关系。

A subset of these platform modules and their dependencies is shown in Figure 2-1. Every edge indicates a unidirectional dependency between modules (we’ll get to the difference between solid and dashed edges later). For example, java.xml depends on java.base. As stated in “Java 9 Modules”, every module implicitly depends on java.base. In Figure 2-1 this implicit dependency is shown only when java.base is the sole dependency for a given module, as is the case with, for example, java.xml.

> 图 2-1 显示了这些平台模块的子集及其依赖关系。每条边表示模块之间的单向依赖关系（稍后介绍实线和虚线之间的区别）。例如，java.xml 依赖于 java.base。如 1.3 节所述，每个模块都隐式依赖于 java.base。在图 2-1 中，只有当 java.base 是给定模块的唯一依赖项时，才会显示这个隐式依赖关系，比如 java.xml。

Even though the dependency graph may look a little overwhelming, we can glean a lot of information from it. Just by looking at the graph, you can get a decent overview of what the Java standard libraries offer and how the functionalities are related. For example, java.logging has many incoming dependencies, meaning it is used by many other platform modules. That makes sense for a central functionality such as logging. Module java.xml.bind (containing the JAXB API for XML binding) has many outgoing dependencies, including an unexpected one on java.desktop. The fact that we can notice this oddity by looking at a generated dependency graph and talk about it is a huge improvement. Because of the modularization of the JDK, there are clean module boundaries and explicit dependencies to reason about. Having an overview of a large codebase like the JDK, based on explicit module information, is invaluable.

> 尽管依赖关系图看起来有点让人无所适从，但是可以从中提取很多信息。只需观察一下该图，就可以大概了解 Java 标准库所提供的功能以及各项功能是如何关联的。例如，java.logging 有许多传入依赖项（incoming dependencies），这意味着许多其他平台模块使用了该模块。对于诸如日志之类的中心功能来说，这么做是很有意义的。模块 java.xml.bind（包含用于 XML 绑定的 JAXB API）有许多传出依赖项（outgoing dependencies），包括 java.desktop（这是一个意料之外的依赖项）。事实上，我们通过查看生成的依赖关系图并进行讨论而发现这个奇异之处，这本身就是一个巨大的进步。由于 JDK 的模块化，形成了清晰的模块边界以及显式的依赖关系。根据显式模块信息了解 JDK 之类的大型代码块是非常有价值的。

Subset of platform modules in the JDK.
Figure 2-1. Subset of platform modules in the JDK

Another thing to note is how all arrows in the dependency graph point downward. There are no cycles in this graph. That’s not by accident: the Java module system does not allow compile-time circular dependencies between modules.

> 另一个需要注意的是，依赖关系图中的所有箭头都是向下的，图中没有循环。这是必然的：Java 模块系统不允许模块之间存在编译时循环依赖。

WARNING

Circular dependencies are generally an indication of bad design. In “Breaking Cycles”, we discuss how to identify and resolve circular dependencies in your codebase.

> 循环依赖通常是一种非常不好的设计。在 5.5.2 节中，将讨论如何识别和解决代码库中的循环依赖。

All modules in Figure 2-1, except jdk.httpserver and jdk.unsupported, are part of the Java SE specification. They share the `java.*` prefix for module names. Every certified Java implementation must contain these modules. Modules such as jdk.httpserver contain implementations of tools and APIs. Where such implementations live is not mandated by the Java SE specification, but of course such modules are essential to a fully functioning Java platform. There are many more modules in the JDK, most of them in the `jdk.*` namespace.

> 除了 jdk.httpserver 和 jdk.unsupported 之外，图 2-1 中的所有模块都是 Java SE 规范的一部分。它们的模块名都共享了前缀`java.*`。每个认证的 Java 实现都必须包含这些模块。诸如 jdk.httpserver 之类的模块包含了工具和 API 的实现，虽然这些实现不受 Java SE 规范的约束，但是这些模块对于一个功能完备的 Java 平台来说是至关重要的。JDK 中还有很多的模块，其中大部分在`jdk.*`命名空间。

TIP

You can get the full list of platform modules by running `java --list-modules`.

> 通过运行`java --list-modules`，可以获取平台模块的完整列表。

Two important modules can be found at the top of Figure 2-1: java.se and java.se.ee. These are so-called aggregator modules, and they serve to logically group several other modules. We’ll see how aggregator modules work later in this chapter.

> 在图 2-1 的顶部可以找到两个重要的模块：java.se 和 java.se.ee。它们就是所谓的聚合器模块（aggregator module），主要用于对其他模块进行逻辑分组。本章稍后将介绍聚合器模块的工作原理。

Decomposing the JDK into modules has been a tremendous amount of work. Splitting up an entangled, organically grown codebase containing tens of thousands of classes into well-defined modules with clear boundaries, while retaining backward compatibility, takes time. This is one of the reasons it took a long time to get a module system into Java. With over 20 years of legacy accumulated, many dubious dependencies had to be untangled. Going forward, this effort will definitely pay off in terms of development speed and increased flexibility for the JDK.

> 将 JDK 分解成模块需要完成大量的工作。将一个错综复杂、有机发展且包含数以万计类的代码库分解成边界清晰且保持向后兼容性的定义良好的模块需要花费大量的时间，这也就是为什么花了如此长的时间才将模块系统植入到 Java 中的原因。经过 20 多年的传统积累，许多存在疑问的依赖关系已经解开。展望未来，这一努力将在 JDK 的快速发展以及更大灵活性方面得到丰厚的回报。

#### INCUBATOR MODULES 孵化器模块

Another example of the improved evolvability afforded by modules is the concept of incubator modules, described in JEP 11 (JEP stands for Java Enhancement Proposal). Incubator modules are a means to ship experimental APIs with the JDK. With Java 9, for example, a new HttpClient API is shipped in the jdk.incubator.httpclient module (all incubator modules have the jdk.incubator prefix). You can depend on such incubator modules if you want to, with the explicit expectation that their APIs may still change. This allows the APIs to mature and harden in a real-world environment, so they can be shipped as a fully supported module in a later JDK release—or be removed, if the API isn’t successful in practice.

> 模块所提供的另一个改进示例是 JEP 11（http://openjdk.java.net/jeps/11 JEP 表示 JavaEnhancement Proposal）中所描述的孵化器模块（incubator module）概念。孵化器模块是一种使用 JDK 提供实验 API 的手段。例如，在使用 Java 9 时，jdk. incubator.httpclient 模块中提供了一个新的 HttpClient API（所有孵化器模块都具有 jdk.incubator 前缀）。如果愿意，可以使用这样的孵化器模块，并且明确地知道模块中 API 仍然可以更改。这样一来，就可以让这些 API 在真实环境中不断变得成熟和稳定，以便在日后的 JDK 版本中可以作为一个完全支持模块来使用或者删除（如果 API 在实践中不成功）。

## 2.2 Module Descriptors 模块描述符

Now that we have a high-level overview of the JDK module structure, let’s explore how modules work. What is a module, and how is it defined? A module has a name, it groups related code and possibly other resources, and is described by a module descriptor. The module descriptor lives in a file called module-info.java. Example 2-1 shows the module descriptor for the java.prefs platform module.

> 到目前为止，我们已经大概了解了 JDK 模块结构，接下来探讨一下模块的工作原理。什么是模块，它是如何定义的？模块拥有一个名称，并对相关的代码以及可能的其他资源进行分组，同时使用一个模块描述符进行描述。模块描述符保存在一个名为 module-info.java 的文件中。示例 2-1 显示了 java.prefs 平台模块的模块描述符。

Example 2-1. module-info.java

> 示例 2-1:module-info.java

```java
module java.prefs {
    requires java.xml; 1

    exports java.util.prefs; 2
}
```

- 1 The requires keyword indicates a dependency, in this case on module java.xml.
- 2 A single package from the java.prefs module is exported to other modules.

---

> 1 关键字 requires 表示一个依赖关系，此时表示对 java.xml 的依赖。
> 2 来自 java.prefs 模块的单个包被导出到其他模块。

Modules live in a global namespace; therefore, module names must be unique. As with package names, you can use conventions such as reverse DNS notation (e.g., com.mycompany.project.somemodule) to ensure uniqueness for your own modules. A module descriptor always starts with the module keyword, followed by the name of the module. Then, the body of module-info.java describes other characteristics of the module, if any.

> 模块都位于一个全局命名空间中，因此，模块名称必须是唯一的。与包名称一样，可以使用反向 DNS 符号（例如 com.mycompany.project.somemodule）等约定来确保模块的唯一性。模块描述符始终以关键字 module 开头，后跟模块名称。而 module-info.java 的主体描述了模块的其他特征（如果有的话）。

Let’s move on to the body of the module descriptor for java.prefs. Code in java.prefs uses code from java.xml to load preferences from XML files. This dependency must be expressed in the module descriptor. Without this dependency declaration, the java.prefs module would not compile (or run), as enforced by the module system. A dependency is declared with the requires keyword followed by a module name, in this case java.xml. The implicit dependency on java.base may be added to a module descriptor. Doing so adds no value, similar to how you can (but generally don’t) add "import java.lang.String" to a class using strings.

> 接下来看一下 java.prefs 模块描述符的主体。java.prefs 使用了 java.xml 中的代码从，以 XML 文件中加载首选项。这种依赖关系必须在模块描述符中表示。如果没有这个依赖关系声明，模块系统就无法编译（或运行）java.prefs 模块。声明依赖关系首先使用关键字 requires，然后紧跟模块名称（此时为 java.xml）。可以将对 java.base 的隐式依赖添加到模块描述符中。但这样做没有任何价值，就好比是将“import java.lang.String”添加到使用字符串的类中（通常并不需要这么做）。

A module descriptor can also contain exports statements. Strong encapsulation is the default for modules. Only when a package is explicitly exported, like java.util.prefs in this example, can it be accessed from other modules. Packages inside a module that are not exported are inaccessible from other modules by default. Other modules cannot refer to types in encapsulated packages, even if they have a dependency on the module. When you look at Figure 2-1, you see that java.desktop has a dependency on java.prefs. That means java.desktop is able to access only types in package java.util.prefs of the java.prefs module.

> 模块描述符还可以包含 exports 语句。强封装性是模块的默认特性。只有当显式地导出一个包时（比如示例中的 java.util.prefs），才可以从其他模块中访问该包。默认情况下，一个模块中若没有导出的包则无法被其他模块所访问。其他模块不能引用封装包中的类型，即使它们与该模块存在依赖关系。从图 2-1 中可以看到，java.desktop 依赖 java.prefs，这意味着 java.desktop 只能访问 java.prefs 模块的 java. util.prefs 包中的类型。

## 2.3 Readability 可读性

An important new concept when reasoning about dependencies between modules is readability. Reading another module means you can access types from its exported packages. You set up readability relations between modules through requires clauses in the module descriptor. By definition, every module reads itself. A module that requires another module reads the other module.

> 在推理模块之间的依赖关系时，需要注意的一个重要的新概念是可读性（readability），读取其他模块意味着可以访问其导出包中的类型。可以在模块描述符中使用 requires 子句设置模块之间的可读性关系。根据定义，每个模块都可以读取自己。而一个模块之所以读取另一个模块是因为需要该模块。

Let’s explore the effects of readability by revisiting the java.prefs module. In this JDK module in Example 2-2, the following class imports and uses classes from the java.xml module.

> 接下来，再次查看 java.prefs 模块，了解一下可读性的影响。在示例 2-2 的 JDK 模块中，XmlSupport 类导入并使用了 java.xml 模块中的类。

Example 2-2. Small excerpt from the class java.util.prefs.XmlSupport

> 示例 2-2：类 java.util.prefs.XmlSupport 的节选

```java
import org.w3c.dom.Document;
// ...

class XmlSupport {

  static void importPreferences(InputStream is)
      throws IOException, InvalidPreferencesFormatException
  {
      try {
          Document doc = loadPrefsDoc(is);
          // ...
        }
  }

  // ...
}
```

Here, org.w3c.dom.Document (among other classes) is imported. It comes from the java.xml module. Because the java.prefs module descriptor contains requires java.xml, as you saw in Example 2-1, this code compiles without issue. Had the author of the java.prefs module left out the requires clause, the Java compiler would report an error. Using code from java.xml in module java.prefs is a deliberate and explicitly recorded choice.

> 此时，导入了 org.w3c.dom.Document（以及其他类），该类来自 java.xml 模块。如示例 2-1 所示，由于 java.prefs 模块描述符包含了 requires java.xml，因此代码可以顺利完成编译。如果 java.prefs 模块的作者省略了 require 子句，那么 Java 编译器将报告错误。在模块 java.prefs 中使用来自 java.xml 的代码是一个经过深思熟虑并显式记录的选择。

## 2.4 Accessibility 可访问性

Readability relations are about which modules read other modules. However, if you read a module, this doesn’t mean you can access everything from its exported packages. Normal Java accessibility rules are still in play after readability has been established.

> 可读性关系表明了哪些模块可以读取其他模块，但读取一个模块并不意味着可以访问其导出包中的所有内容。在建立了可读性之后，正常的 Java 可访问性规则仍然会发挥作用。

Java has had accessibility rules built into the language since the beginning. Table 2-2 provides a refresher on the existing access modifiers and their impact.

> 从一开始，Java 就将可访问性规则内置到语言中。表 2-2 显示当前的访问修饰符及其影响。

Table 2-2. Access modifiers and their associated scopes

| Access modifier | Class | Package | Subclass | Unrestricted |
| --------------- | ----- | ------- | -------- | ------------ |
| public          | ✓     | ✓       | ✓        | ✓            |
| protected       | ✓     | ✓       | ✓        |              |
| - (default)     | ✓     | ✓       |          |              |
| private         | ✓     |         |          |              |

Accessibility is enforced at compile- and run-time. Combining accessibility and readability provides the strong encapsulation guarantees we so desire in a module system. The question of whether you can access a type from module M2 in module M1 becomes twofold:

> 可访问性在编译时和运行时被强制执行。可访问性和可读性的结合可以确保在模块系统中实现强封装性。是否可以在模块 M1 中访问模块 M2 中的类型已经成为一个双重问题：

- Does M1 read M2?
- If yes, is the type accessible in the package exported by M2?

---

> 1）M1 是否可以读取 M2？
> 2）如果可以，M2 导出包中的类型是否可以访问？

Only public types in exported packages are accessible in other modules. If a type is in an exported package but not public, traditional accessibility rules block its use. If it is public but not exported, the module system’s readability rules prevent its use. Violations at compile-time result in a compiler error, whereas violations at run-time result in IllegalAccessError.

> 在其他模块中，只能访问导出包中的公共类型。如果导出包中的一个类型不是公共的，那么传统的可访问性规则将不允许使用该类型。如果类型是公共的，但没有导出，那么模块系统的可读性规则将阻止使用该类型。编译时的违规会导致编译器错误，而运行时的违规会导致 IllegalAccessError。

#### IS PUBLIC STILL PUBLIC? public 仍然表示公开的吗？

No types from a nonexported package can be used by other modules—even if types inside that package are public. This is a fundamental change to the accessibility rules of the Java language.

> 其他模块无法使用未导出包中的任何类型——即使包中的类型是公共的。这是对 Java 语言可访问性规则的根本变化。

Until Java 9, things were quite straightforward. If you had a public class or interface, it could be used by every other class. As of Java 9, public means public only to all other packages inside that module. Only when the package containing the public type is exported can it be used by other modules. This is what strong encapsulation is all about. It forces developers to carefully design a package structure where types meant for external consumption are clearly separated from internal implementation concerns.

> 在 Java 9 出现之前，事情非常简单明了。如果有一个公共类或接口，那么其他类就可以对它进行访问。但自从 Java 9 出现之后，public 意味着仅对模块中的其他包公开。只有当导出包包含了公开类型时，其他模块才可以使用这些类型。这就是强封装的意义所在。它迫使开发人员仔细设计包结构，将需要外部使用的类型与内部实现过程分离开来。

Before modules, the only way to strongly encapsulate implementation classes was to keep them all in a single package and mark them package-private. Since this leads to unwieldy packages, in practice classes were made public just for access across different packages. With modules, you can structure packages any way you like and export only those that really must be accessible to the consumers of the module. Exported packages form the API of a module, if you will.

> 在模块出现之前，强封装实现类的唯一方法是将这些类放置到单个包中，并标记为私有。这种做法使得包变得非常笨重，实际上，将类公开只是为了实现不同包之间的访问。通过使用模块，可以以任何方式构建包，并仅导出模块使用者真正必须访问的包。如果愿意的话，还可以将导出的包构成模块的 API。

Another elephant in the room with regards to accessibility rules is reflection. Before the module system, an interesting but dangerous method called setAccessible was available on all reflected objects. By calling setAccessible(true), any element (regardless of whether it is public or private) becomes accessible. This method is still available but now abides by the same rules as discussed previously. It is no longer possible to invoke setAccessible on an arbitrary element exported from another module and expect it to work as before. Even reflection cannot break strong encapsulation.

> 关于可访问性规则的另一个方面是反射（reflection）。在模块系统出现之前，所有反射对象都有一个有趣而又危险的方法 setAccessible。通过调用 setAccessible(true)，任何元素（不管元素是公共或是私有）都会变为可访问。虽然该方法目前仍然可以使用，但必须遵守前面所讨论的规则。想要在另一个模块导出的任意元素上调用 setAccessible 并期望像以前一样工作是不可能的了（即使反射不会破坏强封装性）。

There are ways around the new accessibility rules imposed by the module system. Most of these workarounds should be viewed as migration aids and are discussed in Part II.

> 可以使用多种方法实现模块系统中新的可访问性规则，其中大多数解决方法应该被视为迁移的辅助工具，详细内容参见第二部分。

## 2.5 Implied Readability 隐式可读性

Readability is not transitive by default. We can illustrate this by looking at the incoming and outgoing read edges of java.prefs, as shown in Figure 2-2.

> 默认情况下，可读性是不可传递的。可以通过查看 java.prefs 的传入和传出“读取边”来说明这一点，如图 2-2 所示。

Readability is not transitive
Figure 2-2. Readability is not transitive: java.desktop does not read java.xml through java.prefs

Here, java.desktop reads java.prefs (among other modules, left out for clarity). We’ve already established that this means java.desktop can access public types from the java.util.prefs package. However, java.desktop cannot access types from java.xml through its dependency on java.prefs. It just so happens that java.desktop does use types from java.xml as well. That’s why java.desktop has its own requires java.xml clause in its module descriptor. In Figure 2-1, this dependency is also visible.

> 此时，java.desktop 可以读取 java.prefs（为了简单起见，排除了其他模块）。这意味着 java.desktop 可以访问 java.util.prefs 包中的公共类型。然而，java. desktop 不能通过与 java.prefs 的依赖关系来访问 java.xml 中的类型，但此时 java.desktop 确实使用了 java.xml 中的类型，这是因为 java.desktop 的模块描述符中使用了 requires java.xml 子句。从图 2-1 中也可以看到这个依赖关系。

Sometimes you do want read relations to be transitive—for example, when a type in an exported package of module M1 refers to a type from another module M2. In that case, modules requiring M1 and thereby referencing types from M2 cannot be used without reading M2 as well.

> 有时确实希望读取关系是可传递的，例如，当模块 M1 导出包中的一个类型需要引用模块 M2 中的一个类型时。此时，如果不能读取 M2，那么需要 M1 进而需要引用 M2 中类型的模块就无法使用了。

That sounds completely abstract, so an illustration is in order. A good example of this phenomenon can be found in the JDK’s java.sql module. It contains two interfaces (Driver, shown in Example 2-3, and SQLXML, shown in Example 2-4) defining method signatures whose result types come from other modules.

> 这听起来非常抽象，所以下面依次提供示例加以说明。可以在 JDK 的 java.sql 模块中找到该现象的一个好示例。该模块包含两个接口（如示例 2-3 所示的 Driver 以及如示例 2-4 所示的 SQLXML），定义了结果类型来自其他模块的方法签名。

Example 2-3. Driver interface (partially shown), allowing a Logger from the java.logging module to be retrieved

> 示例 2-3:Driver 接口（部分显示），允许检索 java.logging 模块中的 Logger

```java
package java.sql;

import java.util.logging.Logger;

public interface Driver {
  public Logger getParentLogger();
  // ..
}
```

Example 2-4. SQLXML interface (partially shown), with Source from module java.xml representing XML coming back from the database

> 示例 2-4 SQLXML 接口（部分显示），来自模块 java. xml 的源代码，表示从数据库返回的 XML

```java
package java.sql;

import javax.xml.transform.Source;

public interface SQLXML {
  <T extends Source> T getSource(Class<T> sourceClass);
  // ..
}
```

If you add a dependency on java.sql to your module descriptor, you can program to these interfaces, since they are in exported packages. But whenever you call get​ParentLogger or getSource, you get back values of a type not exported by java.sql. In the first case, you get a java.util.logging.Logger from java.logging, and in the second case, you get a javax.xml.transform.Source from java.xml. In order to do anything useful with these return values (assign to a local variable, call methods on them), you need to read those other modules as well.

> 如果在模块描述符中添加对 java.sql 的依赖，那么就可以对这些接口进行编程，因为它们都位于导出包中。但每当调用 getParentLogger 或 getSource 时，所获取的并不是由 java.sql 导出的类型的值。在第一种情况下，获取的是来自 java. logging 的 java.util.logging.Logger；而在第二种情况下，获取的是来自 java.xml 的 javax.xml.transform.Source。为了使用这些返回值完成一些有用的事情（比如分配给一个局部变量、调用其方法），还需要读取其他模块。

Of course, you can manually add dependencies on java.logging or java.xml, respectively, to your own module descriptor. But that’s hardly satisfying, especially since the java.sql author already knew the interfaces are unusable without readability on those other modules. Implied readability allows module authors to express this transitive readability relation in module descriptors.

> 当然，可以手动地在模块描述符中分别添加对 java.logging 或 java.xml 的依赖。但这样做是没有必要的，因为 java.sql 的作者已经意识到在其他模块上没有可读性的接口是不可用的。隐式可读性允许模块的作者在模块描述符中表达这种可传递的可读性关系。

For java.sql, it looks like this:

> 对于 java.sql，可完成如下修改：

```java
module java.sql {
    requires transitive java.logging;
    requires transitive java.xml;

    exports java.sql;
    exports javax.sql;
    exports javax.transaction.xa;
}
```

The requires keyword is now followed by the transitive modifier, slightly changing the semantics. A normal requires allows a module to access types in exported packages from the required module only. requires transitive means the same and more. In addition, any module requiring java.sql will now automatically be requiring java.logging and java.xml. That means you get access to the exported packages of those modules as well by virtue of these implied readability relations. With requires transitive, module authors can set up additional readability relations for users of the module.

> 现在，关键字 requires 后面紧跟着修饰符 transitive，从而略微改变了一下语义。常见的 requires 只允许一个模块访问所需模块导出包中的类型，而 requires transitive 的语义更丰富。现在，任何需要 java.sql 的模块都将自动需要 java. logging 和 java.xml，这意味着可以通过隐式可读性关系访问这些模块的导出包。通过使用 requires transitive，模块作者可以为模块用户设置额外的可读性关系。

From the consumer side, this makes it easier to use java.sql. When you require java.sql, you get access to the exported packages java.sql, javax.sql, and javax.transaction.xa (which are all exported by java.sql directly), but also to all packages exported by modules java.logging and java.xml. It’s as if java.sql re-exports those packages for you, courtesy of the implied readability relations it sets up with requires transitive. To be clear, there’s no such thing as re-exporting packages from other modules, but thinking about it this way may help you understand the effects of implied readability.

> 从使用者的角度来看，上述做法可以更容易地使用 java.sql。当你需要 java.sql 时，不仅可以访问导出包 java.sql、javax.sql 和 javax.transaction.xa（这些都直接由 java.sql 导出），还可以访问由模块 java.logging 和 java.xml 导出的所有包。这就好像这些包是由 java.sql 重新导出的一样，这一切都是使用 requires transitive 所设置的隐式可读性关系实现的。很明显，这并没有真的从其他模块重新导出包，但是这样考虑有助于理解隐式可读性的效果。

For an application module app using java.sql, this module definition suffices:

> 对于使用了 java.sql 模块的应用程序来说，其模块定义如下所示：

```java
module app {
  requires java.sql;
}
```

With this module descriptor, the implied readability edges in Figure 2-3 are in effect.

> 使用上述模块描述符生成如图 2-3 所示的隐式可读性边。

Implied readability (requires transitive) shown with double-lined edges.
Figure 2-3. The effect of implied readability (requires transitive) shown with bold edges

Implied readability on java.xml and java.logging (the bold edges in Figure 2-3) is granted to app because java.sql uses requires transitive (solid edges in Figure 2-3) for those modules. Because app does not export anything and uses only java.sql for its encapsulated implementation, a normal requires clause is enough (dashed edge in Figure 2-3). When you need another module for internal uses, a normal requires suffices. If, on the other hand, types from another module are used in exported types, requires transitive is in order. In “API Modules”, we’ll discuss how and when implied readability is important for your own modules in more detail.

> java.xml 和 java.logging 上的隐式可读性被授予给 app，因为 java.sql 针对这些模块使用了 requires transitive（如图 2-3 所示的实边）。因为 app 没有导出任何内容，同时仅使用 java.sql 以完成封装的实现，所以使用常见的 requires 子句就足够了（如图 2-3 所示的虚线）。当需要其他模块供内部使用时，使用 requires 就足够了。但另一方面，如果需要在导出类型中使用另一个模块中的类型，那么就要使用 requires transitive。在 5.3 节中，将会更加详细地讨论隐式可读性的重要性及其何时对模块重要。

Now’s a good time to take another look at Figure 2-1. All solid edges in that graph are requires transitive dependencies too. The dashed edges, on the other hand, are normal requires dependencies. A nontransitive dependency means the dependency is necessary to support the internal implementation of that module. A transitive dependency means the dependency is necessary to support the API of the module. These latter dependencies are more significant; hence they are depicted by solid lines in the diagrams in this book.

> 现在，可以再看看图 2-1。图中所有的实线边都是 requires transitive 依赖关系。而虚线边只是常见的 requires 依赖关系。非传递依赖（nontransitive dependency）意味着依赖关系是支持模块内部实现所必需的。可传递依赖（transitive dependency）则意味着依赖关系是支持模块 API 所必需的。后一种依赖关系更为重要，因此它们在本书的图形中用实线表示。

Looking at Figure 2-1 with these new insights, we can highlight another use case for implied readability: it can be used to aggregate several modules into a single new module. Take, for example, java.se. It’s a module that doesn’t contain any code and consists of just a module descriptor. In this module descriptor, a requires transitive clause is listed for each module that is part of the Java SE specification. When you require java.se in a module, you get access to all exported APIs of every module aggregated by java.se by virtue of implied readability:

> 使用以上新的观点来观察图 2-1，可以看到隐式可读性的另一个用例：可用来将多个模块聚合到一个新模块中。以 java.se 为例，该模块不包含任何代码，只由一个模块描述符构成。在该模块描述符中，针对每个模块（Java SE 规则的一部分）都列出了一个 requires transitive 子句。当需要在一个模块中使用 java.se 时，凭借隐式可读性，可以访问 java.se 所聚合的每个模块导出的所有 API：

```java
module java.se {
    requires transitive java.desktop;
    requires transitive java.sql;
    requires transitive java.xml;
    requires transitive java.prefs;
    // .. many more
}
```

Implied readability itself is transitive as well. Take another aggregator module in the platform, java.se.ee. Figure 2-1 shows that java.se.ee aggregates even more modules than java.se. It does so by using requires transitive java.se and adding several modules containing parts of the Java Enterprise Edition (EE) specification. Here’s what the java.se.ee aggregator module descriptor looks like:

> 隐式可读性自身也是可传递的。以平台中另一个聚合器模块 java.se.ee 为例。从图 2-1 可以看到，相比于 java.se, java.se.ee 聚合了更多的模块。它使用了 requires transitive java.se 并添加了多个模块（包含了 Java EE 规范的部分内容）。Java.se.ee 聚合器模块描述符的内容如下所示：

```java
module java.se.ee {
    requires transitive java.se;
    requires transitive java.xml.ws;
    requires transitive java.xml.bind;
    // .. many more
}
```

The requires transitive on java.se ensures that if java.se.ee is required, implied readability is also established to all modules aggregated by java.se. Furthermore, java.se.ee sets up implied readability to several EE modules.

> java.se 上的 requires transitive 确保一旦需要 java.se.ee，就会为由 java.se 聚合的所有模块建立隐式可读性。此外，java.se.ee 还会对多个 EE 模块设置了隐式可读性。

In the end, java.se and java.se.ee provide implied readability on a huge number of modules reachable through these transitive dependencies.

> 最终，java.se 和 java.se.ee 为大量的模块提供了隐式可读性，只需通过这些可传递依赖关系就可以访问这些模块。

TIP

Requiring java.se.ee or java.se in application modules is rarely the right thing to do. It means you’re effectively replicating the pre-Java 9 behavior of having all of rt.jar accessible in your module. Dependencies should be defined as fine-grained as possible. It pays to be more precise in your module descriptor and require only modules you actually use.

> 在应用程序模块中使用 java.se.ee 或 java.se 是不明智的，这意味着在模块中复制了 Java 9 之前可以访问的 rt.jar 的所有行为。依赖关系应尽可能细化。在模块描述符中要尽可能精确同时只添加实际使用的模块，这是非常重要的。

In “Aggregator Modules”, we’ll explore how the aggregator module pattern helps in modular library design.

> 在 5.4 节将探讨聚合器模块模式如何帮助模块化库设计。

## 2.6 Qualified Exports 限制导出

In some cases, you’ll want to expose a package only to certain other modules. You can do this by using qualified exports in the module descriptor. An example of a qualified export can be found in the java.xml module:

> 在某些情况下，可能只需要将包暴露给特定的某些模块。此时，可以在模块描述符中使用限制导出。可以在 java.xml 模块中找到限制导出的示例：

```java
module java.xml {
  ...
  exports com.sun.xml.internal.stream.writers to java.xml.ws
  ...
}
```

Here we see a platform module sharing useful internals with another platform module. The exported package is accessible only by the modules specified after to. Multiple module names, separated by a comma, can be provided as targets for a qualified export. Any module not mentioned in this to clause cannot access types in this package, even when they read the module.

> 此时可以看到一个与另一个平台模块共享有用的内部代码的平台模块。导出的包只能由 to 之后指定的模块访问。可以用由逗号分隔的多个模块名称作为限制导出的目标。to 子句中没有提到的任何模块都不能访问包中的类型，即使在读取模块时也是如此。

The fact that qualified exports exist doesn’t unequivocally mean you should use them. In general, avoid using qualified exports between modules in an application. Using them creates an intimate bond between the exporting module and its allowable consumers. From a modularity perspective, this is undesirable. One of the great things about modules is that you effectively decouple producers from consumers of APIs. Qualified exports break this property because now the names of consumer modules are part of the provider module’s descriptor.

> 限制导出的存在并不意味着就一定要使用它们。一般来说，应该避免在应用程序的模块之间使用限制导出。使用限制导出意味着在导出模块和允许的使用者之间建立了直接的联系。从模块化的角度来看，这是不可取的。模块之所以伟大，是因为它可以有效地对 API 的生产者与使用者进行解耦。而限制导出破坏了这个属性，因为现在使用者模块名称成为提供者模块描述符的一部分。

For modularizing the JDK, however, this is a lesser concern. Qualified exports have been indispensable to modularizing the platform with all of its legacy. Many platform modules encapsulate part of their code, expose some internal APIs through qualified exports to select other platform modules, and use the normal export mechanism for public APIs used in applications. By using qualified exports, platform modules could be made more fine-grained without duplicating code.

> 然而，对于模块化 JDK 来说，这只是一个小问题。如果想要使用遗留代码对平台进行模块化，那么限制导出是不可或缺的。许多平台模块封装了部分自身的代码，并通过限制导出公开了一些内部 API，以选择其他平台模块，同时对于应用程序中使用的公共 API 则使用正常的导出机制。通过使用限制导出，平台模块可以在不重复代码的情况下变得更细粒度。

## 2.7 Module Resolution and the Module Path 模块解析和模块路径

Having explicit dependencies between modules is not just useful to generate pretty diagrams. The Java compiler and runtime use module descriptors to resolve the right modules when compiling and running modules. Modules are resolved from the module path, as opposed to the classpath. Whereas the classpath is a flat list of types (even when using JAR files), the module path contains only modules. As you’ve learned, these modules carry explicit information on what packages they export, making the module path efficiently indexable. The Java runtime and compiler know exactly which module to resolve from the module path when looking for types from a given package. Previously, a scan through the whole classpath was the only way to locate an arbitrary type.

> 在模块之间建立显式依赖关系不仅仅有助于生成漂亮的图表。当编译和运行模块时，Java 编译器和运行时使用模块描述符来解析正确的模块。模块是从模块路径（module path）中解析出来的，而不是类路径。类路径是一个类型的平面列表（即使使用 JAR 文件也是如此），而模块路径只包含模块。如前所述，这些模块提供了导出包的显式信息，从而能够高效地对模块路径进行索引。当从给定的包中查找类型时，Java 运行时和编译器可以准确地知道从模块路径中解析哪个模块。而在以前，对整个类路径进行扫描是找到任意类型的唯一方法。

When you want to run an application packaged as a module, you need all of its dependencies as well. Module resolution is the process of computing a minimal required set of modules given a dependency graph and a root module chosen from that graph. Every module reachable from the root module ends up in the set of resolved modules. Mathematically speaking, this amounts to computing the transitive closure of the dependency graph. As intimidating as it may sound, the process is quite intuitive:

> 当需要运行一个打包成模块的应用程序时，还需要使用其所有的依赖项。模块解析是根据给定的依赖关系图并从该图中选择一个根模块（root module）来计算最低需求的模块集的过程。从根模块可访问的每个模块最终都在解析模块集中。在数学上讲，这相当于计算依赖关系图的传递闭包（transitive closure）。尽管听起来很吓人，但这个过程还是非常直观的：

1. Start with a single root module and add it to the resolved set.
2. Add each required module (requires or requires transitive in module-info.java) to the resolved set.
3. Repeat step 2 for each new module added to the resolved set in step 2.

---

> 1）首先从一个根模块开始，并将其添加到解析集中。
> 2）向解析集添加所需的模块（module-info.java 中的 requires 或 requires transitive）。
> 3）重复第 2 步，将新的模块添加到解析集中。

This process is guaranteed to terminate because we repeat the process only for newly discovered modules. Also, the dependency graph must be acyclic. If you want to resolve modules for multiple root modules, apply the algorithm to each root module, and then take the union of the resulting sets.

> 上述过程不会无限制地进行下去，因为只有发现了新的模块才会重复该过程。此外，依赖关系图必须是非循环的。如果想要为多个根模块解析模块，则需要首先将该算法应用于每个根模块，然后再合并结果集。

Let’s try this with an example. We have an application module app that will be the root module in the resolution process. It uses only java.sql from the modular JDK:

> 接下来尝试一个示例。此时有一个应用程序模块 app，它是解析过程中的根模块，并且只使用了来自模块化 JDK 的 java.sql：

```java
module app {
    requires java.sql;
}
```

Now we run through the steps of module resolution. We omit java.base when considering the dependencies of modules and assume it always is part of the resolved modules. You can follow along by looking at the edges in Figure 2-1:

> 现在，完成模块解析过程。当考虑到模块的依赖关系时，忽略 java.base，并假定它始终是解析模块的一部分。可以根据图 2-1 所示的边来完成下面的步骤：

1. Add app to the resolved set; observe that it requires java.sql.
2. Add java.sql to the resolved set; observe that it requires java.xml and java.logging.
3. Add java.xml to the resolved set; observe that it requires nothing else.
4. Add java.logging to the resolved set; observe that it requires nothing else.
5. No new modules have been added; resolution is complete.

---

> 1）向解析集添加 app；并观察到它需要使用 java.sql。
> 2）向解析集添加 java.sql；并观察到它需要 java.xml 和 java.logging。
> 3）向解析集添加 java.xml；并观察到它不再需要其他模块。
> 4）向解析集添加 java.logging，并观察到它不再需要其他模块。
> 5）不再需要添加任何新模块；解析过程结束。

The result of this resolution process is a set containing app, java.sql, java.xml, java.logging, and java.base. When running app, the modules are resolved in this way, and the module system gets the modules from the module path.

> 该解析过程的结果是生成了一个包含 app、java.sql、java.xml、java.logging 和 java.base 的集合。当运行 app 时，会按照上述过程解析模块，模块系统从模块路径中获取模块。

Additional checks are performed during this process. For example, two modules with the same name lead to an error at startup (rather than at run-time during inevitable classloading failures). Another check is for uniqueness of exported packages. Only one module on the module path may expose a given package. “Split Packages” discusses problems with multiple modules exporting the same packages.

> 在解析过程中还会完成一些额外的检查。例如，具有相同名称的两个模块在启动时（而不是在运行过程出现类加载失败时）会产生错误。此外，还会检查导出包的唯一性。模块路径上只有一个模块可以公开一个给定的包。5.5.1 节将讨论多个模块导出相同包的问题。

#### VERSIONS 版本

So far, we’ve discussed module resolution without mentioning versions. That may seem odd, since we’re used to specifying versions with dependencies in, for example, Maven POMs. It is a deliberate design decision to leave version selection outside the scope of the Java module system. Versions do not play a role during module resolution. In “Versioned Modules” we discuss this decision in greater depth.

> 前面已经讨论了模块的解析过程，却没有提及版本问题。这看上去可能很奇怪，因为我们习惯于指定具有依赖关系的版本，如 Maven POM。将版本选择（version selection）放置在 Java 模块系统的范围之外是一个经过深思熟虑的设计决策。在模块解析过程中版本不起任何作用，5.7 节将会深入讨论这一决策。

The module resolution process and additional checks ensure that the application runs in a reliable environment and is less likely to fail at run-time. In Chapter 3, you’ll learn how to construct a module path when compiling and running your own modules.

> 模块解析过程以及额外的检查确保了应用程序在一个可靠的环境中运行，降低了运行时失败的可能性。在第 3 章，将会学习如何在编译和运行自己的模块时构建一个模块路径。

## 2.8 Using the Modular JDK Without Modules 在不使用模块的情况下使用模块化 JDK

You’ve learned about many new concepts the module system introduces. At this point, you may be wondering how this all affects existing code, which obviously is not modularized yet. Do you really need to convert your code to modules to start using Java 9? Fortunately not. Java 9 can be used like previous versions of Java, without moving your code into modules. The module system is completely opt-in for application code, and the classpath is still alive and kicking.

> 到目前为止，已经学习了模块系统所引入的许多新概念。此时，你可能想知道模块系统如何影响现有的代码，很显然，这些代码并没有实现模块化。难道真的需要将代码转换为模块才能开始使用 Java 9 吗？幸好不是这样。Java 9 可以像以前的 Java 版本一样使用，而不必将代码移动到模块中。模块系统完全适用于现有的应用程序代码，类路径仍然可以使用。

Still, the JDK itself does consist of modules. How are these two worlds reconciled? Say you have the piece of code in Example 2-5.

> 但 JDK 本身由模块组成。这两个世界是如何调解的呢？接下来看一下示例 2-5 所示的代码片段。

Example 2-5. NotInModule.java

> 示例 2-5:NotInModule.java

```java
import java.util.logging.Level;
import java.util.logging.Logger;
import java.util.logging.LogRecord;


public class NotInModule {

  public static void main(String... args) {
    Logger logger = Logger.getGlobal();
    LogRecord message = new LogRecord(Level.INFO, "This still works!");
    logger.log(message);
  }

}
```

It’s just a class, not put into any module. The code clearly uses types from the java.logging module in the JDK. However, there is no module descriptor to express this dependency. Still, when you compile this code without a module descriptor, put it on the classpath, and run it, it will just work. How can this be? Code compiled and loaded outside a module ends up in the unnamed module. In contrast, all modules you’ve seen so far are explicit modules, defining their name in module-info.java. The unnamed module is special: it reads all other modules, including the java.logging module in this case.

> 上面所示的仅仅是一个类，而不是任何模块。该代码使用了来自 JDK 的 java. logging 模块中的类型，但却没有任何模块描述符来表示这种依赖关系。当编译没有模块描述符的代码，然后将其放置到类路径并运行时，代码可以正常工作。怎么会这样呢？在一个模块之外编译和加载的代码最终都放在未命名模块（unnamed module）中。相比之下，目前所看到的模块都是显式模块，并在 module-info.java 中定义了它们的名称。未命名模块非常特殊：它可以读取所有其他模块，包括此例所读取的 java.logging 模块。

Through the unnamed module, code that is not yet modularized continues to run on JDK 9. Using the unnamed module happens automatically when you put your code on the classpath. That also means you are still responsible for constructing a correct classpath yourself. Almost all guarantees and benefits of the module system we have discussed so far are voided when working through the unnamed module.

> 通过使用未命名模块，尚未模块化的代码可以继续在 JDK 9 上运行。当将代码放在类路径上时，会自动使用未命名模块。这也意味着需要构建一个正确的类路径。可一旦使用了未命名模块，前面讨论的模块系统所带来的保障和好处也就没有了。

You need to be aware of two more things when using the classpath in Java 9. First, because the platform is modularized, it strongly encapsulates internal implementation classes. In Java 8 and earlier, you could use these unsupported internal APIs without repercussions. With Java 9, you cannot compile against encapsulated types in platform modules. To aid migration, code compiled on earlier versions of Java using these internal APIs continues to run on the JDK 9 classpath for now.

> 当在 Java 9 中使用类路径时，还需要注意两件事情。首先，由于平台是模块化的，因此对内部实现类进行了强封装。在 Java 8 和更早的版本中，可以使用这些不被支持的内部 API，而不会有任何不良影响。但如果使用 Java 9，则不能对平台模块中的封装类型进行编译。为了便于迁移，使用了内部 API 在早期版本的 Java 上编译的代码现在可以继续在 JDK 9 类路径上运行。

WARNING

When running (as opposed to compiling) an application on the JDK 9 classpath, a more lenient form of strong encapsulation is activated. All internal classes that were accessible on JDK 8 and earlier remain accessible at run-time on JDK 9. A warning is printed when these encapsulated types are accessed through reflection.

> 当在 Java 9 类路径上运行（而不是编译）一个应用程序时，使用了更为宽松的强封装形式。在 JDK 8 以及更早版本上可访问的所有内部类在 JDK 9 运行时上都是可访问的。但是当通过反射访问这些封装类型时，会出现警告信息。

The second thing to be aware of when compiling code in the unnamed module is that java.se is taken as the root module during compilation. You can access types from any module reachable through java.se and it will work, as shown in Example 2-5. Conversely, this means modules under java.se.ee but not under java.se (such as java.corba and java.xml.ws) are not resolved and therefore not accessible. One of the most prominent examples of this policy is the JAXB API. The rationale behind both restrictions, and how to approach them, is discussed in more detail in Chapter 7.

> 在未命名模块中编译代码时需要注意的第二件事是编译期间 java.se 将作为根模块。如示例 2-5 所示，可以通过 java.se 访问任何可访问模块中的类型。这意味着 java. se.ee（而不是 java.se）下的模块（比如 java.corba 和 java.xml.ws）都无法解析，因此也就无法访问。此策略最突出的示例之一就是 JAXB API。以上两种限制背后的原因以及处理方法将在第 7 章更详细地讨论。

In this chapter, you’ve seen how the JDK has been modularized. Even though modules play a central role in JDK 9, they are optional for applications. Care has been taken to ensure that applications running on the classpath before JDK 9 continue to work, but there are some caveats, as you’ve seen. In the next chapter, we’ll take the module concepts discussed so far and use them to build our own modules.

> 在本章，主要学习了如何对 JDK 进行模块化。虽然模块在 JDK 9 中起到了核心作用，但却是应用程序的可选项。如前所示，虽然目前已经采取谨慎措施，以确保在 JDK 9 之前的类路径上运行的应用程序可以继续运行，但也有一些注意事项。在下一章中，将会更加详细地讨论前面所介绍的模块概念，并使用它们来构建自己的模块。
