# 第 4 章 服务
> Chapter 4. Services

In this chapter, you will learn how to use services, an important feature for creating modular codebases. After learning the basics of providing and consuming services, we will apply them to EasyText, making it more extensible.

Factory Pattern
In the previous chapter, you saw that encapsulation alone doesn’t get us very far when we want to create truly decoupled modules. If we still write

MyInterface i = new MyImpl();
every time we need to use an implementation class, it means the implementation class must be exported. Consequently, strong coupling still remains between the consumer and provider of the implementation: the consumer requires the provider module directly to use its exported implementation class. Changes in the implementation directly affect all consumers. As you will soon see, services are an excellent solution to this problem. But before diving into services, let’s see if we can fix this problem by using an existing pattern, building on our knowledge of the module system so far.

The factory pattern is a well-known creational design pattern that seems to address the very problem we are dealing with. Its goal is to decouple a consumer of objects from the instantiation of specific classes. Many variations on the factory pattern have emerged since it was first described in the iconic Gang of Four Design Patterns book by Gamma et al. (Addison-Wesley). Let’s try to implement a simple variation of this pattern and see how far it gets us with decoupling modules.

We will use the EasyText application again to illustrate the example, by implementing a factory for Analyzer instances. Getting an implementation for a given algorithm name is quite straightforward, as shown in Example 4-1.

Example 4-1. A factory class for Analyzer instances (➥ chapter4/easytext-factory)
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
You can retrieve a list of supported algorithms from the factory, and request an Analyzer instance for an algorithm name. Callers of the AnalyzerFactory are now oblivious to any underlying implementation classes for the analyzers.

But where do we place this factory? For one, the factory itself still needs access to multiple analysis modules with their implementation classes. Otherwise, the instantiation of the various implementation classes in getAnalyzer would not be possible. We could put the factory in the API module, but then the API module would have a compile-time dependency on all implementation modules, which is unsatisfactory. An API should not be tightly coupled to its implementations.

Let’s put the factory in its own module for now, as shown in Figure 4-1.

Factory
Figure 4-1. The factory module decouples the frontends from the analysis implementation modules. There is no requires relation from the frontend modules to the analysis implementation modules.
Now, the frontend modules know about only the API and the factory:

module easytext.cli {
   requires easytext.analysis.api;
   requires easytext.analysis.factory;
}
Getting an Analyzer instance becomes trivial:

Analyzer analyzer = AnalyzerFactory.getAnalyzer("Flesch-Kincaid");
Did we gain anything with this factory approach, besides increased complexity?

On the one hand, yes, the frontend modules are now blissfully unaware of the analysis modules and implementation classes. There is no direct requires relation anymore between the consumer and providers of analyses. Frontend modules can be compiled independently from analysis implementation modules. When the factory offers additional analyses, the frontends will happily use them without any modification. (Remember, with AnalyzerFactory::getSupportedAnalyses, they can discover algorithm names to request instances.)

On the other hand, the same tight coupling issues are still present at the factory module level and below. Whenever a new analysis module comes along, the factory needs to get a dependency on it and expand the getAnalyzer implementation. And the analysis modules still need to export their implementation classes for the factory to use. They could do so through a qualified export (as discussed in “Qualified Exports”) toward the factory module to limit the scope of exposure. But that presumes the analysis modules know about the factory module, which is another form of unwanted coupling.

So the factory pattern provides only a partial solution. We are running into a fundamental limitation of what you can do with modules through requires and exports relations. Programming to interfaces is all well and good, but we have to sacrifice encapsulation to create instances. Fortunately, there is a solution in the Java module system. In the next section, we’ll explore how services provide a way out of this tough spot.

DEPENDENCY INJECTION
Many Java applications use a dependency injection (DI) framework to solve the issue of programming to interfaces without tight coupling to implementation classes. A DI framework takes care of creating implementation instances based on metadata such as annotations or (more traditionally) XML descriptors. The DI framework then injects instances into code depending on the defined interfaces. This principle is more generally known as Inversion of Control (IoC) because the framework is in control of instantiating classes rather than the application code itself.

DI is an excellent way to decouple code, but there are several caveats in the context of modules. This chapter focuses on the module system’s solution for decoupling in the form of services. Services provide IoC through other means than dependency injection. Later, in “Dependency Injection”, you will see how to use DI frameworks in combination with modules for another way to achieve the same level of decoupling.

Services for Implementation Hiding
We tried hiding the implementation classes by using the factory pattern and succeeded only partially. The main problem is that the factory still has to know about all available implementations at compile-time, and the implementation classes must be exported. A solution similar to traditional classpath scanning to discover implementations is not going to solve this, because this would still require readability to all implementation classes in all modules. It still wouldn’t be possible to extend the application with another implementation (new algorithms in the case of EasyText) without changing code and recompiling. This doesn’t sound like seamless extensibility at all!

The decoupling story can be improved a lot by the services mechanism in the Java module system. Using services, we can truly just share public interfaces, and strongly encapsulate implementation code in packages that are not exported. Keep in mind, using services in the module system is completely optional, unlike strong encapsulation (with explicit exports) and explicit dependencies (with requires). You don’t have to use services, but they offer a compelling way of decoupling modules.

Services are expressed both in module descriptors and in code by using the ServiceLoader API. In that sense, using services is intrusive: you need to design your application to use them. As discussed in “Dependency Injection”, there are alternative ways to achieve inversion of control besides using services. In the remainder of this chapter, you will learn how services bring better decoupling and extensibility.

We will refactor the EasyText application to start using services. Our goal is to have several modules provide an analysis implementation. The frontend modules can consume those analysis implementations without knowing the provider modules at compile-time.

Providing Services
Exposing service implementations to another module without exporting implementation classes is not possible without special support from the module system. The Java module system allows for a declarative description of providing and consuming services in module-info.java.

In the EasyText code, we already defined the Analyzer interface, which will be our service interface type. The interface is exported by the easytext.analysis.api module, which is strictly an API-only module.

package javamodularity.easytext.analysis.api;

import java.util.List;

public interface Analyzer {

   String getName();

   double analyze(List<List<String>> text);

}
Typically, the service type is an interface, as is the case here. However, it could also be an abstract or even concrete class; there is no inherent technical limitation. Also, the Analyzer type is meant to be used by service consumers directly. It’s also possible to expose a service type that acts like a factory or proxy. If, for example, Analyzer instances would be expensive to instantiate, or extra steps or arguments are required for initialization, the service type could be more akin to the AnalyzerFactory. This approach allows the consumer to be more in control of the instantiation.

Now let’s refactor our first new analyzer implementation, the Coleman-Liau algorithm (provided by the easytext.algorithm.coleman module) to a service provider. This requires only a change to module-info.java, as shown in Example 4-2.

Example 4-2. Module descriptor providing an Analyzer service (➥ chapter4/easytext-services)
module easytext.analysis.coleman {

   requires easytext.analysis.api;

   provides javamodularity.easytext.analysis.api.Analyzer
       with javamodularity.easytext.analysis.coleman.ColemanAnalyzer;

}
The provides with syntax declares that this module provides an implementation of the Analyzer interface with the ColemanAnalyzer as an implementation class. Both the service type (after provides) and the implementation class (after with) must be fully qualified type names. Most important, the package containing the ColemanAnalyzer implementation class is not exported from this provider module.

This construct works only when the module declaring the provides has access to both the service type and the implementation class. Usually this means that an interface, Analyzer in this example, is either part of the module or is exported by another module that is required. The implementation class is typically part of the provider module, in an encapsulated (nonexported) package.

When you use nonexistent or inaccessible types in the provides clause, the module descriptor won’t compile, and a compiler error is generated. The implementation class used in the with part of the declaration is normally not exported. After all, the whole point of services is to hide implementation details.

No code changes are necessary to the service type or implementation class to provide it as a service. Besides this module-info.java declaration, nothing needs to be done. Service implementations are plain Java classes. There are no special annotations to use, no APIs to implement.

Services allow a module to provide implementations to other modules without exporting the concrete implementation class. The module system has special privileges to reach into the provider module to instantiate the nonexported implementation class on behalf of the consumer. This means consumers of the service can use instances of this implementation class, without having access to it directly. Also, a service consumer doesn’t know which module provided an implementation, nor does it need to. Because the only shared type between provider and consumer is the service type (most often an interface), there is true decoupling.

Now that we’re done providing our first service, we can repeat this process for the other Analyzer implementations, and we’re halfway done. Again, note that these service-providing modules don’t export any packages. Having a module without exports may seem a bit counterintuitive at first. Nevertheless, these analysis implementation modules contribute useful functionality through the services mechanism at run-time, encapsulating their implementation details at compile-time.

The other half of the refactoring is consuming the services. Let’s refactor the CLI module to use the Analyzer services.

Consuming Services
Providing services is useful if other modules can consume them. Consuming a service in the Java module system requires two steps. The first step is adding a uses clause to module-info.java in the CLI module:

module easytext.cli {
   requires easytext.analysis.api;

   uses javamodularity.easytext.analysis.api.Analyzer;
}
The uses clause instructs the ServiceLoader, which you will see in a moment, that this module wants to use implementations of Analyzer. The ServiceLoader then makes Analyzer instances available to the module.

The uses clause does not call for an Analyzer implementation to be available during compile-time. After all, a service implementation could be provided by a module that we don’t have on the module path at compile-time. Services provide extensibility exactly because providers and consumers are bound only at run-time. Compilation will not fail when no service providers are found. The service type (Analyzer), on the other hand, must be accessible at compile-time—hence the requires easytext.analysis.api clause in the module descriptor.

A uses clause also doesn’t guarantee there will be providers during run-time. The application will start successfully without any providers of services. This means there can be zero or more providers available at run-time, and our code has to deal with this.

Now that the module has declared that it wants to use Analyzer implementations, we can start writing code that uses the service. Consuming services happens through the ServiceLoader API. The ServiceLoader API itself has been around since Java 6 already. Although it is widely used in the JDK, few Java developers know or use ServiceLoader. “ServiceLoader Before Java 9” provides more historical background.

The ServiceLoader API is repurposed in the Java module system to work with modules, and is an important programming construct when working with the Java module system. Let’s look at an example; see Example 4-3.

Example 4-3. Main.java
Iterable<Analyzer> analyzers = ServiceLoader.load(Analyzer.class); 1

for (Analyzer analyzer: analyzers) { 2
   System.out.println(analyzer.getName() + ": " + analyzer.analyze(sentences));
}
1
Initialize a ServiceLoader for services of type Analyzer.

2
Iterate over the instances and invoke the analyze method.

The ServiceLoader::load method returns a ServiceLoader instance that also conveniently implements Iterable. When you iterate over it as in the example, instances are created for all the provider types that have been discovered for the requested Analyzer interface. Note that we get only the actual instances here, with no additional information on which modules provided them.

After iterating over the services, we can use them like any other Java object. In fact, they are plain Java objects, except they are instantiated by ServiceLoader for us. Being just normal Java instances, there is zero overhead when invoking a service. Invoking a method on a service is just a direct method call; there are no proxies or other indirections that decrease performance.

With these changes, we have refactored our EasyText code from a partially decoupled factory structure to a fully modular and extensible setup as shown in Figure 4-2.

Structure of Easytext using ServiceLoader for extensibilitys
Figure 4-2. Structure of EasyText using ServiceLoader for extensibility
The code is fully decoupled because the CLI module doesn’t need to know anything about modules providing Analyzer implementations. The application is easily extensible because we can add a new Analyzer implementation by simply adding a new provider module to the module path. Any services provided by these additional modules are picked up automatically through the ServiceLoader service discovery. No code changes or recompilation are necessary. Arguably the best part is that the code is clean. Programming with services is just as simple as writing plain Java code (that’s because it is plain Java code), but the impact on architecture and design is quite positive.

SERVICELOADER BEFORE JAVA 9
ServiceLoader has been around since Java 6. It was designed to make Java more pluggable, and is used in the JDK in several places. It never received widespread use in application development, although several frameworks and libraries rely on the old ServiceLoader besides the JDK itself.

In principle, ServiceLoader had the same goal as services in the Java module system, but the mechanics were different, and there was no true strong encapsulation possible before modules. To register a provider, a file following a specific naming scheme must be created in the META-INF folder of a JAR file. For example, to provide an Analyzer implementation, a file named META-INF/services/javamodularity.easytext.analysis.api.Analyzer must be created. The contents of the file should be a single line representing the fully qualified name of the implementation class—for example, javamodularity.easytext.analysis.coleman.ColemanAnalyzer. Because these files are just text files, it is easy to make mistakes that the compiler won’t catch.

With the Java module system, it’s also possible to consume services provided the “old” way, as long as the service type is accessible to the consuming module.

You have seen that services provide an easy way to achieve decoupling. Think of services as the cornerstone of modular development. Although strong mechanisms for defining module boundaries are the first step toward modular design, services are required to create and use strictly decoupled modules.

Service Life Cycle
If the ServiceLoader is responsible for creating instances of provided services, it’s important to know how this works exactly. In Example 4-3, the iteration caused the Analyzer implementation classes to be instantiated. ServiceLoader works lazily, meaning the ServiceLoader::load call doesn’t immediately instantiate all known provider implementation classes.

A new ServiceLoader is instantiated every time you call ServiceLoader::load. Such a new ServiceLoader in turn reinstantiates provider classes when they are requested. Requesting services from an existing ServiceLoader instance returns cached instances of provider classes.

This is demonstrated by the following code:

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
1
Iterating over first, ServiceLoader instantiates Analyzer implementations.

2
A new ServiceLoader, second, will instantiate its own, fresh, Analyzer implementations. It returns different instances than first.

3
The originally instantiated services are returned from first when iterating again, as they are cached by the first ServiceLoader instance.

4
After reload, the original first ServiceLoader provides fresh instances.

This code outputs something like the following (actual hashCodes will vary, of course):

Using the first analyzers
1379435698
Using the second analyzers
876563773
Using the first analyzers again, hashCode is the same
1379435698
Reloading the first analyzers, hashCode is different
87765719
Because every invocation of ServiceLoader::load leads to new service instances, different modules using the same service will all have their own instance. This is something to remember when working with services that contain state. The state is, without other provisions, not shared between usages across different ServiceLoaders for the same service type. There is no singleton service instance, unlike what is typically the case in dependency injection frameworks.

Service Provider Methods
Service instances can be created in two ways. Either the service implementation class must have a public no-arg constructor, or a static provider method can be used. It’s not always desirable for a service implementation class to have a public no-arg constructor. In cases where more information needs to be passed to the constructor, a static provider method is the better option. Or, you may want to expose an existing class without a no-arg constructor as a service.

A provider method is a public static no-arg method called provider, where the return type is the service type. It must return a service instance of the correct type (or a subtype). How the service is instantiated in this method is completely up to the provider implementation. Possibly a singleton is cached and returned, or it just instantiates a new service instance for each call.

When using the provider method approach, the provides .. with clause refers to the class containing the provider method after with. This can very well be the service implementation class itself, but it can also be another class. A class appearing after with must have either a provider method or a public no-arg constructor. If there is no static provider method, the class is assumed to be the service implementation itself and must have a public no-arg constructor. The compiler will complain when this is not the case.

Let’s look at a provider method example (Example 4-4). We’ll use another Analyzer implementation for this, just to highlight the use of a provider method.

Example 4-4. ExampleProviderMethod.java (➥ chapter4/providers/provider.method.example)
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
The Analyzer implementation is fairly useless, but it does show the usage of a provider method. The module-info.java for this example would be exactly the same as we have seen so far; the Java module system will figure out the right way to instantiate the class. In this example, the provider method is part of the implementation class. Alternatively, we can place the provider method in another class, which then serves as a factory for the service implementation. Example 4-5 shows this approach.

Example 4-5. ExampleProviderFactory.java (➥ chapter4/providers/provider.factory.example)
package javamodularity.providers.factory;

public class ExampleProviderFactory {
  public static ExampleProvider provider() {
    return new ExampleProvider("Analyzer created by factory");
  }
}
Now we do have to change module-info.java to reflect this change. The provides .. with must now point to the class containing the static provider method as shown in Example 4-6.

Example 4-6. module-info.java (➥ chapter4/providers/provider.factory.example)
module provider.factory.example {
    requires easytext.analysis.api;

    provides javamodularity.easytext.analysis.api.Analyzer
        with javamodularity.providers.factory.ExampleProviderFactory;
}
TIP
ServiceLoader can instantiate a service only if the provider class is public. Only the provider class itself needs to be public; our second example shows that the implementation can be package-private as long as the provider class is public.

Note that in all cases the exposed service type Analyzer remains unchanged. From the perspective of the consumer, it makes no difference how the service is instantiated. A static provider method offers more flexibility on the provider side. In many cases, a public no-arg constructor on the service implementation class suffices.

Services in the module system don’t offer a shutdown or service de-registration mechanism. The death of a service is implicit, through garbage collection. Garbage collection behaves the same for service instances as for any other objects in Java. Once there are no hard references to the object anymore, it can be garbage collected.

Factory Pattern Revisited
Consumer modules can obtain services through the ServiceLoader API. You can employ a useful pattern to avoid the use of this API in consumers, if desired. Instead, you can offer an API to consumers similar to the factory example in the beginning of this chapter. It’s based on the ability to have static methods in interfaces as of Java 8.

The service type itself is extended with a static method (factory method) that does the ServiceLoader lookup, as shown in Example 4-7.

Example 4-7. Provide a factory method on the service interface (➥ chapter4/easytext-services-factory)
public interface Analyzer {

   String getName();

   double analyze(List<List<String>> text);

   static Iterable<Analyzer> getAnalyzers() {
     return ServiceLoader.load(Analyzer.class); 1
   }

}
1
Lookup is now done inside the service type itself.

Because the ServiceLoader lookup is done in Analyzer in the API module, its module descriptor must express the uses constraint:

module easytext.analysis.api {
   exports javamodularity.easytext.analysis.api;

   uses javamodularity.easytext.analysis.api.Analyzer;
}
Now, the API module both exports the interface and uses implementations of the Analyzer interface. Consumer modules that want to obtain Analyzer implementations no longer need to use ServiceLoader (though they still can, of course). Instead, all a consumer module needs to do is require the API module and call Analyzer::getAnalyzers. No need for a uses constraint or the ServiceLoader API anymore from the perspective of the consumer.

Through this mechanism, you can use the power of services unobtrusively. Users of an API are not forced to know about services or ServiceLoader but still get the benefits of decoupling and extensibility.

Default Service Implementations
So far, we’ve worked from the assumption that there is an API module, and there are several distinct provider modules implementing this API. That’s not unreasonable, but it’s far from the only way to set things up. It’s perfectly possible to put an implementation into the same module exporting the service type. When a service type has an obvious default implementation, why not provide it from the same module directly?

You see this pattern a lot in the way the JDK itself uses services. Even though it is possible to provide your own implementations for javax.sound.sampled.spi.AudioFileWriter or javax.print.PrintServiceLookup, most of the time the default implementations provided by the java.desktop module are adequate. These service types are exported from java.desktop, and at the same time default implementations are provided.

In fact, java.desktop itself even has uses constraints for those service types. This shows how a module can play the role of API owner, service provider, and consumer at the same time.

Bundling a default service implementation with the service type guarantees that at least one implementation is always available. In that case, no defensive coding is necessary on the consumer’s part. Some service dependencies are intended to be optional. A default implementation in the same module as the service type precludes this scenario. Then, having a separate API module is necessary. In “Implementing Optional Dependencies with Services”, this pattern is explored in more detail.

Service Implementation Selection
When there are multiple providers, you don’t necessarily want to use them all. Sometimes you want to filter and select an implementation based on certain characteristics.

NOTE
It’s always the consumer deciding which service to use based on properties of the providers. Because providers should be completely unaware of each other, there is no way to favor a certain implementation from a provider’s perspective. What would happen if, for example, two providers designate themselves as default or best implementation? The logic to select the right service(s) is application-dependent and belongs to the consumer.

You have seen that the ServiceLoader API itself is fairly limited. Until now, we have iterated only over all existing service implementations. What if we have multiple providers but are interested in only “the best” implementation? The Java module system can’t possibly know what the best implementation is for your needs. Each domain has its own requirements in that regard. Therefore, it’s up to you to equip your service type with methods to discover the capabilities of a service and make decisions based on these methods. This need not be complicated, and usually comes down to adding self-describing methods to a service interface.

For example, the Analyzer service interface offers a getName method. ServiceLoader doesn’t know or care about this method, but we can use it in consumer modules to identify an implementation. Besides selecting an algorithm by name, you can also think of describing different characteristics, for example, with getAccuracy or getCost methods. This way, a consumer of the Analyzer service can make a well-informed choice between implementations. There’s no explicit support from the ServiceLoader API necessary: it all boils down to designing self-describing interfaces.

Service Type Inspection and Lazy Instantiation
In some scenarios, the mechanism described previously is still not sufficient. What if there’s no method on the service interface to distinguish the right implementation? Or instantiation of the services is expensive? We would incur the cost of initialization for all service implementations just to find the right one, using a ServiceLoader iteration. In most scenarios, this is not an issue, but a solution exists for problematic cases.

With Java 9, ServiceLoader is enhanced to support service implementation type inspection before instantiation. Besides iterating over all provided instances as we’ve done so far, it’s also possible to inspect a stream of ServiceLoader.Provider descriptions. The ServiceLoader.Provider class makes it possible to inspect a service provider before requesting an instance. The stream method on ServiceLoader returns a stream of ServiceLoader.Provider objects to inspect.

Let’s look at an example based on EasyText again.

First we introduce our own annotation that can be used to select the right service implementation in Example 4-8. Such an annotation can be part of the API module that is shared between providers and consumers. The example annotation describes whether an Analyzer is fast.

Example 4-8. Define an annotation to annotate the service implementation class (➥ chapter4/easytext-filtering)
package javamodularity.easytext.analysis.api;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Fast {

  public boolean value() default true;

}
We can now use this annotation to add metadata to a service implementation. Here, we add it to an example Analyzer:

@Fast
public class ReallyFastAnalyzer implements Analyzer {
  // Implementation of the analyzer
}
Now we just need some code to filter the Analyzers:

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
Through the type method on Provider, we get access to the java.lang.Class representation of the service implementation, which we pass to the isFast method for filtering.

ENCAPSULATION VERSUS JAVA.LANG.CLASS
It might seem strange that we can get a java.lang.Class representation of the implementation class. Doesn’t that violate strong encapsulation? The package the class comes from isn’t even exported!

This is a clear case of “the map isn’t the territory”—even though Class describes the implementation class, you can’t actually do anything with it. When you try to get an instance through reflection (using provider.type().newInstance()), you will get an IllegalAccessError if the class is indeed not exported. So, having a Class object doesn’t necessarily mean you can instantiate it in your module. All access checks of the module system still apply.

The isFast method checks for the presence of our @Fast annotation, and checks the value explicitly for true (which is the default value). Analyzer implementations that are not annotated as being fast are ignored, but services annotated @Fast or @Fast(true) are instantiated and invoked. If you remove the filter from the stream pipeline, all Analyzers will be invoked indiscriminately.

The examples in this chapter demonstrate that although the ServiceLoader API is basic, the services mechanism is powerful. Services are an important construct in the Java module system when modularizing code.

NOTE
Using services as a means to improve decoupling is not new. For example, OSGi offers a services-based programming model as well. To be successful in creating truly modular code in OSGi, you must use services. So we’re building on a proven concept.

Module Resolution with Service Binding
Remember learning in “Module Resolution and the Module Path” that modules are resolved based on requires clauses in module descriptors? By recursively following all requires relations starting from a root module, the set of resolved modules is built from modules on the module path. During this process, missing modules are detected, giving the benefit of reliable configuration. The application won’t start if a required module is missing.

Service provides and uses clauses add another dimension to the resolution process. Whereas requires clauses indicate strict compile-time relations between modules, service binding happens at run-time. Because service provider and consumer modules both declaratively state their intentions in module descriptors, this information can be used during the module resolution process as well.

In theory, an application can start without any of its services being bound at run-time. Calling ServiceLoader::load won’t result in any instances. That’s hardly useful, so the module system locates service provider modules on the module path at startup in addition to modules that are required.

When a module with a uses clause is resolved, the module system locates all provider modules for the given service type on the module path and adds them to the resolution process. These provider modules, and their dependencies, become part of the run-time module graph.

The implications of this extension to module resolution become clearer by looking at an example. In Figure 4-3 we look at our EasyText example again from the perspective of module resolution.

Service binding influences module resolution.
Figure 4-3. Service binding influences module resolution
We assume there are five modules on the module path: cli (the root module), api, kincaid, coleman, and an imaginary module syllablecounter. Module resolution starts with cli. It has a requires relation to api, so this module is added to the set of resolved modules. So far, nothing new.

However, cli also has a uses clause for Analyzer. There are two provider modules on the module path providing implementations of this interface. Hence the provider modules kincaid and coleman are added to the set of resolved modules. Module resolution stops for cli because it doesn’t have any other requires or uses clauses.

For kincaid, there’s nothing left to add to the resolved modules. The api module that it requires has already been resolved. With coleman, things are more interesting. Service binding caused coleman to be resolved. In this example, the coleman module requires another module: syllablecounter. Therefore, syllablecounter is resolved as well, and both modules are added to the run-time module graph.

If syllablecounter itself would have had requires (or even uses!) clauses, these would be subject to module resolution as well. Conversely, if syllablecounter is not found on the module path, resolution fails, and the application won’t start. Even though cli, the consumer module, doesn’t have any static knowledge of the coleman provider module, it is still resolved with all its dependencies through service binding.

There is no way for a consumer to specify that it needs at least one implementation. When no service provider modules are found, the application starts just as well. Code using ServiceLoader needs to account for this possibility. You’ve already seen that many JDK service types have default implementations. When there’s a default implementation in the module exposing the service type, you’re guaranteed to always have at least one service implementation available.

In the example, module resolution also succeeds when coleman isn’t on the module path. At run-time, the ServiceLoader::load call finds the implementation only from kincaid in that case. However, as we’ve explained, if coleman is on the module path but syllablecounter isn’t, the application won’t start because of module-resolution failure. Silently ignoring this problem would be possible for the module system, but runs counter to the mantra of reliable configuration based on module descriptors.

Services and Linking
In “Linking Modules”, you learned how to use jlink to create custom runtime images. We can create an image for the EasyText implementation with services as well. Based on what you’ve learned in the previous chapter, we can come up with the following jlink command:

$ jlink --module-path mods/:$JAVA_HOME/jmods --add-modules easytext.cli \
        --output image
jlink creates a directory image, containing a bin directory. We can inspect the modules included in the image by using the following command:

$ image/bin/java --list-modules

java.base@9
easytext.analysis.api
easytext.cli
The api and cli modules are part of the image as expected, but what about the two analysis provider modules? If we run the application this way, it starts correctly because service providers are optional. But it is pretty useless without any analyzers.

jlink performs module resolution starting from the root module easytext.cli. All resolved modules are included in the resulting image. However, the resolution process differs from the resolution done by the module system at startup, which we discussed in the previous section. No service binding is done by jlink during module resolution. That means service providers are not automatically included in the image based on uses clauses.

Although this will certainly cause unexpected results for users who are not aware of this, it is a deliberate choice. Services are often used for extensibility. The EasyText application is a good example of this; new types of algorithms can be added by adding new service provider modules to the module path. Services used this way are not necessarily required to run the application. Which service providers you want to combine is an application-dependent concern. At build-time, there is no dependency to the service providers, and at link-time, it’s really up to the creator of the desired image to choose which service providers should be available.

NOTE
A more down-to-earth reason to not do automatic service binding in jlink is that java.base has an enormous number of uses clauses. The providers for all these service types are in various other platform modules. Binding all these services by default would result in a much larger minimum image size. Without automatic service binding in jlink, you can create an image containing just java.base and application modules, as in our example. In general, automatic service binding can lead to unexpectedly large module graphs.

Let’s try to create a runtime image for the EasyText application, configured to run from the command line. To include analyzers, we use the --add-modules argument when executing jlink for each provider module we want to add:

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
This looks better, but we will still find a problem when starting the application:

$ image/bin/java -m easytext.cli input.txt
The application exits with an exception: java.lang.IllegalStateException: SyllableCounter not found. The kincaid module uses another service of type SyllableCounter. This is a case where a service provider uses another service to implement its functionality. We already know that jlink doesn’t automatically include service providers, so the module containing the SyllableCounter example wasn’t included either. We use --add-modules once more to finally get a fully functional image:

$ jlink --module-path mods/:$JAVA_HOME/jmods \
        --add-modules easytext.cli           \
        --add-modules easytext.analysis.coleman \
        --add-modules easytext.analysis.kincaid \
        --add-modules easytext.analysis.naivesyllablecounter \
        --output image
The fact that jlink doesn’t include service providers by default requires some extra work at link-time, especially when services use other services transitively. In return, it does give a lot of flexibility for fine-tuning the contents of a runtime image. Different images can serve different types of users, just by reconfiguring which service providers are included. In “Finding the Right Service Provider Modules”, we’ll see that jlink offers additional options to discover and link relevant service provider modules.

The previous chapters covered the basics of the Java module system. Modularity is a lot about design and architecture, and this is where it really gets interesting. In the next chapter, we are going to look at patterns that improve the maintainability, flexibility, and reusability of systems built by using modules.