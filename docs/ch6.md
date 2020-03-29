# 第 6 章 高级模块化模式
> Chapter 6. Advanced Modularity Patterns

The previous chapter presented general design guidelines and patterns for modular application development. This chapter contains more advanced patterns and module system APIs that may not apply to everyday development. Still, it is an important part of the module system. Not only is the module system meant to be used directly by application developers, but it also serves as a basis for other frameworks to build upon. The advanced APIs primarily revolve around such usage.

The next section explores the need for reflection that many libraries and frameworks currently have. Open modules and packages are introduced as a feature to relax strong encapsulation at run-time. It’s an important feature during migration too, so it will be revisited in Chapter 8.

After open modules and packages, the focus shifts to patterns for dynamically extensible applications. Think of plug-in-based systems or application containers. Central to these systems is the challenge of adding modules at run-time, instead of working only with a fixed configuration of modules from the module path.

TIP
Feel free to skip this latter part of the chapter when first learning about the module system. Most applications get by fine without ever using the module system’s more dynamic and advanced features. You can always come back later to gain a better understanding of these features after getting more experience with typical modular scenarios first.

Strong Encapsulation Revisited
We discussed the virtues of strong encapsulation at length in the previous chapters. In general, it is beneficial to be strict about what is and what is not a public, exported API. But in practice, the line isn’t drawn quite as clearly. Many libraries and frameworks exist that rely on accessing implementation classes of your application to do their work. Think of serialization libraries, object-relational mappers, and dependency injection frameworks. All these libraries want to manipulate classes that you’d otherwise deem internal implementation details.

An object-relational mapper or serialization library needs to access entity classes so it can instantiate them and fill them with the right data. Even if an entity class never leaves the module, the ORM library module somehow needs access. Dependency injection frameworks, as another example, need to inject service instances into service implementation classes. Exporting just the interfaces is not enough. By strongly encapsulating implementation classes within modules, these frameworks are denied the usual access they previously enjoyed on the classpath.

Reflection is almost without exception the tool of choice for these frameworks. Reflection is an important part of the Java platform, allowing code to inspect code at run-time. If that sounds a bit esoteric, that’s because it is. Using reflection in application code is not something to strive for. Still, without reflection, many generic frameworks (such as Hibernate or Spring) would not have been possible. But even reflection cannot break the walls of strong encapsulation around nonexported packages in modules, as you have learned in previous chapters.

The wrong reaction here is to export those packages indiscriminately. Exporting a package means its API can be compiled against and relied upon from different modules. That’s not what we want to express in this case. We need a mechanism to indicate that it’s OK for some libraries to get (reflective) run-time access to certain types.

Deep Reflection
Two orthogonal issues need to be addressed to make traditional reflection-based libraries play nice with strong encapsulation:

Provide access to internal types without exporting packages.

Allow reflective access to all parts of these types.

We’ll zoom in on the second problem first. Let’s assume we export a package so a library can get access. You already know that means we can compile against just the public types in those packages. But does it also mean we can use reflection to break into private parts of those types at run-time? This practice of deep reflection is used by many libraries. For example, Spring or Hibernate inject values into nonpublic fields of classes.

Back to our question: can you do deep reflection on public types from exported packages? The answer is no. It turns out that even if a type is exported, this does not mean you can unconditionally break into private parts of those types with reflection.

From a modularity perspective, this is the right thing to do. When arbitrary modules can break into private parts of exported types, they will do so. In effect, those private parts become part of the official API again. The same scenario played out already in the JDK itself, as discussed in “Using the Modular JDK Without Modules”.

Preventing access to nonpublic parts is not only a matter of API hygiene: private fields tend to be private for good reasons. For example, there is a java.security.KeyStore class in the JDK that manages keys and credentials. The authors of this class specifically do not want anyone to access the private fields guarding those secrets!

Exporting a package does not allow a module using those exported types to reflect over nonpublic parts. Deep reflection is supported in Java by the setAccessible method that is available on reflective objects. It circumvents checks that otherwise block access to inaccessible parts. Before the module system and strong encapsulation, setAccessible would basically never fail. With the module system, the rules have changed. Figure 6-1 shows which scenarios won’t work anymore.

Module `deepreflection` exports a package containing class `Exported` and encapsulates the `NotExported` class. The snippets (assumed to be in another module) show reflection only works on public parts of exported types. Only `Exported.doWork` can be accessed reflectively, all others lead to exceptions.
Figure 6-1. Module deepreflection exports a package containing class Exported and encapsulates the NotExported class. The snippets (assumed to be in another module) show reflection works only on public parts of exported types. Only Exported::doWork can be accessed reflectively; all others lead to exceptions.
So we’re left in the situation where many popular libraries want to perform deep reflection, whether a type is exported or not. Because of strong encapsulation, they can’t. And even if implementation classes were exported, deep reflection on nonpublic parts is prohibited.

Open Modules and Packages
What we need is a way to make types accessible for deep reflection at run-time without exporting them. With such a feature, the frameworks can do their work again, while strong encapsulation is still upheld at compile-time.

Open modules offer exactly this combination of features. When a module is open, all its types are available for deep reflection by other modules at run-time. This property holds regardless of whether any packages are exported.

Making a module open is done by putting the open keyword in the module descriptor:

open module deepreflection {
  exports api;
}
All previous failure modes from Figure 6-1 disappear when a module is open, as shown in Figure 6-2.

The open keyword opens all packages in a module for deep reflection. In addition to a package being open, it can also be exported, as is the case for the api package containing the class Exported in this example. Any module reflectively accessing nonpublic elements from Exported or NotExported can do so after calling setAccessible. Readability to the module deepreflection is assumed by the JVM when using reflection on its types, so no special code has to be written for this to work. At compile-time, NotExported still is inaccessible, while Exported is accessible because of the exports clause in the module descriptor. From the application developer’s point of view, the NotExported class is still strongly encapsulated at compile-time. From the framework’s perspective, the NotExported class is freely accessible at run-time.

TIP
With Java 9, two new methods for reflection objects are added: canAccess and trySetAccessible (both are defined in java.lang.reflect.AccessibleObject). These methods take into account the new reality where deep reflection is not always allowed. You can use these methods instead of dealing with exceptions from setAccessible.

Scenarios showing deep reflection works for all types in an open module.
Figure 6-2. With an open module, all types in all packages are open for deep reflection at run-time. No exceptions are thrown when performing deep reflection from another module.
Opening a whole module is somewhat crude. It’s convenient for when you can’t be sure what types are used at run-time by a library or framework. As such, open modules can play a crucial role in migration scenarios when introducing modules into a codebase. More on this in Chapter 8.

However, when you know which packages need to be open (and in most cases, you should), you can selectively open them from a normal module:

module deepreflection {
  exports api;
  opens internal;
}
Notice the lack of open before the module keyword. This module definition is not equivalent to the previous open module definition. Here, only the types in package internal are open for deep reflection. Types in api are exported, but are not open for deep reflection. At run-time this module definition provides somewhat stronger encapsulation than the fully open module, since no deep reflection is possible on types in package api.

We can make this module definition equivalent to the open module by also opening the api package:

module deepreflection {
  exports api;
  opens api;
  opens internal;
}
A package can be both exported and open at the same time.

NOTE
In practice, this combination is a bit awkward. Try to design exported packages in such a way that other modules don’t need to perform deep reflection on them.

opens clauses can be qualified just like exports:

module deepreflection {
  exports api;
  opens internal to library;
}
The semantics are as you’d expect: only module library can perform deep reflection on types from package internal. A qualified opens reduces the scope to just one or more other explicitly mentioned modules. When you can qualify an opens statement, it’s better to do so. It prevents arbitrary modules from snooping on internal details through deep reflection.

Sometimes you need to be able to perform deep reflection on a third-party module. In some cases, libraries even want to reflectively access private parts of JDK platform modules. It’s not possible to just add the open keyword and recompile the module in these cases. For these scenarios, a command-line flag is introduced for the java command:

--add-opens <module>/<package>=<targetmodule>
This is equivalent to a qualified opens for package in module to targetmodule. If, for example, a framework module myframework wants to use nonpublic parts of java.lang.ClassLoader, you can do so by adding the following option to the java command:

--add-opens java.base/java.lang=myframework
This command-line option should be considered an escape hatch, especially useful during migration of code that’s not written with the module system in mind yet. In Part II, this option and others like it will reappear for these purposes.

Dependency Injection
Open modules and packages are the gateway to supporting existing dependency injection frameworks in the Java module system. In a fully modularized application, a dependency injection framework relies on open packages to access nonexported types.

REPLACING REFLECTION
Java 9 offers an alternative to reflection-based access of frameworks to nonpublic class members in applications: MethodHandles and VarHandles. The latter are introduced in Java 9 through JEP 193. Applications can pass a java.lang.invoke.Lookup instance with the right permissions to the framework, explicitly delegating private lookup capabilities. The framework module can then, using MethodHandles.privateLookupIn(Class, Lookup), access nonpublic members on classes from the application module. It is expected that frameworks, in time, move to this more principled and performance-friendly approach to access application internals. An example of this approach can be found in the code accompanying this chapter (➥ chapter6/lookup).

To illustrate the abstract concept of open modules and packages, we’ll look at a concrete example. Instead of using services with the module system as described in Chapter 4, this example features a fictional third-party dependency injection framework in a module named spruice. Figure 6-3 shows the example. An open package is indicated by an “open” label in front of the types of a package.

Our example application covers two domains: orders and customers. These are clearly visible as separate modules, with the customers domain split into an API and implementation module. The main module uses both services but doesn’t want to be coupled to the implementation details of either service. Both service implementation classes are encapsulated to that end. Only the interfaces are exported and accessible to the main module.

Overview of an application `main` using a dependency injection library `spruice`. Requires relations between modules are shown as solid lines, run-time deep reflection on types in open packages is shown as dashed lines.
Figure 6-3. Overview of an application main using a dependency injection library spruice. Requires relations between modules are shown as solid edges, and run-time deep reflection on types in open packages is shown as dashed edges.
So far, the story is quite similar to the approach described in Chapter 4 on services. The main module is nicely decoupled from implementation specifics. However, this time we’re not going to provide and consume the services through ServiceLoader. Rather, we are going to use the DI framework to inject OrderService and CustomerService implementations into Application. Either we configure spruice explicitly with the correct wiring of services, or we use annotations, or a combination of both approaches. Annotations such as @Inject are often used to identify injection points that are matched on type with injectable instances:

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
In the main method of the Application class, spruice APIs are used to bootstrap the DI framework, Therefore, the main module needs a dependency on spruice for its wiring API. Service implementation classes must be instantiated by spruice and injected into the annotated fields (constructor injection is another viable alternative). Then, Application is ready to receive calls on orderForCustomer.

Unlike the module system’s services mechanism, spruice does not have special privileges to instantiate encapsulated classes. What we can do is add opens clauses for the packages that need to be instantiated or injected. This allows spruice to access those classes at run-time and perform deep reflection where necessary (e.g., to instantiate and inject the OrderServiceImpl class into the orderService private field of Application). Packages that are used only internally in a module, such as cust.internal in the customers module, don’t need to be opened. The opens clauses could be qualified to open to spruice only. Unfortunately, that also ties our orders and customers modules to this specific DI framework. An unqualified opens leaves room for changing DI implementations without recompiling those modules later.

Figure 6-3 exposes spruice for what it really is: a module reaching into almost every dark corner of the application we’re building. Based on the wiring configuration, it finds encapsulated implementation classes, instantiates them, and injects them into private fields of Application. At the same time, this setup allows the application to be just as nicely modularized as with services and ServiceLoader—without having to use the ServiceLoader API to retrieve services. They are injected as if by (reflection) magic.

What we lose is the ability of the Java module system to know about and verify the service dependencies between modules. There are no provides/uses clauses in module descriptors to verify. Also, packages in the application modules need to be opened. It is possible to make all application modules open modules. Application developers then aren’t burdened with making that choice for each package. Of course, this comes at the expense of allowing run-time access and deep reflection on all packages in every application module. With a little insight into what your libraries and frameworks are doing, this heavyweight approach isn’t necessary.

In the next section, we’ll look at reflection on modules themselves. Again, this is an advanced use of the module system APIs, which should not come up in normal application development all that often.

Reflection on Modules
Reflection allows you to reify all Java elements at run-time. Classes, packages, methods, and so on all have a reflective representation. We’ve seen how open modules allow deep reflection on those elements at run-time.

With the addition of a new structural element to Java, the module, reflection needs to be extended as well. In Java 9, java.lang.Module is added to provide a run-time view on a module from code. Methods on this class can be grouped into three distinct responsibilities:

Introspection
Query properties of a given module.

Modification
Change characteristics of a module on-the-fly.

Access
Read resources from inside a module.

The last case was discussed already in “Loading Resources from a Module”. In the remainder of this section, we’ll look at introspecting and modifying modules.

Introspection
The java.lang.Module class is the entry point for reflection on modules. Figure 6-4 shows Module and its related classes. It has a ModuleDescriptor that provides a run-time view on the contents of module-info.

A condensed class diagram of `Module` and related classes.
Figure 6-4. A simplified class diagram of Module and related classes
You can obtain a Module instance through a Class from within a module:

Module module = String.class.getModule();
The getModule method returns the module containing the class. For this example, the String class unsurprisingly comes from the java.base platform module. Later in this chapter, you will see another way to obtain Module instances by name through the new ModuleLayer API, without requiring knowledge of classes in a module.

There are several methods to query information on a Module, shown in Example 6-1.

Example 6-1. Inspecting a module at run-time (➥ chapter6/introspection)
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
The preceding examples are by no means exhaustive but illustrate that all information from module-info.class is available through the ModuleDescriptor class. Instances of ModuleDescriptor are read-only (immutable). It is not possible to, for example, change the name of a module at run-time.

Modifying Modules
You can perform several other operations on a Module that affect the module and its environment. Let’s say you have a package that is not exported, but based on a run-time decision you want to export it:

Module target = ...; // Somehow obtain the module you want to export to
Module module = getClass().getModule(); // Get the module of the current class
module.addExports("javamodularity.export.atruntime", target);
You can add a qualified export only to a specific module through the Module API. Now, the target module can access code that was in the previously encapsulated javamodularity.export.atruntime package.

You may wonder whether this is a security hole: can you just call addExports on an arbitrary module so it gives up its secrets? That’s not the case. When you try to add an export to any module other than the current module where the call is executing from, an exception is thrown by the VM. You cannot escalate privileges of a module from the outside through the module reflection API.

CALLER SENSITIVE
Methods that behave differently when called from different places are caller sensitive methods. You will find methods such as addExports to be annotated with @CallerSensitive in the JDK source code. Caller sensitive methods can find out which class (and module) is calling them, based on the current call-stack. The privilege of getting this information and basing decisions on it is reserved for code in java.base (although the new StackWalker API introduced through JEP 259 in JDK 9 opens this possibility for application code as well). Another example of this mechanism can be found in the implementation of setAccessible, as discussed in “Open Modules and Packages”. Calling this method from a module to which the package is not opened leads to an exception, whereas calling it from a module that is allowed to perform deep reflection succeeds.

Module has four methods that allow run-time modifications:

addExports(String packageName, Module target)
Expose previously nonexported packages to another module.

addOpens(String packageName, Module target)
Opens a package for deep reflection to another module.

addReads(Module other)
Adds a reads relation from the current module to another module.

addUses(Class<?> serviceType)
Indicates that the current module wants to use additional service types with ServiceLoader.

There’s no addProvides method, because exposing new implementations that were not known at compile-time is deemed to be a rare use case.

It’s good to know about the reflection API on modules. However, in practice this API is used only on rare occasions during regular application development. Always try to expose enough information between modules through normal means before reaching for reflection. Using reflection to change module behavior at run-time goes against the philosophy of the Java module system. Implicit dependencies arise that are not taken into account at compile-time or at startup, voiding a lot of guarantees the module system otherwise offers in those early phases.

Annotations
Modules can also be annotated. At run-time, those annotations can be read through the java.lang.Module API. Several default annotations that are part of the Java platform can be applied to modules, for example, the @Deprecated annotation:

@Deprecated
module m {

}
Adding this annotation indicates to users of the module that they should look for a replacement.

TIP
As of Java 9, a deprecated element can also be marked for removal in a future release: @Deprecated(forRemoval=true). Read JEP 277 for more details on the enhanced deprecation features. Several platform modules (such as java.xml.ws and java.corba) are marked for removal in JDK 9.

When you require a deprecated module, the compiler generates a warning. Another default annotation that can be applied to modules is @SuppressWarnings.

It’s also possible to define your own annotations for modules. To this end, a new target element type MODULE is defined, as shown in Example 6-2.

Example 6-2. Annotating a module (➥ chapter6/annotated_module)
package javamodularity.annotatedmodule;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(value={PACKAGE, MODULE})
public @interface CustomAnnotation {

}
This CustomAnnotation can now be applied to packages and modules. Using a custom-defined annotation on a module reveals another curious fact: module declarations can have import statements.

import javamodularity.annotatedmodule.CustomAnnotation;

@CustomAnnotation
module annotated { }
Without the import statement, the module descriptor doesn’t compile. Alternatively, you can use the fully qualified name of the annotation directly, without an import.

TIP
You can also use import in module descriptors to shorten uses/provides clauses.

Figure 6-4 shows how Module implements AnnotatedElement. In code, you can use getAnnotations on a Module instance to get an array of all annotations on a module. The AnnotatedElement interface offers various other methods to find the right annotation.

WARNING
This works only if the retention policy of the annotation is set to RUNTIME.

Besides platform-defined annotations such as @Deprecated, it will most probably be frameworks (or even build tools) that make use of annotations on modules.

Container Application Patterns
At this point, we’re switching gears to even more advanced uses of the module system. This is a good moment to reiterate the advice from the beginning of this chapter: you can safely skip the remainder of this chapter when first learning about the module system.

With that out of the way, let’s dive into the advanced APIs! Up to this point, we’ve looked at modular applications as a single entity. You gather modules, put them on the module path, and start the application. Although that’s a valid view in most cases, another range of applications is structurally different.

You can think of applications acting as a container for other applications. Or, think of applications that define just their core functionality, with the expectation that this will be extended by third parties. In the former case, new applications can be deployed into a running container application. The latter case is often achieved through a plug-in-like architecture. In both cases, you don’t start with a module path containing all modules up front. New modules can come (and go) during the life cycle of the container application. Many of these applications are currently built on module systems such as OSGi or JBoss Modules.

In this section, you’ll look at architecting such a container or plug-in-based system by using the Java module system. Before looking at the realization of these container application patterns, you’ll explore the new APIs that enable them. Keep in mind that these new APIs are introduced specifically for the use cases discussed so far. When you’re not building an extensible, container-like application, you are unlikely to use those APIs.

Layers and Configurations
A module graph is resolved upon starting a module with the java command. The resolver uses modules from the platform itself and the module path to create a consistent set of resolved modules. It does so based on requires clauses and provides/uses clauses in the module descriptors. When the resolver is finished, the resulting module graph cannot be changed anymore.

That seems to run counter to the requirements of container applications. There, the ability to add new modules to a running container is crucial. A new concept must be introduced to allow for this functionality: layers. Layers are to modules as classloaders are to classes: a loading and instantiation mechanism.

A resolved module graph lives within a ModuleLayer. Layers set up coherent sets of modules. Layers themselves can refer to parent layers and form an acyclic graph. But before we get ahead of ourselves, let’s look at the ModuleLayer you already had without knowing.

When plainly starting a module by using java, an initial layer called the boot layer is constructed by the Java runtime. It contains the resulting module graph after resolving the root modules provided to the java command (either as an initial module with -m or through --add-modules).

In Figure 6-5, a simplified example of the boot layer is shown after starting a module application that requires java.sql.

Modules in the boot layer after starting module `application`.
Figure 6-5. Modules in the boot layer after starting module application
(In reality, the boot layer contains many more modules because of service binding of platform modules.)

You can enumerate the modules in the boot layer with the code in Example 6-3.

Example 6-3. Printing all modules in the boot layer (➥ chapter6/bootlayer)
ModuleLayer.boot().modules()
           .forEach(System.out::println);
Because the boot layer is constructed for us, the question of how to create another layer remains. Before you can construct a ModuleLayer, a Configuration must be created describing the module graph inside that layer. Again, for the boot layer this is done implicitly at startup. When constructing a Configuration, a ModuleFinder needs to be provided to locate individual modules. Setting up this Russian doll of classes to create a new ModuleLayer looks as follows:

ModuleFinder finder = ModuleFinder.of(Paths.get("./modules")); 1

ModuleLayer bootLayer = ModuleLayer.boot();

Configuration config = bootLayer.configuration()
   .resolve(finder, ModuleFinder.of(), Set.of("rootmodule")); 2

ClassLoader scl = ClassLoader.getSystemClassLoader();
ModuleLayer newLayer = bootLayer.defineModulesWithOneLoader(config, scl); 3
1
A convenience method to create a ModuleFinder that locates modules in one or more paths on the filesystem.

2
Configurations are resolved relative to parent configurations; in this case, to the configuration of the boot layer.

3
With Configuration, a ModuleLayer can be constructed, materializing the resolved modules from the configuration.

In principle, modules could come from anywhere. Typically, they’re somewhere on the filesystem, so the ModuleFinder::of(Path...) factory method is convenient to use. It returns a ModuleFinder implementation that can load modules from the filesystem. Every ModuleLayer and Configuration points to one or more parents, with the exception of instances returned by the ModuleLayer::empty and Configuration::empty methods. Those special instances serve as root for the ModuleLayer and Configuration hierarchy in the module system. The boot layer and corresponding configuration have their empty counterparts as a parent.

While constructing the new layer, we use the boot layer and configuration as a parent. In the resolve call, the ModuleFinder is passed as the first argument. The second argument to resolve is another ModuleFinder, in this case, an empty one. This second finder is consulted when a module could not be found in the first finder or through the parent configuration.

When resolving modules in the new configuration, modules from the parent configuration are taken into account as well. A module from the newly constructed configuration can read a module from the parent configuration. Root modules to kickstart the resolver are passed as a third argument to the configuration’s resolve method. In this case, rootmodule serves as a initial module for the resolver to start from. Resolution in a new configuration is subject to the same constraints you’ve seen so far. It fails if a root module or one of its dependencies cannot be found. Cycles between modules are not allowed, nor can two modules exporting the same package be read by a single other module.

To expand on the example, let’s say rootmodule requires the javafx.controls platform module and library, a helper module that also lives in the ./modules directory. After resolving the configuration and constructing the new layer, the resulting situation looks like Figure 6-6.

A new layer with the boot layer as its parent.
Figure 6-6. A new layer with the boot layer as its parent
Readability relations can cross layer boundaries. The requires clause of rootmodule to javafx.controls has been resolved to the platform module in the boot layer. On the other hand, the requires clause to library was resolved in the newly constructed layer, because that module is loaded along with rootmodule from the filesystem.

Besides the resolve method on Configuration, there’s also resolveAndBind. This variation also does service binding, taking into account the provides/uses clauses of the modules in the new configuration. Services can cross layer boundaries as well. Modules in the new layer can use services from the parent layer, and vice versa.

Last, the defineModulesWithOneLoader method is called on the parent (boot) layer. This method materializes the module references resolved by the Configuration into real Module instances inside the new layer. The next section discusses the significance of the classloader passed to this method.

All examples of layers you’ve seen so far consisted of new layers pointing to the boot layer as the parent. However, layers can point to a parent layer other than the boot layer as well. It’s even possible for a layer to have multiple parents. This is illustrated in Figure 6-7: Layer 3 points to Layer 1 and Layer 2 as its parents.

Layers can form an acyclic graph.
Figure 6-7. Layers can form an acyclic graph
Static defineModules* methods on ModuleLayer accept a list of parent layers when constructing new layers, instead of calling any of the nonstatic defineModules* methods on a parent layer instance directly. Later, in Figure 6-12, you’ll see those static methods in action. The important thing to remember is that layers can form an acyclic graph, just like modules within a layer.

Classloading in Layers
You may still be wondering about the last two lines of the layer construction code in the previous section:

ClassLoader scl = ClassLoader.getSystemClassLoader();
ModuleLayer newLayer = bootLayer.defineModulesWithOneLoader(config, scl);
What’s up with the classloader and the defineModulesWithOneLoader method? The answer to this question is nontrivial. Before we get to the method in question, it’s good to brush up on what classloaders do and what that means for the module system.

Classloaders, no surprise, load classes at run-time. Classloaders dictate visibility: a class can be loaded if it is visible to a certain classloader, or any other classloaders it delegates to. In a way, it’s peculiar to introduce classloaders this far into the book. Earlier module systems such as OSGi use classloaders as the primary means to enforce encapsulation. Every bundle (OSGi module) gets its own classloader, and the delegation between classloaders follows the wiring of bundles as expressed in the OSGi metadata.

Not so in the Java module system. There, a whole new mechanism encompassing readability and new accessibility rules is put in place, while leaving classloading mostly unchanged. That’s a deliberate choice, because using classloaders for isolation is not a foolproof solution. After classes have been loaded, Class instances can be passed around freely, bypassing any schemes set up through classloader isolation and delegation. You can try to do that in the module system, but you’ve seen that creating instances from Class objects you’re not allowed to access because of encapsulation leads to exceptions. The module system enforces encapsulation at a much deeper level. Furthermore, classloaders are only a run-time construct, whereas the Java module system enforces encapsulation at compile-time as well. Last, many existing codebases make assumptions about the way classes are loaded by default. Changing these defaults (for example, by giving each module its own classloader) would break existing code.

Still, it’s good to be aware of the way classloaders interact with the module system. Let’s revisit Figure 6-5, where a module application is loaded in the boot layer. This time, we’re interested in what classloaders are involved, as shown in Figure 6-8.

Classloaders in the boot layer when starting module `application`.
Figure 6-8. Classloaders in the boot layer when starting module application
Three classloaders are active in the boot layer when running an application from the module path. At the bottom of the delegation hierarchy is BootstrapClassLoader, also known as the primordial classloader. It’s a special classloader loading all essential platform module classes. Care has been taken to load classes from as few modules as possible in this classloader, because classes are granted all security permissions when loaded in the bootstrap loader.

Then there’s PlatformClassLoader, which loads less privileged platform module classes. Last in the chain is AppClassLoader, responsible for loading user-defined modules and some JDK-specific tooling modules (such as jdk.compiler or jdk.javadoc). All classloaders delegate to the underlying classloader. Effectively, this gives AppClassLoader visibility to all classes. This three-way setup is quite similar to the way classloaders worked before the module system, mainly for backward compatibility.

We started this section with the question of why a classloader needs to be passed to the method creating a ModuleLayer:

ClassLoader scl = ClassLoader.getSystemClassLoader();
ModuleLayer newLayer = bootLayer.defineModulesWithOneLoader(config, scl);
Even though the mapping from modules to classloaders is predefined in the boot layer, the creator of a new layer must indicate which classloaders load classes for which modules. The ModuleLayer::defineModulesWithOneLoader(Configuration, ClassLoader) method is a convenience method. It sets up the new layer so that all modules in the layer are loaded by a single, freshly created classloader. This classloader delegates to the parent classloader passed as an argument. In the example, we pass the result of ClassLoader::getSystemClassLoader, which returns AppClass​Loader, the classloader responsible for loading classes of user-defined modules in the boot layer (there’s also a getPlatformClassLoader method).

So, the classloader view on the newly constructed layer from the example (as seen earlier in Figure 6-6) looks like Figure 6-9.

A single classloader is created for the new layer.
Figure 6-9. A single classloader is created for all modules in the new layer
The delegation between classloaders must respect the readability relations between modules, even across layers. It would be problematic if the new classloader didn’t delegate to AppClassLoader in the boot layer, but, for example, to BootstrapClass​Loader. Because rootmodule reads javafx.controls, it must be able to see and load those classes. Parent delegation of the new layer’s classloader to AppClassLoader ensures this. In turn, AppClassLoader delegates to PlatformClassLoader, which loads the classes from javafx.controls.

There are other methods to create a new layer. Another convenience method called defineModulesWithManyLoaders creates a new classloader for each module in the layer, as shown in Figure 6-10.

A single classloader is created for every module in the new layer.
Figure 6-10. Every module within a layer constructed using defineModulesWithManyLoaders gets its own classloader
Again, each of these new classloaders delegates to the parent that is passed as an argument to defineModulesWithManyLoaders. If you need even more control over classloading in a layer, there’s the defineModules method. It takes a function mapping a string (module name) to a classloader. Providing such a mapping gives ultimate flexibility on when to create new classloaders for a new module or to assign existing classloaders to modules in the layer. An example of such a mapping can be found in the JDK itself. The boot layer is created using defineModules with a custom mapping to one of the three classloaders shown previously in Figure 6-8.

Why is it important to control classloading when using layers? Because it can make many limitations in the module system disappear. For example, we discussed how only a single module can contain (or export) a certain package. It turns out this is just a side effect of how the boot layer is created. A package can be defined to a classloader only once. All modules from the module path are loaded by AppClassLoader. Therefore, if any of these modules contain the same package (exported or not), they will be defined to the same AppClassLoader leading to run-time exceptions.

When you instantiate a new layer with a new classloader, the same package may appear in a different module in that layer. In that case, there is no inherent problem because the package gets defined twice to different classloaders. You’ll see later that this also means multiple versions of the same module can live in distinct layers. That’s quite an improvement on the situation we discussed in “Versioned Modules”, although it comes at the expense of the complexity of creating and managing layers.

Plug-in Architectures
Now that you have seen the basics of layers and classloading in layers, it’s time to apply them in practice. First, you’ll look at creating an application that can be extended with new plug-ins at run-time. In many ways, this is similar to what we did with EasyText and services in Chapter 4. With the services approach, when a new analysis provider is put on the module path, it is picked up when starting the application. This is already quite flexible, but what if those new provider modules come from third-party developers? And what if they aren’t put on the module path at startup, but can be introduced at run-time?

Such requirements lead to a more flexible plug-in architecture. A well-known example of a plug-in-based application is the Eclipse IDE. Eclipse itself offers baseline IDE functionality but can be extended in many ways by adding plug-ins. Another example of an application that uses plug-ins is the new jlink tool in the JDK. It can be extended with new optimizations through a plug-in mechanism similar to the one we’re going to look at now. Chapter 13 discusses more details on what plug-ins the jlink tool can use.

In general, we can identify a plug-in host application that gets extended by plug-ins, as shown in Figure 6-11.

Plugins provide additional functionality on top of the host application's functionality.
Figure 6-11. Plug-ins provide additional functionality on top of the host application’s functionality
Users interact with the host application, but experience extended functionality by virtue of the plug-ins. In most cases, the host application can function fine without any plug-ins as well. Plug-ins are usually developed independently of the host application. A clear boundary exists between the host application and the plug-ins. At run-time, the host application calls into the plug-ins for additional functionality. For this to work, there must be an agreed-upon API that the plug-ins implement. Typically, this is an interface that plug-ins implement.

You may have already guessed that layers play a role in the implementation of a dynamic, plug-in-based application. We’re going to create a pluginhost module that spins up a new ModuleLayer for each plug-in. These plug-in modules, plugin.a and plugin.b in this example, live in separate directories along with their dependencies (if any). Crucially, these directories are not on the module path when starting pluginhost.

The example uses services with a pluginhost.api module exposing the interface pluginhost.api.Plugin, which has a single method, doWork. Both plug-in modules require this API module, but otherwise don’t have a compile-time relation to the pluginhost application module. A plug-in module consists of an implementation of the Plugin interface, provided as service. Take the module descriptor of module plugin.a, shown in Example 6-4.

Example 6-4. module-info.java (➥ chapter6/plugins)
module plugin.a {
  requires pluginhost.api;

  provides pluginhost.api.Plugin
      with plugina.PluginA;
}
The plug-in implementation class PluginA is not exported.

In pluginhost, the main method loads plug-in modules from directories provided as arguments:

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
1
For each directory provided as an argument, a ModuleLayer is instantiated in createPluginLayer (implementation shown later).

2
A ServiceLoader::load call is performed with each layer as an argument, giving back Plugin services from that layer.

3
After the services have been flattened into a single stream, we call the doWork method on all plug-ins.

We are using a yet-to-be discussed overload of ServiceLoader::load. It takes a layer as an argument. By passing it the newly constructed layer for the plug-in, the call returns the freshly loaded Plugin service provider from the plug-in module that was loaded into the layer.

The application is started by running the pluginhost module from the module path. Again, none of the plug-in modules are on the module path. Plug-in modules live in separate directories, and will be loaded at run-time by the host application.

After starting pluginhost with the two plug-in directories as arguments, the run-time situation in Figure 6-12 emerges.

Every plugin is instantiated in its own layer.
Figure 6-12. Every plug-in is instantiated in its own layer
The first plug-in consists of a single module. Plug-in B, on the other hand, has a dependency on somelibrary. This dependency is automatically resolved when creating the configuration and layer for this plug-in. As long as somelibrary is in the same directory as plugin.b, things just work. Both plug-ins require the pluginhost.api module, which is part of the boot layer. All other interactions happen through services published by the plug-in modules and consumed by the host application.

Here’s the createPluginLayer method:

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
1
In order to identify the root modules when resolving the Configuration, we retain all modules with a name that starts with plugin.

2
The Configuration is resolved with respect to the boot layer, so that plug-in modules can read pluginhost.api.

3
All modules in the plug-in layer will be defined with the same (fresh) classloader.

Because the createPluginLayer method is called for each plug-in directory, multiple layers are created. Each layer has a single root module (respectively, plugin.a and plugin.b) that gets resolved independently. Because plugin.b requires somelibrary, a ResolutionException will be thrown if that module cannot be found. A configuration and layer will be created only if all constraints expressed in the module descriptors can be satisfied.

NOTE
We can also call resolveAndBind(finder, ModuleFinder.of(), Set.of()) (providing no root modules to resolve) instead of resolve(..., pluginRoots). Because the plugin modules expose a service, service binding causes the resolution of the plugin modules and their dependencies anyway.

There’s another advantage to loading each plug-in module into its own layer with a new classloader. By isolating plug-ins this way, it becomes possible for the plug-ins to depend on different versions of the same module. In “Versioned Modules”, we discussed how only a single module of the same name exporting the same packages can be loaded by the module system. That’s still true, but we now know this depends on the way classloaders are set up. Only when talking about the boot layer constructed from the module path will this be problematic.

Different versions of modules can be loaded simultaneously when constructing multiple layers. If, for example, Plug-in A and B would depend on different versions of somelibrary, that’s perfectly possible, as shown in Figure 6-13.

Different versions of the same module can be loaded in multiple layers.
Figure 6-13. Different versions of the same module can be loaded in multiple layers
No code changes are necessary for this to work. Because in our layers modules are loaded in a new classloader, no clashes in package definitions can occur.

This setup with a new layer for each plug-in brings many advantages. Plug-in authors are free to choose their own dependencies. At run-time, they’re guaranteed not to clash with dependencies from other plug-ins.

One question to ponder is what happens when we create another layer on top of the two plug-in layers. Layers can have multiple parents, so we can create a new layer with both plug-in layers as parents:

List<Configuration> parentConfigs = pluginLayers
  .map(ModuleLayer::configuration)
  .collect(Collectors.toList());
Configuration newconfig = Configuration.resolve(finder, parentConfigs, 1
  ModuleFinder.of(), Set.of("topmodule"));
ModuleLayer.Controller newlayer = ModuleLayer.defineModulesWithOneLoader(
  newconfig, pluginLayers, ClassLoader.getSystemClassLoader()); 2
1
This static method can take multiple configurations as parents.

2
The same holds for the layer construction with multiple parents.

Let’s say this new layer contains a single (root) module named topmodule, which requires somelibrary. Which version of somelibrary will topmodule be resolved against? The fact that the static resolve and defineModulesWithOneLoader methods take a List as a parameter for the parents should serve as a warning sign. Order matters. When the configuration is resolved, the list of parent configurations is consulted in order of the provided list. So depending on which plug-in configuration is put into the parentConfigs list first, the topmodule uses either version 1 or 2 of somelibrary.

Container Architectures
Another architecture that hinges on being able to load new code at run-time is the application container architecture. Isolation between applications in a container is another big theme. Throughout the years, Java has known many application containers, or application servers, most implementing the Java EE standard.

WARNING
Even though application servers offer some level of isolation between applications, in the end all deployed applications still run in the same JVM. True isolation (i.e., restricted memory and CPU utilization) requires a more pervasive approach.

Java EE served as one of the inspirations while designing the requirements around layers. That’s not to say Java EE is aligned with the Java module system already. At the time of writing, it is unclear which version of Java EE will first make use of modules. However, it’s not unthinkable (one might say, quite reasonable to expect) that Java EE will support modular versions of Web Archives (WARs) and Enterprise Application Archives (EARs).

To appreciate how layers enable application container architectures, we’re going to create a small application container. Before looking at the implementation, let’s review how an application container architecture differs from a plug-in-based one (see Figure 6-14).

An application container hosts multiple applications, offering common functionality for use in those applications.
Figure 6-14. An application container hosts multiple applications, offering common functionality for use in those applications
As Figure 6-14 shows, many applications can be loaded into a single container. Applications have their own internal module structure, but crucially, they use common services offered by the container. Functionality such as transaction management or security infrastructure is often provided by containers. Application developers don’t have to reinvent the wheel for every application.

In a way, the call-graph is reversed compared to Figure 6-11: applications use the implementations offered by the container, instead of the host application (container) calling into the newly loaded code. Of course, a shared API between the container and applications must exist for this to work. Java EE is an example of such an API. Another big difference with plug-in-based applications is that individual users of the system interact with the dynamically loaded applications themselves, not the container. Think of deployed applications directly exposing HTTP endpoints, web interfaces, or queues to users. Last, applications in a container can be deployed and undeployed, or replaced by a new version.

Functionally, a container architecture is quite different from a plug-in-based architecture. Implementation-wise, there’s not that much difference. Application modules can be loaded at run-time into a new layer, just like plug-ins. An API module is offered by the container with interfaces for all the services it offers, which applications can compile against. Those services can be consumed in applications through ServiceLoader at run-time.

To make things interesting, we’re going to create a container that can deploy and undeploy applications. In the plug-in example, we relied on the plug-in module to expose a service implementing a common interface. For the container, we’re going to do things differently. After deployment, we’re going to look for a class that implements ContainerApplication, which is part of the API offered by the container. The container will reflectively load the class from the application. By doing so, we cannot rely on the services mechanism to interact with the deployed application (although the deployed application does use services to consume container functionality). After instantiating the class through reflection, the startApp method (defined in Container​Application) is called. Before undeployment, stopApp is called so the application can gracefully shut down.

There are two new concepts to focus on:

How can the container ensure it is able to reflectively instantiate a class from the deployed application? We don’t require the package to be open or exported by the application developer.

How can modules be properly disposed of after undeploying an application?

During layer creation, it’s possible to manipulate the modules that are loaded into the layer and their relations. That’s exactly what a container needs to do to get access to classes in the deployed application. Somehow the container needs to ensure that the package containing the class implementing ContainerApplication is open for deep reflection.

Cleaning up modules is done at the layer level and is surprisingly straightforward. Layers can be garbage collected just like any other Java object. If the container makes sure there are no strong references anymore to the layer, or any of its modules and classes, the layer and everything associated eventually get garbage collected.

It’s time to look at some code. The container provided as an example in this chapter is a simple command-line launcher that lists applications you can deploy and undeploy. Its main class is shown in Example 6-5. If you deploy an application, it keeps running until it is undeployed (or the container is stopped). We pass information about where the deployable apps live as command-line arguments in the format out-appa/app.a/app.a.AppA: first the directory that contains the application modules, then the root module name, and last the name of the class that implements ContainerApplication, all separated by slashes.

Usually, application containers have a deployment descriptor format that would convey such information as part of the application package. There are other ways to achieve similar results, for example, by putting annotations on modules (as shown in “Annotations”) to specify some metadata. For simplicity, we derive AppDescriptor instances containing this information from the command-line arguments.

Example 6-5. Launching the application container (➥ chapter6/container)
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
The main data structures for the container are an array of application descriptors and an array keeping track of started ContainerApplication instances. After the container starts, you can type commands such as deploy 1 or undeploy 2 or exit. The numbers refer to the index in the apps and deployedApps arrays. A layer is created for each application that is deployed. In Figure 6-15, we see two applications deployed (after deploy 1 and deploy 2) in their own layers.

Two application deployed in the container.
Figure 6-15. Two applications deployed in the container
The image is quite similar to Figure 6-12, save for the fact that the provides and uses relations are inverted. In platform.api, we find the service interfaces to the container functionality, such as platform.api.tx.TransactionManager. The platform.container module contains service providers for these interfaces, and contains the Launcher class we saw earlier. Of course, the container and applications in the real world probably consist of many more modules.

Creating a layer for an application looks similar to what we saw when loading plug-ins, with a small twist:

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
1
A ModuleFinder and Configuration are created based on the AppDescriptor metadata.

2
The layer is created with the static ModuleLayer.defineModulesWithOneLoader method, which returns a ModuleLayer.Controller.

Each application is loaded into its own isolated layer, with a single fresh classloader. Even if applications contain modules with the same packages, they won’t conflict. By using the static ModuleLayer::defineModulesWithOneLoader, we get back a ModuleLayer.Controller object.

This controller is used by the calling method to open the package in the root module that contains the class implementing ContainerApplication in the deployed application:

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
1
Calling the previously defined createAppLayer method to obtain the ModuleLayer.Controller.

2
From the controller, we can get the actual layer and locate the root module that has been loaded.

3
Before using reflection to instantiate the application class in the root module, we ensure that the given package is open.

4
Now, the instantiateApp implementation can use reflection to instantiate the application class without restrictions.

5
Finally, the deployed application is started by calling startApp.

The most interesting line in the deployApp is where ModuleLayer.Controller::addOpens is invoked. It opens the package mentioned in the AppDescriptor from the application’s root module to the container module, as shown in Figure 6-16.

At layer creation the packages `app.a` and `app.b` are opened to the `platform.container` module.
Figure 6-16. At layer creation, the packages app.a and app.b are opened to the platform.container module
This qualified opens enables the container to reflect over the packages app.a and app.b. In addition to addOpens(Module source, String pkg, Module target), you can call addExports(Module source, String pkg, Module target) or addReads(Module source, Module target) on a layer controller. With addExports, a previously encapsulated package can be exported (without being opened). And, by establishing readability with addReads, the target module can access the exported packages of the source module. In all cases, source modules must come from the layer that has been constructed. Layers can, in effect, reshape the dependencies and encapsulation boundaries of modules. As always, with great power comes great responsibility.

Resolving Platform Modules in Containers
Isolation in the container architectures described so far is achieved by resolving a new application or plug-in within its own ModuleLayer. Each application or plug-in can require its own libraries, possibly even different versions. As long as ModuleFinder can locate the necessary modules for ModuleLayer, everything is fine.

But what about dependencies to platform modules from these freshly loaded applications or plug-ins? At first sight, it might seem there is no problem. ModuleLayer can resolve modules in parent layers, eventually reaching the boot layer. The boot layer contains platform modules, so all is well. Or is it? That depends on which modules were resolved into the boot layer while starting the container.

Normal module resolution rules apply: the root module that is started determines which platform modules are resolved. When the root module is the container launcher, only the dependencies of the container launcher module are taken into account. Only the (transitive) dependencies of this root module end up in the boot layer at run-time.

When a new application or plug-in is loaded after startup, these might require platform modules that were not required by the container itself. In that case, module resolution fails for the new layer. This can be prevented by using --add-modules ALL-SYSTEM when starting the container module. All platform modules are in that case resolved, even if the module that is started doesn’t depend on them. The boot layer has all possible platform modules because of the ALL-SYSTEM option. This ensures that applications or plug-ins loaded at run-time can require arbitrary platform modules.

You have seen how layers enable dynamic architectures. They allow constructing and loading new module graphs at run-time, possibly changing the openness of packages and readability relations to suit the container. Modules in different layers do not interfere, allowing for multiple versions of the same module to coexist in different layers. Services are a natural fit when interacting with newly loaded modules in layers. However, it’s also still possible to take a more traditional, reflection-based approach, as you have seen.

The ModuleLayer API is not expected to be widely used in normal application development. In some ways, the API is similar in nature to classloaders: a powerful feature, mostly wielded by frameworks to make the lives of developers easier. Existing Java module frameworks are expected to use layers as a means of interoperability. It’s up to frameworks to put the new ModuleLayer API to good use, just as they did with classloaders over the past 20 years.