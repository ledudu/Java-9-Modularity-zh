# 第 7 章 没有模块的迁移

> Chapter 7. Migration Without Modules
Backward compatibility has always been a major goal for Java. Usually, migration to a new Java version is mostly trivial from the developer perspective. The module system and the modularized JDK arguably represent the biggest change to the whole Java platform since its inception. Even so, backward compatibility is a top priority.

Migrating an existing application to Java 9 is best approached as a two-step process. This chapter focuses on migrating existing code to build and run on Java 9, without migrating code to modules. The next chapter dives into migrating code to modules, providing strategies to accomplish this.

NOTE
Why migrate to Java 9 when you don’t anticipate using its flagship feature, the module system? An upgrade to Java 9 also gives access to the other features that are part of Java 9. Think of new APIs, tools, and performance improvements.

Whether you go all the way to modules or leave it at the first step depends. Is the application likely to see lots of extensions and new features? In that case, reaping the benefits of modularity may justify the cost of taking the second step. When an application is in maintenance mode and only has to run on Java 9, it makes sense to only take the first step, as described in this chapter.

For library maintainers, the question isn’t if Java 9 support is necessary, but when. Migrating a library to Java 9 and modules raises different concerns than migrating applications. In Chapter 10, we address those concerns.

But first, what does it take to bring an application to Java 9, without adopting modules for the application yet? It should be clear that an application that has been developed for Java 8 or earlier, while following best practices such that only public JDK APIs are used, will just work. JDK 9 is still backward compatible, but many internal changes have been made. Migration problems you might run into are often caused by improper use of the JDK, either by the application’s code itself, or, more likely, by its libraries.

Libraries can be a source of frustration when it comes to migration. Many frameworks and libraries have made assumptions on (nonpublic, and therefore unsupported) implementation details of the JDK. Technically, the JDK can’t be blamed for breaking this code. In reality, things are more nuanced. “Libraries, Strong Encapsulation, and the JDK 9 Classpath” explains the compromise that was reached to work toward stronger encapsulation while not breaking existing libraries.

In an ideal world, libraries and frameworks update their implementations to be Java 9 compatible before Java 9 is released. That’s not the world we live in, unfortunately. As a user of libraries and frameworks, you should know how to work around potential problems. The remainder of this chapter focuses on strategies to get your applications running on Java 9, even in a nonideal world. Hopefully, with time, this chapter becomes obsolete.

The Classpath Is Dead, Long Live the Classpath
Previous chapters introduced the module path. In many ways, you can view the module path as the successor of the classpath. Does this mean the classpath is gone in Java 9? Or that it’s going away, at all? Absolutely not! History will tell whether the classpath is ever removed from Java. Meanwhile, the classpath is still available in Java 9, and works largely the same as in previous releases. The classpath can even be combined with the new module path, as you will see in the next chapter.

When we ignore the module path, and use the classpath to build and run applications, we’re simply not using the new module features in our application. This requires minimal (if any) changes to existing code. Roughly speaking, when your application and its dependencies use only officially sanctioned APIs from the JDK, it should compile and run without issues on JDK 9.

If changes are necessary, they arise from the fact that the JDK itself has been modularized. Whether or not your application uses modules, the JDK it runs on always consists of modules as of Java 9. Although the module system is mostly ignored from an application perspective in this scenario, the changes to the JDK structure are still there. In many cases, the modular JDK doesn’t pose any problems for classpath-based applications, but there are definitely some caveats. Those caveats are in most cases related to libraries. The remainder of the chapter covers the possible problems, and more important, their workarounds.

Libraries, Strong Encapsulation, and the JDK 9 Classpath
One of the problems you can run into when migrating a classpath-based application to Java 9 is caused by the strong encapsulation of code in platform modules. Many libraries use classes from the platform that are now encapsulated with Java 9. Or, they use deep reflection to pry their way into nonpublic parts of platform classes.

Deep reflection is using the reflection API to get access to nonpublic elements of a class. In “Deep Reflection”, you learned that exporting a package from a module does not make its nonpublic elements accessible for reflection. Unfortunately, many libraries call setAccessible on private elements found through reflection.

You have seen that when using modules, JDK 9 by default disallows access to encapsulated packages and deep reflection on code in other modules, which includes platform modules. There is a good reason for this: abuse of platform internals has been the source of many security issues, and allowing it hampers evolution of APIs. However, in this chapter, we’re still dealing with classpath-based applications on top of a modular JDK. On the classpath, strong encapsulation of platform internals is not enforced as strictly, although it still plays a role.

Using deep reflection on JDK types is an obscure use case. Why would you want to make private parts of JDK classes accessible? It turns out some commonly used libraries do this. An example of this is the javassist runtime code-generation library, which is used by many other frameworks.

To ease migration of classpath-based applications to Java 9, the JVM by default shows a warning when deep reflection is applied on classes in platform modules. Or, when reflection is used to access types in nonexported packages. For example, when running code that uses the javassist library, we see the following warning:

WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by javassist.util.proxy.SecurityActions
  (...javassist-3.20.0-GA.jar) to method
  java.lang.ClassLoader.defineClass(...)
WARNING: Please consider reporting this to the maintainers of
  javassist.util.proxy.SecurityActions
WARNING: Use --illegal-access=warn to enable warnings of further illegal
  reflective access operations
WARNING: All illegal access operations will be denied in a future release
Let that sink in for a bit. Code that ran without any issues on JDK 8 and earlier now prints a prominent warning to the console—even in production. It shows how seriously the breach of strong encapsulation is taken.

Besides this warning, the application will still run as usual. As indicated by the warning message, the behavior will change in a next version of Java. In the future, the JDK will enforce strong encapsulation of platform modules even for code on the classpath. The same application will not run on default settings in a future Java release. Therefore, it is important to investigate the warnings and to fix the underlying problems. When the warnings are caused by libraries, that usually means reporting the issue to the maintainers.

By default, only a single warning is generated on the first illegal access attempt. Following attempts will not generate extra errors or warnings. If we want to further investigate the cause of the problem, we can use different settings for the --illegal-access command-line flag to tweak the behavior:

--illegal-access=permit
The default behavior. Illegal access to encapsulated types is allowed. Generates a warning on the first illegal access attempt through reflection.

--illegal-access=warn
Like permit, but generates an error on every illegal access attempt.

--illegal-access=debug
Also shows stack traces for illegal access attempts.

--illegal-access=deny
Does not allow illegal access attempts. This will be the default in the future.

Notice that none of the settings allow you to suppress the printed warnings. This is by design. In this chapter, you’ll learn how to address the underlying issues, in order to resolve the illegal access warnings. Because --illegal-access=deny will be the future default, your goal is to run your application with this setting.

If we run code that uses javassist with --illegal-access=deny, the application fails to run and we see the following error:

java.lang.reflect.InaccessibleObjectException: Unable to make protected final
    java.lang.Class java.lang.ClassLoader.defineClass(java.lang.String,byte[],
                                        int,int,java.security.ProtectionDomain)
  throws java.lang.ClassFormatError accessible: module java.base does not
  "opens java.lang" to unnamed module @0x7b3300e5
This error explains that javassist tries to make the defineClass method on java.lang.Class public. We can use the --add-opens flag to grant the classpath deep reflection access to a specific package in a module. Open modules and open packages are discussed in detail in “Deep Reflection”. As a refresher, a package needs to be open to allow deep reflection. This is even true when the package is exported as is the case here with java.lang. A package is usually opened in a module descriptor, similar to the way packages are exported. We can do the same from the command line for modules that we don’t control (for example, platform modules):

java --add-opens java.base/java.lang=ALL-UNNAMED
In this example, java.base/java.lang is the module/package we grant access to. The last argument is the module that gets the access. Because the code is still on the classpath, we use ALL-UNNAMED, which represents the classpath. The package is now open, so the deep reflection is no longer illegal. This will remove the warning (or error, when running with --illegal-access=deny). Similarly, when code on the classpath tries to access a type in a nonexported package, you can use --add-exports to force the package to be exported. We’ll see an example of this scenario in the next section. Remember that this is still just a workaround. Ask the maintainers of a library that causes illegal access problems for an updated version of the library with a proper fix.

WARNING
Illegal access is allowed by the default setting --illegal-access=permit only on packages that already existed before JDK 9, but are now encapsulated. Any new encapsulated packages in JDK 9 are not exempt from strong encapsulation, even when code is on the classpath.

SECURITY IMPACT
How does the existence of --add-opens and --add-exports impact security? One of the reasons to not allow deep reflection by default for platform modules in a future Java version is to prevent malicious code from reaching dangerous JDK internals. Doesn’t a flag to simply disable these checks void this important security benefit? One the one hand, yes, it selectively opens up a bigger attack surface when you choose to do so.

But consider this: there’s no way to gain the privileges afforded by --add-opens or --add-exports at run-time by merely executing Java code. An attacker needs to have access to the startup scripts (the command line) of an application to add these flags. When that level of access is established, the breach already allows the attacker to make arbitrary modifications reaching much further than just adding JVM options.

Compilation and Encapsulated APIs
The JDK contains many private, internal APIs. They are not supposed to be used by anyone other than the JDK itself. This has been clearly documented since the early beginnings. Examples are the sun.* and jdk.internal.* packages. As an application developer, you likely are not using these types directly. Most of these internal classes serve obscure corner cases, which typical applications don’t need. For this book, we even found it difficult to come up with a good example from the application development perspective.

Of course, some applications and (especially older) libraries do still use those internal classes. JDK internals were not strongly encapsulated previously, because there was no mechanism to do so. Pre-Java 9 compilers do emit warnings when using internal classes, but those are easily overlooked or ignored. We have seen that for the time being, code compiled with older versions of Java that uses encapsulated JDK types will still run on Java 9, because of the --illegal-access=permit default setting.

The same code will not compile on Java 9, however! Let’s say we have code (see Example 7-1) compiled with the JDK 8 compiler that uses types from the sun.security.x509 package.

Example 7-1. EncapsulatedTypes.java (➥ chapter7/encapsulation)
package encapsulated;

import sun.security.x509.X500Name;

public class EncapsulatedTypes {
    public static void main(String... args) throws Exception {
        System.out.println(new X500Name("test.com", "test",
                     "test", "US"));

    }
}
Compiling this code with JDK 9 results in the following compiler error:

./src/encapsulated/EncapsulatedTypes.java:3: error: package sun.security.x509
is not visible
import sun.security.x509.X500Name;
                   ^
  (package sun.security.x509 is declared in module java.base, which does not
   export it to the unnamed module)
By default, this code will still run successfully on Java 9, although the code is using an encapsulated package. You might wonder why there’s a difference between javac and java when it comes to accessing encapsulated types. What’s the point of being able to run code that accesses encapsulated types when you can’t compile the same code?

The reason that such code is still able run is to provide backward compatibility for existing libraries. The reason compiling with those same encapsulated types is prohibited is to prevent future compatibility nightmares. For code that you control, you should take immediate action when it comes to encapsulated types and replace them with nonencapsulated alternatives. When using a library (compiled with an older Java version) that’s using encapsulated types or deep reflection on JDK internals, you’re in a more difficult spot. You can’t fix the problem yourself, which would block you in your attempt to move to Java 9. Because of the lenient runtime, the library can still be used for the time being.

Allowing the usage of encapsulated JDK types at run-time is only a temporary situation. In a future Java release, this will be disabled. We can already prepare for this today by setting the --illegal-access=deny flag that we have seen in the previous section. Running the same code with java --illegal-access=deny generates an error:

Exception in thread "main" java.lang.IllegalAccessError:
class encapsulated.EncapsulatedTypes (in unnamed module @0x2e5c649) cannot
access class sun.security.x509.X500Name (in module java.base) because module
java.base does not export sun.security.x509 to unnamed module @0x2e5c649
        at encapsulated.EncapsulatedTypes.main(EncapsulatedTypes.java:7)
TIP
Notice that no warnings are shown for this scenario if we configure --illegal-access with anything other than deny. Only reflective illegal access triggers the warnings we have seen, not static references to encapsulated types as in this case. This restriction is a pragmatic one: changing the VM to also generate warnings for static references to encapsulated types would be too invasive.

The right course of action is to report the issue to the maintainers of the library. But what if this is our own code, and we need to recompile with JDK 9 but can’t make code changes right away? Changing code is always risky, so we have to find the right moment to do so.

We can use command-line flags to break encapsulation at compile-time as well. In the previous section, you saw how to use --add-opens to open a package from the command line. Both java and javac also support --add-exports. As the name suggests, we can use this to export an otherwise encapsulated package from a module. The syntax is --add-exports <module>/<package>=<targetmodule>. Because our code is still running on the classpath, we can use ALL-UNNAMED as the target module. Note that exporting an encapsulated package still does not allow deep reflection on its types. The package needs to be open for that. In this case, exporting the package is sufficient. In Example 7-1, we’re referencing the encapsulated type directly, without any reflection involved. For our (admittedly contrived) sun.security.​x509.X500Name example, we can compile and run with the following commands:

javac --add-exports java.base/sun.security.x509=ALL-UNNAMED \
 encapsulated/EncapsulatedTypes.java

java --add-exports java.base/sun.security.x509=ALL-UNNAMED  \
  encapsulated.EncapsulatedTypes
The --add-exports and --add-opens flags can be used for any module and package, not only for JDK internals. During compilation, warnings are still emitted for the use of internal APIs. Ideally, the --add-exports flag is a temporary migration step. Use it until you adapt your code to the public APIs, or (if a library is in violation) until there is new release of the third-party library using the replacement API.

TOO MANY COMMAND-LINE FLAGS!
Some operating systems limit the length of the command line that can be executed. When you need to add many flags during migration, you can hit these limits. You can use a file to provide all the command-line arguments to java/javac instead:

$ java @arguments.txt
The argument files must contain all necessary command-line flags. Each line in the file contains a single option. For instance, arguments.txt could contain the following:

-cp application.jar:javassist.jar
--add-opens java.base/java.lang=ALL-UNNAMED
--add-exports java.base/sun.security.x509=ALL-UNNAMED
-jar application.jar
Even if you’re not running into command-line limits, argument files can be clearer than a very long line somewhere in a script.

Removed Types
Code also could use internal types, which are now removed entirely. This is not directly related to the module system, but is still worth mentioning. One of the removed internal classes in Java 9 is sun.misc.BASE64Encoder, which was popular before Java 8 introduced the java.util.Base64 class. Example 7-2 shows code using BASE64Decoder.

Example 7-2. RemovedTypes.java (➥ chapter7/removedtypes)
package removed;

import sun.misc.BASE64Decoder;

// Compile with Java 8, run on Java 9: NoClassDefFoundError.
public class RemovedTypes {
    public static void main(String... args) throws Exception {
        new BASE64Decoder();
    }
}
This code will no longer compile or run on Java 9. When we try to compile, we see the following error:

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
If we compile the code with an older Java version, but try to run it with Java 9, it also fails:

Exception in thread "main" java.lang.NoClassDefFoundError: sun/misc/BASE64Decoder
  at removed.RemovedTypes.main(RemovedTypes.java:8)
Caused by: java.lang.ClassNotFoundException: sun.misc.BASE64Decoder
  ...
For an encapsulated type, we can work around the problem by forcing access to it with command-line flags. We can’t do this for this BASE64Decoder example, because the class doesn’t exist anymore. It’s important to understand this difference.

USING JDEPS TO FIND REMOVED OR ENCAPSULATED TYPES AND THEIR ALTERNATIVES
jdeps is a tool shipped with the JDK. One of the things jdeps can do is find usages of removed or encapsulated JDK types, and suggest replacements. jdeps always works on class files, not on source code. If we compile Example 7-2 with Java 8, we can run jdeps on the resulting class:

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
Similarly, encapsulated types such as X500Name in Example 7-1 are reported by jdeps with suggested replacements. More details on how to work with jdeps are discussed in “Using jdeps”.

Since Java 8, the JDK includes java.util.Base64, which is a much better alternative to use. The solution in this case is simple: we must migrate to the public API in order to run on JDK 9. In general, moving to Java 9 will expose a lot of technical debt in the areas discussed in this chapter.

Technical Debts by Oliver Widder link:http://geek-and-poke.com/geekandpoke/2013/11/20/technical-debts, CC-BY link:https://creativecommons.org/licenses/by/3.0/deed.en_US
Using JAXB and Other Java EE APIs
Certain Java EE technologies, such as JAXB, shipped with the JDK alongside Java SE APIs in the past. These technologies are still present in Java 9, but require special attention. They are shipped in the following list of modules:

java.activation

java.corba

java.transaction

java.xml.bind

java.xml.ws

java.xml.ws.annotation

In Java 9, these modules are deprecated for removal. The @Deprecated annotation has a new argument forRemoval in Java 9. When set to true, this means the API element will be removed in a future release. For API elements that are part of the JDK, this means removal may happen in a next major release. More details about deprecation can be found in JEP 277.

There is good reason for removing Java EE technologies from the JDK. The overlap between Java SE and Java EE in the JDK has always been confusing. Java EE application servers usually provide custom implementations of the APIs. Slightly simplified, this is done by putting the alternative implementation on the classpath, overriding the default JDK version. In Java 9, this becomes a problem. The module system does not allow the same package to be provided by multiple modules. If a duplicate package is found on the classpath (hence in the unnamed module), it is ignored. In any case, a situation where both Java SE and an application server provide java.xml.bind would not result in the expected behavior.

This is a serious practical problem, which would break many existing application servers and related tools. To avoid this problem, these modules are not resolved by default in classpath-based scenarios. Let’s take a look at the module graph of the platform in Figure 7-1.

Subset of the JDK module graph showing modules only reachable through `java.se.ee`, not `java.se`.
Figure 7-1. Subset of the JDK module graph showing modules reachable only through java.se.ee, not java.se
At the very top are the java.se and java.se.ee modules. Both are aggregator modules, modules that don’t contain code but group a set of more fine-grained modules. Aggregator modules are discussed in detail in “Aggregator Modules”. Most platform modules reside under java.se and are not shown here (but you can see the whole graph in Figure 2-1). The java.se.ee module aggregates the modules we are discussing, which are not part of the java.se aggregator module. This includes the java.xml.bind module, containing JAXB types.

By default, both javac and java use java.se as the root when compiling and running classes in the unnamed module. Code can access any package exported by the transitive dependencies of java.se. Modules under java.se.ee but not under java.se are therefore not resolved, so they are not read by the unnamed module. Even though package javax.xml.bind is exported from module java.xml.bind, it doesn’t matter because it is not resolved during compilation and run-time.

If modules under java.se.ee are necessary, we need to add them explicitly to the set of resolved platform modules. We can do so by adding them as root modules with the --add-modules flag of both javac and java.

Let’s try this with Example 7-3, based on JAXB. This example serializes a Book to XML.

Example 7-3. JaxbExample.java (➥ chapter7/jaxb)
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
On Java 8, this example compiles and runs without problems. On Java 9, we get several errors at compile-time:

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
When you compile with Java 8 and run the code with Java 9, an exception reporting the same problem is generated at run-time. We already know how to fix this issue: add --add-modules java.xml.bind to both the javac and java invocation.

WARNING
Instead of adding the platform module that contains JAXB, you can add a JAR that provides JAXB to the classpath. Several popular (open source) libraries provide JAXB implementations. Since the Java EE modules in the JDK are marked for removal, this is a more future-proof solution.

Note that we wouldn’t encounter this problem with the module path. If our code lives in a module, it must explicitly define a requirement on any modules other than java.base. That includes a dependency on java.xml.bind for the example code. Based on this, the module system resolves these modules without the need for a command-line flag.

Summing up, beware when using Java EE code from the JDK. When you’re getting errors that the packages are not visible, add the relevant modules by using --add-modules. Be aware that they will be removed in a next major Java release, however. Adding your own versions of these technologies to the classpath instead avoids future problems.

The jdk.unsupported Module
Some internal classes from the JDK have proven to be harder to encapsulate. Chances are that you have never used sun.misc.Unsafe and the like. These have always been unsupported classes, meant to be used only in the JDK internally.

Some of these classes are widely used by libraries for performance reasons. Although it’s easy to argue that this should never be done, in some cases it’s the only option. A well-known example is the sun.misc.Unsafe class, which can perform low-level operations bypassing Java’s memory model and other safety nets. The same functionality cannot be implemented by libraries outside the JDK.

If such classes would simply be encapsulated, libraries depending on them would no longer work with JDK 9, at least, not without warnings. Theoretically, this is not a backward-compatibility issue. Those libraries abuse nonsupported implementation classes, after all. For some of these highly used internal APIs, the real-world implications would be too severe to ignore, however—especially because there are no supported alternatives to the functionality they provide.

With that in mind, a compromise was reached. The JDK team researched which JDK platform internals are used by libraries the most, and which of those can be implemented only inside the JDK. Those classes are not encapsulated in Java 9.

Here’s the resulting list of specific classes and methods that are kept accessible:

sun.misc.{Signal,SignalHandler}

sun.misc.Unsafe

sun.reflect.Reflection::getCallerClass(int)

sun.reflect.ReflectionFactory::newConstructorForSerialization

Remember, if these names don’t mean anything to you, that’s a good thing. Popular libraries such as Netty, Mockito, and Akka use these classes, though. Not breaking these libraries is a good thing as well.

Because these methods and classes were not primarily designed to be used outside the JDK, they are moved to a platform module called jdk.unsupported. This indicates that it is expected the classes in this module will be replaced by other APIs in a future Java version. The jdk.unsupported module exports and/or opens the internal packages containing the classes discussed. Many existing uses involve deep reflection. Using these classes through reflection does not lead to warnings at run-time, unlike the scenarios discussed in “Libraries, Strong Encapsulation, and the JDK 9 Classpath”. That’s because jdk.unsupported opens the necessary packages in its module descriptor, so there is no illegal access from that point of view.

WARNING
Although these types can be used without breaking encapsulation, they are still unsupported; their use is still discouraged. The plan is to provide supported alternatives in the future. For example, some of the functionality in Unsafe is superseded by variable handles as proposed in JEP 193. Until then, the status quo is maintained.

When code still lives on the classpath, nothing changes. Libraries can use these classes from the classpath as before, running without any warnings or errors. The compiler generates warnings when compiling against classes from jdk.unsupported, rather than errors as with encapsulated types:

warning: Unsafe is internal proprietary API and may be
         removed in a future release
If you want to use these types from a module, you must require jdk.unsupported. Having such a requires statement in your module descriptor serves as a warning sign. In a future Java release, changes may be necessary to adapt to publicly supported APIs instead of the unsupported APIs.

Other Changes
Many other changes in JDK 9 can potentially break code. These changes affect, for example, tool authors, and applications that use the JDK extension mechanisms. Some of the changes include the following:

JDK layout
Because of the platform modularization, the big rt.jar containing all platform classes doesn’t exist anymore. The layout of the JDK itself has changed considerably as well, as is documented in JEP 220. Tools or code relying on the JDK layout must adapt to this new reality.

Version string
Gone are the days that all Java platform versions start with the 1.x prefix. Java 9 is shipped with version 9.0.0. The syntax and semantics of the version string have changed considerably. If an application does any kind of parsing on the Java version, read JEP 223 for all the details.

Extension mechanisms
Features such as the Endorsed Standard Override Mechanism and the extension mechanism through the java.ext.dirs property are removed. They are replaced by upgradeable modules. More information can be found in JEP 220.

These are all highly specialized features of the JDK. If your application does rely on them, it will not work with JDK 9. Because these changes are not really related to the Java module system, we won’t go into further detail. The linked JDK Enhancement Proposals (JEPs) contain guidance on how to proceed in these cases.

Congratulations! You now know how to run your existing application on JDK 9. Even though several things could go wrong, in many cases things will just work. Remember to run your application with --illegal-access=deny as well, to be prepared for the future. After fixing all issues when running existing applications from the classpath, it’s time to look at how to make them more modular.