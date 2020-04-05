# 第 7 章 没有模块的迁移

> Chapter 7. Migration Without Modules

Backward compatibility has always been a major goal for Java. Usually, migration to a new Java version is mostly trivial from the developer perspective. The module system and the modularized JDK arguably represent the biggest change to the whole Java platform since its inception. Even so, backward compatibility is a top priority.

> 向后兼容性一直是Java的主要目标。通常，从开发人员的角度来看迁移到新的Java版本通常是微不足道的。模块化系统和模块化JDK可以说是自Java平台出现以来对整个平台最大的改变。但即便如此，向后兼容性也是重中之重。

Migrating an existing application to Java 9 is best approached as a two-step process. This chapter focuses on migrating existing code to build and run on Java 9, without migrating code to modules. The next chapter dives into migrating code to modules, providing strategies to accomplish this.

> 将现有的应用程序迁移到Java 9最好分两步进行。本章重点介绍如何迁移现有代码以便在Java 9上构建和运行，而无须将代码迁移到模块。下一章将深入讨论如何将代码迁移到模块，并提供了实现这一目标的策略。

NOTE

Why migrate to Java 9 when you don’t anticipate using its flagship feature, the module system? An upgrade to Java 9 also gives access to the other features that are part of Java 9. Think of new APIs, tools, and performance improvements.

> 当不需要使用Java 9的旗舰功能——模块系统时，为什么还要迁移到Java 9呢？升级到Java 9后就可以访问属于Java 9的其他功能了，想一想新的API、工具和性能改进。

Whether you go all the way to modules or leave it at the first step depends. Is the application likely to see lots of extensions and new features? In that case, reaping the benefits of modularity may justify the cost of taking the second step. When an application is in maintenance mode and only has to run on Java 9, it makes sense to only take the first step, as described in this chapter.

> 究竟是一直使用模块还是从一开始就不使用模块要视情况而定。应用程序是否可以得到更多扩展和新功能？如果可以，那么模块化所带来的好处可以证明采取第二步所付出的代价是值得的。正如本章所述，当应用程序处于维护模式并且只需在Java 9上运行时，那么只需要完成执行第一步就可以了。

For library maintainers, the question isn’t if Java 9 support is necessary, but when. Migrating a library to Java 9 and modules raises different concerns than migrating applications. In Chapter 10, we address those concerns.

> 对于库的维护者来说，问题不在于Java 9的支持是否必要，而在于何时提供支持。相比于迁移应用程序，将库迁移到Java 9和模块会引发完全不同的问题，在第10章中将会解决这些问题。

But first, what does it take to bring an application to Java 9, without adopting modules for the application yet? It should be clear that an application that has been developed for Java 8 or earlier, while following best practices such that only public JDK APIs are used, will just work. JDK 9 is still backward compatible, but many internal changes have been made. Migration problems you might run into are often caused by improper use of the JDK, either by the application’s code itself, or, more likely, by its libraries.

> 但首先，如果没有为应用程序采用模块，那么将应用程序迁移到Java 9上需要做些什么呢？应该清楚的是，为Java 8或更早版本所开发的应用程序必须遵循最佳实践（仅使用公共JDK API）才能正常工作。虽然JDK 9仍然是向后兼容的，但是已经做了许多内部改变。可能遇到的迁移问题通常是由JDK的错误使用造成的，而这些错误使用可能是应用程序代码本身的问题，或者更可能是库的问题。

Libraries can be a source of frustration when it comes to migration. Many frameworks and libraries have made assumptions on (nonpublic, and therefore unsupported) implementation details of the JDK. Technically, the JDK can’t be blamed for breaking this code. In reality, things are more nuanced. “Libraries, Strong Encapsulation, and the JDK 9 Classpath” explains the compromise that was reached to work toward stronger encapsulation while not breaking existing libraries.

> 当谈到迁移问题时，库可能是迁移失败的主要根源。许多框架和库对JDK的（非公共的，因此也是不支持的）实现细节进行了假设。从技术上讲，不能责怪JDK破坏了代码。事实上，事情更加微妙。7.2节介绍了在不破坏现有库的情况下实现强封装的折中办法。

In an ideal world, libraries and frameworks update their implementations to be Java 9 compatible before Java 9 is released. That’s not the world we live in, unfortunately. As a user of libraries and frameworks, you should know how to work around potential problems. The remainder of this chapter focuses on strategies to get your applications running on Java 9, even in a nonideal world. Hopefully, with time, this chapter becomes obsolete.

> 在一个理想的世界中，在Java 9发布之前库和框架就将它们的实现更新为与Java 9兼容。但不幸的是，这不是我们生活的世界。作为库和框架的用户，应该知道如何解决潜在的问题。本章的其余部分将重点介绍即使在非理想的世界中，应该采取哪些策略使应用程序在Java 9上运行。希望随着时间的流逝，这一章会变得过时。

## 7.1 The Classpath Is Dead, Long Live the Classpath 类路径已经“死”了？
Previous chapters introduced the module path. In many ways, you can view the module path as the successor of the classpath. Does this mean the classpath is gone in Java 9? Or that it’s going away, at all? Absolutely not! History will tell whether the classpath is ever removed from Java. Meanwhile, the classpath is still available in Java 9, and works largely the same as in previous releases. The classpath can even be combined with the new module path, as you will see in the next chapter.

> 前面的章节介绍了模块路径。在许多方面，可以将模块路径视为类路径的继任者。这是否意味着类路径在Java 9中不存在了？或者说它消失了吗？绝对不是！历史将会告诉我们类路径是否应该从Java中移除。与此同时，类路径在Java 9中仍然可用，并且工作方式与以前的版本大致相同。在下一章将会看到，类路径甚至可以与新的模块路径结合使用。

When we ignore the module path, and use the classpath to build and run applications, we’re simply not using the new module features in our application. This requires minimal (if any) changes to existing code. Roughly speaking, when your application and its dependencies use only officially sanctioned APIs from the JDK, it should compile and run without issues on JDK 9.

> 如果忽略模块路径，而使用类路径来构建和运行应用程序，那么我们根本就没有在应用程序中使用新的模块功能。此时需要对现有代码进行最少的（如果有的话）更改。简单地说，当应用程序及其依赖项仅使用来自JDK的官方认可的API时，它应该可以在JDK 9上编译并运行而不会出现任何问题。

If changes are necessary, they arise from the fact that the JDK itself has been modularized. Whether or not your application uses modules, the JDK it runs on always consists of modules as of Java 9. Although the module system is mostly ignored from an application perspective in this scenario, the changes to the JDK structure are still there. In many cases, the modular JDK doesn’t pose any problems for classpath-based applications, but there are definitely some caveats. Those caveats are in most cases related to libraries. The remainder of the chapter covers the possible problems, and more important, their workarounds.

> 如果说更改是必须的，那是因为JDK本身已经实现了模块化。无论应用程序是否使用了模块，从Java 9开始，它运行的JDK始终由模块组成。虽然此时从应用程序的角度来看，可以忽略模块系统，但对JDK结构的更改仍然存在。在大多数情况下，模块化JDK不会给基于类路径的应用程序带来任何问题，但肯定会有一些警告。这些警告在大多数情况下与库有关。本章的其余部分将介绍可能存在的问题，更重要的是这些问题的解决方法。

## 7.2 Libraries, Strong Encapsulation, and the JDK 9 Classpath 库、强封装和JDK 9类路径
One of the problems you can run into when migrating a classpath-based application to Java 9 is caused by the strong encapsulation of code in platform modules. Many libraries use classes from the platform that are now encapsulated with Java 9. Or, they use deep reflection to pry their way into nonpublic parts of platform classes.

> 将基于类路径的应用程序迁移到Java 9时可能遇到的一个问题是由平台模块中代码的强大封装所引起的。许多库使用了平台中用Java 9封装的类，或者使用深度反射来窥探平台类的非公共部分。

Deep reflection is using the reflection API to get access to nonpublic elements of a class. In “Deep Reflection”, you learned that exporting a package from a module does not make its nonpublic elements accessible for reflection. Unfortunately, many libraries call setAccessible on private elements found through reflection.

> 深度反射使用反射API来访问类的非公共元素。在6.1.1节中曾经讲过，从模块中导出包不会使其非公共元素可用于反射。但不幸的是，许多库调用了通过反射找到的私有元素上的setAccessible。

You have seen that when using modules, JDK 9 by default disallows access to encapsulated packages and deep reflection on code in other modules, which includes platform modules. There is a good reason for this: abuse of platform internals has been the source of many security issues, and allowing it hampers evolution of APIs. However, in this chapter, we’re still dealing with classpath-based applications on top of a modular JDK. On the classpath, strong encapsulation of platform internals is not enforced as strictly, although it still plays a role.

> 可以看到，当使用模块时，默认情况下JDK 9不允许访问封装的包以及深度反射其他模块（包括平台模块）中的代码。这样做的一个很好的理由是：平台内部类的滥用已经成为许多安全问题的源头，并且阻碍了API的发展。但是，在本章中仍然需要在模块化JDK之上处理基于类路径的应用程序。在类路径上，平台内部类的强封装并不是严格执行的，尽管它仍然起作用。

Using deep reflection on JDK types is an obscure use case. Why would you want to make private parts of JDK classes accessible? It turns out some commonly used libraries do this. An example of this is the javassist runtime code-generation library, which is used by many other frameworks.

> 对JDK类型使用深度反射有时是非常让人费解的。为什么要使JDK类的私有部分可访问呢？事实证明，一些常用的库都是这么做的。javassist运行时代码生成库就是一个例子，其他许多框架都使用它。

To ease migration of classpath-based applications to Java 9, the JVM by default shows a warning when deep reflection is applied on classes in platform modules. Or, when reflection is used to access types in nonexported packages. For example, when running code that uses the javassist library, we see the following warning:

> 为了便于将基于类路径的应用程序迁移到Java 9，在对平台模块中的类应用深度反射时，或者使用反射来访问非导出包中的类型时，JVM默认显示警告。例如，当运行使用javassist库的代码时，会看到以下警告：
```log
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by javassist.util.proxy.SecurityActions
  (...javassist-3.20.0-GA.jar) to method
  java.lang.ClassLoader.defineClass(...)
WARNING: Please consider reporting this to the maintainers of
  javassist.util.proxy.SecurityActions
WARNING: Use --illegal-access=warn to enable warnings of further illegal
  reflective access operations
WARNING: All illegal access operations will be denied in a future release
```
Let that sink in for a bit. Code that ran without any issues on JDK 8 and earlier now prints a prominent warning to the console—even in production. It shows how seriously the breach of strong encapsulation is taken.

> 接下来仔细思考一下。那些在JDK 8和更早的版本上运行没有任何问题的代码现在会在控制台上显示一个醒目的警告——即使是在生产环境中也是如此。这表明严重破坏了强封装。

Besides this warning, the application will still run as usual. As indicated by the warning message, the behavior will change in a next version of Java. In the future, the JDK will enforce strong encapsulation of platform modules even for code on the classpath. The same application will not run on default settings in a future Java release. Therefore, it is important to investigate the warnings and to fix the underlying problems. When the warnings are caused by libraries, that usually means reporting the issue to the maintainers.

> 除了这个警告之外，应用程序仍然照常运行。如警告消息所示，在下一个Java版本中行为将发生变化。将来，即使是类路径上的代码，JDK也会强制执行平台模块的强封装。在未来的Java版本中，相同的应用程序将不能在默认设置下运行。因此，好好研究一下警告信息并解决潜在的问题是非常重要的。如果警告是由库引起的，那么通常意味着向维护者报告了相关问题。

By default, only a single warning is generated on the first illegal access attempt. Following attempts will not generate extra errors or warnings. If we want to further investigate the cause of the problem, we can use different settings for the --illegal-access command-line flag to tweak the behavior:

> 在默认情况下，只会在第一次非法访问尝试时产生一个警告，而后续的尝试将不会产生额外的错误或警告。如果想要进一步调查问题的原因，可以使用--illegal-access命令行标志的不同设置来调整行为：
```
--illegal-access=permit
```
The default behavior. Illegal access to encapsulated types is allowed. Generates a warning on the first illegal access attempt through reflection.

> 默认行为。允许对封装类型进行非法访问。当第一次尝试通过反射进行非法访问时会生成一个警告。
```
--illegal-access=warn
```
Like permit, but generates an error on every illegal access attempt.

> 与permit一样，但每次非法访问尝试时都会产生错误。
```
--illegal-access=debug
```

Also shows stack traces for illegal access attempts.

> 同时显示非法访问尝试的堆栈跟踪。
```
--illegal-access=deny
```
Does not allow illegal access attempts. This will be the default in the future.

> 不允许非法的访问尝试。这将是未来的默认行为。

Notice that none of the settings allow you to suppress the printed warnings. This is by design. In this chapter, you’ll learn how to address the underlying issues, in order to resolve the illegal access warnings. Because --illegal-access=deny will be the future default, your goal is to run your application with this setting.

> 请注意，没有允许取消打印的警告的设置。这是由设计所决定的。在本章中，将学习如何解决潜在的问题，以消除非法访问警告。由于--illegal-access=deny将是未来的默认设置，因此下一步目标是使用此设置运行应用程序。

If we run code that uses javassist with --illegal-access=deny, the application fails to run and we see the following error:

> 如果使用--illegal-access=deny运行使用javassist的代码，那么应用程序将无法运行，并且会看到以下错误：
```log
java.lang.reflect.InaccessibleObjectException: Unable to make protected final
    java.lang.Class java.lang.ClassLoader.defineClass(java.lang.String,byte[],
                                        int,int,java.security.ProtectionDomain)
  throws java.lang.ClassFormatError accessible: module java.base does not
  "opens java.lang" to unnamed module @0x7b3300e5
```
This error explains that javassist tries to make the defineClass method on java.lang.Class public. We can use the --add-opens flag to grant the classpath deep reflection access to a specific package in a module. Open modules and open packages are discussed in detail in “Deep Reflection”. As a refresher, a package needs to be open to allow deep reflection. This is even true when the package is exported as is the case here with java.lang. A package is usually opened in a module descriptor, similar to the way packages are exported. We can do the same from the command line for modules that we don’t control (for example, platform modules):

> 这个错误解释了javassist试图调用java.lang.Class上的defineClass方法。可以使用--add-opens标志授予对模块中特定包的类路径深度反射访问。“深度反射”一节中已经详细地讨论了开放式模块和开放式包。现在复习一下相关内容，为了进行深度反射，需要开放包。当导出包时也是如此，就像此时所使用的java.lang一样。通常在模块描述符中开放一个包，这与导出包的方式类似。可以通过命令行对那些无法控制的模块（例如平台模块）执行相同的操作：
```
java --add-opens java.base/java.lang=ALL-UNNAMED
```
In this example, java.base/java.lang is the module/package we grant access to. The last argument is the module that gets the access. Because the code is still on the classpath, we use ALL-UNNAMED, which represents the classpath. The package is now open, so the deep reflection is no longer illegal. This will remove the warning (or error, when running with --illegal-access=deny). Similarly, when code on the classpath tries to access a type in a nonexported package, you can use --add-exports to force the package to be exported. We’ll see an example of this scenario in the next section. Remember that this is still just a workaround. Ask the maintainers of a library that causes illegal access problems for an updated version of the library with a proper fix.

> 在本示例中，java.base/java.lang是授权访问的模块/包。最后一个参数是获取访问权限的模块。因为代码仍然在类路径中，所以使用了ALL-UNNAMED，它表示类路径。现在，这个包是开放的，所以深度反射不再是非法的，从而消除了警告（或者避免使用--illegal-access = deny运行时出错）。同样，当类路径上的代码尝试访问非导出包中的类型时，可以使用--add-exports来强制导出包。在下一节中将会看到这种情况的一个例子。请记住，这仅仅是一种解决方法。可以询问一个库的维护者，即使是使用正确修复的已更新库版本，也会导致非法访问问题。

WARNING

Illegal access is allowed by the default setting --illegal-access=permit only on packages that already existed before JDK 9, but are now encapsulated. Any new encapsulated packages in JDK 9 are not exempt from strong encapsulation, even when code is on the classpath.

> 默认设置--illegalaccess=permit允许对JDK 9之前已经存在但现在被封装的包进行非法访问。即使代码位于类路径中，JDK 9中的任何新的封装包都会被强封装。

#### SECURITY IMPACT 安全影响
How does the existence of --add-opens and --add-exports impact security? One of the reasons to not allow deep reflection by default for platform modules in a future Java version is to prevent malicious code from reaching dangerous JDK internals. Doesn’t a flag to simply disable these checks void this important security benefit? One the one hand, yes, it selectively opens up a bigger attack surface when you choose to do so.

> --add-opens和--add-exports如何影响安全性？在未来的Java版本中，默认情况下不允许对平台模块进行深度反射的原因之一是为了防止恶意代码到达危险的JDK内部。是否可以使用一个标志来简单地禁用这些检查，同时继续保持这种安全优势呢？一方面，答案是肯定的，但是当选择这么做时，无形间就打开了一个更大的攻击面。

But consider this: there’s no way to gain the privileges afforded by --add-opens or --add-exports at run-time by merely executing Java code. An attacker needs to have access to the startup scripts (the command line) of an application to add these flags. When that level of access is established, the breach already allows the attacker to make arbitrary modifications reaching much further than just adding JVM options.

> 但考虑到这点：仅仅通过执行Java代码，无法在运行时获得由--add-opens或--add-exports提供的权限。攻击者需要访问应用程序的启动脚本（命令行）来添加这些标志。当建立了这种访问级别时，攻击者已经可以进行任意修改了，而不仅仅是添加JVM选项了。

## 7.3 Compilation and Encapsulated APIs 编译和封装的API
The JDK contains many private, internal APIs. They are not supposed to be used by anyone other than the JDK itself. This has been clearly documented since the early beginnings. Examples are the sun.* and jdk.internal.* packages. As an application developer, you likely are not using these types directly. Most of these internal classes serve obscure corner cases, which typical applications don’t need. For this book, we even found it difficult to come up with a good example from the application development perspective.

> JDK包含许多私有的内部API，这些API应该仅能被JDK所使用。从早期开始，这一规定就已被清楚地记载下来了。比如sun.*和jdk.internal.*包。作为应用程序开发人员，可能并不会直接使用这些类型。大多数的内部类仅用于一些极端情况，典型的应用程序通常不需要。在编写本书的时候，甚至很难从应用程序开发的角度提供一个恰当的示例。

Of course, some applications and (especially older) libraries do still use those internal classes. JDK internals were not strongly encapsulated previously, because there was no mechanism to do so. Pre-Java 9 compilers do emit warnings when using internal classes, but those are easily overlooked or ignored. We have seen that for the time being, code compiled with older versions of Java that uses encapsulated JDK types will still run on Java 9, because of the --illegal-access=permit default setting.

> 当然，一些应用程序以及（较旧的）库仍然使用这些内部类。在以前，JDK内部没有进行强封装，因为当时没有这样的机制。当使用内部类时，Java 9之前的编译器会发出警告，但那些警告很容易被忽略。前面已经看到，随着时间的推移，因为--illegal-access=permit默认设置，使用了封装JDK类型且在旧版本Java上编译的代码仍然可以在Java 9上运行。

The same code will not compile on Java 9, however! Let’s say we have code (see Example 7-1) compiled with the JDK 8 compiler that uses types from the sun.security.x509 package.

> 然而，相同的代码在Java 9上却无法通过编译！假设使用JDK 8编译器编译示例7-1所示的代码，该代码使用了sun.security.x509包中的类型。

Example 7-1. EncapsulatedTypes.java (➥ chapter7/encapsulation)

> 示例7-1:EncapsulatedTypes.java（chapter7/encapsulation）
```java
package encapsulated;

import sun.security.x509.X500Name;

public class EncapsulatedTypes {
    public static void main(String... args) throws Exception {
        System.out.println(new X500Name("test.com", "test",
                     "test", "US"));

    }
}
```
Compiling this code with JDK 9 results in the following compiler error:

> 使用JDK 9编译上面的代码，会产生如下所示编译器错误：
```log
./src/encapsulated/EncapsulatedTypes.java:3: error: package sun.security.x509
is not visible
import sun.security.x509.X500Name;
                   ^
  (package sun.security.x509 is declared in module java.base, which does not
   export it to the unnamed module)
```
By default, this code will still run successfully on Java 9, although the code is using an encapsulated package. You might wonder why there’s a difference between javac and java when it comes to accessing encapsulated types. What’s the point of being able to run code that accesses encapsulated types when you can’t compile the same code?

> 默认情况下，尽管代码使用封装包，但代码仍然可以在Java 9上成功运行。你可能想知道为什么在访问封装类型时，javac和java之间存在区别。当无法编译相同的代码时，能够运行访问封装类型的代码又有什么意义呢？

The reason that such code is still able run is to provide backward compatibility for existing libraries. The reason compiling with those same encapsulated types is prohibited is to prevent future compatibility nightmares. For code that you control, you should take immediate action when it comes to encapsulated types and replace them with nonencapsulated alternatives. When using a library (compiled with an older Java version) that’s using encapsulated types or deep reflection on JDK internals, you’re in a more difficult spot. You can’t fix the problem yourself, which would block you in your attempt to move to Java 9. Because of the lenient runtime, the library can still be used for the time being.

> 这样的代码之所以仍然能够运行，其原因是为现有的库提供向后兼容性。而禁止相同封装类型编译的原因是为了防止将来可能出现的兼容性噩梦。对于自己可以控制的代码，当涉及封装类型时，应立即采取行动，并用非封装的替代类型替换它们。当所使用的库（用较旧的Java版本进行编译）使用了封装类型或对JDK内部进行深度反射时，情况就更加复杂了。此时无法自己修复这个问题，这样一来，会阻止你向Java 9的迁移。但由于较宽松的运行时，因此库仍然可以暂时使用。

Allowing the usage of encapsulated JDK types at run-time is only a temporary situation. In a future Java release, this will be disabled. We can already prepare for this today by setting the --illegal-access=deny flag that we have seen in the previous section. Running the same code with java --illegal-access=deny generates an error:

> 允许在运行时使用封装的JDK类型只是一种临时情况。在未来的Java版本中，这种情况将被禁止。通过设置上一节中所介绍的--illegal-access=deny标志可以防止这类情况的出现。使用java--illegal-access=deny运行相同的代码会生成一个错误：
```log
Exception in thread "main" java.lang.IllegalAccessError:
class encapsulated.EncapsulatedTypes (in unnamed module @0x2e5c649) cannot
access class sun.security.x509.X500Name (in module java.base) because module
java.base does not export sun.security.x509 to unnamed module @0x2e5c649
        at encapsulated.EncapsulatedTypes.main(EncapsulatedTypes.java:7)
```
TIP

Notice that no warnings are shown for this scenario if we configure --illegal-access with anything other than deny. Only reflective illegal access triggers the warnings we have seen, not static references to encapsulated types as in this case. This restriction is a pragmatic one: changing the VM to also generate warnings for static references to encapsulated types would be too invasive.

> 请注意，如果使用deny之外的任何其他值配置--illegal-access，则不会显示任何警告。只有反射性的非法访问会触发前面所看到的警告，而静态引用封装类型则不会。该限制非常实用：如果将VM更改为对封装类型的静态引用也会产生警告，那么这种更改就显得过于侵入性了。

The right course of action is to report the issue to the maintainers of the library. But what if this is our own code, and we need to recompile with JDK 9 but can’t make code changes right away? Changing code is always risky, so we have to find the right moment to do so.

> 正确的做法是将问题报告给库的维护者。但是，如果是自己的代码，同时需要使用JDK 9重新编译，但又不能立即更改代码，应该怎么做呢？更改代码总是有风险的，所以必须找到更改的合适时机。

We can use command-line flags to break encapsulation at compile-time as well. In the previous section, you saw how to use --add-opens to open a package from the command line. Both java and javac also support --add-exports. As the name suggests, we can use this to export an otherwise encapsulated package from a module. The syntax is `--add-exports <module>/<package>=<targetmodule>`. Because our code is still running on the classpath, we can use ALL-UNNAMED as the target module. Note that exporting an encapsulated package still does not allow deep reflection on its types. The package needs to be open for that. In this case, exporting the package is sufficient. In Example 7-1, we’re referencing the encapsulated type directly, without any reflection involved. For our (admittedly contrived) sun.security.​x509.X500Name example, we can compile and run with the following commands:

> 也可以使用命令行标志在编译时打破封装。在上一节中，看到了如何使用--add-opens从命令行公开一个包。java和javac都支持--add-exports，顾名思义，可以使用它从模块导出一个封装的包。语法是`--add-exports <module>/<package>= <targetmodule>`。因为代码仍然在类路径上运行，所以可以使用ALL-UNNAMED作为目标模块。请注意，导出封装的包并不意味着可以对其类型进行深度反射。要进行深度反射，必须开放包。目前，导出包就足够了。在示例7-1中，直接引用了封装类型，而没有涉及任何反射。而对于sun.security.x509.X500Name示例，可以使用下面的命令编译并运行：
```
javac --add-exports java.base/sun.security.x509=ALL-UNNAMED \
 encapsulated/EncapsulatedTypes.java

java --add-exports java.base/sun.security.x509=ALL-UNNAMED  \
  encapsulated.EncapsulatedTypes
```
The --add-exports and --add-opens flags can be used for any module and package, not only for JDK internals. During compilation, warnings are still emitted for the use of internal APIs. Ideally, the --add-exports flag is a temporary migration step. Use it until you adapt your code to the public APIs, or (if a library is in violation) until there is new release of the third-party library using the replacement API.

> 不仅对JDK内部，--add-exports和--add-opens标志可用于任何模块和包。在编译过程中，内部API的使用仍然会发出警告。理想情况下，--add-exports标志是临时迁移步骤。可以一直使用它，直到将代码调整为公共API，或（如果库违反规则）直到使用替代API的第三方库的新版本发布。

#### TOO MANY COMMAND-LINE FLAGS! 太多命令行标志
Some operating systems limit the length of the command line that can be executed. When you need to add many flags during migration, you can hit these limits. You can use a file to provide all the command-line arguments to java/javac instead:

> 一些操作系统限制了可执行命令行的长度。当在迁移期间需要添加许多标志时，可能会触犯这些限制。此时的替代做法是使用一个文件将所有的命令行参数提供给java/javac：
```
$ java @arguments.txt
```
The argument files must contain all necessary command-line flags. Each line in the file contains a single option. For instance, arguments.txt could contain the following:

> 参数文件必须包含所有必要的命令行标志。文件中的每一行都包含一个选项。例如，arguments.txt可以包含以下内容：
```
-cp application.jar:javassist.jar
--add-opens java.base/java.lang=ALL-UNNAMED
--add-exports java.base/sun.security.x509=ALL-UNNAMED
-jar application.jar
```
Even if you’re not running into command-line limits, argument files can be clearer than a very long line somewhere in a script.

> 即使没有命令行限制，相比于脚本中非常长的命令行，参数文件可能更清晰。

## 7.4 Removed Types 删除的类型
Code also could use internal types, which are now removed entirely. This is not directly related to the module system, but is still worth mentioning. One of the removed internal classes in Java 9 is sun.misc.BASE64Encoder, which was popular before Java 8 introduced the java.util.Base64 class. Example 7-2 shows code using BASE64Decoder.

> 代码也可以使用目前已经完全删除的内部类型，虽然这与模块系统没有直接关系，但仍值得一提。其中一个在Java 9中删除的内部类是sun.misc.BASE64Encoder，在Java 8引入java.util.Base64类之前，该类是非常流行的。示例7-2显示了使用BASE64Decoder的代码：

Example 7-2. RemovedTypes.java (➥ chapter7/removedtypes)

> 示例7-2:RemovedTypes.java（chapter7/removedtypes）
```java
package removed;

import sun.misc.BASE64Decoder;

// Compile with Java 8, run on Java 9: NoClassDefFoundError.
public class RemovedTypes {
    public static void main(String... args) throws Exception {
        new BASE64Decoder();
    }
}
```
This code will no longer compile or run on Java 9. When we try to compile, we see the following error:

> 该代码在Java 9上无法编译或运行。当尝试编译时，会看到以下所示错误：
```log
removed/RemovedTypes.java:3: error: cannot find symbol
import sun.misc.BASE64Decoder;
               ^
  symbol:   class BASE64Decoder
  location: package sun.misc
removed/RemovedTypes.java:8: error: cannot find symbol
        new BASE64Decoder();
            ^
  symbol:   class BASE64Decoder
  location: class RemovedTypes
2 errors
```
If we compile the code with an older Java version, but try to run it with Java 9, it also fails:

> 如果使用旧版本的Java编译上面的代码，但试图在Java 9上运行，也会失败：
```log
Exception in thread "main" java.lang.NoClassDefFoundError: sun/misc/BASE64Decoder
  at removed.RemovedTypes.main(RemovedTypes.java:8)
Caused by: java.lang.ClassNotFoundException: sun.misc.BASE64Decoder
  ...
```
For an encapsulated type, we can work around the problem by forcing access to it with command-line flags. We can’t do this for this BASE64Decoder example, because the class doesn’t exist anymore. It’s important to understand this difference.

> 对于封装类型，可以通过使用命令行标志进行强制访问来解决此问题。但此时针对Base64Decoder示例无法这么做，因为该类不存在。理解这种差异是很重要的。

#### USING JDEPS TO FIND REMOVED OR ENCAPSULATED TYPES AND THEIR ALTERNATIVES 使用jdeps查找已删除或封装的类型及其替代方法
jdeps is a tool shipped with the JDK. One of the things jdeps can do is find usages of removed or encapsulated JDK types, and suggest replacements. jdeps always works on class files, not on source code. If we compile Example 7-2 with Java 8, we can run jdeps on the resulting class:

> jdeps是JDK附带的一个工具。jdeps可以做的一件事是找到使用被删除或封装的JDK类型的地方，并建议替换。jdeps总是在类文件上工作，而不是在源代码上。如果使用Java 8编译示例7-2，则可以在生成的类上运行jdeps：
```log
jdeps -jdkinternals removed/RemovedTypes.class

RemovedTypes.class -> JDK removed internal API
   removed.RemovedTypes -> sun.misc.BASE64Decoder
   JDK internal API (JDK removed internal API)

Warning: JDK internal APIs are unsupported and private to JDK implementation
that are subject to be removed or changed incompatibly and could
break your application.
Please modify your code to eliminate dependence on any JDK internal APIs.
For the most recent update on JDK internal API replacements, please check:
https://wiki.openjdk.java.net/display/JDK8/Java+Dependency+Analysis+Tool

JDK Internal API                         Suggested Replacement
----------------                         ---------------------
sun.misc.BASE64Decoder                   Use java.util.Base64 @since 1.8
```
Similarly, encapsulated types such as X500Name in Example 7-1 are reported by jdeps with suggested replacements. More details on how to work with jdeps are discussed in “Using jdeps”.

> 类似地，示例7-1中诸如X500Name之类的封装类型也被jdeps报告并带有建议的替换方法。有关如何使用jdeps的更多详细信息，请参阅8.9节。

Since Java 8, the JDK includes java.util.Base64, which is a much better alternative to use. The solution in this case is simple: we must migrate to the public API in order to run on JDK 9. In general, moving to Java 9 will expose a lot of technical debt in the areas discussed in this chapter.

> 从Java 8开始，JDK就包含了java.util.Base64，这是一个更好的选择。在这种情况下，解决方案就很简单：必须迁移到公共API以便在JDK 9上运行。一般来说，迁移到Java 9将会暴露本章讨论的许多技术债务（technical debt）。

Technical Debts by Oliver Widder link:http://geek-and-poke.com/geekandpoke/2013/11/20/technical-debts, CC-BY link:https://creativecommons.org/licenses/by/3.0/deed.en_US

## 7.5 Using JAXB and Other Java EE APIs 使用JAXB和其他Java EE API
Certain Java EE technologies, such as JAXB, shipped with the JDK alongside Java SE APIs in the past. These technologies are still present in Java 9, but require special attention. They are shipped in the following list of modules:

> 在过去，JDK附带的某些Java EE技术（如JAXB）是与Java SE API一起提供的。这些技术在Java9中仍然存在，但是需要特别注意。它们主要包含在以下模块列表中：

- java.activation
- java.corba
- java.transaction
- java.xml.bind
- java.xml.ws
- java.xml.ws.annotation

In Java 9, these modules are deprecated for removal. The @Deprecated annotation has a new argument forRemoval in Java 9. When set to true, this means the API element will be removed in a future release. For API elements that are part of the JDK, this means removal may happen in a next major release. More details about deprecation can be found in JEP 277.

> 在Java 9中，这些模块已被弃用。@Deprecated注释在Java 9中有一个新的参数forRemoval。如果将其设置为true，则意味着API元素将在未来版本中被删除。对于属于JDK的API元素来说，这意味着在下一个主要版本中可能会被删除。有关弃用的更多细节可以在JEP277（http://openjdk.java.net/jeps/277）中找到。

There is good reason for removing Java EE technologies from the JDK. The overlap between Java SE and Java EE in the JDK has always been confusing. Java EE application servers usually provide custom implementations of the APIs. Slightly simplified, this is done by putting the alternative implementation on the classpath, overriding the default JDK version. In Java 9, this becomes a problem. The module system does not allow the same package to be provided by multiple modules. If a duplicate package is found on the classpath (hence in the unnamed module), it is ignored. In any case, a situation where both Java SE and an application server provide java.xml.bind would not result in the expected behavior.

> 从JDK中删除Java EE技术有很好的理由。JDK中Java SE和Java EE之间的重叠一直是令人困惑的。Java EE应用程序服务器通常提供了API的自定义实现。简单地讲，是通过将替代实现放在类路径上，覆盖默认的JDK版本来完成的。而在Java 9中，这就成为一个问题。模块系统不允许多个模块提供相同的包。如果在类路径中找到重复的包（因此在未命名的模块中），则会将其忽略。在任何情况下，若Java SE和应用程序服务器都提供java.xml.bind，则不会实现预期的行为。

This is a serious practical problem, which would break many existing application servers and related tools. To avoid this problem, these modules are not resolved by default in classpath-based scenarios. Let’s take a look at the module graph of the platform in Figure 7-1.

> 这是一个严重的实际问题，会破坏许多现有的应用服务器和相关工具。为了避免出现该问题，在基于类路径的场景中，默认情况下不会解析这些模块。接下来看一下图7-1所示的平台模块图。

Subset of the JDK module graph showing modules only reachable through `java.se.ee`, not `java.se`.
Figure 7-1. Subset of the JDK module graph showing modules reachable only through java.se.ee, not java.se

At the very top are the java.se and java.se.ee modules. Both are aggregator modules, modules that don’t contain code but group a set of more fine-grained modules. Aggregator modules are discussed in detail in “Aggregator Modules”. Most platform modules reside under java.se and are not shown here (but you can see the whole graph in Figure 2-1). The java.se.ee module aggregates the modules we are discussing, which are not part of the java.se aggregator module. This includes the java.xml.bind module, containing JAXB types.

> 最顶层是java.se和java.se.ee模块。两者都是聚合器模块，这些模块不包含代码，而是将一组更细粒度的模块组合在一起。聚合器模块已经在5.4节中详细讨论过。大多数平台模块驻留在java.se中，图中并没有显示（但可以从图2-1中看到整个模块图）。java.se.ee模块聚合了正在讨论的模块，它们不是java.se聚合器模块的一部分，其中包括包含了JAXB类型的java.xml.bind模块。

By default, both javac and java use java.se as the root when compiling and running classes in the unnamed module. Code can access any package exported by the transitive dependencies of java.se. Modules under java.se.ee but not under java.se are therefore not resolved, so they are not read by the unnamed module. Even though package javax.xml.bind is exported from module java.xml.bind, it doesn’t matter because it is not resolved during compilation and run-time.

> 默认情况下，在编译和运行未命名模块中的类时，javac和java都使用java.se作为根，代码可以访问由java.se的可传递依赖项导出的任何包。因此，在java.se.ee中却不在java.se下的模块不会被解析，所以它们也不会被未命名的模块读取。即使从模块java.xml.bind中导出包javax.xml.bind也没关系，因为在编译和运行时不会对其进行解析。

If modules under java.se.ee are necessary, we need to add them explicitly to the set of resolved platform modules. We can do so by adding them as root modules with the --add-modules flag of both javac and java.

> 如果java.se.ee中的模块是必需的，那么就需要将它们显式地添加到已解析的平台模块集合中。可以使用javac和java的--add-modules标志将这些模块添加为根模块。

Let’s try this with Example 7-3, based on JAXB. This example serializes a Book to XML.

> 接下来看一下基于JAXB的示例7-3。该示例将Book序列化为XML。

Example 7-3. JaxbExample.java (➥ chapter7/jaxb)

> 示例7-3:JaxbExample.java（chapter7/jaxb）
```java
package example;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Marshaller;

public class JaxbExample {
    public static void main(String... args) throws Exception {
      Book book = new Book();
      book.setTitle("Java 9 Modularity");

      JAXBContext jaxbContext = JAXBContext.newInstance(Book.class);
      Marshaller jaxbMarshaller = jaxbContext.createMarshaller();

      jaxbMarshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);

      jaxbMarshaller.marshal(book, System.out);
    }
}
```
On Java 8, this example compiles and runs without problems. On Java 9, we get several errors at compile-time:

> 在Java 8上，该示例可以编译和运行而不会出现任何问题。但在Java 9上，则会在编译时出现几个错误：
```log
example/JaxbExample.java:3: error: package javax.xml.bind is not visible
import javax.xml.bind.JAXBContext;
               ^
 (package javax.xml.bind is declared in module java.xml.bind, which is not
   in the module graph)
example/JaxbExample.java:4: error: package javax.xml.bind is not visible
import javax.xml.bind.JAXBException;
               ^
 (package javax.xml.bind is declared in module java.xml.bind, which is not
   in the module graph)
example/JaxbExample.java:5: error: package javax.xml.bind is not visible
import javax.xml.bind.Marshaller;
               ^
 (package javax.xml.bind is declared in module java.xml.bind, which is not
   in the module graph)
3 errors
```
When you compile with Java 8 and run the code with Java 9, an exception reporting the same problem is generated at run-time. We already know how to fix this issue: add --add-modules java.xml.bind to both the javac and java invocation.

> 当使用Java 8进行编译并使用Java 9运行代码时，就会在运行时生成报告相同问题的异常。前面已经介绍了如何解决这个问题：将--add-modules java.xml.bind添加到javac和java调用中。

WARNING

Instead of adding the platform module that contains JAXB, you can add a JAR that provides JAXB to the classpath. Several popular (open source) libraries provide JAXB implementations. Since the Java EE modules in the JDK are marked for removal, this is a more future-proof solution.

> 除了添加包含JAXB的平台模块之外，还可以添加一个为类路径提供JAXB的JAR。几个流行的（开源）库提供了JAXB实现。由于JDK中的Java EE模块被标记为“删除”，因此这是一个更具前瞻性的解决方案。

Note that we wouldn’t encounter this problem with the module path. If our code lives in a module, it must explicitly define a requirement on any modules other than java.base. That includes a dependency on java.xml.bind for the example code. Based on this, the module system resolves these modules without the need for a command-line flag.

> 请注意，如果使用模块路径，则不会遇到该问题。如果代码存在于一个模块中，那么它必须在java.base以外的任何模块上显式地定义一个需求。对于示例代码来说，包括对java.xml.bind的依赖。基于此原因，模块系统不需要命令行标志即可解析这些模块。

Summing up, beware when using Java EE code from the JDK. When you’re getting errors that the packages are not visible, add the relevant modules by using --add-modules. Be aware that they will be removed in a next major Java release, however. Adding your own versions of these technologies to the classpath instead avoids future problems.

> 总结一下，当从JDK使用Java EE代码时要格外小心。当接收到包不可见的错误时，可以通过使用--add-modules添加相关模块。但是请注意，所添加的模块将在下一个主要的Java版本中被删除。可以将自己的这些技术版本添加到类路径中，以避免将来出现问题。

## 7.6 The jdk.unsupported Module jdk.unsupported模块
Some internal classes from the JDK have proven to be harder to encapsulate. Chances are that you have never used sun.misc.Unsafe and the like. These have always been unsupported classes, meant to be used only in the JDK internally.

> JDK的一些内部类已经被证明更难以进行封装。一般来说，使用sun.misc.Unsafe等类的机会是比较少的。这些一直是不受支持的类，意味着只能在JDK内部使用。

Some of these classes are widely used by libraries for performance reasons. Although it’s easy to argue that this should never be done, in some cases it’s the only option. A well-known example is the sun.misc.Unsafe class, which can perform low-level operations bypassing Java’s memory model and other safety nets. The same functionality cannot be implemented by libraries outside the JDK.

> 由于性能原因，这些类中的一些被许多库广泛使用。显而易见这种做法是不对的，但在某些情况下这是唯一的选择。一个著名的示例是sun.misc.Unsafe类，它可以绕过Java的内存模型和其他安全网执行一些低级操作，而JDK之外的库无法实现相同的功能。

If such classes would simply be encapsulated, libraries depending on them would no longer work with JDK 9, at least, not without warnings. Theoretically, this is not a backward-compatibility issue. Those libraries abuse nonsupported implementation classes, after all. For some of these highly used internal APIs, the real-world implications would be too severe to ignore, however—especially because there are no supported alternatives to the functionality they provide.

> 如果简单地封装这些类，那么依赖于它们的库将不再使用JDK 9，至少没有警告。从理论上讲，这不是一个向后兼容的问题，毕竟这些库滥用了不支持的实现类。对于这些使用频率比较高的内部API来说，由于其在现实世界中的作用太大而无法忽视，特别是当它们所提供的功能没有替代方案时更是如此。

With that in mind, a compromise was reached. The JDK team researched which JDK platform internals are used by libraries the most, and which of those can be implemented only inside the JDK. Those classes are not encapsulated in Java 9.

> 考虑到这一点，达成了妥协。JDK团队研究了哪些JDK平台内部类是库使用最多的，哪些只能在JDK内部实现。这些类没有封装在Java 9中。

Here’s the resulting list of specific classes and methods that are kept accessible:

> 下面所示的是可以访问的特定类和方法的列表：

- sun.misc.{Signal,SignalHandler}
- sun.misc.Unsafe
- sun.reflect.Reflection::getCallerClass(int)
- sun.reflect.ReflectionFactory::newConstructorForSerialization

Remember, if these names don’t mean anything to you, that’s a good thing. Popular libraries such as Netty, Mockito, and Akka use these classes, though. Not breaking these libraries is a good thing as well.

> 请记住，如果这些名字对你来说没有任何意义，那是一件好事。诸如Netty、Mockito和Akka之类的流行库都使用了这些类。不破坏这些库也是一件好事。

Because these methods and classes were not primarily designed to be used outside the JDK, they are moved to a platform module called jdk.unsupported. This indicates that it is expected the classes in this module will be replaced by other APIs in a future Java version. The jdk.unsupported module exports and/or opens the internal packages containing the classes discussed. Many existing uses involve deep reflection. Using these classes through reflection does not lead to warnings at run-time, unlike the scenarios discussed in “Libraries, Strong Encapsulation, and the JDK 9 Classpath”. That’s because jdk.unsupported opens the necessary packages in its module descriptor, so there is no illegal access from that point of view.

> 由于这些方法和类主要不是为在JDK之外使用而设计的，因此可以将它们移动到名为jdk.unsupported的平台模块中。这表明在未来的Java版本中，该模块中的类将被其他API所替换。jdk.unsupported模块导出和/或公开了包含所讨论类的内部包。现有的用途包括深度反射。与7.2节中所讨论的场景不同，通过反射使用这些类不会导致警告。这是因为jdk.unsupported在其模块描述符中公开了必需的包，所以从这个角度来看不存在非法访问。

WARNING

Although these types can be used without breaking encapsulation, they are still unsupported; their use is still discouraged. The plan is to provide supported alternatives in the future. For example, some of the functionality in Unsafe is superseded by variable handles as proposed in JEP 193. Until then, the status quo is maintained.

> 虽然可以在不破坏封装的情况下使用这些类型，但它们仍然不受支持，依旧不鼓励使用这些类型。预计在将来会提供支持的替代类型。例如，Unsafe中一些功能被JEP193（http://openjdk.java.net/jeps/193）中所提出的变量句柄（variable handle）所取代。但在此之前，仍然维持现状。

When code still lives on the classpath, nothing changes. Libraries can use these classes from the classpath as before, running without any warnings or errors. The compiler generates warnings when compiling against classes from jdk.unsupported, rather than errors as with encapsulated types:

> 当代码仍然存在于类路径上时，不需要进行任何更改。库可以像以前一样通过类路径使用这些类，并且在没有任何警告或错误的情况下运行。编译器针对jdk.unsupported中的类进行编译时会生成警告，而不是像编译封装类型一样产生错误：
```log
warning: Unsafe is internal proprietary API and may be
         removed in a future release
```
If you want to use these types from a module, you must require jdk.unsupported. Having such a requires statement in your module descriptor serves as a warning sign. In a future Java release, changes may be necessary to adapt to publicly supported APIs instead of the unsupported APIs.

> 如果想从模块中使用这些类型，则必须依靠jdk.unsupported。在模块描述符中有这样一个requires语句就相当于是一个警告标志。在未来的Java版本中，其可能需要进行更改以适应公开支持的API，而不是不支持的API。

## 7.7 Other Changes 其他更改
Many other changes in JDK 9 can potentially break code. These changes affect, for example, tool authors, and applications that use the JDK extension mechanisms. Some of the changes include the following:

> JDK 9中的许多其他更改可能会破坏代码。比如，这些更改会影响工具作者以及使用JDK扩展机制的应用程序。其中一些变化如下：

- JDK layout JDK布局
  - Because of the platform modularization, the big rt.jar containing all platform classes doesn’t exist anymore. The layout of the JDK itself has changed considerably as well, as is documented in JEP 220. Tools or code relying on the JDK layout must adapt to this new reality.
  - > 由于平台模块化，包含所有平台类的大rt.jar不再存在。正如JEP 220（http://penjdk.java.net/jeps/220）中所记载的那样，JDK本身的布局也发生了相当大的变化。依靠JDK布局的工具或代码必须适应这个新的现实。
- Version string 版本字符串
  - Gone are the days that all Java platform versions start with the 1.x prefix. Java 9 is shipped with version 9.0.0. The syntax and semantics of the version string have changed considerably. If an application does any kind of parsing on the Java version, read JEP 223 for all the details.
  - > 所有Java平台版本都以1.x为前缀开头的日子已经一去不复返了。Java 9对应的是版本9.0.0。版本字符串的语法和语义已经发生了很大的变化。如果应用程序对Java版本进行任何类型的解析，请阅读JEP 223（http://openjdk.java.net/jeps/223）以了解所有的细节。
- Extension mechanisms 扩展机制
  - Features such as the Endorsed Standard Override Mechanism and the extension mechanism through the java.ext.dirs property are removed. They are replaced by upgradeable modules. More information can be found in JEP 220.
  - > 诸如授权标准覆盖机制（Endorsed Standard Override Mechanism）以及通过java.ext. dirs属性的扩展机制等功能已被删除。它们被可升级的模块所取代。更多的信息可以在JEP220（http://openjdk.java.net/jeps/220）中找到。

These are all highly specialized features of the JDK. If your application does rely on them, it will not work with JDK 9. Because these changes are not really related to the Java module system, we won’t go into further detail. The linked JDK Enhancement Proposals (JEPs) contain guidance on how to proceed in these cases.

> 这些都是JDK的高度专业化功能。如果应用程序确实需要它们，那么它就不能使用JDK 9。因为这些更改与Java模块系统没有真正的关系，所以在此不做详细讨论。链接相关的JEP（JDKEnhancement Proposal）包含有关如何在这些情况下继续进行操作的指导。

Congratulations! You now know how to run your existing application on JDK 9. Even though several things could go wrong, in many cases things will just work. Remember to run your application with --illegal-access=deny as well, to be prepared for the future. After fixing all issues when running existing applications from the classpath, it’s time to look at how to make them more modular.

> 恭喜！现在你已经知道如何在JDK 9上运行现有的应用程序了。虽然有些事情可能出错，但在大多数情况下，还是比较顺利的。请记住，用--illegal-access = deny来运行你的应用程序，以便为将来做好准备。在解决了从类路径中运行现有的应用程序所存在的问题之后，接下来看一下如何使现有的应用程序更加模块化。