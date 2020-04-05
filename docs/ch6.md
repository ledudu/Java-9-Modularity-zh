# 第 6 章 高级模块化模式

> Chapter 6. Advanced Modularity Patterns

The previous chapter presented general design guidelines and patterns for modular application development. This chapter contains more advanced patterns and module system APIs that may not apply to everyday development. Still, it is an important part of the module system. Not only is the module system meant to be used directly by application developers, but it also serves as a basis for other frameworks to build upon. The advanced APIs primarily revolve around such usage.

> 前一章介绍了模块化应用程序开发的通用设计指南和模式，本章包含了更多可能不适用于日常开发的高级模式和模块系统 API。不过，它们也是模块系统的重要组成部分。模块系统不仅可以被应用程序开发人员直接使用，而且还可以作为其他框架的基础，而高级 API 主要是用于后一种用法。

The next section explores the need for reflection that many libraries and frameworks currently have. Open modules and packages are introduced as a feature to relax strong encapsulation at run-time. It’s an important feature during migration too, so it will be revisited in Chapter 8.

> 下一节将讨论目前许多库和框架所使用的反射。将开放式模块和包作为一项功能引入，可以在运行时减弱强封装性，这也是迁移过程中的一个重要功能，所以在第 8 章中还会重新讨论。

After open modules and packages, the focus shifts to patterns for dynamically extensible applications. Think of plug-in-based systems or application containers. Central to these systems is the challenge of adding modules at run-time, instead of working only with a fixed configuration of modules from the module path.

> 在介绍完开放式模块和包之后，关注点将转移到动态扩展应用程序的模式。思考一下基于插件的系统或应用程序容器，这些系统的核心是在运行时添加模块所面临的挑战，而不仅仅是使用来自模块路径中的模块的固定配置。

TIP

Feel free to skip this latter part of the chapter when first learning about the module system. Most applications get by fine without ever using the module system’s more dynamic and advanced features. You can always come back later to gain a better understanding of these features after getting more experience with typical modular scenarios first.

> 如果是第一次学习模块系统，可以跳过本章的后半部分，大多数应用程序都不会使用模块系统中更动态和更高级的功能。在获得了有关典型模块化方案的更多经验之后，可以再返回阅读本章的后半部分，从而更好地了解这些功能。

## 6.1 Strong Encapsulation Revisited 重温强封装性

We discussed the virtues of strong encapsulation at length in the previous chapters. In general, it is beneficial to be strict about what is and what is not a public, exported API. But in practice, the line isn’t drawn quite as clearly. Many libraries and frameworks exist that rely on accessing implementation classes of your application to do their work. Think of serialization libraries, object-relational mappers, and dependency injection frameworks. All these libraries want to manipulate classes that you’d otherwise deem internal implementation details.

> 在前面的章节中已经详细讨论了强封装的优点。一般来说，严格限制哪些是公开导出的 API 大有裨益。但实际上，限制的依据是很难确定的，存在许多依靠访问应用程序的实现类来完成工作的库和框架，如序列化库、对象关系映射器以及依赖注入框架。所有这些库都希望操作那些应该是内部实现细节的类。

An object-relational mapper or serialization library needs to access entity classes so it can instantiate them and fill them with the right data. Even if an entity class never leaves the module, the ORM library module somehow needs access. Dependency injection frameworks, as another example, need to inject service instances into service implementation classes. Exporting just the interfaces is not enough. By strongly encapsulating implementation classes within modules, these frameworks are denied the usual access they previously enjoyed on the classpath.

> 对象关系映射器或序列化库需要访问实体类，以便可以实例化它们并填充正确的数据。即使实体类从不离开模块，ORM 库模块也需要访问它们。又如，依赖注入框架需要将服务实例注入服务实现类中，仅导出接口是不够的。通过强封装模块中的实现类，这些框架无法实现以前在类路径上所享有的访问。

Reflection is almost without exception the tool of choice for these frameworks. Reflection is an important part of the Java platform, allowing code to inspect code at run-time. If that sounds a bit esoteric, that’s because it is. Using reflection in application code is not something to strive for. Still, without reflection, many generic frameworks (such as Hibernate or Spring) would not have been possible. But even reflection cannot break the walls of strong encapsulation around nonexported packages in modules, as you have learned in previous chapters.

> 反射毫无疑问是这些框架的首选工具。反射是 Java 平台的重要组成部分，允许代码在运行时检查代码。也许这听起来有点深奥，那是因为反射确实比较难理解。虽然在应用程序代码中并不一定要使用反射，但如果没有反射，许多通用框架（比如 Hibernate 或 Spring）是无法实现的。但正如前面的章节中所学到的，即使是反射也不能破坏模块中非导出包周围强封装所形成的壁垒。

The wrong reaction here is to export those packages indiscriminately. Exporting a package means its API can be compiled against and relied upon from different modules. That’s not what we want to express in this case. We need a mechanism to indicate that it’s OK for some libraries to get (reflective) run-time access to certain types.

> 此时，某些人可能会不加选择地导出所需的包。导出包意味着其 API 可以根据不同的模块进行编译和依赖。这并不是所希望的。此时需要一种机制来指示某些库可以获得（反射）某些类型的运行时访问权限。

### 6.1.1 Deep Reflection 深度反射

Two orthogonal issues need to be addressed to make traditional reflection-based libraries play nice with strong encapsulation:

> 为了让传统的基于反射的库更好地利用强封装性，需要解决两个正交问题：

- Provide access to internal types without exporting packages.
- Allow reflective access to all parts of these types.

---

> - 在不导出包的情况下访问内部类型。
> - 允许反射访问这些类型的所有部分。

We’ll zoom in on the second problem first. Let’s assume we export a package so a library can get access. You already know that means we can compile against just the public types in those packages. But does it also mean we can use reflection to break into private parts of those types at run-time? This practice of deep reflection is used by many libraries. For example, Spring or Hibernate inject values into nonpublic fields of classes.

> 先解决第二个问题。假设导出一个可以让库访问的包。如前所述，这意味着可以针对包中的公共类型进行编译，但这是否也意味着可以在运行时使用反射来“闯入”这些类型的私有部分？这种深度反射（deep reflection）的做法被许多库所使用。例如，Spring 或 Hibernate 使用该方法将值注入类的非公共字段中。

Back to our question: can you do deep reflection on public types from exported packages? The answer is no. It turns out that even if a type is exported, this does not mean you can unconditionally break into private parts of those types with reflection.

> 回到前面的问题：能否从导出包对公共类型进行深度反射？答案是否定的。事实证明，即使一个类型被导出，也并不意味着可以无条件地通过反射“闯入”这些类型的私有部分。

From a modularity perspective, this is the right thing to do. When arbitrary modules can break into private parts of exported types, they will do so. In effect, those private parts become part of the official API again. The same scenario played out already in the JDK itself, as discussed in “Using the Modular JDK Without Modules”.

> 从模块化的角度来看，这是正确的做法。当任意模块可以“闯入”导出类型的私有部分时，它们就会这样做。实际上，那些私有部分再次成为官方 API 的一部分。如 2.8 节所述，JDK 本身已经出现了类似情况。

Preventing access to nonpublic parts is not only a matter of API hygiene: private fields tend to be private for good reasons. For example, there is a java.security.KeyStore class in the JDK that manages keys and credentials. The authors of this class specifically do not want anyone to access the private fields guarding those secrets!

> 防止访问非公开部分不仅是 API 是否“卫生”的问题：出于很多原因的考虑，私有的字段往往是私有的。例如，JDK 中有一个用于管理密钥和凭证的 java.security.Key Store 类，该类的作者特别不希望任何人访问保护这些秘密的私有字段！

Exporting a package does not allow a module using those exported types to reflect over nonpublic parts. Deep reflection is supported in Java by the setAccessible method that is available on reflective objects. It circumvents checks that otherwise block access to inaccessible parts. Before the module system and strong encapsulation, setAccessible would basically never fail. With the module system, the rules have changed. Figure 6-1 shows which scenarios won’t work anymore.

> 导出一个包并不意味着允许使用这些导出类型的模块反射包中的非公共部分。在 Java 中，深度反射由反射对象上的 setAccessible 方法提供支持。它绕过了检查，从而可以访问不可访问的部分。在模块系统和强封装出现之前，setAccessible 基本上不会失败。但如果使用了模块系统，那么规则就发生了变化。图 6-1 显示了不再适用的场景。

<Figures figure="6-1">Module deepreflection exports a package containing class Exported and encapsulates the NotExported class. The snippets (assumed to be in another module) show reflection works only on public parts of exported types. Only Exported::doWork can be accessed reflectively; all others lead to exceptions.</Figures>

So we’re left in the situation where many popular libraries want to perform deep reflection, whether a type is exported or not. Because of strong encapsulation, they can’t. And even if implementation classes were exported, deep reflection on nonpublic parts is prohibited.

> 不管类型是否导出，许多流行的库都希望可以进行深度反射。但由于强封装性，它们很难做到这一点。而且，即使导出了实现类，也会禁止对非公共部分进行深度反射。

### 6.1.2 Open Modules and Packages 开放式模块和包

What we need is a way to make types accessible for deep reflection at run-time without exporting them. With such a feature, the frameworks can do their work again, while strong encapsulation is still upheld at compile-time.

> 此时需要的是一种可以在不导出类型的情况下在运行时让类型可用于深度反射的方法。一旦具备了这样的功能，框架可以完成自己的工作，同时在编译时仍然可以保持强封装性。

Open modules offer exactly this combination of features. When a module is open, all its types are available for deep reflection by other modules at run-time. This property holds regardless of whether any packages are exported.

> 开放式模块提供了这些功能的组合。当开放一个模块时，所有类型都可以在运行时被其他模块深度反射。无论是否导出包，该属性都是成立的。

Making a module open is done by putting the open keyword in the module descriptor:

> 只需在模块描述符中添加关键字 open，就可以开放一个模块：

```java
open module deepreflection {
  exports api;
}
```

All previous failure modes from Figure 6-1 disappear when a module is open, as shown in Figure 6-2.

> 当开放了一个模块后，如图 6-1 所示的失败模式都消失了，如图 6-2 所示。

The open keyword opens all packages in a module for deep reflection. In addition to a package being open, it can also be exported, as is the case for the api package containing the class Exported in this example. Any module reflectively accessing nonpublic elements from Exported or NotExported can do so after calling setAccessible. Readability to the module deepreflection is assumed by the JVM when using reflection on its types, so no special code has to be written for this to work. At compile-time, NotExported still is inaccessible, while Exported is accessible because of the exports clause in the module descriptor. From the application developer’s point of view, the NotExported class is still strongly encapsulated at compile-time. From the framework’s perspective, the NotExported class is freely accessible at run-time.

> 为了进行深度反射，open 关键字开放了模块中的所有包。除了开放包之外，还可以导出包，就像本例中导出包含 Exported 类的 api 包一样。在调用 setAccessible 之后，任何模块都可以反射访问 Exported 或 NotExported 中的非公共元素。当 JVM 在模块 deepreflection 的类型上使用反射时，会假定对该模块具有可读性，因此不需要编写特殊的代码就可以工作。在编译时，NotExported 仍然是不可访问的，而 Exported 因模块描述符中的 exports 子句而可以访问。从应用程序开发人员的角度来看，NotExported 类在编译时仍被强封装。但从框架的角度来看，NotExported 类在运行时是可以自由访问的。

TIP

With Java 9, two new methods for reflection objects are added: canAccess and trySetAccessible (both are defined in java.lang.reflect.AccessibleObject). These methods take into account the new reality where deep reflection is not always allowed. You can use these methods instead of dealing with exceptions from setAccessible.

> 在 Java 9 中，添加了两个反射对象的新方法：canAccess 和 trySetAccessible（都是在 java.lang.reflect.AccessibleObject 中定义的）。这些方法考虑到这样一个新的现实，即深度反射并不总是被允许的。你可以使用这些方法，而不是处理来自 setAccessible 的异常。

<Figures figure="6-2">With an open module, all types in all packages are open for deep reflection at run-time. No exceptions are thrown when performing deep reflection from another module.</Figures>

Opening a whole module is somewhat crude. It’s convenient for when you can’t be sure what types are used at run-time by a library or framework. As such, open modules can play a crucial role in migration scenarios when introducing modules into a codebase. More on this in Chapter 8.

> 开放整个模块看似有点不妥。但是当不能确定在运行时库或框架使用什么类型时，这种做法是很方便的。因此，当将模块引入代码库时，开放式模块可以在迁移场景中发挥至关重要的作用。更多内容请参阅第 8 章。

However, when you know which packages need to be open (and in most cases, you should), you can selectively open them from a normal module:

> 然而，当知道需要开放哪些包时（大多数情况下都应该知道），可以有选择性地从一个普通模块中开放所需的包：

```java
module deepreflection {
  exports api;
  opens internal;
}
```

Notice the lack of open before the module keyword. This module definition is not equivalent to the previous open module definition. Here, only the types in package internal are open for deep reflection. Types in api are exported, but are not open for deep reflection. At run-time this module definition provides somewhat stronger encapsulation than the fully open module, since no deep reflection is possible on types in package api.

> 请注意在关键字 module 之前缺少了关键字 open。该模块定义不等于前面的开放式模块定义。此时，只有包 internal 中的类型可用于深度反射。虽然 api 中的类型被导出，但是不能被深度反射。在运行时，该模块定义比完全开放的模块提供了更强的封装，因为无法在包 api 中的类型上进行深度反射。

We can make this module definition equivalent to the open module by also opening the api package:

> 可以通过开放 api 包，从而使这个模块定义等同于开放式模块：

```java
module deepreflection {
  exports api;
  opens api;
  opens internal;
}
```

A package can be both exported and open at the same time.

> 一个包可以同时导出和开放。

NOTE

In practice, this combination is a bit awkward. Try to design exported packages in such a way that other modules don’t need to perform deep reflection on them.

> 实际上，这种组合有点尴尬。一旦设计了导出包，其他模块就不需要对它们进行深度反射了。

opens clauses can be qualified just like exports:

> opens 子句可以像 exports 子句那样被限定：

```java
module deepreflection {
  exports api;
  opens internal to library;
}
```

The semantics are as you’d expect: only module library can perform deep reflection on types from package internal. A qualified opens reduces the scope to just one or more other explicitly mentioned modules. When you can qualify an opens statement, it’s better to do so. It prevents arbitrary modules from snooping on internal details through deep reflection.

> 此时的语义与期望的一样：只有模块 library 可以对包 internal 中的类型进行深度反射。合适的 opens 可以将范围缩小到一个或多个明确提到的模块。如果可以限定 opens 声明，那么最好这样做。这样一来，就可以防止任意模块通过深度反射窥探内部细节信息。

Sometimes you need to be able to perform deep reflection on a third-party module. In some cases, libraries even want to reflectively access private parts of JDK platform modules. It’s not possible to just add the open keyword and recompile the module in these cases. For these scenarios, a command-line flag is introduced for the java command:

> 有时需要对第三方模块进行深度反射。在某些情况下，有些库甚至想反射访问 JDK 平台模块的私有部分。此时添加 open 关键字并重新编译模块是不可能的。针对这些情况，为 Java 命令引入了一个命令行标志：

```
--add-opens <module>/<package>=<targetmodule>
```

This is equivalent to a qualified opens for package in module to targetmodule. If, for example, a framework module myframework wants to use nonpublic parts of java.lang.ClassLoader, you can do so by adding the following option to the java command:

> 上面的命令行标志等同于将 module 中的 package 限定开放给 targetmodule。例如，如果框架模块 myframework 想要使用 java.lang.ClassLoader 的非公共部分，则可以将以下选项添加到 Java 命令：

```java
--add-opens java.base/java.lang=myframework
```

This command-line option should be considered an escape hatch, especially useful during migration of code that’s not written with the module system in mind yet. In Part II, this option and others like it will reappear for these purposes.

> 该命令行选项应该被认为是一个逃生入口，在对那些没有使用模块系统编写的代码进行迁移时该选项是非常有用的。在第二部分中，还会重复出现该选项以及其他类似的选项。

### 6.1.3 Dependency Injection 依赖注入

Open modules and packages are the gateway to supporting existing dependency injection frameworks in the Java module system. In a fully modularized application, a dependency injection framework relies on open packages to access nonexported types.

> 在 Java 模块系统中，开放式模块和包是支持现有依赖注入框架的关键。在完全模块化的应用程序中，依赖注入框架使用开放式包来访问非导出类型。

#### REPLACING REFLECTION 取代反射

Java 9 offers an alternative to reflection-based access of frameworks to nonpublic class members in applications: MethodHandles and VarHandles. The latter are introduced in Java 9 through JEP 193. Applications can pass a java.lang.invoke.Lookup instance with the right permissions to the framework, explicitly delegating private lookup capabilities. The framework module can then, using MethodHandles.privateLookupIn(Class, Lookup), access nonpublic members on classes from the application module. It is expected that frameworks, in time, move to this more principled and performance-friendly approach to access application internals. An example of this approach can be found in the code accompanying this chapter (➥ chapter6/lookup).

> Java 9 为应用程序中非公共类成员的基于反射的框架访问提供了一种替代方案：MethodHandles 和 VarHandles。后者是通过 JEP193（http://openjdk.java.net/jeps/193）在Java 9 中引入的。应用程序可以将具有适当权限的 java.lang.invoke.Lookup 实例传递给框架，显式委派私有查找功能。然后，框架模块使用 MethodHandles.privateLookupIn（Class, Lookup）访问应用程序模块类中的非公共成员。随着时间的推移，框架将会转向使用更具原则性和性能友好的方法来访问应用程序内部构件。可以在本章附带的代码中找到该方法的一个示例（chapter6/lookup）。

To illustrate the abstract concept of open modules and packages, we’ll look at a concrete example. Instead of using services with the module system as described in Chapter 4, this example features a fictional third-party dependency injection framework in a module named spruice. Figure 6-3 shows the example. An open package is indicated by an “open” label in front of the types of a package.

> 为了说明开放式模块和包的抽象概念，接下来看一个具体的示例。该示例没有像第 4 章所描述的那样在模块系统中使用服务，而是在一个名为 spruice 的模块中提供了一个虚构的第三方依赖注入框架。图 6-3 显示了该示例。在包类型前面添加一个“open”标签来表示开放式图。

Our example application covers two domains: orders and customers. These are clearly visible as separate modules, with the customers domain split into an API and implementation module. The main module uses both services but doesn’t want to be coupled to the implementation details of either service. Both service implementation classes are encapsulated to that end. Only the interfaces are exported and accessible to the main module.

> 该示例应用程序涵盖两个领域：orders 和 customers。显而易见，它们都是单独的模块，同时，customers 域拆分为 API 和实现模块。main 模块使用了这两种服务，但又不希望与这两种服务的实现细节产生任何耦合。为此，对这两个服务实现类进行了封装，只有接口被导出并可以被 main 模块访问。

<Figures figure="6-3">Overview of an application main using a dependency injection library spruice. Requires relations between modules are shown as solid edges, and run-time deep reflection on types in open packages is shown as dashed edges.</Figures>

So far, the story is quite similar to the approach described in Chapter 4 on services. The main module is nicely decoupled from implementation specifics. However, this time we’re not going to provide and consume the services through ServiceLoader. Rather, we are going to use the DI framework to inject OrderService and CustomerService implementations into Application. Either we configure spruice explicitly with the correct wiring of services, or we use annotations, or a combination of both approaches. Annotations such as @Inject are often used to identify injection points that are matched on type with injectable instances:

> 到目前为止，该示例所采用的方法与第 4 章关于服务的方法非常相似。main 模块与实现细节实现了很好的解耦。但此时并不会通过 ServiceLoader 来提供和消费这些服务，相反，将使用 DI 框架将 OrderService 和 CustomerService 实现注入 Application 中。要么使用正确的服务布线来显式配置 spruice，要么使用注释，又或者结合使用这两种方法。@Inject 之类的注释通常用于标识与可注入实例类型匹配的注入点：

```java
public class Application {

  @Inject
  private OrderService orderService;

  @Inject
  private CustomerService customerService;

  public void orderForCustomer(String customerId, String[] productIds) {
    Customer customer = customerService.find(customerId)
    orderService.order(customer, productIds);
  }

  public static void main(String... args) {
    // Bootstrap Spruice and set up wiring
  }
}
```

In the main method of the Application class, spruice APIs are used to bootstrap the DI framework, Therefore, the main module needs a dependency on spruice for its wiring API. Service implementation classes must be instantiated by spruice and injected into the annotated fields (constructor injection is another viable alternative). Then, Application is ready to receive calls on orderForCustomer.

> 在 Application 类的 main 方法中，使用了 spruice API 来引导 DI 框架，因此，main 模块需要依赖 spruice 来实现其布线 API。服务实现类必须通过 spruice 进行实例化并注入注释字段（构造函数注入是另一个可行的选择）。然后，Application 准备好接收对 orderForCustomer 的调用。

Unlike the module system’s services mechanism, spruice does not have special privileges to instantiate encapsulated classes. What we can do is add opens clauses for the packages that need to be instantiated or injected. This allows spruice to access those classes at run-time and perform deep reflection where necessary (e.g., to instantiate and inject the OrderServiceImpl class into the orderService private field of Application). Packages that are used only internally in a module, such as cust.internal in the customers module, don’t need to be opened. The opens clauses could be qualified to open to spruice only. Unfortunately, that also ties our orders and customers modules to this specific DI framework. An unqualified opens leaves room for changing DI implementations without recompiling those modules later.

> 与模块系统的服务机制不同，spruice 没有实例化封装类的特殊权限。可以做的是为需要实例化或注入的包添加 opens 子句，从而允许 spruice 在运行时访问这些类，并在必要时执行深度反射（例如，将 OrderServiceImpl 类实例化并注入到 Application 的 OrderService 私有字段）。仅在模块内部使用的包不需要开放，如 customers 模块中的 cust.internal。opens 子句可以限定仅对 spruice 开放。但不幸的是，这样做会将 orders 和 customers 模块绑定到这个特定的 DI 框架。如果对开放没有任何限制，那么就可能在不重新编译这些模块的情况下改变 DI 实现。

Figure 6-3 exposes spruice for what it really is: a module reaching into almost every dark corner of the application we’re building. Based on the wiring configuration, it finds encapsulated implementation classes, instantiates them, and injects them into private fields of Application. At the same time, this setup allows the application to be just as nicely modularized as with services and ServiceLoader—without having to use the ServiceLoader API to retrieve services. They are injected as if by (reflection) magic.

> 图 6-3 揭示了 spruice 的真正含义：一个模块可以深入到正在构建的应用程序的每个角落。根据布线配置，它可以找到封装的实现类，然后进行实例化并注入 Application 的私有字段中。同时，该设置允许应用程序像使用服务和 ServiceLoader 一样实现很好的模块化（无须使用 ServiceLoader API 来检索服务）。它们就像是由魔法（反射）注入的一样。

What we lose is the ability of the Java module system to know about and verify the service dependencies between modules. There are no provides/uses clauses in module descriptors to verify. Also, packages in the application modules need to be opened. It is possible to make all application modules open modules. Application developers then aren’t burdened with making that choice for each package. Of course, this comes at the expense of allowing run-time access and deep reflection on all packages in every application module. With a little insight into what your libraries and frameworks are doing, this heavyweight approach isn’t necessary.

> 此时失去的是 Java 模块系统了解和验证模块之间服务依赖关系的能力。模块描述符中没有 provides/uses 子句来验证。并且，应用程序模块中的包需要开放。可以使所有应用程序模块公开模块，因此应用程序开发人员不必为每个包做出选择。当然，这是以允许对每个应用程序模块中所有程序包进行运行时访问和深度反射为代价的。只需稍微了解一下库和框架所要完成的工作，就没有必要使用这个重量级的方法。

In the next section, we’ll look at reflection on modules themselves. Again, this is an advanced use of the module system APIs, which should not come up in normal application development all that often.

> 6.2 节将介绍对模块自身的反射。同样，这也是模块系统 API 的高级使用，不应该经常出现在正常应用程序开发中。

## 6.2 Reflection on Modules 对模块的反射

Reflection allows you to reify all Java elements at run-time. Classes, packages, methods, and so on all have a reflective representation. We’ve seen how open modules allow deep reflection on those elements at run-time.

> 反射允许在运行时对所有 Java 元素进行具体化。类、包、方法等都有反射表示。前面已经看到了开放式模块如何在运行时允许对这些元素进行深度反射。

With the addition of a new structural element to Java, the module, reflection needs to be extended as well. In Java 9, java.lang.Module is added to provide a run-time view on a module from code. Methods on this class can be grouped into three distinct responsibilities:

> 随着 Java 新结构元素——模块的增加，反射也需要扩展。在 Java 9 中，可以使用 java.lang.Module 提供模块的运行时视图。该类的方法可以分为三个不同的职责：

- Introspection 内省
  - Query properties of a given module. 查询给定模块的属性。
- Modification 修改
  - Change characteristics of a module on-the-fly. 动态更改模块的特性。
- Access 访问
  - Read resources from inside a module. 从模块内部读取资源。

The last case was discussed already in “Loading Resources from a Module”. In the remainder of this section, we’ll look at introspecting and modifying modules.

> 最后一种情况在 5.8.1 节已经讨论过了，在本节的其余部分将主要介绍内省和修改模块。

### 6.2.1 Introspection 内省

The java.lang.Module class is the entry point for reflection on modules. Figure 6-4 shows Module and its related classes. It has a ModuleDescriptor that provides a run-time view on the contents of module-info.

> java.lang.Module 类是对模块反射的入口点。Module 及其相关类如图 6-4 所示。它有一个 ModuleDescriptor，提供了有关 module-info 内容的运行时视图。

<Figures figure="6-4">A simplified class diagram of Module and related classes</Figures>

You can obtain a Module instance through a Class from within a module:

> 可以通过模块中的一个类获取 Module 实例：

```java
Module module = String.class.getModule();
```

The getModule method returns the module containing the class. For this example, the String class unsurprisingly comes from the java.base platform module. Later in this chapter, you will see another way to obtain Module instances by name through the new ModuleLayer API, without requiring knowledge of classes in a module.

> getModule 方法返回包含该类的模块。在该示例中，String 类无疑来自于 java. base 平台模块。在本章的后面，将会看到通过新的 ModuleLayer API 按名称获取模块实例的另一种方式，而不需要知道模块中的类。

There are several methods to query information on a Module, shown in Example 6-1.

> 可以使用多种方法查询模块中的信息，如示例 6-1 所示。

Example 6-1. Inspecting a module at run-time (➥ chapter6/introspection)

> 示例 6-1：在运行时检查模块（chapter6/introspection）

```java
String name1 = module.getName(); // Name as defined in module-info.java
Set<String> packages1 = module.getPackages(); // Lists all packages in the module

// The methods above are convenience methods that return
// information from the Module's ModuleDescriptor:
ModuleDescriptor descriptor = module.getDescriptor();
String name2 = descriptor.name(); // Same as module.getName();
Set<String> packages2 = descriptor.packages(); // Same as module.getPackages();

// Through ModuleDescriptor, all information from module-info.java is exposed:
Set<Exports> exports = descriptor.exports(); // All exports, possibly qualified
Set<String> uses = descriptor.uses(); // All services used by this module
```

The preceding examples are by no means exhaustive but illustrate that all information from module-info.class is available through the ModuleDescriptor class. Instances of ModuleDescriptor are read-only (immutable). It is not possible to, for example, change the name of a module at run-time.

> 上面的示例虽然并不详尽，但却说明了 module-info.class 中的所有信息都可以通过 ModuleDescriptor 类获得。ModuleDescriptor 的实例是只读的（不可变的）。例如，不可能在运行时更改模块的名称。

### 6.2.2 Modifying Modules 修改模块

You can perform several other operations on a Module that affect the module and its environment. Let’s say you have a package that is not exported, but based on a run-time decision you want to export it:

> 可以执行模块上其他几个影响该模块及其环境的操作。假设有一个未导出的包，但根据运行时的决策需要将其导出：

```java
Module target = ...; // Somehow obtain the module you want to export to
Module module = getClass().getModule(); // Get the module of the current class
module.addExports("javamodularity.export.atruntime", target);
```

You can add a qualified export only to a specific module through the Module API. Now, the target module can access code that was in the previously encapsulated javamodularity.export.atruntime package.

> 可以通过 Module API 向特定的模块添加限制导出。现在，目标模块可以访问先前封装的 javamodularity.export.atruntime 包中的代码。

You may wonder whether this is a security hole: can you just call addExports on an arbitrary module so it gives up its secrets? That’s not the case. When you try to add an export to any module other than the current module where the call is executing from, an exception is thrown by the VM. You cannot escalate privileges of a module from the outside through the module reflection API.

> 此时你可能会怀疑这是否是一个安全漏洞：是否可以在任意模块上调用 addExports，从而放弃自身的秘密？事实并非如此。当尝试将导出添加到正在执行调用的当前模块之外的任何其他模块时，VM 就会引发异常。无法通过模块反射 API 从外部升级模块的权限。

#### CALLER SENSITIVE 调用者敏感

Methods that behave differently when called from different places are caller sensitive methods. You will find methods such as addExports to be annotated with @CallerSensitive in the JDK source code. Caller sensitive methods can find out which class (and module) is calling them, based on the current call-stack. The privilege of getting this information and basing decisions on it is reserved for code in java.base (although the new StackWalker API introduced through JEP 259 in JDK 9 opens this possibility for application code as well). Another example of this mechanism can be found in the implementation of setAccessible, as discussed in “Open Modules and Packages”. Calling this method from a module to which the package is not opened leads to an exception, whereas calling it from a module that is allowed to perform deep reflection succeeds.

> 那些从不同地方调用时行为方式不同的方法被称为调用者敏感（caller sensitive）方法。可以在 JDK 源代码中找到许多用@CallerSensitive 进行注释的方法，比如 addExports。调用者敏感方法可以根据当前的调用堆栈找出哪个类（和模块）正在调用它们。获取该信息和基于该信息做出决策的权限是为 java.base 中的代码保留的（虽然在 JDK 9 中通过 JEP 259 引入的新 StackWalker API 也为应用程序代码提供了这种可能性）。该机制的另一个示例可以在 setAccessible 的实现中找到，如“开放式模块和包”一节（6.1.2 节）中所述。从包未对其开放的模块上调用此方法会导致异常，而从允许进行深度反射的模块中调用它时则会成功。

Module has four methods that allow run-time modifications:

> Module 提供了四个允许运行时修改的方法：

```java
addExports(String packageName, Module target)
```

Expose previously nonexported packages to another module.

> 将先前没有导出的包公开给另一个模块。

```java
addOpens(String packageName, Module target)
```

Opens a package for deep reflection to another module.

> 开放一个包，以便另一个模块进行深度反射。

```java
addReads(Module other)
```

Adds a reads relation from the current module to another module.

> 添加当前模块到另一个模块的读取关系。

```java
addUses(Class<?> serviceType)
```

Indicates that the current module wants to use additional service types with ServiceLoader.

> 表明当前模块想要通过 ServiceLoader 使用额外的服务类型。

There’s no addProvides method, because exposing new implementations that were not known at compile-time is deemed to be a rare use case.

> 此时，没有 addProvides 方法，因为公开在编译时未知的新实现是一种比较少见的情况。

It’s good to know about the reflection API on modules. However, in practice this API is used only on rare occasions during regular application development. Always try to expose enough information between modules through normal means before reaching for reflection. Using reflection to change module behavior at run-time goes against the philosophy of the Java module system. Implicit dependencies arise that are not taken into account at compile-time or at startup, voiding a lot of guarantees the module system otherwise offers in those early phases.

> 了解模块上的反射 API 是很有好处的。但是，事实上该 API 在正常应用程序开发过程中很少使用。在使用反射之前通常会试图通过正常的方式在模块之间公开足够的信息。在运行时使用反射会改变模块的行为，从而违背 Java 模块系统的原理。如果出现了在编译时或启动时没有考虑到的隐式依赖关系，就会使模块系统在早期阶段所提供的许多保证变得无效。

### 6.2.3 Annotations 注释

Modules can also be annotated. At run-time, those annotations can be read through the java.lang.Module API. Several default annotations that are part of the Java platform can be applied to modules, for example, the @Deprecated annotation:

> 模块也可以注释。在运行时，可以通过 java.lang.Module API 读取这些注释。可以将一些作为 Java 平台一部分的默认注释应用于模块，如@Deprecated 注释：

```java
@Deprecated
module m {

}
```

Adding this annotation indicates to users of the module that they should look for a replacement.

> 添加该注释表明模块的用户应该寻找一个替代模块。

TIP

As of Java 9, a deprecated element can also be marked for removal in a future release: @Deprecated(forRemoval=true). Read JEP 277 for more details on the enhanced deprecation features. Several platform modules (such as java.xml.ws and java.corba) are marked for removal in JDK 9.

> 从 Java 9 开始，可以对那些不建议使用的元素进行标记，以便在未来的版本中将其删除：@Deprecated(forRemoval = true)。有关增强弃用功能的更多详细信息，请参阅 JEP277（http://openjdk.java.net/jeps/277）。在JDK 9 中，有几个平台模块（例如，java.xml.ws 和 java.corba）被标记为删除。

When you require a deprecated module, the compiler generates a warning. Another default annotation that can be applied to modules is @SuppressWarnings.

> 当需要使用一个不建议使用的模块时，编辑器会生成一条警告信息。另一个可应用于模块的默认注释是@SuppressWarnings。

It’s also possible to define your own annotations for modules. To this end, a new target element type MODULE is defined, as shown in Example 6-2.

> 也可以为模块定义自己的注释。为此，定义新的目标元素类型 MODULE，如示例 6-2 所示。

Example 6-2. Annotating a module (➥ chapter6/annotated_module)

> 示例 6-2：注释一个模块（chapter6/annotated_module）

```java
package javamodularity.annotatedmodule;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(value={PACKAGE, MODULE})
public @interface CustomAnnotation {

}
```

This CustomAnnotation can now be applied to packages and modules. Using a custom-defined annotation on a module reveals another curious fact: module declarations can have import statements.

> 现在，CustomAnnotation 可以应用于包和模块。在一个模块上使用自定义注释揭示了另一个奇怪的事实：模块声明可以使用 import 语句。

```java
import javamodularity.annotatedmodule.CustomAnnotation;

@CustomAnnotation
module annotated { }
```

Without the import statement, the module descriptor doesn’t compile. Alternatively, you can use the fully qualified name of the annotation directly, without an import.

> 如果没有 import 语句，模块描述符将无法编译。或者，也可以直接使用注释的完全限定名称，而不是 import。

TIP

You can also use import in module descriptors to shorten uses/provides clauses.

> 也可以在模块描述符中使用 import，从而缩短 uses/provides 子句。

Figure 6-4 shows how Module implements AnnotatedElement. In code, you can use getAnnotations on a Module instance to get an array of all annotations on a module. The AnnotatedElement interface offers various other methods to find the right annotation.

> 图 6-4 显示了 Module 如何实现 AnnotatedElement。在代码中，可以使用 Module 实例上的 getAnnotations 来获取模块上所有注释的数组。AnnotatedElement 接口还提供了其他方法来查找正确的注释。

WARNING

This works only if the retention policy of the annotation is set to RUNTIME.

> 只有在注释的保留策略设置为 RUNTIME 时才有效。

Besides platform-defined annotations such as @Deprecated, it will most probably be frameworks (or even build tools) that make use of annotations on modules.

> 除了平台定义的注释（如@Deprecated）之外，框架（甚至是构建工具）也会使用模块注释。

## 6.3 Container Application Patterns 容器应用程序模式

At this point, we’re switching gears to even more advanced uses of the module system. This is a good moment to reiterate the advice from the beginning of this chapter: you can safely skip the remainder of this chapter when first learning about the module system.

> 接下来重点关注模块系统更高级的应用。此时重申一下本章开头所给出的建议：如果是第一次学习模块系统，那么可以放心地跳过本章的其余部分。

With that out of the way, let’s dive into the advanced APIs! Up to this point, we’ve looked at modular applications as a single entity. You gather modules, put them on the module path, and start the application. Although that’s a valid view in most cases, another range of applications is structurally different.

> 在初步了解了模块系统之后，可以开始深入研究高级 API！到目前为止，都是将模块化应用看作一个单一的实体。首先收集模块，并将它们放在模块路径，然后启动应用程序。尽管在大多数情况下这是一个行之有效的方法，但是另外一种类型的应用程序在结构上与其存在很大不同。

You can think of applications acting as a container for other applications. Or, think of applications that define just their core functionality, with the expectation that this will be extended by third parties. In the former case, new applications can be deployed into a running container application. The latter case is often achieved through a plug-in-like architecture. In both cases, you don’t start with a module path containing all modules up front. New modules can come (and go) during the life cycle of the container application. Many of these applications are currently built on module systems such as OSGi or JBoss Modules.

> 可以将一个应用程序视为其他应用程序的容器，或者假设有一个仅定义了核心功能的应用程序，并且期望由第三方对其进行扩展。在前一种情况下可以将新应用程序部署到正在运行的容器应用程序中，而后一种情况通常是通过类似插件的架构来实现的。这两种情况都不会从一个包含所有模块的模块路径开始。在容器应用程序的生命周期内，新的模块会不断地“来来去去”。目前，许多容器应用程序都建立在诸如 OSGi 或 JBoss Modules 之类的模块系统上。

In this section, you’ll look at architecting such a container or plug-in-based system by using the Java module system. Before looking at the realization of these container application patterns, you’ll explore the new APIs that enable them. Keep in mind that these new APIs are introduced specifically for the use cases discussed so far. When you’re not building an extensible, container-like application, you are unlikely to use those APIs.

> 本节将会学习如何使用 Java 模块系统构建一个容器或基于插件的系统。在研究这些容器应用程序模式的实现之前，首先将探索启用这些模式的新 API。请记住，这些新的 API 是专门为目前讨论的用例而引入的。如果不需要构建一个可扩展的容器式应用程序时，那么就不太可能使用这些 API。

### 6.3.1 Layers and Configurations 层和配置

A module graph is resolved upon starting a module with the java command. The resolver uses modules from the platform itself and the module path to create a consistent set of resolved modules. It does so based on requires clauses and provides/uses clauses in the module descriptors. When the resolver is finished, the resulting module graph cannot be changed anymore.

> 当使用 Java 命令启动模块时，将会解析模块图。解析器使用来自平台本身和模块路径的模块来创建一组解析模块。这主要是根据模块描述符中的 requires 子句和 provides/uses 子句完成的。完成解析后就不能再更改所生成的模块图了。

That seems to run counter to the requirements of container applications. There, the ability to add new modules to a running container is crucial. A new concept must be introduced to allow for this functionality: layers. Layers are to modules as classloaders are to classes: a loading and instantiation mechanism.

> 这似乎与容器应用程序的要求背道而驰。此时，向正在运行的容器添加新模块的能力是至关重要的。必须引入一个新的概念来实现这个功能：层。层对于模块来说就像是类加载器对类那样——一种加载和实例化机制。

A resolved module graph lives within a ModuleLayer. Layers set up coherent sets of modules. Layers themselves can refer to parent layers and form an acyclic graph. But before we get ahead of ourselves, let’s look at the ModuleLayer you already had without knowing.

> 已解析的模块图位于 ModuleLayer 内。层设置了一组连贯的模块。层本身可以引用父层，并形成一个非循环图。但是在进一步学习之前，先了解一下 ModuleLayer。

When plainly starting a module by using java, an initial layer called the boot layer is constructed by the Java runtime. It contains the resulting module graph after resolving the root modules provided to the java command (either as an initial module with -m or through --add-modules).

> 当使用 Java 启动一个模块时，Java 运行时将构建一个被称为引导层（boot layer）的初始层。该层包含了在解析提供给 Java 命令（以带有-m 的初始模块形式提供，或者使用--add-modules）的根模块之后所生成的模块图。

In Figure 6-5, a simplified example of the boot layer is shown after starting a module application that requires java.sql.

> 在图 6-5 中，显示了在启动需要 java.sql 的 application 模块之后的引导层的简化示例。

<Figures figure="6-5">Modules in the boot layer after starting module application</Figures>

(In reality, the boot layer contains many more modules because of service binding of platform modules.)

> （实际上，由于平台模块的服务绑定，引导层包含了更多的模块。）

You can enumerate the modules in the boot layer with the code in Example 6-3.

> 可以使用示例 6-3 所示的代码枚举引导层中的模块。

Example 6-3. Printing all modules in the boot layer (➥ chapter6/bootlayer)

> 示例 6-3：打印引导层中的所有模块（chapter6/bootlayer）

```java
ModuleLayer.boot().modules()
           .forEach(System.out::println);
```

Because the boot layer is constructed for us, the question of how to create another layer remains. Before you can construct a ModuleLayer, a Configuration must be created describing the module graph inside that layer. Again, for the boot layer this is done implicitly at startup. When constructing a Configuration, a ModuleFinder needs to be provided to locate individual modules. Setting up this Russian doll of classes to create a new ModuleLayer looks as follows:

> 由于引导层是 Java 运行时所构建的，因此如何创建另一个层的问题仍然存在。在构建 ModuleLayer 之前，必须在该层中创建一个描述模块图的 Configuration。同样，对于引导层，该过程也是在启动时隐式完成的。在构建 Configuration 时，需要提供一个 ModuleFinder 来定位各个模块。设置一连串类以创建一个新的 ModuleLayer，如下代码所示：

```java
ModuleFinder finder = ModuleFinder.of(Paths.get("./modules")); 1

ModuleLayer bootLayer = ModuleLayer.boot();

Configuration config = bootLayer.configuration()
   .resolve(finder, ModuleFinder.of(), Set.of("rootmodule")); 2

ClassLoader scl = ClassLoader.getSystemClassLoader();
ModuleLayer newLayer = bootLayer.defineModulesWithOneLoader(config, scl); 3
```

1. A convenience method to create a ModuleFinder that locates modules in one or more paths on the filesystem.
2. Configurations are resolved relative to parent configurations; in this case, to the configuration of the boot layer.
3. With Configuration, a ModuleLayer can be constructed, materializing the resolved modules from the configuration.

---

> 1. 一种创建 ModuleFinder 的便捷方法，ModuleFinder 可以在文件系统的一个或者多个路径上查找模块。
> 2. 相对于父配置进行配置解析；此时是相对于引导层的配置。
> 3. 通过 Configuration，可以构建一个 ModuleLayer，从而根据配置具体化已解析的模块。

In principle, modules could come from anywhere. Typically, they’re somewhere on the filesystem, so the ModuleFinder::of(Path...) factory method is convenient to use. It returns a ModuleFinder implementation that can load modules from the filesystem. Every ModuleLayer and Configuration points to one or more parents, with the exception of instances returned by the ModuleLayer::empty and Configuration::empty methods. Those special instances serve as root for the ModuleLayer and Configuration hierarchy in the module system. The boot layer and corresponding configuration have their empty counterparts as a parent.

> 原则上，模块可以来自任何地方。通常它们位于文件系统的某个位置，所以 ModuleFinder::of(Path...)工厂方法使用非常方便，它返回一个可以从文件系统加载模块的 ModuleFinder 实现。除 ModuleLayer::empty 和 Configuration::empty 方法返回的实例之外，每个 ModuleLayer 和 Configuration 都指向一个或多个父级。这些特殊实例作为模块系统中的 ModuleLayer 和 Configuration 层次结构的根。引导层和对应配置将空项作为父项。

While constructing the new layer, we use the boot layer and configuration as a parent. In the resolve call, the ModuleFinder is passed as the first argument. The second argument to resolve is another ModuleFinder, in this case, an empty one. This second finder is consulted when a module could not be found in the first finder or through the parent configuration.

> 在构建新层时，使用引导层及其配置作为父级。在 resolve 调用中，ModuleFinder 作为第一个参数传递，而第二个参数是另一个 ModuleFinder（在本示例中，第二个参数为空），当第一个查找器或父配置中找不到模块时就会查阅第二个查找器。

When resolving modules in the new configuration, modules from the parent configuration are taken into account as well. A module from the newly constructed configuration can read a module from the parent configuration. Root modules to kickstart the resolver are passed as a third argument to the configuration’s resolve method. In this case, rootmodule serves as a initial module for the resolver to start from. Resolution in a new configuration is subject to the same constraints you’ve seen so far. It fails if a root module or one of its dependencies cannot be found. Cycles between modules are not allowed, nor can two modules exporting the same package be read by a single other module.

> 在解析新配置中的模块时也会考虑父配置中的模块，新构建配置中的模块可以从父配置中读取模块。启动解析器的根模块将作为第三个参数传递给配置的 resolve 方法。在这种情况下，rootmodule 充当解析器的初始模块。新配置中的解析受到前面所介绍的约束的限制。如果找不到根模块或其某个依赖项，则会失败。模块之间的循环是不允许的，同时导出同一个包的两个模块也不能被另外一个模块读取。

To expand on the example, let’s say rootmodule requires the javafx.controls platform module and library, a helper module that also lives in the ./modules directory. After resolving the configuration and constructing the new layer, the resulting situation looks like Figure 6-6.

> 为了扩展这个示例，假设 rootmodule 需要 javafx.controls 平台模块和 library（一个驻留在．/modules 目录中的辅助模块）。解析完配置并构建新层后，结果如图 6-6 所示。

<Figures figure="6-6">A new layer with the boot layer as its parent</Figures>

Readability relations can cross layer boundaries. The requires clause of rootmodule to javafx.controls has been resolved to the platform module in the boot layer. On the other hand, the requires clause to library was resolved in the newly constructed layer, because that module is loaded along with rootmodule from the filesystem.

> 可读性关系可以跨越层边界。rootmodule 到 javafx.controls 的 requires 子句被解析到引导层的平台模块中。另一方面，到 library 的 requires 子句在新构建的层中被解析，因为该模块与来自文件系统的 rootmodule 一起被加载。

Besides the resolve method on Configuration, there’s also resolveAndBind. This variation also does service binding, taking into account the provides/uses clauses of the modules in the new configuration. Services can cross layer boundaries as well. Modules in the new layer can use services from the parent layer, and vice versa.

> 除了 Configuration 上的 resolve 方法外，还有 resolveAndBind。考虑到新配置中模块的 provides/uses 子句，resolveAndBind 方法也会进行服务绑定。服务也可以跨越层边界。新层中的模块可以使用来自父层的服务，反之亦然。

Last, the defineModulesWithOneLoader method is called on the parent (boot) layer. This method materializes the module references resolved by the Configuration into real Module instances inside the new layer. The next section discusses the significance of the classloader passed to this method.

> 最后，在父（引导）层调用 defineModulesWithOneLoader 方法，该方法将 Configuration 所解析的模块引用转化为新层内的 Module 实例。下一节将讨论传递给 defineModulesWithOneLoader 方法的类加载器的重要性。

All examples of layers you’ve seen so far consisted of new layers pointing to the boot layer as the parent. However, layers can point to a parent layer other than the boot layer as well. It’s even possible for a layer to have multiple parents. This is illustrated in Figure 6-7: Layer 3 points to Layer 1 and Layer 2 as its parents.

> 到目前为止，所看到的所有层示例都是以引导层作为父层的新层所组成的。但是，层也可以指向除引导层之外的父层，甚至有可能一个层有多个父层。如图 6-7 所示：第 3 层指向第 1 层和第 2 层作为其父节点。

<Figures figure="6-7">Layers can form an acyclic graph</Figures>

Static defineModules* methods on ModuleLayer accept a list of parent layers when constructing new layers, instead of calling any of the nonstatic defineModules* methods on a parent layer instance directly. Later, in Figure 6-12, you’ll see those static methods in action. The important thing to remember is that layers can form an acyclic graph, just like modules within a layer.

> ModuleLayer 上的静态 defineModules*方法在构建新层时接收一个父层列表，而不是直接调用父层实例上任何非静态的 defineModules*方法。稍后在图 6-12 中将会看到这些静态方法是如何使用的。重要的是要记住层可以形成一个非循环图，就像一个层内的模块一样。

### 6.3.2 Classloading in Layers 层中的类加载

You may still be wondering about the last two lines of the layer construction code in the previous section:

> 此时，你可能仍然在想上一节中层构造代码的最后两行：

```java
ClassLoader scl = ClassLoader.getSystemClassLoader();
ModuleLayer newLayer = bootLayer.defineModulesWithOneLoader(config, scl);
```

What’s up with the classloader and the defineModulesWithOneLoader method? The answer to this question is nontrivial. Before we get to the method in question, it’s good to brush up on what classloaders do and what that means for the module system.

> 什么是类加载器和 defineModulesWithOneLoader 方法？回答这个问题不是那么容易的。在找到回答问题的方法之前，最好先了解一下什么是类加载器，及其对模块系统意味着什么。

Classloaders, no surprise, load classes at run-time. Classloaders dictate visibility: a class can be loaded if it is visible to a certain classloader, or any other classloaders it delegates to. In a way, it’s peculiar to introduce classloaders this far into the book. Earlier module systems such as OSGi use classloaders as the primary means to enforce encapsulation. Every bundle (OSGi module) gets its own classloader, and the delegation between classloaders follows the wiring of bundles as expressed in the OSGi metadata.

> 毫无疑问，类加载器在运行时加载类。类加载器决定了可见性：如果某个类对于某个类加载器或其委托的其他类加载器可见，那么该类就可以加载。从某种意义上讲，在本书中介绍类加载器是非常奇怪的。像 OSGi 这样的早期模块系统使用类加载器作为强制封装的主要手段，每个包（OSGi 模块）都有它自己的类加载器，类加载器之间的委托遵循 OSGi 元数据中所表示的 bundle 布线。

Not so in the Java module system. There, a whole new mechanism encompassing readability and new accessibility rules is put in place, while leaving classloading mostly unchanged. That’s a deliberate choice, because using classloaders for isolation is not a foolproof solution. After classes have been loaded, Class instances can be passed around freely, bypassing any schemes set up through classloader isolation and delegation. You can try to do that in the module system, but you’ve seen that creating instances from Class objects you’re not allowed to access because of encapsulation leads to exceptions. The module system enforces encapsulation at a much deeper level. Furthermore, classloaders are only a run-time construct, whereas the Java module system enforces encapsulation at compile-time as well. Last, many existing codebases make assumptions about the way classes are loaded by default. Changing these defaults (for example, by giving each module its own classloader) would break existing code.

> 在 Java 模块系统中却不是这样的。它建立了一个全新的机制，包括可读性和新的可访问性规则，而类加载通常是不变的。这是一个慎重的选择，因为使用类加载器进行隔离并不是一个万无一失的解决方案。加载类之后，Class 实例可以自由传递，绕过通过类加载器隔离和委托所设置的任何方案。虽然可以尝试在模块系统中执行此操作，但可以看到，由于存在封装，从不允许访问的 Class 对象创建实例会导致异常。模块系统在更深的层面上执行了封装。此外，类加载器只是一个运行时构造，而 Java 模块系统在编译时也会执行封装。最后，许多现有的代码库对默认情况下类的加载方式进行了假设，改变这些默认值（例如，给每个模块一个自己的类加载器）会破坏现有的代码。

Still, it’s good to be aware of the way classloaders interact with the module system. Let’s revisit Figure 6-5, where a module application is loaded in the boot layer. This time, we’re interested in what classloaders are involved, as shown in Figure 6-8.

> 不过，了解类加载器与模块系统的交互方式是很有帮助的。再来看一下图 6-5，其中一个模块 application 被加载到引导层。此时仅对所参与的类加载器感兴趣，如图 6-8 所示。

<Figures figure="6-8">Classloaders in the boot layer when starting module application</Figures>

Three classloaders are active in the boot layer when running an application from the module path. At the bottom of the delegation hierarchy is BootstrapClassLoader, also known as the primordial classloader. It’s a special classloader loading all essential platform module classes. Care has been taken to load classes from as few modules as possible in this classloader, because classes are granted all security permissions when loaded in the bootstrap loader.

> 当从模块路径运行应用程序时，有三个类加载器在引导层中处于活动状态。在委托层的底部是 BootstrapClassLoader，也被称为原始类加载器（primordial classloader）。这是一个加载所有基本平台模块类的特殊类加载器。在该类加载器中已经采取了谨慎措施，以便从尽可能少的模块加载类，因为类在引导加载程序中加载时被授予所有安全权限。

Then there’s PlatformClassLoader, which loads less privileged platform module classes. Last in the chain is AppClassLoader, responsible for loading user-defined modules and some JDK-specific tooling modules (such as jdk.compiler or jdk.javadoc). All classloaders delegate to the underlying classloader. Effectively, this gives AppClassLoader visibility to all classes. This three-way setup is quite similar to the way classloaders worked before the module system, mainly for backward compatibility.

> 接下来是 PlatformClassLoader，加载较少权限的平台模块类。最后一个是 AppClassLoader，负责加载用户定义的模块和一些特定于 JDK 的工具模块（比如，jdk.compiler 或 jdk.javadoc）。所有的类加载器都委托给底层的类加载器。实际上，这使得所有类都对 AppClassLoader 可见。这种三路设置类似于类加载器在模块系统之前的工作方式，主要是为了向后兼容。

We started this section with the question of why a classloader needs to be passed to the method creating a ModuleLayer:

> 在本节的开头，曾经提过一个问题：为什么需要将一个类加载器传递给创建 ModuleLayer 的方法：

```java
ClassLoader scl = ClassLoader.getSystemClassLoader();
ModuleLayer newLayer = bootLayer.defineModulesWithOneLoader(config, scl);
```

Even though the mapping from modules to classloaders is predefined in the boot layer, the creator of a new layer must indicate which classloaders load classes for which modules. The ModuleLayer::defineModulesWithOneLoader(Configuration, ClassLoader) method is a convenience method. It sets up the new layer so that all modules in the layer are loaded by a single, freshly created classloader. This classloader delegates to the parent classloader passed as an argument. In the example, we pass the result of ClassLoader::getSystemClassLoader, which returns AppClass​Loader, the classloader responsible for loading classes of user-defined modules in the boot layer (there’s also a getPlatformClassLoader method).

> 即使在引导层中预定义了从模块到类加载器的映射，新层的创建者也必须指出哪些类加载器为哪些模块加载类。ModuleLayer::defineModulesWithOneLoader（Configuration,ClassLoader）方法是一个方便的方法。它设置了新的层，以便层中的所有模块都由一个新创建的类加载器加载。这个类加载器委托给作为参数传递的父类加载器。在本示例中，传递了 ClassLoader::getSystemClassLoader 的结果，并返回 AppClassLoader，它是负责在引导层中加载用户定义模块类的类加载器（还有一个 getPlatformClassLoader 方法）。

So, the classloader view on the newly constructed layer from the example (as seen earlier in Figure 6-6) looks like Figure 6-9.

> 因此，示例中新构建的层（如图 6-6 所示）的类加载器视图如图 6-9 所示。

<Figures figure="6-9">A single classloader is created for all modules in the new layer</Figures>

The delegation between classloaders must respect the readability relations between modules, even across layers. It would be problematic if the new classloader didn’t delegate to AppClassLoader in the boot layer, but, for example, to BootstrapClass​Loader. Because rootmodule reads javafx.controls, it must be able to see and load those classes. Parent delegation of the new layer’s classloader to AppClassLoader ensures this. In turn, AppClassLoader delegates to PlatformClassLoader, which loads the classes from javafx.controls.

> 即使是跨层，类加载器之间的委托也必须遵守模块之间的可读性关系。如果新的类加载器没有在引导层中委托给 AppClassLoader，而是委托给 BootstrapClass Loader，则会产生问题。由于 rootmodule 读取 javafx.controls，因此必须能够查看并加载这些类。而将新层的类加载器委托给 AppClassLoader 可以确保这一点。继而，AppClassLoader 委托给 PlatformClassLoader，后者从 javafx. controls 加载类。

There are other methods to create a new layer. Another convenience method called defineModulesWithManyLoaders creates a new classloader for each module in the layer, as shown in Figure 6-10.

> 还可以使用其他方法创建一个新层。另一个被称为 defineModulesWithManyLoaders 的简便方法为层中的每个模块创建一个新的类加载器，如图 6-10 所示。

<Figures figure="6-10">Every module within a layer constructed using defineModulesWithManyLoaders gets its own classloader</Figures>

Again, each of these new classloaders delegates to the parent that is passed as an argument to defineModulesWithManyLoaders. If you need even more control over classloading in a layer, there’s the defineModules method. It takes a function mapping a string (module name) to a classloader. Providing such a mapping gives ultimate flexibility on when to create new classloaders for a new module or to assign existing classloaders to modules in the layer. An example of such a mapping can be found in the JDK itself. The boot layer is created using defineModules with a custom mapping to one of the three classloaders shown previously in Figure 6-8.

> 同样，这些新的类加载器都委托给作为参数传递给 defineModulesWithManyLoaders 的父类加载器。如果需要对层中的类加载进行更多的控制，可以使用 defineModules 方法。它接收一个将字符串（模块名称）映射到类加载器的函数。当需要为新模块创建新类加载器或将现有类加载器分配给层中模块时，这种映射提供了极大的灵活性。可以在 JDK 本身找到这种映射的示例。引导层是使用 defineModules 创建的，具有自定义映射到图 6-8 所示的三个类加载器之一的功能。

Why is it important to control classloading when using layers? Because it can make many limitations in the module system disappear. For example, we discussed how only a single module can contain (or export) a certain package. It turns out this is just a side effect of how the boot layer is created. A package can be defined to a classloader only once. All modules from the module path are loaded by AppClassLoader. Therefore, if any of these modules contain the same package (exported or not), they will be defined to the same AppClassLoader leading to run-time exceptions.

> 为什么在使用层时控制类的加载非常重要？因为可以取消模块系统中许多限制。例如，前面讨论了单个模块如何才能包含（或导出）某个包。事实证明，这只是创建引导层的一个副作用。一个包只能定义一个类加载器。来自模块路径的所有模块都由 AppClassLoader 加载。因此，如果这些模块中的任何一个模块包含相同的包（不管是否导出），那么一旦将它们定义到相同的 AppClassLoader，就会导致运行时异常。

When you instantiate a new layer with a new classloader, the same package may appear in a different module in that layer. In that case, there is no inherent problem because the package gets defined twice to different classloaders. You’ll see later that this also means multiple versions of the same module can live in distinct layers. That’s quite an improvement on the situation we discussed in “Versioned Modules”, although it comes at the expense of the complexity of creating and managing layers.

> 当使用新的类加载器实例化新层时，相同的程序包可能会出现在该层的不同模块中。在这种情况下，不会出现上述问题，因为包被定义为两个不同的类加载器。稍后会看到，这也意味着同一模块的多个版本可以存在于不同的层中。这是对 5.7 节中讨论的情况的很大改进，尽管这是以提高创建和管理层的复杂性为代价的。

### 6.3.3 Plug-in Architectures 插件体系结构

Now that you have seen the basics of layers and classloading in layers, it’s time to apply them in practice. First, you’ll look at creating an application that can be extended with new plug-ins at run-time. In many ways, this is similar to what we did with EasyText and services in Chapter 4. With the services approach, when a new analysis provider is put on the module path, it is picked up when starting the application. This is already quite flexible, but what if those new provider modules come from third-party developers? And what if they aren’t put on the module path at startup, but can be introduced at run-time?

> 到目前为止，已经学习了层以及层中类加载的基础知识，接下来可以在实践中应用这些知识了。首先，创建一个可以在运行时使用新插件进行扩展的应用程序。在很多方面与第 4 章中使用 EasyText 和服务所完成的事情是相类似的。当使用服务方法时，只需将新的分析提供者放在模块路径上，应用程序在启动时就会提取它。虽然这种方法已经非常灵活了，但是如果这些新的提供者模块来自第三方开发者呢？如果这些分析提供者不是在启动时放在模块路径上，而是在运行时引入，又该怎么办呢？

Such requirements lead to a more flexible plug-in architecture. A well-known example of a plug-in-based application is the Eclipse IDE. Eclipse itself offers baseline IDE functionality but can be extended in many ways by adding plug-ins. Another example of an application that uses plug-ins is the new jlink tool in the JDK. It can be extended with new optimizations through a plug-in mechanism similar to the one we’re going to look at now. Chapter 13 discusses more details on what plug-ins the jlink tool can use.

> 为了满足上述需求，需要使用更加灵活的插件体系结构。Eclipse IDE 是一个众所周知的基于插件的应用程序示例。Eclipse 本身仅提供了基本的 IDE 功能，但可以通过添加插件以多种方式进行扩展。使用插件的应用程序的另一个例子是 JDK 中的新 jlink 工具，它可以通过与接下来将要介绍的相类似的插件机制进行新的优化扩展。第 13 章讨论了 jlink 工具可以使用的插件的更多细节。

In general, we can identify a plug-in host application that gets extended by plug-ins, as shown in Figure 6-11.

> 一般来说，可以找到一个通过插件扩展的插件主机应用程序，如图 6-11 所示。

<Figures figure="6-11">Plug-ins provide additional functionality on top of the host application’s functionality</Figures>

Users interact with the host application, but experience extended functionality by virtue of the plug-ins. In most cases, the host application can function fine without any plug-ins as well. Plug-ins are usually developed independently of the host application. A clear boundary exists between the host application and the plug-ins. At run-time, the host application calls into the plug-ins for additional functionality. For this to work, there must be an agreed-upon API that the plug-ins implement. Typically, this is an interface that plug-ins implement.

> 虽然用户与主机应用程序进行交互，但凭借插件可以体验到扩展功能。在大多数情况下，主机应用程序可以在没有任何插件的情况下正常运行。插件通常独立于主机应用程序开发，主机应用程序和插件之间存在明确的界限。在运行时，主机应用程序通过调用插件来获取额外功能。为此，插件必须实现一致的 API。通常，插件实现的是一个接口。

You may have already guessed that layers play a role in the implementation of a dynamic, plug-in-based application. We’re going to create a pluginhost module that spins up a new ModuleLayer for each plug-in. These plug-in modules, plugin.a and plugin.b in this example, live in separate directories along with their dependencies (if any). Crucially, these directories are not on the module path when starting pluginhost.

> 你可能已经猜到，层在实现一个动态的、基于插件的应用程序中扮演着重要的角色。接下来将要创建一个 pluginhost 模块，并为每个插件创建一个新的 ModuleLayer。这些插件模块（在本示例中为 plugin.a 和 plugin.b）与其依赖项（如果有的话）位于不同的目录中。关键是，当启动 pluginhost 时这些目录不在模块路径上。

The example uses services with a pluginhost.api module exposing the interface pluginhost.api.Plugin, which has a single method, doWork. Both plug-in modules require this API module, but otherwise don’t have a compile-time relation to the pluginhost application module. A plug-in module consists of an implementation of the Plugin interface, provided as service. Take the module descriptor of module plugin.a, shown in Example 6-4.

> 示例使用了带有 pluginhost.api 模块的服务，该模块公开了接口 pluginhost. api.Plugin，而该接口具有单一方法 doWork。虽然两个插件模块都需要该 API 模块，但与 pluginhost 应用程序模块之间不存在编译时关系。插件模块由作为服务提供的 Plugin 接口实现组成。示例 6-4 显示了 plugin.a 模块的模块描述符。

Example 6-4. module-info.java (➥ chapter6/plugins)

> 示例 6-4:module-info.java（chapter6/plugins）

```java
module plugin.a {
  requires pluginhost.api;

  provides pluginhost.api.Plugin
      with plugina.PluginA;
}
```

The plug-in implementation class PluginA is not exported.

> 插件实现类 PluginA 没有被导出。

In pluginhost, the main method loads plug-in modules from directories provided as arguments:

> 在 pluginhost 中，main 方法从参数所提供的目录中加载插件模块：

```java
  if (args.length < 1) {
    System.out.println("Please provide plugin directories");
    return;
  }

  System.out.println("Loading plugins from " + Arrays.toString(args));

  Stream<ModuleLayer> pluginLayers = Stream
    .of(args)
    .map(dir -> createPluginLayer(dir)); 1

  pluginLayers
    .flatMap(layer -> toStream(ServiceLoader.load(layer, Plugin.class))) 2
    .forEach(plugin -> {
       System.out.println("Invoking " + plugin.getName());
       plugin.doWork(); 3
    });
}
```

1. For each directory provided as an argument, a ModuleLayer is instantiated in createPluginLayer (implementation shown later).
2. A ServiceLoader::load call is performed with each layer as an argument, giving back Plugin services from that layer.
3. After the services have been flattened into a single stream, we call the doWork method on all plug-ins.

---

> 1. 针对作为参数提供的每一个目录，都会在 createPluginLayer（稍后实现）中实例化一个 ModuleLayer。
> 2. ServiceLoader::load 调用是以每一层作为参数执行的，并从该层返回插件服务。
> 3. 在将服务整合为单个流之后，就可以在所有插件上调用 doWork 方法。

We are using a yet-to-be discussed overload of ServiceLoader::load. It takes a layer as an argument. By passing it the newly constructed layer for the plug-in, the call returns the freshly loaded Plugin service provider from the plug-in module that was loaded into the layer.

> 现在使用的是 ServiceLoader::load 的一个尚未讨论过的重载。它需要以层作为参数。通过为插件传递新构建的层，该调用将从加载到层的插件模块中返回新加载的 Plugin 服务提供者。

The application is started by running the pluginhost module from the module path. Again, none of the plug-in modules are on the module path. Plug-in modules live in separate directories, and will be loaded at run-time by the host application.

> 通过从模块路径运行 pluginhost 模块来启动应用程序。同样，没有任何插件模块在模块路径上。插件模块位于不同的目录中，并由主机应用程序在运行时加载。

After starting pluginhost with the two plug-in directories as arguments, the run-time situation in Figure 6-12 emerges.

> 在以两个插件目录作为参数启动 pluginhost 之后，会出现如图 6-12 所示的运行时情况。

<Figures figure="6-12">Every plug-in is instantiated in its own layer</Figures>

The first plug-in consists of a single module. Plug-in B, on the other hand, has a dependency on somelibrary. This dependency is automatically resolved when creating the configuration and layer for this plug-in. As long as somelibrary is in the same directory as plugin.b, things just work. Both plug-ins require the pluginhost.api module, which is part of the boot layer. All other interactions happen through services published by the plug-in modules and consumed by the host application.

> 第一个插件由单个模块组成。而插件 B 依赖于 somelibrary，当为此插件创建配置和层时，将会自动解析此依赖关系。只要 somelibrary 和 plugin.b 在同一个目录下，程序就可以顺利运行。这两个插件都需要 pluginhost.api 模块，它是引导层的一部分。所有其他交互都是通过由插件模块发布并由主机应用程序使用的服务产生的。

Here’s the createPluginLayer method:

> createPluginLayer 方法如下所示：

```java
static ModuleLayer createPluginLayer(String dir) {
  ModuleFinder finder = ModuleFinder.of(Paths.get(dir));

  Set<ModuleReference> pluginModuleRefs = finder.findAll();
  Set<String> pluginRoots = pluginModuleRefs.stream()
           .map(ref -> ref.descriptor().name())
           .filter(name -> name.startsWith("plugin")) 1
           .collect(Collectors.toSet());

  ModuleLayer parent = ModuleLayer.boot();
  Configuration cf = parent.configuration()
    .resolve(finder, ModuleFinder.of(), pluginRoots); 2

  ClassLoader scl = ClassLoader.getSystemClassLoader();
  ModuleLayer layer = parent.defineModulesWithOneLoader(cf, scl); 3

  return layer;
}
```

1. In order to identify the root modules when resolving the Configuration, we retain all modules with a name that starts with plugin.
2. The Configuration is resolved with respect to the boot layer, so that plug-in modules can read pluginhost.api.
3. All modules in the plug-in layer will be defined with the same (fresh) classloader.

---

> 1. 为了在解析 Configuration 时识别根模块，保留了所有以 plugin 开头的模块。
> 2. 相对于引导层，Configuration 已被解析，所以插件模块可以读取 pluginhost.api。
> 3. 插件层中的所有模块将使用相同的（新鲜的）类加载器来定义。

Because the createPluginLayer method is called for each plug-in directory, multiple layers are created. Each layer has a single root module (respectively, plugin.a and plugin.b) that gets resolved independently. Because plugin.b requires somelibrary, a ResolutionException will be thrown if that module cannot be found. A configuration and layer will be created only if all constraints expressed in the module descriptors can be satisfied.

> 由于针对每个插件目录都会调用 createPluginLayer 方法，所以会创建多个层。每个层都有一个能独立解析的根模块（分别为 plugin.a 和 plugin.b）。由于 plugin.b 需要 somelibrary，如果找不到该模块，将会抛出一个 ResolutionException 异常。只有满足模块描述符中表达的所有约束条件时才能创建配置和层。

NOTE

We can also call resolveAndBind(finder, ModuleFinder.of(), Set.of()) (providing no root modules to resolve) instead of resolve(..., pluginRoots). Because the plugin modules expose a service, service binding causes the resolution of the plugin modules and their dependencies anyway.

> 除了调用 resolve(…, pluginRoots)之外，也可以调用 resolveAndBind (finder,ModuleFinder.of(), Set.of())（没有提供要解析的根模块）。由于 plugin 模块公开了一个服务，因此服务绑定导致了 plugin 模块及其依赖项的解析。

There’s another advantage to loading each plug-in module into its own layer with a new classloader. By isolating plug-ins this way, it becomes possible for the plug-ins to depend on different versions of the same module. In “Versioned Modules”, we discussed how only a single module of the same name exporting the same packages can be loaded by the module system. That’s still true, but we now know this depends on the way classloaders are set up. Only when talking about the boot layer constructed from the module path will this be problematic.

> 使用新的类加载器将每个插件模块加载到自己的层中还有另一个好处。通过这种方式来隔离插件，可以让插件依赖于相同模块的不同版本。在 5.7 节中，讨论了模块系统如何加载导出了相同包的同名模块。现在我们知道这取决于类加载器的设置方式。但是当谈到从模块路径构建的引导层时，这种做法会出现问题。

Different versions of modules can be loaded simultaneously when constructing multiple layers. If, for example, Plug-in A and B would depend on different versions of somelibrary, that’s perfectly possible, as shown in Figure 6-13.

> 当构建多个层时，可以同时加载模块的不同版本。例如，如果插件 A、B 依赖于 somelibrary 不同的版本，就会出现上述情况，如图 6-13 所示。

<Figures figure="6-13">Different versions of the same module can be loaded in multiple layers</Figures>

No code changes are necessary for this to work. Because in our layers modules are loaded in a new classloader, no clashes in package definitions can occur.

> 此时，不需要进行任何代码修改。因为在层中，模块被加载到一个新的类加载器中，所以在包定义中不会发生冲突。

This setup with a new layer for each plug-in brings many advantages. Plug-in authors are free to choose their own dependencies. At run-time, they’re guaranteed not to clash with dependencies from other plug-ins.

> 这种为每个插件设置一个新层的设置带来了许多好处。插件作者可以自由选择自己的依赖项。在运行时，保证不会与其他插件的依赖项发生冲突。

One question to ponder is what happens when we create another layer on top of the two plug-in layers. Layers can have multiple parents, so we can create a new layer with both plug-in layers as parents:

> 需要思考的一个问题是，当在两个插件层之上创建另一个层时会发生什么事情。层可以有多个父级，所以可以创建一个新层，并将两个插件层作为父级：

```java
List<Configuration> parentConfigs = pluginLayers
  .map(ModuleLayer::configuration)
  .collect(Collectors.toList());
Configuration newconfig = Configuration.resolve(finder, parentConfigs, 1
  ModuleFinder.of(), Set.of("topmodule"));
ModuleLayer.Controller newlayer = ModuleLayer.defineModulesWithOneLoader(
  newconfig, pluginLayers, ClassLoader.getSystemClassLoader()); 2
```

1. This static method can take multiple configurations as parents.
2. The same holds for the layer construction with multiple parents.

---

> 1. 此静态方法可以接收多个配置作为父级。
> 2. 层结构也带有多个父级。

Let’s say this new layer contains a single (root) module named topmodule, which requires somelibrary. Which version of somelibrary will topmodule be resolved against? The fact that the static resolve and defineModulesWithOneLoader methods take a List as a parameter for the parents should serve as a warning sign. Order matters. When the configuration is resolved, the list of parent configurations is consulted in order of the provided list. So depending on which plug-in configuration is put into the parentConfigs list first, the topmodule uses either version 1 or 2 of somelibrary.

> 假设这个新层包含一个名为 topmodule 的单个（根）模块，该模块依赖 somelibrary。使用 somelibrary 的哪个版本来解析 topmodule 模块呢？静态 resolve 和 defineModulesWithOneLoader 方法都接收了一个 List 作为父类参数，这应该视作一个警告标志。由此可以看出，顺序非常重要。当解析完配置后，将按照所提供列表的顺序查阅父配置列表。因此，根据优先放入 parentConfigs 列表中的插件配置，topmodule 将使用 somelibrary 的版本 1 或版本 2。

### 6.3.4 Container Architectures 容器体系结构

Another architecture that hinges on being able to load new code at run-time is the application container architecture. Isolation between applications in a container is another big theme. Throughout the years, Java has known many application containers, or application servers, most implementing the Java EE standard.

> 另一种能够在运行时加载新代码的体系结构是应用程序容器体系结构。容器中应用程序之间的隔离是另一个大的主题。多年来，Java 已经了解了许多实现 Java EE 标准的应用程序容器或应用程序服务器。

WARNING

Even though application servers offer some level of isolation between applications, in the end all deployed applications still run in the same JVM. True isolation (i.e., restricted memory and CPU utilization) requires a more pervasive approach.

> 尽管应用程序服务器在应用程序之间提供了一定程度的隔离，但最终所有部署的应用程序仍然运行在同一个 JVM 中。要实现真正的隔离（即限制内存和 CPU 利用率）需要一种更普遍的方法。

Java EE served as one of the inspirations while designing the requirements around layers. That’s not to say Java EE is aligned with the Java module system already. At the time of writing, it is unclear which version of Java EE will first make use of modules. However, it’s not unthinkable (one might say, quite reasonable to expect) that Java EE will support modular versions of Web Archives (WARs) and Enterprise Application Archives (EARs).

> 在围绕层设计需求时，Java EE 充当了灵感之一。这并不是说 Java EE 已经与 Java 模块系统保持一致。在编写本书时，尚不清楚哪个版本的 Java EE 将首先使用模块。但是，Java EE 支持 WebArchive（WAR）和 Enterprise Application Archive（EAR）的模块化版本并不是不可能的（可以说是相当合理的预期）。

To appreciate how layers enable application container architectures, we’re going to create a small application container. Before looking at the implementation, let’s review how an application container architecture differs from a plug-in-based one (see Figure 6-14).

> 为了了解层如何启用应用程序容器体系结构，接下来将创建一个小的应用程序容器。在查看实现之前，先回顾一下应用程序容器体系结构与基于插件的体系结构之间的不同之处（请参见图 6-14）。

<Figures figure="6-14">An application container hosts multiple applications, offering common functionality for use in those applications</Figures>

As Figure 6-14 shows, many applications can be loaded into a single container. Applications have their own internal module structure, but crucially, they use common services offered by the container. Functionality such as transaction management or security infrastructure is often provided by containers. Application developers don’t have to reinvent the wheel for every application.

> 如图 6-14 所示，可以将多个应用程序加载到单个容器中。虽然应用程序拥有各自的内部模块结构，但更重要的是它们都使用了容器提供的公共服务。诸如事务管理或安全基础结构之类的功能通常由容器提供。应用程序开发人员不必为每个应用程序的运行重新设计“轮子”。

In a way, the call-graph is reversed compared to Figure 6-11: applications use the implementations offered by the container, instead of the host application (container) calling into the newly loaded code. Of course, a shared API between the container and applications must exist for this to work. Java EE is an example of such an API. Another big difference with plug-in-based applications is that individual users of the system interact with the dynamically loaded applications themselves, not the container. Think of deployed applications directly exposing HTTP endpoints, web interfaces, or queues to users. Last, applications in a container can be deployed and undeployed, or replaced by a new version.

> 在某种程度上，调用图与图 6-11 正好相反：应用程序使用容器所提供的实现，而不是主机应用程序（容器）调用新加载的代码。当然，前提是必须存在容器和应用程序之间的共享 API。JavaEE 就是此类 API 的一个示例。与基于插件的应用程序的另一个较大的区别是，系统的个人用户与动态加载的应用程序本身进行交互，而不是容器。可以考虑已部署的应用程序直接向用户公开 HTTP 端点、Web 界面或队列。最后，可以部署或取消部署容器中的应用程序，也可以由新版本取代。

Functionally, a container architecture is quite different from a plug-in-based architecture. Implementation-wise, there’s not that much difference. Application modules can be loaded at run-time into a new layer, just like plug-ins. An API module is offered by the container with interfaces for all the services it offers, which applications can compile against. Those services can be consumed in applications through ServiceLoader at run-time.

> 从功能上讲，容器体系结构与基于插件的体系结构完全不同，但在实施方面却没有太大的区别。就像插件一样，可以在运行时将应用程序模块加载到新层。容器提供了一个 API 模块，该模块包含了所提供服务的相关接口，这些应用程序根据该模块进行编译。这些服务可以在运行时通过 ServiceLoader 在应用程序中使用。

To make things interesting, we’re going to create a container that can deploy and undeploy applications. In the plug-in example, we relied on the plug-in module to expose a service implementing a common interface. For the container, we’re going to do things differently. After deployment, we’re going to look for a class that implements ContainerApplication, which is part of the API offered by the container. The container will reflectively load the class from the application. By doing so, we cannot rely on the services mechanism to interact with the deployed application (although the deployed application does use services to consume container functionality). After instantiating the class through reflection, the startApp method (defined in Container​Application) is called. Before undeployment, stopApp is called so the application can gracefully shut down.

> 为了让事情变得更有趣，接下来将创建一个可以部署和取消部署应用程序的容器。在前面的插件示例中，我们依靠插件模块公开了一个实现通用接口的服务。而对于容器，需要做完全不同的事情。部署完成后，查找一个实现了 ContainerApplication 的类，该类是容器提供的 API 的一部分。容器将以反射的方式从应用程序加载类。这样一来，就不能依赖服务机制与已部署的应用程序进行交互（虽然已部署的应用程序确实通过服务来使用容器功能）。在通过反射实例化该类之后，调用 startApp 方法（在 ContainerApplication 中定义）。而在取消部署之前，调用 stopApp，以便应用程序可以正常关闭。

There are two new concepts to focus on:

> 此时需要关注两个新的概念：

- How can the container ensure it is able to reflectively instantiate a class from the deployed application? We don’t require the package to be open or exported by the application developer.
- How can modules be properly disposed of after undeploying an application?

---

> - 容器如何确保能够从已部署的应用程序中以反射的方式实例化一个类？不要求应用程序开发者公开或导出包。
> - 取消部署应用程序之后，如何妥善地处理模块？

During layer creation, it’s possible to manipulate the modules that are loaded into the layer and their relations. That’s exactly what a container needs to do to get access to classes in the deployed application. Somehow the container needs to ensure that the package containing the class implementing ContainerApplication is open for deep reflection.

> 在层创建过程中，可以操作加载到层中的模块及其关系，而这正是容器访问已部署应用程序中的类所需要做的事情。不管怎样，容器都需要确保包含实现了 ContainerApplication 的类的包是开放的，以便进行深度反射。

Cleaning up modules is done at the layer level and is surprisingly straightforward. Layers can be garbage collected just like any other Java object. If the container makes sure there are no strong references anymore to the layer, or any of its modules and classes, the layer and everything associated eventually get garbage collected.

> 清理模块是在层级别完成的，并且非常简单明了，就像其他任何 Java 对象一样，层可以被垃圾回收。如果容器确定对层及其任何模块和类不再有强引用，那么层和相关的内容最终都会被垃圾回收。

It’s time to look at some code. The container provided as an example in this chapter is a simple command-line launcher that lists applications you can deploy and undeploy. Its main class is shown in Example 6-5. If you deploy an application, it keeps running until it is undeployed (or the container is stopped). We pass information about where the deployable apps live as command-line arguments in the format out-appa/app.a/app.a.AppA: first the directory that contains the application modules, then the root module name, and last the name of the class that implements ContainerApplication, all separated by slashes.

> 现在可以看一些代码了。本章作为示例所提供的容器是一个简单的命令行启动器，列出了可以部署和取消部署的应用程序。示例 6-5 给出了它的主类。如果部署应用程序，它将一直运行，直到取消部署（或停止容器）。示例以 out-appa/app.a/app.a.AppA 格式将有关可部署应用程序的位置信息作为命令行参数传递：首先是包含应用程序模块的目录，然后是根模块名称，最后是实现了 ContainerApplication 的类的名称，全部以斜线分隔。

Usually, application containers have a deployment descriptor format that would convey such information as part of the application package. There are other ways to achieve similar results, for example, by putting annotations on modules (as shown in “Annotations”) to specify some metadata. For simplicity, we derive AppDescriptor instances containing this information from the command-line arguments.

> 通常，应用程序容器具有一个部署描述符格式，它将相关信息作为应用程序包的一部分来传递。当然，也可以使用其他方法来实现类似结果，例如，通过将注释放在模块上（如 6.2.3 节所示）来指定一些元数据。为了简单起见，从命令行参数中导出包含此信息的 AppDescriptor 实例。

Example 6-5. Launching the application container (➥ chapter6/container)

> 示例 6-5：启动应用程序容器（chapter6/container）

```java
public class Launcher {

  private static AppDescriptor[] apps;
  private static ContainerApplication[] deployedApps;

  public static void main(String... args) {
    System.out.println("Starting container");

    deployedApps = new ContainerApplication[args.length];
    apps = new AppDescriptor[args.length];
    for (int i = 0; i < args.length; i++)
      apps[i] = new AppDescriptor(args[i]);

    // handling keyboard input and calling deploy/undeploy omitted
  }

  // methods for deploy/undeploy discussed later
}
```

The main data structures for the container are an array of application descriptors and an array keeping track of started ContainerApplication instances. After the container starts, you can type commands such as deploy 1 or undeploy 2 or exit. The numbers refer to the index in the apps and deployedApps arrays. A layer is created for each application that is deployed. In Figure 6-15, we see two applications deployed (after deploy 1 and deploy 2) in their own layers.

> 该容器的主要数据结构是一个应用程序描述符数组，以及一个用于跟踪已启动的 ContainerApplication 实例的数组。容器启动后，可以键入命令，比如 deploy 1 或 undeploy2，又或者 exit。这些数字指的是 apps 和 deployedApps 数组中的索引。为每个已部署的应用程序创建一个层。在图 6-15 中，可以看到两个应用程序部署（在输入了 deploy 1 和 deploy 2 之后）在各自的层中。

<Figures figure="6-15">Two applications deployed in the container</Figures>

The image is quite similar to Figure 6-12, save for the fact that the provides and uses relations are inverted. In platform.api, we find the service interfaces to the container functionality, such as platform.api.tx.TransactionManager. The platform.container module contains service providers for these interfaces, and contains the Launcher class we saw earlier. Of course, the container and applications in the real world probably consist of many more modules.

> 除了 provides 和 uses 关系相反之外，该图与图 6-12 非常相似。在 platform.api 中，找到了容器功能的服务接口，比如 platform.api.tx.TransactionManager。platform.container 模块包含了这些接口的服务提供者，以及之前所看到的 Launcher 类。当然，现实世界中的容器和应用程序可能包含更多模块。

Creating a layer for an application looks similar to what we saw when loading plug-ins, with a small twist:

> 为应用程序创建一个层看起来与在加载插件时所完成的事情类似，但略微有所不同：

```java
private static ModuleLayer.Controller createAppLayer(AppDescriptor appDescr) {
  ModuleFinder finder = ModuleFinder.of(Paths.get(appDescr.appDir));
  ModuleLayer parent = ModuleLayer.boot();

  Configuration cf = parent.configuration()
     .resolve(finder, ModuleFinder.of(), Set.of(appDescr.rootmodule)); 1

  ClassLoader scl = ClassLoader.getSystemClassLoader();
  ModuleLayer.Controller layer =
    ModuleLayer.defineModulesWithOneLoader(cf, List.of(parent), scl); 2

  return layer;
}
```

1. A ModuleFinder and Configuration are created based on the AppDescriptor metadata.
2. The layer is created with the static ModuleLayer.defineModulesWithOneLoader method, which returns a ModuleLayer.Controller.

---

> 1. 根据 AppDescriptor 元数据创建 ModuleFinder 和 Configuration。
> 2. 使用静态 ModuleLayer.defineModulesWithOneLoader 方法创建层，并返回 ModuleLayer.Controller。

Each application is loaded into its own isolated layer, with a single fresh classloader. Even if applications contain modules with the same packages, they won’t conflict. By using the static ModuleLayer::defineModulesWithOneLoader, we get back a ModuleLayer.Controller object.

> 使用一个新的类加载器将每个应用程序都加载到自己的隔离层中，即使应用程序包含具有相同包的模块，也不会发生冲突。通过使用静态 ModuleLayer::defineModules WithOneLoader，可以得到一个 ModuleLayer.Controller 对象。

This controller is used by the calling method to open the package in the root module that contains the class implementing ContainerApplication in the deployed application:

> 通过调用方法使用此控制器公开根模块中的包，该包包含了在部署应用程序中实现 ContainerApplication 的类：

```java
private static void deployApp(int appNo) {
  AppDescriptor appDescr = apps[appNo];
  System.out.println("Deploying " + appDescr);

  ModuleLayer.Controller appLayerCtrl = createAppLayer(appDescr); 1
  Module appModule = appLayerCtrl.layer() 2
    .findModule(appDescr.rootmodule)
    .orElseThrow(() -> new IllegalStateException("No " + appDescr.rootmodule));

  appLayerCtrl.addOpens(appModule, appDescr.appClassPkg,
    Launcher.class.getModule()); 3

  ContainerApplication app = instantiateApp(appModule, appDescr.appClass); 4
  deployedApps[appNo] = app;
  app.startApp(); 5
}
```

1. Calling the previously defined createAppLayer method to obtain the ModuleLayer.Controller.
2. From the controller, we can get the actual layer and locate the root module that has been loaded.
3. Before using reflection to instantiate the application class in the root module, we ensure that the given package is open.
4. Now, the instantiateApp implementation can use reflection to instantiate the application class without restrictions.
5. Finally, the deployed application is started by calling startApp.

---

> 1. 调用前面所定义的 createAppLayer 方法，获取 ModuleLayer.Controller。
> 2. 从控制器中可以获取实际层并找到所加载的根模块。
> 3. 在使用反射以实例化根模块中的应用程序类之前，首先确保给定的包是开放的。
> 4. 现在，instantiateApp 实现可以使用反射以在不受限制的情况下实例化应用程序类。
> 5. 最后，通过调用 startApp 启动已部署的应用程序。

The most interesting line in the deployApp is where ModuleLayer.Controller::addOpens is invoked. It opens the package mentioned in the AppDescriptor from the application’s root module to the container module, as shown in Figure 6-16.

> deployApp 中最有趣的一行代码是调用 ModuleLayer.Controller::addOpens。它将 AppDescriptor 中所提及的包从应用程序的根模块开放给容器模块，如图 6-16 所示。

<Figures figure="6-16">At layer creation, the packages app.a and app.b are opened to the platform.container module</Figures>

This qualified opens enables the container to reflect over the packages app.a and app.b. In addition to addOpens(Module source, String pkg, Module target), you can call addExports(Module source, String pkg, Module target) or addReads(Module source, Module target) on a layer controller. With addExports, a previously encapsulated package can be exported (without being opened). And, by establishing readability with addReads, the target module can access the exported packages of the source module. In all cases, source modules must come from the layer that has been constructed. Layers can, in effect, reshape the dependencies and encapsulation boundaries of modules. As always, with great power comes great responsibility.

> 这个受限 opens 方法能够让容器反射包 app.a 和 app.b。除了 addOpens(Module source, Stringpkg, Module target)之外，还可以在层控制器上调用 addExports (Module source, Stringpkg, Module target)或者 add Reads (Module source, Module target)。如果使用 addExports，可以导出先前封装的包（而无须开放）。此外，通过使用 addReads 建立可读性，目标模块可以访问源模块的导出包。无论何种情况，源模块必须来自已经构建的层。实际上，层可以重塑模块的依赖关系和封装边界。一如既往，权力越大，责任也就越大。

### 6.3.5 Resolving Platform Modules in Containers 解析容器中的平台模块

Isolation in the container architectures described so far is achieved by resolving a new application or plug-in within its own ModuleLayer. Each application or plug-in can require its own libraries, possibly even different versions. As long as ModuleFinder can locate the necessary modules for ModuleLayer, everything is fine.

> 到目前为止，所描述的容器体系结构中的隔离都是通过在自己的 ModuleLayer 中解析新应用程序或插件来实现的。每个应用程序或插件可能需要自己的库，甚至可能需要不同的版本。只要 ModuleFinder 可以找到 ModuleLayer 所需的模块，一切都没有问题。

But what about dependencies to platform modules from these freshly loaded applications or plug-ins? At first sight, it might seem there is no problem. ModuleLayer can resolve modules in parent layers, eventually reaching the boot layer. The boot layer contains platform modules, so all is well. Or is it? That depends on which modules were resolved into the boot layer while starting the container.

> 但是，这些新加载的应用程序或插件与平台模块之间的依赖关系如何呢？乍一看，这个问题非常容易回答。ModuleLayer 可以解析父层中的模块，并最终到达引导层。引导层包含平台模块，所以一切应该没有问题。但事实真的是这样的吗？这取决于在启动容器时将哪些模块解析到引导层。

Normal module resolution rules apply: the root module that is started determines which platform modules are resolved. When the root module is the container launcher, only the dependencies of the container launcher module are taken into account. Only the (transitive) dependencies of this root module end up in the boot layer at run-time.

> 正常的模块解析规则是：启动的根模块决定哪些平台模块被解析。当根模块是容器启动器时，只考虑容器启动器模块的依赖项。只有根模块的（可传递）依赖项在运行时最终到达引导层。

When a new application or plug-in is loaded after startup, these might require platform modules that were not required by the container itself. In that case, module resolution fails for the new layer. This can be prevented by using --add-modules ALL-SYSTEM when starting the container module. All platform modules are in that case resolved, even if the module that is started doesn’t depend on them. The boot layer has all possible platform modules because of the ALL-SYSTEM option. This ensures that applications or plug-ins loaded at run-time can require arbitrary platform modules.

> 在启动后加载新应用程序或插件时，可能会使用一些容器本身不需要的平台模块。在这种情况下，新层的模块解析将会失败。可以通过在启动容器模块时使用--add-modules ALL-SYSTEM 来防止此类情况的发生。此时，即使启动的模块不依赖于这些模块，所有平台模块也会被解析。由于使用了 ALL-SYSTEM 选项，引导层具有了所有可能的平台模块，这样做可以确保在运行时加载的应用程序或插件可以要求任意的平台模块。

You have seen how layers enable dynamic architectures. They allow constructing and loading new module graphs at run-time, possibly changing the openness of packages and readability relations to suit the container. Modules in different layers do not interfere, allowing for multiple versions of the same module to coexist in different layers. Services are a natural fit when interacting with newly loaded modules in layers. However, it’s also still possible to take a more traditional, reflection-based approach, as you have seen.

> 前面介绍了层如何启用动态体系结构。它们允许在运行时构建和加载新的模块图，从而可能会改变包的开放性和可读性关系以适应容器。不同层中的模块不会互相干扰，从而允许同一模块的多个版本共存于不同的层中。当需要与层中新加载的模块进行交互时，服务是非常合适的。但是，正如所看到的那样，也可以采取更传统的、基于反射的方法。

The ModuleLayer API is not expected to be widely used in normal application development. In some ways, the API is similar in nature to classloaders: a powerful feature, mostly wielded by frameworks to make the lives of developers easier. Existing Java module frameworks are expected to use layers as a means of interoperability. It’s up to frameworks to put the new ModuleLayer API to good use, just as they did with classloaders over the past 20 years.

> ModuleLayer API 在日常的应用程序开发中并没有广泛使用。从某些方面讲，该 API 在性质上类似于类加载器：一个非常强大的功能，主要由框架实现，使开发人员的工作更加轻松。现有的 Java 模块框架预计将使用层作为互操作性的一种手段，就像在过去的 20 年里使用类加载器一样，是否可以好好利用新的 ModuleLayer API 取决于框架。
