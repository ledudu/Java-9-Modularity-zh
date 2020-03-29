# 第 2 章 模块化和模块化 JDK
> Chapter 2. Modules and the Modular JDK

Java is over 20 years old. As a language, it’s still popular, proving that Java has held up well. The platform’s long evolution becomes especially apparent when looking at the standard libraries. Prior to the Java module system, the runtime library of the JDK consisted of a hefty rt.jar (as shown previously in Figure 1-1), weighing in at more than 60 megabytes. It contains most of the runtime classes for Java: the ultimate monolith of the Java platform. In order to regain a flexible and future-proof platform, the JDK team set out to modularize the JDK—an ambitious goal, given the size and structure of the JDK. Over the course of the past 20 years, many APIs have been added. Virtually none have been removed.

Take CORBA—once considered the future of enterprise computing, and now a mostly forgotten technology. (To those who are still using it: we feel for you.) The classes supporting CORBA in the JDK are still present in rt.jar to this day. Each and every distribution of Java, regardless of the applications it runs, includes those CORBA classes. No matter whether you use CORBA or not, the classes are there. Carrying this legacy in the JDK results in unnecessary use of disk space, memory, and CPU time. In the context of using resource-constrained devices, or creating small containers for the cloud, these resources are in short supply. Not to mention the cognitive overhead of obsolete classes showing up in IDE autocompletions and documentation during development.

Simply removing these technologies from the JDK isn’t a viable option, though. Backward compatibility is one of the most important guiding principles for Java. Removal of APIs would break a long streak of backward compatibility. Although it may affect only a small percentage of users, plenty of people are still using technologies like CORBA. In a modular JDK, people who aren’t using CORBA can choose to ignore the module containing CORBA.

Alternatively, an aggressive deprecation schedule for truly obsolete technologies could work. Still, it would take several major releases before the JDK sheds the excess weight. Also, deciding what technology is truly obsolete would be at the discretion of the JDK team, which is a difficult position to be in.

NOTE
In the specific case of CORBA, the module is marked as deprecated, meaning it will likely be removed in a subsequent major Java release.

But the desire to break up the monolithic JDK is not just about removing obsolete technology. A vast array of technologies are useful to certain types of applications, while useless for others. JavaFX is the latest user-interface technology in Java, after AWT and Swing. This is certainly not something to be removed, but clearly it’s not required in every application either. Web applications, for example, use none of the GUI toolkits in Java. Yet there is no way to deploy and run them without all three GUI toolkits being carried along.

Aside from convenience and waste, consider the security perspective. Java has experienced a considerable number of security exploits in the past. Many of these exploits share a common trait: somehow attackers gain access to sensitive classes inside the JDK to bypass the JVM’s security sandbox. Strongly encapsulating dangerous internal classes within the JDK is a big improvement from a security standpoint. Also, decreasing the number of available classes in the runtime decreases the attack surface. Having tons of unused classes around in your application runtime only for them to be exploited later is an unfortunate trade-off. With a modular JDK, only those modules your application needs are resolved.

By now, it’s abundantly clear that a modular approach for the JDK itself is sorely needed.

The Modular JDK
The first step toward a more modular JDK was taken in Java 8 with the introduction of compact profiles. A profile defines a subset of packages from the standard library available to applications targeting that profile. Three profiles are defined, imaginatively called compact1, compact2, and compact3. Each profile is a superset of the previous, adding more packages that can be used. The Java compiler and runtime were updated with knowledge of these predefined profiles. Java SE Embedded 8 (Linux only) offers low-footprint runtimes matching the compact profiles.

If your application fits one of the profiles described in Table 2-1, this is a good way to target a smaller runtime. But if you require even so much as a single class outside of the predefined profiles, you’re out of luck. In that sense, compact profiles are far from flexible. They also don’t address strong encapsulation. As an intermediate solution, compact profiles fulfilled their purpose. Ultimately, a more flexible approach is needed.

Table 2-1. Profiles defined for Java 8
Profile	Description
compact1

Smallest profile with Java core classes and logging and scripting APIs

compact2

Extends compact1 with XML, JDBC, and RMI APIs

compact3

Extends compact2 with security and management APIs

You already saw a glimpse of how JDK 9 is split into modules in Figure 1-2. The JDK now consists of about 90 platform modules, instead of a monolithic library. A platform module is part of the JDK, unlike application modules, which you can create yourself. There is no technical distinction between platform modules and application modules. Every platform module constitutes a well-defined piece of functionality of the JDK, ranging from logging to XML support. All modules explicitly define their dependencies on other modules.

A subset of these platform modules and their dependencies is shown in Figure 2-1. Every edge indicates a unidirectional dependency between modules (we’ll get to the difference between solid and dashed edges later). For example, java.xml depends on java.base. As stated in “Java 9 Modules”, every module implicitly depends on java.base. In Figure 2-1 this implicit dependency is shown only when java.base is the sole dependency for a given module, as is the case with, for example, java.xml.

Even though the dependency graph may look a little overwhelming, we can glean a lot of information from it. Just by looking at the graph, you can get a decent overview of what the Java standard libraries offer and how the functionalities are related. For example, java.logging has many incoming dependencies, meaning it is used by many other platform modules. That makes sense for a central functionality such as logging. Module java.xml.bind (containing the JAXB API for XML binding) has many outgoing dependencies, including an unexpected one on java.desktop. The fact that we can notice this oddity by looking at a generated dependency graph and talk about it is a huge improvement. Because of the modularization of the JDK, there are clean module boundaries and explicit dependencies to reason about. Having an overview of a large codebase like the JDK, based on explicit module information, is invaluable.

Subset of platform modules in the JDK.
Figure 2-1. Subset of platform modules in the JDK
Another thing to note is how all arrows in the dependency graph point downward. There are no cycles in this graph. That’s not by accident: the Java module system does not allow compile-time circular dependencies between modules.

WARNING
Circular dependencies are generally an indication of bad design. In “Breaking Cycles”, we discuss how to identify and resolve circular dependencies in your codebase.

All modules in Figure 2-1, except jdk.httpserver and jdk.unsupported, are part of the Java SE specification. They share the java.* prefix for module names. Every certified Java implementation must contain these modules. Modules such as jdk.httpserver contain implementations of tools and APIs. Where such implementations live is not mandated by the Java SE specification, but of course such modules are essential to a fully functioning Java platform. There are many more modules in the JDK, most of them in the jdk.* namespace.

TIP
You can get the full list of platform modules by running
java --list-modules.

Two important modules can be found at the top of Figure 2-1: java.se and java.se.ee. These are so-called aggregator modules, and they serve to logically group several other modules. We’ll see how aggregator modules work later in this chapter.

Decomposing the JDK into modules has been a tremendous amount of work. Splitting up an entangled, organically grown codebase containing tens of thousands of classes into well-defined modules with clear boundaries, while retaining backward compatibility, takes time. This is one of the reasons it took a long time to get a module system into Java. With over 20 years of legacy accumulated, many dubious dependencies had to be untangled. Going forward, this effort will definitely pay off in terms of development speed and increased flexibility for the JDK.

INCUBATOR MODULES
Another example of the improved evolvability afforded by modules is the concept of incubator modules, described in JEP 11 (JEP stands for Java Enhancement Proposal). Incubator modules are a means to ship experimental APIs with the JDK. With Java 9, for example, a new HttpClient API is shipped in the jdk.incubator.httpclient module (all incubator modules have the jdk.incubator prefix). You can depend on such incubator modules if you want to, with the explicit expectation that their APIs may still change. This allows the APIs to mature and harden in a real-world environment, so they can be shipped as a fully supported module in a later JDK release—or be removed, if the API isn’t successful in practice.

Module Descriptors
Now that we have a high-level overview of the JDK module structure, let’s explore how modules work. What is a module, and how is it defined? A module has a name, it groups related code and possibly other resources, and is described by a module descriptor. The module descriptor lives in a file called module-info.java. Example 2-1 shows the module descriptor for the java.prefs platform module.

Example 2-1. module-info.java
module java.prefs {
    requires java.xml; 1

    exports java.util.prefs; 2
}
1
The requires keyword indicates a dependency, in this case on module java.xml.

2
A single package from the java.prefs module is exported to other modules.

Modules live in a global namespace; therefore, module names must be unique. As with package names, you can use conventions such as reverse DNS notation (e.g., com.mycompany.project.somemodule) to ensure uniqueness for your own modules. A module descriptor always starts with the module keyword, followed by the name of the module. Then, the body of module-info.java describes other characteristics of the module, if any.

Let’s move on to the body of the module descriptor for java.prefs. Code in java.prefs uses code from java.xml to load preferences from XML files. This dependency must be expressed in the module descriptor. Without this dependency declaration, the java.prefs module would not compile (or run), as enforced by the module system. A dependency is declared with the requires keyword followed by a module name, in this case java.xml. The implicit dependency on java.base may be added to a module descriptor. Doing so adds no value, similar to how you can (but generally don’t) add "import java.lang.String" to a class using strings.

A module descriptor can also contain exports statements. Strong encapsulation is the default for modules. Only when a package is explicitly exported, like java.util.prefs in this example, can it be accessed from other modules. Packages inside a module that are not exported are inaccessible from other modules by default. Other modules cannot refer to types in encapsulated packages, even if they have a dependency on the module. When you look at Figure 2-1, you see that java.desktop has a dependency on java.prefs. That means java.desktop is able to access only types in package java.util.prefs of the java.prefs module.

Readability
An important new concept when reasoning about dependencies between modules is readability. Reading another module means you can access types from its exported packages. You set up readability relations between modules through requires clauses in the module descriptor. By definition, every module reads itself. A module that requires another module reads the other module.

Let’s explore the effects of readability by revisiting the java.prefs module. In this JDK module in Example 2-2, the following class imports and uses classes from the java.xml module.

Example 2-2. Small excerpt from the class java.util.prefs.XmlSupport
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
Here, org.w3c.dom.Document (among other classes) is imported. It comes from the java.xml module. Because the java.prefs module descriptor contains requires java.xml, as you saw in Example 2-1, this code compiles without issue. Had the author of the java.prefs module left out the requires clause, the Java compiler would report an error. Using code from java.xml in module java.prefs is a deliberate and explicitly recorded choice.

Accessibility
Readability relations are about which modules read other modules. However, if you read a module, this doesn’t mean you can access everything from its exported packages. Normal Java accessibility rules are still in play after readability has been established.

Java has had accessibility rules built into the language since the beginning. Table 2-2 provides a refresher on the existing access modifiers and their impact.

Table 2-2. Access modifiers and their associated scopes
Access modifier	Class	Package	Subclass	Unrestricted
public

✓

✓

✓

✓

protected

✓

✓

✓

- (default)

✓

✓

private

✓

Accessibility is enforced at compile- and run-time. Combining accessibility and readability provides the strong encapsulation guarantees we so desire in a module system. The question of whether you can access a type from module M2 in module M1 becomes twofold:

Does M1 read M2?

If yes, is the type accessible in the package exported by M2?

Only public types in exported packages are accessible in other modules. If a type is in an exported package but not public, traditional accessibility rules block its use. If it is public but not exported, the module system’s readability rules prevent its use. Violations at compile-time result in a compiler error, whereas violations at run-time result in IllegalAccessError.

IS PUBLIC STILL PUBLIC?
No types from a nonexported package can be used by other modules—even if types inside that package are public. This is a fundamental change to the accessibility rules of the Java language.

Until Java 9, things were quite straightforward. If you had a public class or interface, it could be used by every other class. As of Java 9, public means public only to all other packages inside that module. Only when the package containing the public type is exported can it be used by other modules. This is what strong encapsulation is all about. It forces developers to carefully design a package structure where types meant for external consumption are clearly separated from internal implementation concerns.

Before modules, the only way to strongly encapsulate implementation classes was to keep them all in a single package and mark them package-private. Since this leads to unwieldy packages, in practice classes were made public just for access across different packages. With modules, you can structure packages any way you like and export only those that really must be accessible to the consumers of the module. Exported packages form the API of a module, if you will.

Another elephant in the room with regards to accessibility rules is reflection. Before the module system, an interesting but dangerous method called setAccessible was available on all reflected objects. By calling setAccessible(true), any element (regardless of whether it is public or private) becomes accessible. This method is still available but now abides by the same rules as discussed previously. It is no longer possible to invoke setAccessible on an arbitrary element exported from another module and expect it to work as before. Even reflection cannot break strong encapsulation.

There are ways around the new accessibility rules imposed by the module system. Most of these workarounds should be viewed as migration aids and are discussed in Part II.

Implied Readability
Readability is not transitive by default. We can illustrate this by looking at the incoming and outgoing read edges of java.prefs, as shown in Figure 2-2.

Readability is not transitive
Figure 2-2. Readability is not transitive: java.desktop does not read java.xml through java.prefs
Here, java.desktop reads java.prefs (among other modules, left out for clarity). We’ve already established that this means java.desktop can access public types from the java.util.prefs package. However, java.desktop cannot access types from java.xml through its dependency on java.prefs. It just so happens that java.desktop does use types from java.xml as well. That’s why java.desktop has its own requires java.xml clause in its module descriptor. In Figure 2-1, this dependency is also visible.

Sometimes you do want read relations to be transitive—for example, when a type in an exported package of module M1 refers to a type from another module M2. In that case, modules requiring M1 and thereby referencing types from M2 cannot be used without reading M2 as well.

That sounds completely abstract, so an illustration is in order. A good example of this phenomenon can be found in the JDK’s java.sql module. It contains two interfaces (Driver, shown in Example 2-3, and SQLXML, shown in Example 2-4) defining method signatures whose result types come from other modules.

Example 2-3. Driver interface (partially shown), allowing a Logger from the java.logging module to be retrieved
package java.sql;

import java.util.logging.Logger;

public interface Driver {
  public Logger getParentLogger();
  // ..
}
Example 2-4. SQLXML interface (partially shown), with Source from module java.xml representing XML coming back from the database
package java.sql;

import javax.xml.transform.Source;

public interface SQLXML {
  <T extends Source> T getSource(Class<T> sourceClass);
  // ..
}
If you add a dependency on java.sql to your module descriptor, you can program to these interfaces, since they are in exported packages. But whenever you call get​ParentLogger or getSource, you get back values of a type not exported by java.sql. In the first case, you get a java.util.logging.Logger from java.logging, and in the second case, you get a javax.xml.transform.Source from java.xml. In order to do anything useful with these return values (assign to a local variable, call methods on them), you need to read those other modules as well.

Of course, you can manually add dependencies on java.logging or java.xml, respectively, to your own module descriptor. But that’s hardly satisfying, especially since the java.sql author already knew the interfaces are unusable without readability on those other modules. Implied readability allows module authors to express this transitive readability relation in module descriptors.

For java.sql, it looks like this:

module java.sql {
    requires transitive java.logging;
    requires transitive java.xml;

    exports java.sql;
    exports javax.sql;
    exports javax.transaction.xa;
}
The requires keyword is now followed by the transitive modifier, slightly changing the semantics. A normal requires allows a module to access types in exported packages from the required module only. requires transitive means the same and more. In addition, any module requiring java.sql will now automatically be requiring java.logging and java.xml. That means you get access to the exported packages of those modules as well by virtue of these implied readability relations. With requires transitive, module authors can set up additional readability relations for users of the module.

From the consumer side, this makes it easier to use java.sql. When you require java.sql, you get access to the exported packages java.sql, javax.sql, and javax.transaction.xa (which are all exported by java.sql directly), but also to all packages exported by modules java.logging and java.xml. It’s as if java.sql re-exports those packages for you, courtesy of the implied readability relations it sets up with requires transitive. To be clear, there’s no such thing as re-exporting packages from other modules, but thinking about it this way may help you understand the effects of implied readability.

For an application module app using java.sql, this module definition suffices:

module app {
  requires java.sql;
}
With this module descriptor, the implied readability edges in Figure 2-3 are in effect.

Implied readability (requires transitive) shown with double-lined edges.
Figure 2-3. The effect of implied readability (requires transitive) shown with bold edges
Implied readability on java.xml and java.logging (the bold edges in Figure 2-3) is granted to app because java.sql uses requires transitive (solid edges in Figure 2-3) for those modules. Because app does not export anything and uses only java.sql for its encapsulated implementation, a normal requires clause is enough (dashed edge in Figure 2-3). When you need another module for internal uses, a normal requires suffices. If, on the other hand, types from another module are used in exported types, requires transitive is in order. In “API Modules”, we’ll discuss how and when implied readability is important for your own modules in more detail.

Now’s a good time to take another look at Figure 2-1. All solid edges in that graph are requires transitive dependencies too. The dashed edges, on the other hand, are normal requires dependencies. A nontransitive dependency means the dependency is necessary to support the internal implementation of that module. A transitive dependency means the dependency is necessary to support the API of the module. These latter dependencies are more significant; hence they are depicted by solid lines in the diagrams in this book.

Looking at Figure 2-1 with these new insights, we can highlight another use case for implied readability: it can be used to aggregate several modules into a single new module. Take, for example, java.se. It’s a module that doesn’t contain any code and consists of just a module descriptor. In this module descriptor, a requires transitive clause is listed for each module that is part of the Java SE specification. When you require java.se in a module, you get access to all exported APIs of every module aggregated by java.se by virtue of implied readability:

module java.se {
    requires transitive java.desktop;
    requires transitive java.sql;
    requires transitive java.xml;
    requires transitive java.prefs;
    // .. many more
}
Implied readability itself is transitive as well. Take another aggregator module in the platform, java.se.ee. Figure 2-1 shows that java.se.ee aggregates even more modules than java.se. It does so by using requires transitive java.se and adding several modules containing parts of the Java Enterprise Edition (EE) specification. Here’s what the java.se.ee aggregator module descriptor looks like:

module java.se.ee {
    requires transitive java.se;
    requires transitive java.xml.ws;
    requires transitive java.xml.bind;
    // .. many more
}
The requires transitive on java.se ensures that if java.se.ee is required, implied readability is also established to all modules aggregated by java.se. Furthermore, java.se.ee sets up implied readability to several EE modules.

In the end, java.se and java.se.ee provide implied readability on a huge number of modules reachable through these transitive dependencies.

TIP
Requiring java.se.ee or java.se in application modules is rarely the right thing to do. It means you’re effectively replicating the pre-Java 9 behavior of having all of rt.jar accessible in your module. Dependencies should be defined as fine-grained as possible. It pays to be more precise in your module descriptor and require only modules you actually use.

In “Aggregator Modules”, we’ll explore how the aggregator module pattern helps in modular library design.

Qualified Exports
In some cases, you’ll want to expose a package only to certain other modules. You can do this by using qualified exports in the module descriptor. An example of a qualified export can be found in the java.xml module:

module java.xml {
  ...
  exports com.sun.xml.internal.stream.writers to java.xml.ws
  ...
}
Here we see a platform module sharing useful internals with another platform module. The exported package is accessible only by the modules specified after to. Multiple module names, separated by a comma, can be provided as targets for a qualified export. Any module not mentioned in this to clause cannot access types in this package, even when they read the module.

The fact that qualified exports exist doesn’t unequivocally mean you should use them. In general, avoid using qualified exports between modules in an application. Using them creates an intimate bond between the exporting module and its allowable consumers. From a modularity perspective, this is undesirable. One of the great things about modules is that you effectively decouple producers from consumers of APIs. Qualified exports break this property because now the names of consumer modules are part of the provider module’s descriptor.

For modularizing the JDK, however, this is a lesser concern. Qualified exports have been indispensable to modularizing the platform with all of its legacy. Many platform modules encapsulate part of their code, expose some internal APIs through qualified exports to select other platform modules, and use the normal export mechanism for public APIs used in applications. By using qualified exports, platform modules could be made more fine-grained without duplicating code.

Module Resolution and the Module Path
Having explicit dependencies between modules is not just useful to generate pretty diagrams. The Java compiler and runtime use module descriptors to resolve the right modules when compiling and running modules. Modules are resolved from the module path, as opposed to the classpath. Whereas the classpath is a flat list of types (even when using JAR files), the module path contains only modules. As you’ve learned, these modules carry explicit information on what packages they export, making the module path efficiently indexable. The Java runtime and compiler know exactly which module to resolve from the module path when looking for types from a given package. Previously, a scan through the whole classpath was the only way to locate an arbitrary type.

When you want to run an application packaged as a module, you need all of its dependencies as well. Module resolution is the process of computing a minimal required set of modules given a dependency graph and a root module chosen from that graph. Every module reachable from the root module ends up in the set of resolved modules. Mathematically speaking, this amounts to computing the transitive closure of the dependency graph. As intimidating as it may sound, the process is quite intuitive:

Start with a single root module and add it to the resolved set.

Add each required module (requires or requires transitive in module-info.java) to the resolved set.

Repeat step 2 for each new module added to the resolved set in step 2.

This process is guaranteed to terminate because we repeat the process only for newly discovered modules. Also, the dependency graph must be acyclic. If you want to resolve modules for multiple root modules, apply the algorithm to each root module, and then take the union of the resulting sets.

Let’s try this with an example. We have an application module app that will be the root module in the resolution process. It uses only java.sql from the modular JDK:

module app {
    requires java.sql;
}
Now we run through the steps of module resolution. We omit java.base when considering the dependencies of modules and assume it always is part of the resolved modules. You can follow along by looking at the edges in Figure 2-1:

Add app to the resolved set; observe that it requires java.sql.

Add java.sql to the resolved set; observe that it requires java.xml and java.logging.

Add java.xml to the resolved set; observe that it requires nothing else.

Add java.logging to the resolved set; observe that it requires nothing else.

No new modules have been added; resolution is complete.

The result of this resolution process is a set containing app, java.sql, java.xml, java.logging, and java.base. When running app, the modules are resolved in this way, and the module system gets the modules from the module path.

Additional checks are performed during this process. For example, two modules with the same name lead to an error at startup (rather than at run-time during inevitable classloading failures). Another check is for uniqueness of exported packages. Only one module on the module path may expose a given package. “Split Packages” discusses problems with multiple modules exporting the same packages.

VERSIONS
So far, we’ve discussed module resolution without mentioning versions. That may seem odd, since we’re used to specifying versions with dependencies in, for example, Maven POMs. It is a deliberate design decision to leave version selection outside the scope of the Java module system. Versions do not play a role during module resolution. In “Versioned Modules” we discuss this decision in greater depth.

The module resolution process and additional checks ensure that the application runs in a reliable environment and is less likely to fail at run-time. In Chapter 3, you’ll learn how to construct a module path when compiling and running your own modules.

Using the Modular JDK Without Modules
You’ve learned about many new concepts the module system introduces. At this point, you may be wondering how this all affects existing code, which obviously is not modularized yet. Do you really need to convert your code to modules to start using Java 9? Fortunately not. Java 9 can be used like previous versions of Java, without moving your code into modules. The module system is completely opt-in for application code, and the classpath is still alive and kicking.

Still, the JDK itself does consist of modules. How are these two worlds reconciled? Say you have the piece of code in Example 2-5.

Example 2-5. NotInModule.java
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
It’s just a class, not put into any module. The code clearly uses types from the java.logging module in the JDK. However, there is no module descriptor to express this dependency. Still, when you compile this code without a module descriptor, put it on the classpath, and run it, it will just work. How can this be? Code compiled and loaded outside a module ends up in the unnamed module. In contrast, all modules you’ve seen so far are explicit modules, defining their name in module-info.java. The unnamed module is special: it reads all other modules, including the java.logging module in this case.

Through the unnamed module, code that is not yet modularized continues to run on JDK 9. Using the unnamed module happens automatically when you put your code on the classpath. That also means you are still responsible for constructing a correct classpath yourself. Almost all guarantees and benefits of the module system we have discussed so far are voided when working through the unnamed module.

You need to be aware of two more things when using the classpath in Java 9. First, because the platform is modularized, it strongly encapsulates internal implementation classes. In Java 8 and earlier, you could use these unsupported internal APIs without repercussions. With Java 9, you cannot compile against encapsulated types in platform modules. To aid migration, code compiled on earlier versions of Java using these internal APIs continues to run on the JDK 9 classpath for now.

WARNING
When running (as opposed to compiling) an application on the JDK 9 classpath, a more lenient form of strong encapsulation is activated. All internal classes that were accessible on JDK 8 and earlier remain accessible at run-time on JDK 9. A warning is printed when these encapsulated types are accessed through reflection.

The second thing to be aware of when compiling code in the unnamed module is that java.se is taken as the root module during compilation. You can access types from any module reachable through java.se and it will work, as shown in Example 2-5. Conversely, this means modules under java.se.ee but not under java.se (such as java.corba and java.xml.ws) are not resolved and therefore not accessible. One of the most prominent examples of this policy is the JAXB API. The rationale behind both restrictions, and how to approach them, is discussed in more detail in Chapter 7.

In this chapter, you’ve seen how the JDK has been modularized. Even though modules play a central role in JDK 9, they are optional for applications. Care has been taken to ensure that applications running on the classpath before JDK 9 continue to work, but there are some caveats, as you’ve seen. In the next chapter, we’ll take the module concepts discussed so far and use them to build our own modules.