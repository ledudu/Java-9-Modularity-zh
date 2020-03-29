# 第 1 章 模块化概述
> Chapter 1. Modularity Matters

Have you ever scratched your head in bewilderment, asking yourself, “Why is this code here? How does it relate to the rest of this gigantic codebase? Where do I even begin?” Or did your eyes glaze over after scanning the multitude of Java Archives (JARs) bundled with your application code? We certainly have.

The art of structuring large codebases is an undervalued one. This is neither a new problem, nor is it specific to Java. However, Java is one of the mainstream languages in which very large applications are built all the time—often making heavy use of many libraries from the Java ecosystem. Under these circumstances, systems can outgrow our capacity for understanding and efficient development. A lack of structure is dearly paid for in the long run, experience shows.

Modularity is one of the techniques you can employ to manage and reduce this complexity. Java 9 introduces a new module system that makes modularization easier and more accessible. It builds on top of abstractions Java already has for modular development. In a sense, it promotes existing best practices on large-scale Java development to be part of the Java language.

The Java module system will have a profound impact on Java development. It represents a fundamental shift to modularity as a first-class citizen for the whole Java platform. Modularization is addressed from the ground up, with changes to the language, Java Virtual Machine (JVM), and standard libraries. While this represents a monumental effort, it’s not as flashy as, for example, the addition of streams and lambdas in Java 8. There’s another fundamental difference between a feature like lambdas and the Java module system. A module system is concerned with the large-scale structure of whole applications. Turning an inner class into a lambda is a fairly small and localized change within a single class. Modularizing an application affects design, compilation, packaging, deployment, and so on. Clearly, it’s much more than just another language feature.

With every new Java release, it’s tempting to dive right in and start using the new features. To make the most out of the module system, we should first take a step back and focus on what modularity is. And, more important, why we should care.

What Is Modularity?
So far, we’ve touched upon the goal of modularity (managing and reducing complexity), but not what modularity entails. At its heart, modularization is the act of decomposing a system into self-contained but interconnected modules. Modules are identifiable artifacts containing code, with metadata describing the module and its relation to other modules. Ideally, these artifacts are recognizable from compile-time all the way through run-time. An application then consists of multiple modules working together.

So, modules group related code, but there’s more to it than that. Modules must adhere to three core tenets:

Strong encapsulation
A module must be able to conceal part of its code from other modules. By doing so, a clear line is drawn between code that is publicly usable and code that is deemed an internal implementation detail. This prevents accidental or unwanted coupling between modules: you simply cannot use what has been encapsulated. Consequently, encapsulated code may change freely without affecting users of the module.

Well-defined interfaces
Encapsulation is fine, but if modules are to work together, not everything can be encapsulated. Code that is not encapsulated is, by definition, part of the public API of a module. Since other modules can use this public code, it must be managed with great care. A breaking change in nonencapsulated code can break other modules that depend on it. Therefore, modules should expose well-defined and stable interfaces to other modules.

Explicit dependencies
Modules often need other modules to fulfill their obligations. Such dependencies must be part of the module definition, in order for modules to be self-contained. Explicit dependencies give rise to a module graph: nodes represent modules, and edges represent dependencies between modules. Having a module graph is important for both understanding an application and running it with all necessary modules. It provides the basis for a reliable configuration of modules.

Flexibility, understandability, and reusability all come together with modules. Modules can be flexibly composed into different configurations, making use of the explicit dependencies to ensure that everything works together. Encapsulation ensures that you never have to know implementation details and that you will never accidentally rely on them. To use a module, knowing its public API is enough. Also, a module exposing well-defined interfaces while encapsulating its implementation details can readily be swapped with alternative implementations conforming to the same API.

Modular applications have many advantages. Experienced developers know all too well what happens when codebases are nonmodular. Endearing terms like spaghetti architecture, messy monolith, or big ball of mud do not even begin to cover the associated pain. Modularity is not a silver bullet, though. It is an architectural principle that can prevent these problems to a high degree when applied correctly.

That being said, the definition of modularity provided in this section is deliberately abstract. It might make you think of component-based development (all the rage in the previous century), service-oriented architecture, or the current microservices hype. Indeed, these paradigms try to solve similar problems at various levels of abstraction.

What would it take to realize modules in Java? It’s instructive to take a moment and think about how the core tenets of modularity are already present in Java as you know it (and where it is lacking).

Done? Then you’re ready to proceed to the next section.

Before Java 9
Java is used for development of all sorts and sizes. Applications comprising millions of lines of code are no exception. Evidently, Java has done something right when it comes to building large-scale systems—even before Java 9 arrived on the scene. Let’s examine the three core tenets of modularity again in the light of Java before the arrival of the Java 9 module system.

Encapsulation of types can be achieved by using a combination of packages and access modifiers (such as private, protected, or public). By making a class protected, for example, you can prevent other classes from accessing it unless they reside in the same package. That raises an interesting question: what if you want to access that class from another package in your component, but still want to prevent others from using it? There’s no good way to do this. You can, of course, make the class public. But, public means public to every other type in the system, meaning no encapsulation. You can hint that using such a class is not smart by putting it in an .impl or .internal package. But really, who looks at that? People use it anyway, just because they can. There’s no way to hide such an implementation package.

In the well-defined interfaces department, Java has been doing great since its inception. You guessed it, we’re talking about Java’s very own interface keyword. Exposing a public interface, while hiding the implementation class behind a factory or through dependency injection, is a tried-and-true method. As you will see throughout this book, interfaces play a central role in modular systems.

Explicit dependencies are where things start to fall apart. Yes, Java does have explicit import statements. Unfortunately, those imports are strictly a compile-time construct. Once you package your code into a JAR, there’s no telling which other JARs contain the types your JAR needs to run. In fact, this problem is so bad, many external tools evolved alongside the Java language to solve this problem. The following sidebar provides more details.

EXTERNAL TOOLING TO MANAGE DEPENDENCIES: MAVEN AND OSGI
Maven
One of the problems solved by the Maven build tool is compile-time dependency management. Dependencies between JARs are defined in an external Project Object Model (POM) file. Maven’s great success is not the build tool per se, but the fact that it spawned a canonical repository called Maven Central. Virtually all Java libraries are published along with their POMs to Maven Central. Various other build tools such as Gradle or Ant (with Ivy) use the same repository and metadata. They all automatically resolve (transitive) dependencies for you at compile-time.

OSGi
What Maven does at compile-time, OSGi does at run-time. OSGi requires imported packages to be listed as metadata in JARs, which are then called bundles. You must also explicitly define which packages are exported, that is, visible to other bundles. At application start, all bundles are checked: can every importing bundle be wired to an exporting bundle? A clever setup of custom classloaders ensures that at run-time no types are loaded in a bundle besides what is allowed by the metadata. As with Maven, this requires the whole world to provide correct OSGi metadata in their JARs. However, where Maven has unequivocally succeeded with Maven Central and POMs, the proliferation of OSGi-capable JARs is less impressive.

Both Maven and OSGi are built on top of the JVM and Java language, which they do not control. Java 9 addresses some of the same problems in the core of the JVM and the language. The module system is not intended to completely replace those tools. Both Maven and OSGi (and similar tools) still have their place, only now they can build on a fully modular Java platform.

As it stands, Java offers solid constructs for creating large-scale modular applications. It’s also clear there is definitely room for improvement.

JARs as Modules?
JAR files seem to be the closest we can get to modules pre-Java 9. They have a name, group related code, and can offer well-defined public interfaces. Let’s look at an example of a typical Java application running on top of the JVM to explore the notion of JARs as modules; see Figure 1-1.

A typical application running on the JVM
Figure 1-1. MyApplication is a typical Java application, packaged as a JAR and using other libraries
There’s an application JAR called MyApplication.jar containing custom application code. Two libraries are used by the application: Google Guava and Hibernate Validator. There are three additional JARs as well. Those are transitive dependencies of Hibernate Validator, possibly resolved for us by a build tool like Maven. MyApplication runs on a pre-Java 9 runtime which itself exposes Java platform classes through several bundled JARs. The pre-Java 9 runtime may be a Java Runtime Environment (JRE) or a Java Development Kit (JDK), but in both cases it includes rt.jar (runtime library), which contains the classes of the Java standard library.

When you look closely at Figure 1-1, you can see that some of the JARs list classes in italic. These classes are supposed to be internal classes of the libraries. For example, com.google.common.base.internal.Finalizer is used in Guava itself, but is not part of the official API. It’s a public class, since other Guava packages use Finalizer. Unfortunately, this also means there’s no impediment for com.myapp.Main to use classes like Finalizer. In other words, there’s no strong encapsulation.

The same holds for internal classes from the Java platform itself. Packages such as sun.misc have always been accessible to application code, even though documentation sternly warns they are unsupported APIs that should not be used. Despite this warning, utility classes such as sun.misc.BASE64Encoder are used in application code all the time. Technically, that code may break with any update of the Java runtime, since they are internal implementation classes. Lack of encapsulation essentially forced those classes to be considered semipublic APIs anyway, since Java highly values backward compatibility. This is an unfortunate situation, arising from the lack of true encapsulation.

What about explicit dependencies? As you’ve already learned, there is no dependency information anymore when looking strictly at JARs. You run MyApplication as follows:

java -classpath lib/guava-19.0.jar:\
                lib/hibernate-validator-5.3.1.jar:\
                lib/jboss-logging-3.3.0Final.jar:\
                lib/classmate-1.3.1.jar:\
                lib/validation-api-1.1.0.Final.jar \
     -jar MyApplication.jar
Setting up the correct classpath is up to the user. And, without explicit dependency information, it is not for the faint of heart.

Classpath Hell
The classpath is used by the Java runtime to locate classes. In our example, we run Main, and all classes that are directly or indirectly referenced from this class need to be loaded at some point. You can view the classpath as a list of all classes that may be loaded at runtime. While there is more to it behind the scenes, this view suffices to understand the issues with the classpath.

A condensed view of the resulting classpath for MyApplication looks like this:

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
There’s no notion of JARs or logical grouping anymore. All classes are sequenced into a flat list, in the order defined by the -classpath argument. When the JVM loads a class, it reads the classpath in sequential order to find the right one. As soon as the class is found, the search ends and the class is loaded.

What if a class cannot be found on the classpath? Then you will get a run-time exception. Because classes are loaded lazily, this could be triggered when some unlucky user clicks a button in your application for the first time. The JVM cannot efficiently verify the completeness of the classpath upon starting. There is no way to tell in advance whether the classpath is complete, or whether you should add another JAR. Obviously, that’s not good.

Graph Theory for Geeks by Oliver Widder link:http://geek-and-poke.com/geekandpoke/2011/7/27/graph-theory-for-geeks.html, CC-BY link:https://creativecommons.org/licenses/by/3.0/deed.en_US
More insidious problems arise when duplicate classes are on the classpath. Let’s say you try to circumvent the manual setup of the classpath. Instead, you let Maven construct the right set of JARs to put on the classpath, based on the explicit dependency information in POMs. Since Maven resolves dependencies transitively, it’s not uncommon for two versions of the same library (say, Guava 19 and Guava 18) to end up in this set, through no fault of your own. Now both library JARs are flattened into the classpath, in an undefined order. Whichever version of the library classes comes first is loaded. However, other classes may expect a class from the (possibly incompatible) other version. Again, this leads to run-time exceptions. In general, whenever the classpath contains two classes with the same (fully qualified) name, even if they are completely unrelated, only one “wins.”

It now becomes clear why the term classpath hell (also known as JAR hell) is so infamous in the Java world. Some people have perfected the art of tuning a classpath through trial-and-error—a rather sad occupation when you think about it. The fragile classpath remains a leading cause of problems and frustration. If only more information were available about the relations between JARs at run-time. It’s as if a dependency graph is hiding in the classpath and is just waiting to come out and be exploited. Enter Java 9 modules!

Java 9 Modules
By now, you have a solid understanding of Java’s current strengths and limitations when it comes to modularity. With Java 9, we get a new ally in the quest for well-structured applications: the Java module system. While designing the Java Platform Module System to overcome current limitations, two main goals were defined:

Modularize the JDK itself.

Offer a module system for applications to use.

These goals are closely related. Modularizing the JDK is done by using the same module system that we, as application developers, can use in Java 9.

The module system introduces a native concept of modules into the Java language and runtime. Modules can either export or strongly encapsulate packages. Furthermore, they express dependencies on other modules explicitly. As you can see, all three tenets of modularity are addressed by the Java module system.

Let’s revisit the MyApplication example, now based on the Java 9 module system, in Figure 1-2.

Each JAR becomes a module, containing explicit references to other modules. The fact that hibernate-validator uses jboss-logging, classmate, and validation-api is part of its module descriptor. A module has a publicly accessible part (on the top) and an encapsulated part (on the bottom, indicated with the padlock). That’s why MyApplication can no longer use Guava’s Finalizer class. Through this diagram, we discover that MyApplication uses validation-api, as well, to annotate some of its classes. What’s more, MyApplication has an explicit dependency on a module in the JDK called java.sql.

MyApplication as modular application on top of modular Java 9.
Figure 1-2. MyApplication as a modular application on top of modular Java 9
Figure 1-2 tells us much more about the application than in the classpath situation shown in Figure 1-1. All that could be said there is that MyApplication uses classes from rt.jar, like all Java applications—and that it runs with a bunch of JARs on the (possibly incorrect) classpath.

That’s just the application layer. It’s modules all the way down. At the JDK layer, there are modules as well (Figure 1-2 shows a small subset). Like the modules in the application layer, they have explicit dependencies and expose some packages while concealing others. The most essential platform module in the modular JDK is java.base. It exposes packages such as java.lang and java.util, which no other module can do without. Because you cannot avoid using types from these packages, every module requires java.base implicitly. If the application modules require any functionality from platform modules other than what’s in java.base, these dependencies must be explicit as well, as is the case with MyApplication’s dependency on java.sql.

Finally, there’s a way to express dependencies between separate parts of the code at a higher level of granularity in the Java language. Now imagine the advantages of having all this information available at compile-time and run-time. Accidental dependencies on code from other nonreferenced modules can be prevented. The toolchain knows which additional modules are necessary for running a module by inspecting its (transitive) dependencies, and optimizations can be applied using this knowledge.

Strong encapsulation, well-defined interfaces, and explicit dependencies are now part of the Java platform. In short, these are the most important benefits of the Java Platform Module System:

Reliable configuration
The module system checks whether a given combination of modules satisfies all dependencies before compiling or running code. This leads to fewer run-time errors.

Strong encapsulation
Modules explicitly choose what to expose to other modules. Accidental dependencies on internal implementation details are prevented.

Scalable development
Explicit boundaries enable teams to work in parallel while still creating maintainable codebases. Only explicitly exported public types are shared, creating boundaries that are automatically enforced by the module system.

Security
Strong encapsulation is enforced at the deepest layers inside the JVM. This limits the attack surface of the Java runtime. Gaining reflective access to sensitive internal classes is not possible anymore.

Optimization
Because the module system knows which modules belong together, including platform modules, no other code needs to be considered during JVM startup. It also opens up the possibility to create a minimal configuration of modules for distribution. Furthermore, whole-program optimizations can be applied to such a set of modules. Before modules, this was much harder, because explicit dependency information was not available and a class could reference any other class from the classpath.

In the next chapter, we explore how modules are defined and what concepts govern their interactions. We do this by looking at modules in the JDK itself. There are many more platform modules than shown in Figure 1-2.

Exploring the modular JDK in Chapter 2 is a great way to get to know the module system concepts, while at the same time familiarizing yourself with the modules in the JDK. These are, after all, the modules you’ll be using first and foremost in your modular Java 9 applications. After that, you’ll be ready to start writing your own modules in Chapter 3.