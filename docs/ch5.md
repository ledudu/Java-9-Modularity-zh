# 第 5 章 模块化模式

> Chapter 5. Modularity Patterns

Mastering a new technology or language feature in some ways feels like acquiring a new superpower. You immediately see the potential, and you want to change the world by applying it everywhere. If anything, superheroes from comics have shown us the world isn’t this black-and-white—even when you believe you have a superpower. Rarely is good and bad use of new powers immediately apparent; with great superpower comes great responsibility.

Learning the Java module system is no different. Knowing the module system does nothing for you if you do not also know about modular design principles to wield its power. Modularity is more than just an implementation concern, eased by the introduction of new language features. It’s a matter of design and architecture as well. Applying modular design also is a long-term investment. With it, you can hedge against changes in requirements, environments, teams, and other unforeseen events.

In this chapter, we discuss common patterns to improve the maintainability, flexibility, and reusability of systems built using modules. It’s essential to remember that many of these patterns and design practices are technology agnostic. Although there is code in this chapter, it serves to illustrate these sometimes abstract patterns in the context of the Java module system. The focus is on effectively modularizing a system by applying established patterns of modularity.

If these patterns seem obvious to you, congratulations! You’ve been doing modular development all along. Still, the Java module system offers more support for writing modular code than ever before in the language. And not just for you, but across your whole team and even the Java ecosystem at large. If, on the other hand, these patterns are new to you, congratulations as well. By learning and applying these patterns and design practices, your applications become easier to maintain and extend.

This chapter discusses basic modularity patterns that you will encounter frequently in application development. We start with some general module design guidelines, followed by more concrete module patterns. In Chapter 6, we discuss more advanced patterns that may appeal only to developers of extremely flexible applications, such as generic application containers or plug-in-based systems.

Determining Module Boundaries
What makes a good module? It may surprise you that this question is in fact age-old. Dividing systems into small, manageable modules has been identified as a winning strategy since the inception of our profession. Take, for example, this quote from a 1972 paper:

The effectiveness of a “modularization” is dependent upon the criteria used in dividing the system into modules.

D.L. Parnas, “On the Criteria To Be Used in Decomposing Systems into Modules”

One of the main points this paper makes is that modularization starts before any code is written. Module boundaries should be derived from the system’s design and intent. In the following sidebar, you can read how Parnas approached this challenge. Like the quote says, the criteria used to draw boundaries determine the success of a modularization effort. So what are those criteria? As always, it depends.

PARNAS PARTITIONING
Based on his 1972 paper, D.L. Parnas devised an approach to modularization called Parnas partitioning. It’s always a good idea to take possible change into account when thinking about module boundaries. With Parnas partitioning, you construct a hiding assumption list. The list contains areas from the system that are expected to change or have contentious design decisions, along with their estimated probability of changing later. Functionality is divided into modules using this list to minimize the impact of change. High-probability items on the hiding assumption list are prime candidates to be encapsulated in a module. Creating such a list is not a hard science. It’s something you can do together with technical and nontechnical stakeholders.

Read more about constructing a hiding assumption list in this primer on Parnas partitioning.

There’s a big difference between creating a modular library designed for reuse versus building a large enterprise application where understandability and maintainability are chief concerns. In general, we can distinguish several axes you can align with when designing modules:

Comprehension
Modules and their relations reflect the overall structure and intent of the system. Whenever someone looks at the codebase without prior knowledge, the high-level structure and functionality is immediately apparent. The module structure guides developers through the system when looking for specific functionality.

Changeability
Requirements change constantly. By using modules to encapsulate decisions that are likely to change, the impact of change decreases. Two systems with similar functionality but different anticipated areas of change may have different optimal module boundaries.

Reuse
Modules are an ideal unit of reuse. To increase reusability, modules should be narrowly focused and as independent as possible. Reusable modules can be composed in many ways across different applications.

Teamwork
Sometimes you want to use module boundaries to clearly divide work across multiple teams. Rather than using technical considerations, you align module boundaries with organizational boundaries.

Some tension exists between these axes. There’s no one-size-fits-all answer here. By designing for change in a particular area, abstractions can be introduced that decrease comprehension at first sight. Say, for example, a step from a larger process is put in a separate module because it’s expected to change more frequently than the surrounding parts. Logically, the step still belongs with the main process, but it’s now in a separate module. This makes the whole a bit less comprehensible in a way, while changeability increases.

Another important trade-off comes from the tension between reuse and use. Generic components or reusable libraries may become complex because they need to be adaptable to different usage scenarios. Nonreusable application components, not directly burdened by the needs of many consumers, can be more straightforward and specific.

When designing for reuse, you have two main drivers:

Adhering to the Unix philosophy of doing only one thing and doing it well.

Minimizing the number of dependencies the module has itself. Otherwise, you’re burdening all reusing consumers with those transitive dependencies.

These drivers do not necessarily work for application modules, where ease of use, comprehension, and speed of development are more important. If you can use a library module to speed up the development of your application module, and make the application module simpler in the process, you do so. On the other hand, if you want to create a reusable library module, your future consumers will thank you for not bringing in a multitude of transitive dependencies. There is no right or wrong here, only deliberate trade-offs. The module system makes these choices explicit.

Typically, reusable modules will be smaller and more focused than single-use application modules. On the other hand, orchestrating many small modules brings about its own complexity. When reuse is of no immediate concern, having larger modules can make sense.

In the next section, we’ll explore the notion of module size.

Lean Modules
How big should a module be? Although it sounds like a natural question, it’s like asking how big your application should be. Just big enough to serve its purpose, but no bigger—which isn’t terribly useful advice.

There are more important concerns to address when thinking about module design than just size. When thinking about the measure of a module, take into account two metrics: the size of its public surface area and of its internal implementation.

Simplifying and minimizing the publicly exported part of your module is beneficial for two reasons. First, a simple and small API is easier to use than a large and convoluted one. Users of the module are not burdened with unnecessary details. The whole point of modularity is to break down concerns into manageable chunks.

Second, minimizing the public part of a module reduces the liability of the module’s maintainer. You don’t have to support what others can’t access, leaving the module authors free to change internal details without grave consequences. The less is revealed, the less is relied upon by consumers, and the more stable an API can be. Whatever is put in the exported part of a module becomes a contract between the module producer and its consumers. This is not something to be taken lightly if you want to evolve the module in a backward-compatible manner (which should be the default position). Ergo, minimizing the public surface area of a module is recommended.

That leaves the other metric: the measure of the nonexported part of a module. Here it makes less sense to talk about raw size as with the public part of a module. Again, a module’s private implementation should be as big as it needs to be to fulfill its API contract. More interesting is the question: how many other modules does it need to accomplish its goals? A lean module is as independent as possible, avoiding dependencies on other modules where possible. Nothing is more discouraging than to see a load of (transitive) dependencies being added to your system because you want to use a specific module. As discussed previously, when wide reuse is not a main concern for the module, this becomes less of an issue.

You may have noticed parallels between developing lean modules and the prevailing best practices around reusable microservices. Strive to be as small as possible. Have a well-defined contract to the outside world while at the same time being as independent as possible. Indeed, these are similar concerns at different levels in your system architecture. Modules with their public APIs and module descriptors facilitate intraprocess reuse and composition. Microservices function at a higher level in the architecture, through (networked) interprocess communication. Modules and microservices are therefore complementary concepts. A microservice may very well be implemented using Java modules on the inside. One important difference is that with modules and their descriptors, we can explicitly describe not only what they offer (export), but also what they require. Modules in the Java module system can therefore be resolved and linked reliably, something that cannot be said of most microservices environments.

API Modules
So far, you’ve seen that being deliberate about the API of a module is a must. API design becomes a first-class citizen when building proper modules. It may sound like this is a concern for only library authors, but that’s not at all the case. When you modularize an application, crafting the public API of application modules is every bit as important. The API of a module is by definition the sum of its exported packages. Applications consisting of modules exporting everything they contain are typically a warning sign, leaving you no better off than before you used modules. A modular application hides implementation details from other parts of the application just as a good library hides its internals from applications. Whenever your module is used in different parts of your application, or by different teams in your organization, having a well-defined and stable API is paramount.

What Should Be in an API Module?
If you expect to have only a single implementation of an interface, you can combine the API and implementation in a single module. In that case, the exported part is visible to the consumers of the module, and your implementation packages are concealed. The implementation can be exposed as a service through the ServiceLoader. Even if you expect multiple implementations, in some cases it makes sense to bundle a default implementation into the API module. For the remainder of this discussion, we assume the API module does not contain such a default implementation, but stands on its own. In “API Module with a Default Implementation”, we discuss some caveats when a module contains both an exported API and an implementation of that API.

We already established that the public part of a module should be as lean as possible. But what do you export from an API module? Interfaces have been mentioned a lot, as they form the backbone of most APIs. Of course, there’s more.

An interface contains methods with parameters and result types. In its most basic form, the interface stands on its own, using only types from java.base:

public interface SimpleTextRepository {
  String findText(String id);
}
Having such a simple interface is rare in practice. In an application such as EasyText, you’d expect a repository implementation to return a domain-specific type from getText (see Example 5-1).

Example 5-1. An interface with nonprimitive types
public interface TextRepository {
  Text findText(String id);
}
In this case, the Text class is placed in the API module as well. It could be a typical JavaBean-style class, describing the data the caller can expect to get out of a service. As such, it’s part of the public API. Exceptions declared in methods on an interface are part of the API, too. They should be colocated in the API module and exported alongside the methods that (declare to) throw them.

Interfaces are the primary means of achieving decoupling between the API provider and consumer. Of course, API modules can contain much more than interfaces—for example, (abstract) base classes you expect the API consumer to extend, enums, annotations, and so on. Whatever you put into an API module, remember: minimalism goes a long way.

When another module requires the API module, you want it to be usable as is. You don’t want to require additional modules just to get readability of all the types used in the interface. Making the API module fully self-contained, as discussed so far, is one way to do this. Still, that’s not always feasible. Often, an interface method returns or accepts a parameter of a type that resides in another module. To streamline this scenario, the module system offers implied readability.

Implied Readability
“Implied Readability” provided an introduction to implied readability based on platform modules. Let’s look at an example from the EasyText domain to see how implied readability helps create self-contained and fully self-describing API modules. You’re going to look at an example consisting of three modules, shown in Figure 5-1.

Three modules, without implied readability yet.
Figure 5-1. Three modules, without implied readability yet
In this example, the TextRepository interface in Example 5-1 lives in the module easytext.repository.api, and the Text class that its findText method returns lives in another module, easytext.domain.api. The module-info.java of easytext.client (which calls the TextRepository) starts out like Example 5-2.

Example 5-2. Module descriptor of a module using TextRepository (➥ chapter5/implied_readability)
module easytext.client {
  requires easytext.repository.api; 1

  uses easytext.repository.api.TextRepository; 2
}
1
Requires the API module because we need to access TextRepository in it

2
Indicates this client wants to use a service implementing TextRepository

The easytext.repository.api in turn depends on the easytext.domain.api, since it uses Text as the return type in the TextRepository interface:

module easytext.repository.api {
  exports easytext.repository.api; 1
  requires easytext.domain.api; 2
}
1
Exposes the API package containing TextRepository

2
Requires the domain API module because it contains Text, which is referenced in the TextRepository interface

Last, the easytext.domain.api module contains the Text class:

public class Text {

   private String theText;

   public String getTheText() {
      return this.theText;
   }

   public void setTheText(String theText) {
      this.theText = theText;
   }

   public int wordcount() {
      return 42; // Why not
   }

}
Note that Text has a wordcount method, which we’ll be using later in the client code. The easytext.domain.api module exports the package containing this Text class:

module easytext.domain.api {
   exports easytext.domain.api;
}
The client module contains the following invocation of the repository:

TextRepository repository = ServiceLoader.load(TextRepository.class)
   .iterator().next();

repository.findText("HHGTTG").wordcount();
If we compile this, the following error is produced by the compiler:

./src/easytext.client/easytext/client/Client.java:13: error: wordcount() in
Text is defined in an inaccessible class or interface
      repository.findText("HHGTTG").wordcount();
                                 ^
Even though we’re not mentioning the Text type directly in easytext.client, we’re trying to call a method on this type as it is returned from the repository. Therefore, the client module needs to read the easytext.domain.api module, which exports Text. One way to solve this compilation error is to add a requires easytext.domain.api clause to the client’s module descriptor. That’s not a great solution, though; why should the client module have to handle the transitive dependencies of the repository module? A better solution is to improve the repository’s module descriptor:

module easytext.repository.api {
  exports easytext.repository.api;
  requires transitive easytext.domain.api; 1
}
1
Sets up implied readability by adding the transitive keyword

Note the additional transitive keyword in the exports clause. It effectively says that the repository module reads easytext.domain.api, and every module requiring easytext.repository.api also automatically reads easytext.domain.api, as illustrated in Figure 5-2.

Implied readability edge
Figure 5-2. Implied readability relation set up by requires transitive, shown as a bold edge
Now, the example compiles without problems. The client module can read the Text class through the requires transitive clause in the repository’s module descriptor. Implied readability allows the repository module to express that its own exported packages are not enough to use the module.

You’ve seen an example where the return type comes from a different module, necessitating the use of requires transitive. Whenever a public, exported type refers to types from another module, use implied readability. Besides return types, this applies to argument types and thrown exceptions from different modules as well.

A compiler flag helps you spot dependencies in an API module that should be marked as transitive. If you compile with -Xlint:exports, any types that are part of exported types which are not transitively required (but should be) result in a warning. This way, you can find the problem while compiling the API module itself. Otherwise, the error surfaces only when compiling a consuming module that is missing said dependency because no implied readability is set up. For example, when we leave out the transitive modifier in the previous easytext.repository.api module descriptor and compile it with -Xlint:exports, the following warning is produced:

$ javac -Xlint:exports --module-source-path src -d out -m easytext.repository.api
src/easytext.repository.api/easytext/repository/api/TextRepository.java:6:
warning: [exports] class Text in module easytext.domain.api is not indirectly
 exported using requires transitive
  Text findText(String id);
  ^
1 warning
When designing an API module, implied readability is a powerful technique to make modules easier to consume. There is a subtle danger to relying on implied readability in consuming modules. In our example, the client module can access the Text class by virtue of the transitive nature of implied readability. This works fine when the type is used through the interface, as in the example repository.findText("HHGTTG").wordcount(). However, what if inside the client module we start using the Text class directly, without getting it as a return value through the interface’s method—say, by instantiating Text directly and storing it in a field? That continues to compile and run correctly. But does the client’s module descriptor now truly reflect our intentions? You can argue that the client module must take an explicit, direct dependency on the easytext.domain.api module in this case. That way, should the client module stop using the repository module (and thus lose the implied readability on the domain API), the client code continues to compile.

This may seem like an inconsequential issue. It compiles and works, so what’s the big deal? However, as a module author, you are responsible for declaring the correct dependencies based on your code. Implied readability is meant only to prevent surprises such as the compiler error you saw previously. It is not a carte blanche to be lazy about real dependencies your module has.

API Module with a Default Implementation
Whether the implementation of the API should live in the same module or in a separate module is an interesting question. Separating the API and implementation is a useful pattern when you expect there to be multiple implementations of the API. On the other hand, when only a single implementation exists, bundling the API and implementation in the same module makes the most sense. It also makes sense when you want to offer a default implementation as a convenience.

A combined API/implementation module doesn’t preclude alternative implementations in separate modules, at a later point in time. However, those alternative implementation modules then need a dependency on the combined module for the API. This combined module also contains an implementation. That may lead to superfluous transitive dependencies, because of the combined module’s implementation dependencies that are of no use to the alternative implementation but still need to be resolved.

Figure 5-3 illustrates this problem. Modules easytext.analysis.coleman and easytext.analysis provide an implementation of the Analyzer interface as service. The latter module exports the API in addition to providing an implementation. However, the implementation inside easytext.analysis (but not the API) requires the syllablecounter module. Now, the alternative implementation of the API in easytext.analysis.coleman cannot run without syllablecounter being present on the module path, even though the alternative implementation doesn’t need it. Separating the API into its own module avoids such problems, as shown in Figure 5-4.

When the API is in a module with an implementation (like in the `easytext.analysis` module here), the dependencies of the implementation (in this case `syllablecounter`) transitively burden alternative implementations well.
Figure 5-3. When the API is in a module with an implementation (as in the easytext.analysis module here), the dependencies of the implementation (in this case syllablecounter) transitively burden alternative implementations as well
Separate API module
Figure 5-4. A separate API module works better when there are multiple implementations
Towards the end of Chapter 3 and in Chapter 4, we introduced a separate API module for the EasyText Analyzer interface. Whenever you have (or expect to have) multiple implementations of the API, it makes sense to extract the public API into its own module. This is certainly the case here: the whole idea was to make EasyText extensible with new analysis functionality. When the implementations are provided as a service, this leads to the pattern in Figure 5-4 when extracting an API module.

In this case, you end up with multiple implementation modules that do not export anything, which depend on the API module and publish their implementation as a service. The API module, in turn, only exports packages and does not contain any encapsulated implementation code. With this setup, the implementation dependency of the easytext.analysis.kincaid module on syllablecounter is not imposed on the alternative easytext.analysis.coleman module. You can use easytext.analysis.coleman without having easytext.analysis.kincaid or syllablecounter on your module path.

Aggregator Modules
With implied readability fresh in your mind, you’re ready for a new module pattern: the aggregator module. Imagine you have a library consisting of several loosely related modules. A user of your imaginary library can use either one or several modules, depending on their needs. So far, nothing new.

Building a Facade over Modules
Sometimes you don’t want to burden the user of the library with the question of which exact modules to use. Maybe the modules of the library are split up in a way that benefits the library maintainer but might confuse the user. Or maybe you just want to have a way for people to quickly get started by having to depend on only a single module representing the whole library. One way to do this would be to build both the individual modules and a “super-module” combining all content from the individual modules. That works but is not a particularly nice solution.

Another way to achieve a similar result is to use implied readability to construct an aggregator module. Essentially, you are building a facade over the existing library modules. An aggregator module contains no code; it has only a module descriptor setting up implied readability to all other modules:

module library {
  requires transitive library.one;
  requires transitive library.two;
  requires transitive library.three;
}
Now, if the library user adds a dependency on library, all three library modules are transitively resolved, and their exported types are readable for the application. Figure 5-5 shows the new situation. Of course, it’s still possible to depend on a single specific module if so desired.

Implied readability edge
Figure 5-5. The application module can use all exported types from the three library modules through the aggregator module library. Implied readability edges are shown with bold edges.
Because aggregator modules are lightweight, it’s perfectly possible to create several different aggregator modules. For example, you could provide several profiles or distributions of a library, targeted towards specific users.

The JDK contains a good example of this approach, as we saw before in Chapter 2. It has several aggregator modules, listed in Table 5-1.

Table 5-1. Aggregator modules in the JDK
Module	Aggregates
java.se

All modules officially belonging to the Java SE specification

java.se.ee

The java.se modules, plus all Java EE modules bundled with the Java SE platform

On the one hand, creating an aggregator module is convenient for consumers. Consumer modules can just require the aggregator module, without thinking too hard about the underlying structure. On the other hand, it poses a risk as well. The consuming module can now transitively access all exported types of the aggregated modules through the aggregator module. That may include types from modules you don’t want to depend on as a consumer. Because of implied readability, you will not be warned by the module system anymore when you do use those types. Specifying precise dependencies on the exact underlying modules you need protects you from these pitfalls.

Having aggregator modules available is a convenience. From the point of view of the JDK developers, it’s more than just a convenience. Aggregator modules are a great way to compose the platform into manageable chunks without having to repeat yourself. Let’s take a look at one of the platform aggregator modules, java.se.ee, to see what this means. As you’ve seen before, you can use java --describe-module <modulename> to view the module descriptor content of a module:

$ java --describe-modules java.se.ee
java.se.ee@9
requires java.se transitive
requires java.xml.bind transitive
requires java.corba transitive
...
The aggregator module java.se.ee transitively requires relevant EE modules. It also transitively requires another aggregator module, java.se. Implied readability works transitively, so when a module reads java.se.ee, it also reads everything that is required transitively by java.se, and so on. This pattern of hierarchical aggregation is a clean way to organize a large number of modules.

Safely Splitting a Module
There is another useful application of the aggregator module pattern. It’s when you have a single, monolithic module that you need to split up after it has been released. It may have grown too large to maintain, for example, or you might want to separate out unrelated functionality for improved reusability.

Suppose that a module largelibrary (shown in Figure 5-6) is in need of further modularization. However, there are already users of largelibrary in the wild, using its public API. The split needs to happen in a backward-compatible manner. That is, we can’t rely on existing users of largelibrary to switch to the new, smaller modules right away.

The solution, shown in Figure 5-7, is to replace largelibrary with an aggregator module of the same name. In turn, this aggregator module arranges implied readability to the new, smaller modules.

Module `largelibrary` before the split.
Figure 5-6. Module largelibrary before the split
Module `largelibrary` after the split.
Figure 5-7. Module largelibrary after the split
The existing packages, both exported and encapsulated, are distributed over the newly introduced modules. New users of the library now have a choice of using either one of the individual modules, or largelibrary for readability on the whole API. Existing users of the library do not have to change their code or module descriptors when upgrading to this new version of largelibrary.

It’s not necessary to always create a pure aggregator module, containing only a module descriptor and no code of its own. Often a library consists of core functionality that is independently useful. Going with the largelibrary example, package largelibrary.part2 might build on top of largelibrary.part1.

In that case, it makes sense to create two modules, as shown in Figure 5-8.

Alternative approach to splitting `largelibrary`.
Figure 5-8. Alternative approach to splitting largelibrary
Consumers of largelibrary can keep using it as is, or require only the largelibrary.core module to use a subset of the functionality. This approach is implemented with the following module descriptor for largelibrary:

module largelibrary {
  exports largelibrary.part2;

  requires transitive largelibrary.core;
}
As you have seen, implied readability offers a safe way to split modules. Consumers of the new aggregator modules won’t notice any difference to the situation where all code was in a single module.

Avoiding Cyclic Dependencies
In practice, safely splitting an existing module will be hard. The previous example assumed that the original package boundaries allow for cleanly splitting the module. What if different classes from the same package need to end up in different modules? You can’t export the same package from two different modules for your users. Or, what if types across two packages have a mutual dependency? Putting each package into a separate module won’t work in that case, because module dependencies cannot be circular.

We’ll look at the problem of split packages first. Then we’ll investigate how to refactor circular dependencies between modules.

Split Packages
One scenario you can run into when splitting modules is the introduction of split packages. A split package is a single package that spans multiple modules, as shown in Figure 5-9. It occurs when partitioning a module doesn’t nicely align with existing package boundaries.

Two modules containing the same packages, but different classes.
Figure 5-9. Two modules containing the same packages but different classes
In this example, module.one and module.two both contain classes from the same packages called splitpackage and splitpackage.internal.

TIP
Remember, packages in Java are nonhierarchical. Despite their appearance, splitpackage and splitpackage.internal are two unrelated packages that happen to share the same prefix.

Putting both module.one and module.two on the module path results in an error when starting the JVM. The Java module system doesn’t allow split packages. Only one module may export a given package to another module. Because exports are declared in terms of package names, having two modules export the same package leads to inconsistencies. If this were allowed, a class with the exact same fully qualified name might be exported from both modules. When another module depends on both these modules and wants to use this class, conflicts arise as to which module the class should come from.

Even if the split package is not exported from the modules, the module system won’t allow it. Theoretically, having a nonexported split package (such as splitpackage.internal in Figure 5-9) isn’t a problem. It’s all encapsulated, after all. In practice, the way the module system loads modules from the module path prohibits this arrangement. All modules from the module path are loaded within the same classloader. A classloader can have only a single definition of a package, and whether it’s exported or encapsulated doesn’t matter. In “Container Application Patterns”, you’ll see how more advanced uses of the module system allow for multiple modules with the same encapsulated packages.

The obvious way to avoid split packages is to not create them in the first place. That works when you create modules from scratch, but is harder when you’re transitioning existing JARs to modules.

The example in Figure 5-9 illustrates a cleanly split package, meaning no types with the same fully qualified name appear in the different modules. When converting existing JARs to modules, encountering unclean splits (where multiple JARs have the same types) is not uncommon. Of course, on the classpath these JARs might have worked together (but only accidentally). Not in the module system, though. In that case, merging the JARs and their overlapping packages into a single module is the solution.

Keep in mind that the module system checks for package overlap with all modules. That includes platform modules. Several JARs in the wild try to add classes to packages owned by modules in the JDK. Modularizing these JARs and putting them on the module path will not work, because they overlap with these platform modules.

WARNING
When JARs containing packages that overlap with JDK packages are placed on the classpath, their types will be ignored and will not be loaded.

Breaking Cycles
Now that we’ve addressed the problem of split packages, we are still left with the issue of cyclic dependencies between packages. When splitting up a module with mutually dependent packages, this would give rise to cyclic module dependencies. You can create these modules, but they won’t compile.

In “Module Resolution and the Module Path”, you learned that readability relations between modules must be acyclic at compile-time. Two modules cannot require each other from their module descriptors. Then, in “Creating a GUI Module”, you learned that cyclic readability relations can arise at run-time. Also note that services can use each other, forming a cyclic call-graph at run-time.

So, why this strict enforcement against cycles at compile-time? The JVM can load classes lazily at run-time, allowing for a multistage resolution strategy in the case of cycles. However, the compiler can compile a type only if all the other types it uses are either already compiled, or being compiled in the same compilation run.

The easiest way to achieve this would be to always compile mutually dependent modules together. Although not impossible, this leads to hard-to-manage builds and codebases. Indeed, that is a subjective statement. Disallowing cyclic module dependencies at compile-time is an opinionated choice made by the Java module system based on the premise that cyclic dependencies are generally bad news for modularity.

It’s not controversial to say code containing cycles is harder to understand—especially because cycles can hide behind many levels of indirection. It is not always simply two classes in two different packages in two JARs that depend on each other. Cyclic dependencies significantly muddy the waters, and have no place in applications modularized with the Java module system.

What to do when you need to modularize an application that does have cyclic dependencies between existing JARs? Or, when splitting packages from a JAR would result in such cycles? You cannot just convert them to two modules that require each other, because the compiler disallows such a configuration.

One obvious solution is to merge these JARs into a single module. When there is such a tight (cyclic) relation between two components, it’s not much of a stretch to conclude they’re effectively one module. Of course, this solution assumes the cyclic relationship is benign to start with. It also breaks down when the cycle is indirect and involves several components, not just two, unless you want to merge all components participating in the cycle, which is unlikely.

Often, a cycle indicates questionable design. That means breaking the cycle involves a bit of redesign. As the saying goes, all problems in computer science can be solved by introducing another level of indirection (except the problem of too many indirections, of course). Let’s see how adding an indirection can help break cycles.

We start out with two JAR files: authors.jar and books.jar. Each JAR contains a single class (respectively Author and Book), referencing each other. By naively turning the existing JARs into modular JARs, the cyclic dependency becomes apparent, as shown in Figure 5-10.

These modules won't compile or resolve because of their cyclic dependency.
Figure 5-10. These modules won’t compile or resolve because of their cyclic dependency
The first question we should answer is: what is the correct relation between those modules? There’s no one-size-fits-all approach to be taken here. We need to zoom in on the code, and see what it’s trying to accomplish. Only then can the question of what’s appropriate be answered. We’ll explore this question based on Example 5-3.

Example 5-3. Author.java (➥ chapter5/cyclic_dependencies/cycle)
public class Author {
  private String name;
  private List<Book> books = new ArrayList<>();

  public Author(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public void writeBook(String title, String text) {
    this.books.add(new Book(this, title, text));
  }

}
An Author has a name and can write a book, which is added to the list of books:

public class Book {
  private Author author;
  private String title;
  private String text;

  public Book(Author author, String title, String text) {
    this.author = author;
    this.text = text;
    this.title = title;
  }

  public void printBook() {
    System.out.printf("%s, by %s\n\n%s", title, author.getName(), text);
  }
}
A book can be created given an Author, a title, and some text. After creation, a Book can be printed using printBook. Looking at that method’s code, we can see that its functionality is causing the dependency from Book to Author. In the end, the Author is just there to get a name for printing. This points to a new abstraction. The Book cares only about getting a name. Why should it be coupled to Author? Maybe there are even other ways of creating books than with authors (I heard deep learning is taking over our jobs any day now…).

All of these deliberations point to a new abstraction: the indirection we are looking for. Because the book’s module is interested only in the name of things, let’s introduce an interface Named, as shown in Example 5-4.

Example 5-4. Named.java (➥ chapter5/cyclic_dependencies/without_cycle)
public interface Named {
  String getName();
}
Now Author can implement this interface. It already has the required getName implementation. The Book implementation needs to switch out usages of Author for Named. In the end, this results in the module structure shown in Figure 5-11. As an added bonus, the javamodularity.authors package need not be exported anymore.

The `Named` interface breaks the cyclic dependency.
Figure 5-11. The Named interface breaks the cyclic dependency
This is far the from the only solution. In a larger system where multiple other components use Author similarly, the Named interface can also be lifted into its own module. For this example, that would lead to a triangular dependency graph with Named in the top module, and both the books and authors modules pointing to it.

In general, interfaces play a big role in breaking cyclic dependencies. An interface is one of the most powerful means of abstraction in Java. They enable the pattern of dependency inversion by which cycles can be broken.

Until now, we’ve assumed that the exact nature of the cyclic dependency is known. When cycles are indirect and arise through many steps, they can be hard to spot, though. Tools (for example SonarQube) can help detect cyclic dependencies in existing code. Use them to your advantage.

Optional Dependencies
One of the hallmarks of a modular application is its explicit dependency graph. So far, you’ve seen how to construct such a graph through requires statements in module descriptors. But what if a module isn’t strictly necessary at run-time, but nice to have?

Many frameworks currently work this way: by adding a JAR file (let’s say fastjsonlib.jar) to the classpath, you get additional functionality. Whenever fastjsonlib.jar is not available, the framework uses a fallback mechanism or just doesn’t offer the enhanced functionality. For the given example, parsing JSON might be a bit slower but it still works. The framework effectively has an optional dependency on fastjsonlib. If your application already makes use of fastjsonlib, the framework also uses it; otherwise, it won’t.

NOTE
The Spring Framework is a famous example of a framework that has many optional dependencies. For instance, it has a Base64Utils helper class that delegates to either Java 8’s Base64 class or to an Apache Commons Codec Base64 class if it’s on the classpath. Regardless of the run-time environment, Spring itself must be compiled against both implementations.

Such optional dependencies cannot be expressed with our current knowledge of module descriptors. You can express a single module dependency graph for compile-time and run-time only through requires statements.

Services offer this flexibility and are a great way to address optional dependencies in your application, as you will see later in “Implementing Optional Dependencies with Services”. However, services are bound to the ServiceLoader API, which can be an intrusive change for existing code. Since optional dependencies are often used by frameworks and libraries, they might not want to force the use of the services API on its users. When adopting services is not an option, another feature of the module system can be used to model optional dependencies: compile-time dependencies.

Compile-Time Dependencies
As the name already implies, compile-time dependencies are dependencies that are only required during compilation. You can express a compile-time dependency on a module by adding the static modifier to a requires clause, as shown in Example 5-5.

Example 5-5. Declaring a compile-time dependency (➥ chapter5/optional_dependencies)
module framework {
  requires static fastjsonlib;
}
By adding static, the fastjsonlib module needs to be present when compiling, but not when running framework. This effectively makes fastjsonlib an optional dependency of framework from the perspective of the module consumer.

NOTE
Compile-time dependencies have other applications as well. For example, a module can export Java annotations that are used only during compilation. Requiring such a module should not lead to a run-time dependency, so requires static suits this scenario perfectly.

The question of course is, what happens when framework is run without fastjsonlib? Let’s explore the scenario where framework uses the class FastJson exported from fastjsonlib directly:

package javamodularity.framework;

import javamodularity.fastjsonlib.FastJson;

public class MainBad {

  public static void main(String... args) {
    FastJson fastJson = new FastJson();
  }

}
Running framework without fastjsonlib in this case leads to a NoClassDefFound​Error. The resolver doesn’t complain about the missing fastjsonlib module, because it is a compile-time only dependency. Still, we get an error at run-time because Fast​Json clearly is necessary for framework in this case.

The onus is on the module expressing a compile-time dependency to guard against these problems at run-time. That means framework needs to code defensively around the usage of classes from compile-time dependencies. A direct reference to FastJson as in MainBad is problematic, because the VM will always try to load the class and instantiate it, leading to the NoClassDefFoundError.

Fortunately, Java has lazy loading semantics for classes: they are loaded at the last possible time. If we can somehow tentatively try to use the FastJson class and gracefully recover if it’s not available, we achieve our goal. By using reflection with appropriate try-catch blocks, framework can prevent run-time errors:

package javamodularity.framework;

import javamodularity.fastjsonlib.FastJson;

public class Main {

  public static void main(String... args) {
    try {
      Class<?> clazz = Class.forName("javamodularity.fastjsonlib.FastJson");
      FastJson instance =
        (FastJson) clazz.getConstructor().newInstance();
      System.out.println("Using FastJson");
    } catch (ReflectiveOperationException e) {
      System.out.println("Oops, we need a fallback!");
    }
  }

}
When the FastJson class isn’t found, the catch block can be used as a fallback. On the other hand, if fastjsonlib is there, we can use FastJson without any problems after the reflective instantiation.

WARNING
The requires static fastjsonlib clause does not cause fast​jsonlib to be resolved at run-time, even if fast​jsonlib is on the module path! There needs to be a direct requires clause on fast​jsonlib, or you can add it as root module through --add-modules fastjsonlib for it to be resolved. In both cases, fastjsonlib is then resolved, and framework reads and uses it.

Guarding each and every use of classes from a compile-time dependency this way can be quite painful. Lazy loading also means that the moment a class is loaded can be surprising. A notorious example is the static initializer block in a class. That should already be a warning sign that requires static may not be the best way to create optional coupling between modules.

Whenever your module uses compile-time dependencies, try to concentrate the use of these optional types in one part of the module. Put the guard on the instantiation of a single top-level class, that in turn has all the (direct) references to types in the optional dependency. This way, the module won’t get littered with defensive guard code.

There are other legitimate uses for requires static—for example, to reference compile-time-only annotations from another module. A requires clause (without static) would cause the dependency to be required at run-time as well.

Sometimes annotations on a class are used only at compile-time—for example, to perform static analysis (such as checking @Nullable or @NonNull), or to mark a type as input for code generation. When you retrieve annotations for an element at run-time and the annotation class is not present, the JVM gracefully degrades and won’t throw any classloading exceptions. Still, you need access to the annotation types when compiling the code.

Let’s look at a fictional @GenerateSchema annotation that can be put on entities. At build-time, these annotations are used to find classes to generate a database schema based on their signature. The annotation is not used at run-time. Therefore, we want code annotated with @GenerateSchema not to require the schemagenerator module (which exports the annotation) at run-time. Say we have the class in Example 5-6 in module application.

Example 5-6. An annotated class (➥ chapter5/optional_dependencies_annotations)
package javamodularity.application;

import javamodularity.schemagenerator.GenerateSchema;

@GenerateSchema
public class BookEntity {

  public String title;
  public String[] authors;

}
The module descriptor for application should have the right compile-time dependency:

module application {
  requires static schemagenerator;
}
In application, there’s also a main class that instantiates the BookEntity and tries to obtain the annotations on that class:

package javamodularity.application;

public class Main {
  public static void main(String... args) {
    BookEntity b = new BookEntity();
    assert BookEntity.class.getAnnotations().length == 0;
    System.out.println("Running without annotation @GenerateSchema present.");
  }
}
When you run the application without the schemagenerator module, everything works out fine:

$ java -ea --module-path out/application \
       -m application/javamodularity.application.Main
Running without annotation @GenerateSchema present.
(The -ea flag enables run-time assertions in the JVM.)

There are no classloading or module resolution problems due to the missing schemagenerator module at run-time. Leaving out the module at run-time is possible because it’s a compile-time dependency. Subsequently, the JVM gracefully handles the absence of annotation classes at run-time, as it always has. The call to getAnnotations returns an empty array.

However, if we add the schemagenerator module explicitly, the @GenerateSchema annotation is found and returned:

$ java -ea --module-path out/application:out/schemagenerator \
       -m application/javamodularity.application.Main
Exception in thread "main" java.lang.AssertionError
        at application/javamodularity.application.Main.main(Main.java:6)
An AssertionError is thrown because now the @GenerateSchema annotation is returned. No longer is the annotations array empty.

There’s no need for guard code to cope with missing annotation types at run-time, unlike the previous examples of compile-time dependencies we’ve seen. The JVM already takes care of this during classloading and reflective access to annotations.

It is also possible to combine the static modifier on requires with transitive:

module questionable {
  exports questionable.api;
  requires transitive static fastjsonlib;
}
You need to do this when a type from the optional dependency is referenced in an exported package. The fact that it’s possible to combine static and transitive doesn’t make it a good idea. It puts the responsibility of creating proper guards on the consumer of the API, which certainly doesn’t adhere to the principle of the least surprise. In fact, the only reason this combination of modifiers is possible is to enable modularization of legacy code by using this pattern.

Optional dependencies can be approximated through requires static, but with services in the module system we can do better!

Implementing Optional Dependencies with Services
Using compile-time dependencies to model optional dependencies is possible but requires diligent use of reflection to safeguard classloading. Services are a better fit for this usage. You already saw in Chapter 4 that a service consumer can obtain zero or more implementations of a service type from service provider modules. Getting zero or one service implementation is just a special case of this general mechanism.

Let’s apply this to the previous example with framework by using the optional fast​jsonlib. Fair warning: we start with a naive refactoring, refining it into a real solution in several steps.

In Example 5-7, framework becomes a consumer of an optional service provided by fastjsonlib.

Example 5-7. Consuming a service, which may or may not be available at run-time (➥ chapter5/optional_dependencies_service)
module framework {
  requires static fastjsonlib;
  uses javamodularity.fastjsonlib.FastJson;
}
By virtue of the uses clause, we can now load FastJson with ServiceLoader in the framework code:

FastJson fastJson =
  ServiceLoader.load(FastJson.class)
               .findFirst()
               .orElse(getFallBack());
We no longer need reflection in the framework code to get FastJson when it’s available. If there’s no service found by ServiceLoader (meaning findFirst returns an empty Optional), we assume we can get a fallback implementation with getFallBack.

Of course, fastjsonlib must provide the class we’re interested in as a service:

module fastjsonlib {
  exports javamodularity.fastjsonlib;
  provides javamodularity.fastjsonlib.FastJson
      with javamodularity.fastjsonlib.FastJson;
}
With this setup, the resolver even resolves fastjsonlib without having to add it explicitly, based on the uses and provides clauses.

There are several awkward problems with this refactoring from a pure compile-time dependency to a service, though. First of all, it’s a bit strange to expose a class directly as a service, rather than exposing an interface and hiding the implementation. Splitting FastJson into an exported interface and encapsulated implementation solves this. This refactoring also enables the framework to implement a fallback class implementing the same interface.

A bigger problem surfaces when trying to run framework without fastjsonlib. After all, fastjsonlib is supposed to be optional so this should be possible. When framework is started without fastjsonlib on the module path, the following error occurs:

Error occurred during initialization of VM
java.lang.module.ResolutionException: Module framework does not read a module
  that exports javamodularity.fastjsonlib
You cannot declare a service dependency with uses on a type that you can’t read at run-time, regardless of whether there is a provider module. The obvious solution is to change the compile-time dependency on fastjsonlib into a normal dependency (requires without static). However, that’s not what we want: the dependency on the library should be optional.

At this point, a more intrusive refactoring is warranted. It becomes clear that not everything can be optional in the relation between framework and fastjsonlib for this service setup to work. Why does the FastJson interface (assuming we refactored it into an interface) live in fastjsonlib? In the end, it’s the framework which dictates that functionality it wants to use. The functionality is optionally provided by a library or by fallback code in the framework itself. Through this realization, it makes sense to put the interface in framework or in a separate API module that is shared between the framework and library.

This is an intrusive redesign. It almost inverts the relationship between the framework and the library. Instead of the framework optionally requiring the library, the library must require the framework (or its API module) to implement an interface and offer the implementation as a service. However, when such a redesign can be pulled off, it results in an elegant decoupled interaction between the framework and library.

Versioned Modules
Talking about modules for frameworks and libraries inevitably leads to questions around versioning. Modules are independently deployable and reusable units. Applications are built by combining the right deployment units (modules). Having just a module name is not sufficient to select the right modules that work together. You need versions as well.

Modules in the Java module system cannot declare a version in module-info.java. Still, it is possible to attach version information when creating a modular JAR. You can set a version by using the --module-version=<V> flag of the jar tool. The version V is set as an attribute on the compiled module-info.class and is available at run-time. Adding a version to your modular JARs is a good practice, especially the for API modules discussed earlier in this chapter.

SEMANTIC VERSIONING
There are many different ways to version modules. The most important goal in versioning is to communicate the impact of changes to consumers of a module. Is a newer version a safe drop-in replacement for the previous version? Are there any changes to the API of the module? Semantic versioning formalizes a versioning scheme that is widely used and understood:

MAJOR.MINOR.PATCH
Breaking changes, such as changing a method signature in an interface, lead to a bump in the MAJOR version part. Changes to a public API that are backward compatible, such as adding a method on a public class, bump the MINOR part of the version string. Last, the PATCH part is incremented when implementation details change. This can be a bug fix, or optimizations to the code. In any case, a PATCH increment should always be a safe drop-in replacement for the previous version.

Note that it is not always straightforward to determine whether something is a major or minor change. For example, adding a method to an interface is a major change only if consumers of the interface are supposed to implement it. If, on the other hand, the interface is only called by consumers (but not implemented), consumers won’t break when a method is added. To muddy the waters further, default methods on interfaces (introduced in Java 8) can turn the addition of a method on an interface into a minor change even in the first scenario. The important thing is to always reason from the perspective of the consumer of the module. The module’s next version should reflect the impact of the upcoming changes for the consumers of a module.

Module Resolution and Versioning
Even though there’s support for adding a version to modular JARs, it is not used in any way by the module system (yet). Modules are resolved purely by name. This may seem strange, because we already established that versions play an important role in deciding which modules play nice together. Ignoring versions during module resolution is not an oversight, but a deliberate design choice in the Java module system.

How you indicate which versions of deployment units work together is a rather controversial topic. What are the syntax and semantics of the version string? Do you specify a version range for dependencies? Or only exact versions? What happens if you end up requiring two versions at the same time, in the latter case? Such conflicts must be addressed.

Tools and frameworks (such as Maven and OSGi) have opinionated answers to each of those questions. It turns out these version-selection algorithms and associated conflict-resolution heuristics are both complex and (sometimes subtly) different. That’s why the Java module system, for now, eschews the notion of version selection during the module-resolution process. Whatever strategy adopted would end up deeply ingrained in the module system and, hence, in the compiler, language specification, and JVM. The cost of not getting it right was simply too high. Therefore, the requires clause in module descriptors takes only a module name, not a module version (or range).

That still leaves us developers with a challenge. How do we select the right module versions to put on the module path? The answer is surprisingly straightforward, if a bit unsatisfactory: just as we did before with the classpath. Selection and retrieval of the right versions of dependencies are delegated to existing build tools. Maven and Gradle et al. handle this by externalizing the dependency version information into a POM file. Other tools may use other means, but the fact remains this information must be stored outside the module.

Figure 5-12 shows the steps involved with building a source module application that depends on two other modular JARs, lib and foo. At build-time, build tools download the right versions of the dependencies from a repository, using information from POM files. Any version conflicts arising during this process must be resolved by the build tool’s conflict-resolution algorithm. The downloaded modular JARs are put on the module path for compilation. Then, the Java compiler resolves the module graph guided by information from module-info descriptors on the module path and application itself. More details on how existing build tools handle modules can be found in Chapter 11.

Build tools for version selection
Figure 5-12. Build tools select the right version of a dependency to put on the module path
1
A build tool such as Maven or Gradle downloads dependencies from a repository such as Maven Central. Version information in a build descriptor (for example, pom.xml) controls which versions are downloaded.

2
The build tool sets up the module path with the downloaded versions.

3
Then, the Java compiler or runtime resolves the module graph from the module path, which contains the correct versions without duplicates for the application to work. Duplicate versions of a module result in an error.

The Java module system ensures that all necessary modules are present when compiling and running application through the module-resolution process. However, the module system is oblivious to which version of a module is resolved. As long as there is a module with the correct name, it will be resolved. In addition, if multiple modules with the same name (but possibly a different version) are found in a directory on the module path, this leads to an error.

WARNING
If two modules with the same name are in different directories on the module path, the resolver uses the first one it encounters and ignores the second. No errors are produced in this case.

There are genuine situations where having multiple versions of the same module at the same time is expedient. We’ve seen that by default, this scenario is not supported when launching an application from the module path. This is similar to the situation before the module system. On the classpath, two JARs of a different version can lead to undefined run-time behavior, which is arguably worse.

In application development, it is highly advisable to find a way to unify dependencies on a single module version. Often, the desire for running multiple concurrent versions of the same module stems from laziness rather than a pressing need. When developing generic, container-like applications, this may not always be possible. In Chapter 6, you will see that there is a way to resolve multiple versions of a module by using a more sophisticated module system API to construct module graphs at run-time. Another option in these cases is to adopt an existing module system such as OSGi, which offers the possibility to run multiple versions concurrently out of the box.

Resource Encapsulation
We’ve spent a considerable amount of time talking about strongly encapsulating code in modules. Although that’s the most obvious use of strong encapsulation, applications typically have more resources than just code. Think of files containing translations (localized resource bundles), configuration files, images used in user interfaces, and so on. It’s worthwhile to encapsulate such resources in a module as well, colocating them with the code using the resources. Having modules nose into another module’s resources is just as bad as depending on private implementation classes.

Historically, resources on the classpath were even more free-for-all than code, because no access modifiers can be applied to resources. Any class can read any resource on the classpath. With modules, that changes. By default, resources within packages in a module are strongly encapsulated. Those resources can be used only from within the module, just as with classes in nonexported packages.

However, many tools and frameworks depend on finding resources regardless of where they come from. Frameworks might scan for certain configuration files (for example, persistence.xml or beans.xml in Java EE) or otherwise depend on resources from application code. This requires resources within a module to be accessible from other modules. To handle these cases gracefully and maintain backward compatibility, many exceptions have been added to the default of encapsulating resources in modules.

First, we are going to look at loading resources from within the same module. Second, we are going to look at how modules can share resources and which exceptions to the default of strong encapsulation are in place. Last, we turn our attention to a special case of resource loading: ResourceBundles. This mechanism is mainly used for localization and has been updated for the module system.

Loading Resources from a Module
Here’s an example of a compiled module firstresourcemodule containing both code and resources:

firstresourcemodule
├── javamodularity
│   └── firstresourcemodule
│       ├── ResourcesInModule.class
│       ├── ResourcesOtherModule.class
│       └── resource_in_package.txt
├── module-info.class
└── top_level_resource.txt
There are two resources in this example alongside two classes and the module descriptor: resource_in_package.txt and top_level_resource.txt. We assume the resources are put into the module during the build process.

Resources are typically loaded through resource loading methods provided by the Class API. This still works within a module, as shown in Example 5-8.

Example 5-8. Various ways of loading resources in a module (➥ chapter5/resource_encapsulation)
   public static void main(String... args) throws Exception {
      Class clazz = ResourcesInModule.class;
      InputStream cz_pkg = clazz.getResourceAsStream("resource_in_package.txt"); 1
      URL cz_tl = clazz.getResource("/top_level_resource.txt"); 2

      Module m = clazz.getModule(); 3
      InputStream m_pkg = m.getResourceAsStream(
        "javamodularity/firstresourcemodule/resource_in_package.txt"); 4
      InputStream m_tl = m.getResourceAsStream("top_level_resource.txt"); 5

      assert Stream.of(cz_pkg, cz_tl, m_pkg, m_tl)
                   .noneMatch(Objects::isNull);
   }

}
1
Reading a resource with Class::getResource resolves the name relative to the package the class is in (javamodularity.firstresourcemodule in this case).

2
When reading a top-level resource, a slash must be prefixed to avoid relative resolution of the resource name.

3
You can obtain a java.lang.Module instance from Class, representing the module the class is from.

4
The Module API introduces new ways to obtain resources from a module.

5
This getResourceAsStream method also works for top-level resources. The Module API always assumes absolute names, so no leading slash is necessary for a top-level resource.

All of these methods work for loading resources within the same module. Existing code does not have to be changed for loading resources. As long as the Class instance you’re calling getResource{AsStream} on belongs to the current module containing the resource, you get back a proper InputStream or URL. On the other hand, when the Class instance comes from another module, null will be returned because of resource encapsulation.

WARNING
You can also load resources through ClassLoader::getResource* methods. In a module context, it is better to use the methods on Class and Module instead. The ClassLoader methods do not take into account the current module context like the methods Class and Module do, leading to potentially confusing results.

There’s a new way to load resources as well. It’s through the new Module API, which is discussed in more detail in “Reflection on Modules”. The Module API exposes getResourceAsStream to load a resource from that module. Resources in a package can be referred to in an absolute way by replacing the dots in the package name with slashes and appending the filename. For example, javamodularity.firstresourcemodule becomes javamodularity/firstresourcemodule. After adding the filename, the argument to load a resource in a package becomes javamodularity/firstresourcemodule/resource_in_package.txt.

Any resource in the same module, whether it is in a package or at the top level, can be loaded by the methods discussed so far.

Loading Resources Across Modules
What happens if you obtain a Module instance representing a module other than the current module? It might seem you can call getResourceAsStream on this other Module instance and get access to all resources in that module. Thanks to strong encapsulation of resources, this is not (always) the case. Several exceptions to this rule exist, so let’s add a module secondresourcemodule to our example to explore the different scenarios:

secondresourcemodule
├── META-INF
│   └── resource_in_metainf.txt
├── javamodularity
│   └── secondresourcemodule
│       ├── A.class
│       └── resource_in_package2.txt
├── module-info.class
└── top_level_resource2.txt
We assume that both module descriptors for firstresourcemodule and secondresourcemodule have an empty body, meaning no package is exported. There’s a package containing class A and a resource, there’s a top-level resource, and a resource inside the META-INF directory. While looking at the following code, keep in mind resource encapsulation applies only to resources inside packages in a module.

We’re going to try to access those resources in secondresourcemodule from a class in firstresourcemodule:

Optional<Module> otherModule =
   ModuleLayer.boot().findModule("secondresourcemodule"); 1

otherModule.ifPresent(other -> {
   try {
      InputStream m_tl = other.getResourceAsStream("top_level_resource2.txt"); 2
      InputStream m_pkg = other.getResourceAsStream(
          "javamodularity/secondresourcemodule/resource_in_package2.txt"); 3
      InputStream m_class = other.getResourceAsStream(
          "javamodularity/secondresourcemodule/A.class"); 4
      InputStream m_meta =
          other.getResourceAsStream("META-INF/resource_in_metainf.txt"); 5
      InputStream cz_pkg =
        Class.forName("javamodularity.secondresourcemodule.A")
             .getResourceAsStream("resource_in_package2.txt"); 6

      assert Stream.of(m_tl, m_class, m_meta)
                   .noneMatch(Objects::isNull);
      assert Stream.of(m_pkg, cz_pkg)
                   .allMatch(Objects::isNull);

   } catch (Exception e) {
      throw new RuntimeException(e);
1
A Module can be obtained through the boot layer. The corresponding ModuleLayer API is introduced in the next chapter.

2
Top-level resources from other modules can always be loaded.

3
A resource from a package in another module is encapsulated by default, so this returns null.

4
An exception is made for .class files; these can always be loaded from another module.

5
Because META-INF is never a valid package name, resources from that directory can be accessed as well.

6
While we can get a Class<A> instance by using Class::forName, loading the encapsulated resource through it will return null, just as in (3).

Resource encapsulation applies only to resources in packages. Class file resources (ending with .class) are the exception; they’re not encapsulated even if they’re inside a package. All other resources can be freely used by other modules. Even though you can doesn’t mean you should do this. Relying on resources from another module is not quite modular. It’s best to load resources only from within the same module. When you do need resources from another module, consider exposing the content of the resources through a method on an exported class or even as a service. This way, the dependency becomes explicit in the module descriptor.

EXPOSING RESOURCES IN PACKAGES
You can expose encapsulated resources in packages to other modules by using open modules or open packages. These concepts are introduced in “Open Modules and Packages”. A resource that is in an open module or package can be loaded as if no resource encapsulation is applied.

Working with ResourceBundles
An example where the JDK itself heeds the advice of exporting resources through services is with ResourceBundles. ResourceBundles provide a mechanism for localization as part of the JDK. They’re essentially lists of key-value pairs, specific to a locale. You can implement the ResourceBundle interface yourself, or use, for example, a properties file. The latter is convenient, because there is default support for loading properties files following a predefined format, as shown in Example 5-9.

Example 5-9. Properties files to be loaded by the ResourceBundle mechanism, where Translations is a user-defined basename
Translations_en.properties
Translations_en_US.properties
Translations_nl.properties
You then load a bundle for a specific locale, and get a translation for a key:

Locale locale = Locale.ENGLISH;
ResourceBundle translations =
   ResourceBundle.getBundle("javamodularity.resourcebundle.Translations",
                            locale);
String translation = translations.getString("modularity_key");
The translation properties files live in a package in a module. Traditionally, the getBundle implementation could scan the classpath for files to load. Then, the most appropriate properties file would be selected based on the locale, regardless of which JAR it comes from.

TIP
Explaining how ResourceBundle::getBundle selects the right bundle given a basename and locale is beyond the scope of this book. If you’re unfamiliar with this process, the ResourceBundle JavaDoc contains extensive information about how it loads the most specific file with fallback mechanisms. You will also find that an additional class-based format is supported besides properties files.

With modules, there is no classpath to scan. And you have already seen that resources in modules are encapsulated. Only files within the module calling getBundle are considered.

It’s desirable to put translations for different locales into separate modules. At the same time, opening these modules or packages (see “Exposing Resources in Packages”) has more consequences than just exposing resources. This is why a services-based mechanism for ResourceBundles is introduced in Java 9. It’s based on a new interface called ResourceBundleProvider, containing a single method with the signature ResourceBundle getBundle(String basename, Locale locale). Whenever a module wants to offer additional translations, it can create an implementation of this interface and register it as a service. The implementation then locates the correct resource inside the module and returns it, or returns null if no suitable translation for the given locale is in the module.

Using this pattern, you can now extend the supported locales in an application by adding a module. As long as it registers a ResourceBundleProvider implementation, it’s automatically picked up by the module requesting the translations through ResourceBundle::getBundle. A complete example can be found in the code accompanying this chapter (➥ chapter5/resourcebundles).