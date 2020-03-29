# 第 8 章 迁移到模块

> Chapter 8. Migration to Modules
With all the module goodness from the previous chapters, you are hopefully excited to start using the Java module system. Writing new code based on modules is pretty straightforward now that you understand the basic concepts.

Back in the real world, there’s also a lot of existing code that we may want to migrate to modules. The previous chapter showed how to migrate existing code to Java 9, without transforming the codebase to modules. This is the first step in any migration scenario. With that taken care of, we can focus on migrating toward the Java module system in this chapter.

NOTE
We’re not suggesting that every existing application should be migrated to use the Java module system. If an application is not actively developed anymore, it might not be worth the work. Similarly, small applications may not really benefit from structuring in modules. Migrate to improve maintainability, changeability, and reusability when it makes sense—not just for the sake of it.

The amount of work required for a migration largely depends on how well-structured a codebase is. But even for a well-structured codebase, migration to a modular run-time can be a challenging task. Most applications use third-party libraries, which are an important factor when migrating. These libraries aren’t necessarily modularized yet, nor do you want to take on that responsibility.

Luckily, the Java module system is designed with backward compatibility and migration as a primary concern. Several constructs were introduced in the Java module system to make gradual migration of existing code possible. In this chapter, you will learn about these constructs to migrate your own code to modules. Migration from the perspective of a library maintainer is, of course, related but requires a slightly different process. Chapter 10 focuses on that perspective.

Migration Strategies
A typical application has application code (your code) and library code. The application code uses code from third-party libraries. Ideally, all the libraries used are already modules, and we can focus on modularizing our own code. For the first years after the release of Java 9, this is probably not a realistic scenario. Some libraries may not be available as modules yet, and perhaps never will be, because they are no longer maintained.

If we were to wait until the whole ecosystem moved toward modules, we might have to wait very long. Also, this would require updating to new versions of those libraries, potentially causing its own set of issues. We could also manually patch the libraries, adding a module descriptor and transforming them to a module. This is clearly a lot of work, and requires forking the library, which makes future updates more painful. It would be much better if we can focus on migrating our own code, while leaving libraries the way they are for the moment.

A Simple Example
You will look at several migration examples in this chapter to understand the various cases you can run into in practice. To start, we take a simple application that uses the Jackson library to convert a Java object to JSON. For this application, we need three JAR files from the Jackson project:

com.fasterxml.jackson.core

com.fasterxml.jackson.databind

com.fasterxml.jackson.annotations

The Jackson JAR files in the version used for this example (2.8.8) are not modules yet. They are plain JAR files without module descriptors.

The application consists of two classes, with the main class listed in Example 8-1. Not listed here is the Book class, a simple class with getters and setters representing a book. The Main class contains a main method using ObjectMapper from com.fasterxml.jackson.databind to convert a Book instance to JSON.

Example 8-1. Main.java (➥ chapter8/jackson-classpath)
package demo;

import com.fasterxml.jackson.databind.ObjectMapper;

public class Main {

  public static void main(String... args) throws Exception {
    Book modularityBook =
      new Book("Java 9 Modularity", "Modularize all the things!");

    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(modularityBook);
    System.out.println(json);

  }
}
The com.fasterxml.jackson.databind.ObjectMapper class in the example is part of jackson-databind-2.8.8.jar. This JAR file has a dependency on both jackson-core-2.8.8.jar and jackson-annotations-2.8.8.jar. However, this dependency information is implicit because the JAR files are not modules. The example project has the following file structure to start with:

├── lib
│   ├── jackson-annotations-2.8.8.jar
│   ├── jackson-core-2.8.8.jar
│   └── jackson-databind-2.8.8.jar
└── src
    └── demo
        ├── Book.java
        └── Main.java
As you saw in the previous chapter, the classpath is still available in Java 9. Let’s start with building and running on the classpath, before we start migration to modules. We can build and run the application with the commands in Example 8-2.

Example 8-2. run.sh (➥ chapter8/jackson-classpath)
CP=lib/jackson-annotations-2.8.8.jar:
CP+=lib/jackson-core-2.8.8.jar:
CP+=lib/jackson-databind-2.8.8.jar

javac -cp $CP -d out -sourcepath src $(find src -name '*.java')

java -cp $CP:out demo.Main
This application compiles and runs with Java 9 without any changes.

The Jackson libraries are not directly under our control, but the Main and Book code is, so that’s what we focus on for migration. This is a common migration scenario. We want to move our own code toward modules, without worrying about libraries. The Java module system has some tricks up its sleeve to make gradual migration possible.

Mixing Classpath and Module Path
To make gradual migration possible, we can mix the usage of classpath and module path. This is not an ideal situation, since we only partially benefit from the advantages of the Java module system. However, it is extremely helpful for migrating in small steps.

Because the Jackson libraries are not our own source code, ideally we would not change them at all. Instead we start the migration top-down, by migrating our own code first. Let’s put this code in a module named books. You will see in a moment that this isn’t sufficient, but let’s start by creating a simple module-info.java for our module:

module books {

}
Note that the module doesn’t have any requires statements yet. This is suspicious because we clearly do have a dependency on classes from the jackson-databind-2.8.8.jar JAR file. Because we now have a real module, we can compile our code by using the --module-source-path flag. The Jackson libraries are not modules, so they stay on the classpath for now:

CP=lib/jackson-annotations-2.8.8.jar:
CP+=lib/jackson-core-2.8.8.jar:
CP+=lib/jackson-databind-2.8.8.jar

javac -cp $CP -d out --module-source-path src -m books

src/books/demo/Main.java:3: error:
package com.fasterxml.jackson.databind does not exist

import com.fasterxml.jackson.databind.ObjectMapper;
                                     ^
src/books/demo/Main.java:11: error: cannot find symbol
    ObjectMapper mapper = new ObjectMapper();
    ^
  symbol:   class ObjectMapper
  location: class Main
src/books/demo/Main.java:11: error: cannot find symbol
    ObjectMapper mapper = new ObjectMapper();
                              ^
  symbol:   class ObjectMapper
  location: class Main
3 errors
The compiler is clearly not happy! Although jackson-databind-2.8.8.jar is still on the classpath, the compiler tells us that it is not usable in our module. Modules cannot read the classpath, so our module can’t access types on the classpath, as illustrated in Figure 8-1.

Modules don't read the classpath.
Figure 8-1. Modules don’t read the classpath
Not being able to read from the classpath is a good thing, even when it requires some work during migration, because we want to be explicit about our dependencies. When modules can read the classpath, besides their explicit dependencies, all bets are off.

Even so, our application is not compiling now, so let’s try to fix that first. If we can’t rely on the classpath, the only way forward is to make the code that our module relies on available as a module as well. This requires us to turn jackson-databind-2.8.8.jar into a module.

Automatic Modules
The source code of the Jackson libraries is open source, so we could patch the code to turn it into a module ourselves. In a large application that uses a long list of (transitive) dependencies, patching all of them isn’t appealing. Moreover, we probably don’t have enough knowledge of the libraries to properly modularize them.

The Java module system has a useful feature to deal with code that isn’t a module yet: automatic modules. An automatic module can be created by moving an existing JAR file from the classpath to the module path, without changing its contents. This turns the JAR into a module, with a module descriptor generated on the fly by the module system. In contrast, explicit modules always have a user-defined module descriptor. All modules we’ve seen so far, including platform modules, are explicit modules. Automatic modules behave differently than explicit modules. An automatic module has the following characteristics:

It does not contain module-info.class.

It has a module name specified in META-INF/MANIFEST.MF or derived from its filename.

It requires transitive all other resolved modules.

It exports all its packages.

It reads the classpath (or more precisely, the unnamed module as discussed later).

It cannot have split packages with other modules.

This makes the automatic module immediately usable for other modules. Mind you, it’s not a well-designed module. Requiring all modules and exporting all packages doesn’t sound like proper modularization, but at least it’s usable.

What does it mean to require all other resolved modules? An automatic module requires every module in the already resolved module graph. Remember, there still is no explicit information in an automatic module telling the module system which other modules it really needs. This means the JVM can’t warn at startup when dependencies of automatic modules are missing. We, as developers, are responsible for making sure the module path (or classpath) contains all required dependencies. This is not very different from working with the classpath.

All modules in the module graph are required transitive by automatic modules. This effectively means that if you require one automatic module, you get implied readability to all other modules “for free.” This is a trade-off, which we will discuss in more detail soon.

Let’s take the jackson-databind-2.8.8.jar JAR file and turn it into an automatic module, by moving it to the module path. First we move the JAR file to a new directory that we name mods in this example:

├── lib
│   ├── jackson-annotations-2.8.8.jar
│   └── jackson-core-2.8.8.jar
├── mods
│   └── jackson-databind-2.8.8.jar
└── src
    └── books
        ├── demo
            ├── Book.java
            └── Main.java
        └── module-info.java
Next we have to modify the module-info.java in our books module to require jackson.databind:

module books {
  requires jackson.databind;
}
The books module requires jackson.databind as if it were a normal module. But where did the module name come from? The name of an automatic module can be specified in the newly introduced Automatic-Module-Name field of a META-INF/MANIFEST.MF file. This provides a way for library maintainers to choose a module name even before they fully migrate the library to be a module. See “Choosing a Library Module Name” for more details about naming modules this way.

If no name is specified, the module name is derived from the JAR’s filename. The naming algorithm is roughly the following:

Dashes (-) are replaced by dots (.).

Version numbers are omitted.

In the Jackson example, the module name is based on the filename.

We can now successfully compile the program by using the following command:

CP=lib/jackson-annotations-2.8.8.jar:
CP+=lib/jackson-core-2.8.8.jar

javac -cp $CP --module-path mods -d out --module-source-path src -m books
The jackson-databind-2.8.8.jar JAR file is removed from the classpath, and a module path is now configured, pointing to the mods directory. Figure 8-2 provides an overview of where all the code lives.

Non-modular JARs on the module path become automatic modules. The classpath becomes the unnamed module.
Figure 8-2. Nonmodular JARs on the module path become automatic modules. The classpath becomes the unnamed module.
To run the program, we also have to update the java invocation:

java -cp $CP --module-path mods:out -m books/demo.Main
We made the following change to the java command:

Move the out directory to the module path.

Move jackson-databind-2.8.8.jar from the classpath (lib) to the module path (mods).

Start the application by using the -m flag to specify our module.

TIP
Instead of moving just jackson-databind to the module path, we could have moved all the JARs to the module path. This makes the process a little easier, but makes it harder to see what happens. Feel free to move all JARs to the module path when migrating your own applications.

We’re close to our first migrated application, but unfortunately we still get an exception when starting the application:

Exception in thread "main" java.lang.reflect.InaccessibleObjectException:
  Unable to make public java.lang.String demo.Book.getTitle() accessible:
  module books does not "exports demo" to module jackson.databind
  ...
This is a problem specific to Jackson Databind, but not an uncommon scenario. We use Jackson Databind to marshal the Book class, which is part of the books module. Jackson Databind uses reflection to look at the fields of a class to be able to serialize it. Therefore, Jackson Databind needs to access the Book class; otherwise, it can’t use reflection to look at its fields. For this to be possible, the package containing the class must be exported or opened by its containing module (books, in this example). Exporting the package restricts Jackson Databind to reflect only over public elements, whereas opening the package allows for deep reflection as well. In our example, reflecting over public elements is enough.

This puts us in a difficult position. We don’t necessarily want to export the package containing Book to other modules, just because Jackson needs it. If we did, we would give up on encapsulation, and that was one of the primary reasons to move to modules! There are multiple ways to work around this problem, each with its own trade-offs. The first way is to use a qualified export. Using a qualified export, we can export the package to just jackson.databind, so we don’t lose encapsulation with respect to other modules:

module books {
  requires jackson.databind;

  exports demo to jackson.databind;
}
After recompiling, we can now successfully run the application! We have another option besides exporting that may better fit our needs when it comes to reflection, and we will explore this in the next section.

WARNINGS WHEN USING AUTOMATIC MODULES
Although automatic modules are essential to migration, they should be used with some caution. Whenever you write a requires on an automatic module, make a mental note to come back to this later. If the library is released as an explicit module, you want to use that instead.

Two warnings were added to the compiler to help with this as well. Note that it is only a recommendation that Java compilers support these warnings, so different compiler implementations may have different results. The first warning is opt out (enabled by default), and will give a warning for every requires transitive on an automatic module. The warning can be disabled with the -Xlint:-requires-transitive-automatic flag. Notice the dash (-) after the colon. The second warning is opt in (disabled by default), and will give a warning for every requires on an automatic module. This warning can be enabled with the -Xlint:requires-automatic (no dash after the colon) flag. The reason that the first warning is enabled by default is that it is a more dangerous scenario. You expose a (possibly volatile) automatic module to consumers of your module through implied readability.

Replace automatic modules with explicit modules when available, and ask the library maintainer for one if not available yet. Also remember that such a module may have a more restricted API, because the default of having all packages exported is likely not what the library maintainer intended. This can lead to extra work when switching from an automatic module to an explicit module, where the library maintainer has created a module descriptor.

Open Packages
Using exports in the context of reflection has some caveats. First of all, it’s arguably strange that we need to give compile-time readability to a package, while we only expect run-time (reflection) usage. Frameworks often use reflection to work on application code, but they don’t need compile-time readability. Also, we might not always know which module requires readability up front, so a qualified export isn’t possible.

Using the Java Persistence API (JPA) is an example of such a scenario. When using JPA, you typically program to the standardized API. At run-time, you would use an implementation of this API such as Hibernate or EclipseLink. The API and the implementation live in separate modules. In the end, the implementation needs accessibility to your classes. If we put an exports com.mypackage to hibernate.core or something like that in our module, we’d be suddenly coupled to the implementation. Changing JPA implementations would require us to change the module descriptor of our code, which would be a clear sign of leaking implementation details.

As discussed in detail in “Open Modules and Packages”, another problem arises when it comes to reflection. Exporting a package exports only the public types in the package. Protected or package-private classes, and nonpublic methods and fields in exported classes, are not accessible. Deep reflection, using the setAccessible method, will not work even when a package is exported. To allow deep reflection (which many frameworks need), a package must be open.

Going back to our Jackson example, we can use the opens keyword instead of the qualified export to jackson.databind:

module books {
  requires jackson.databind;

  opens demo;
}
An open package gives run-time access (including deep reflection) to its types to any module, but no compile-time access. This avoids others using your implementation code accidentally at compile-time, while frameworks can do their magic without problems at run-time. When only run-time access is required, opens is a good choice in most cases. Remember that an open package is not truly encapsulated. Another module can always access the package by using reflection. But at least we’re protected against accidental usage during development, and it clearly documents that the package isn’t meant to be used directly by other modules.

Like the exports keyword, the opens keyword can be qualified. This way, a package can be opened just to a limited set of modules:

module books {
  requires jackson.databind;

  opens demo to jackson.databind;
}
Now that you have seen two ways to work around the run-time accessibility problem, one question remains: why did we find out about this problem only when running the application, and not during compilation? Let’s revisit the rules of readability again to understand this scenario better. For a class to able to read another class from another module, the following needs be true:

The class needs to be public (ignoring the case of deep reflection).

The package in the other module must be exported, or opened in the case of deep reflection.

The consuming module must have a readability relation to the other module (requires).

Usually, this can all be checked at compile-time. Jackson Databind doesn’t have a compile-time dependency on our code, however. It learns about our Book class only because we pass it in as an argument to ObjectMapper. This means the compiler can’t help us. When doing reflection, the runtime takes care of automatically setting up a readability relation (requires), so this step is taken care of. Next it will find out that the class is not exported nor opened (and therefore not accessible) at run-time, and this is not automatically “fixed” by the runtime.

If the runtime is smart enough to add a readability relation automatically, why doesn’t it also take care of opening the package? This is about intentions and module ownership. When code uses reflection to access code in another module, the intention from that module’s perspective is clearly to read the other module. It would be unnecessary extra boilerplate to be (even more) explicit about this. The same is not true for exports/opens. The module owner should decide which packages are exported or opened. Only the module itself should define this intention, so it can’t be automatically inferred by the behavior of some other module.

Many frameworks use reflection in a similar way, so it’s always important to test well after migration.

TIP
In “Libraries, Strong Encapsulation, and the JDK 9 Classpath”, you learned that by default Java 9 runs with --illegal-acces=permit. Why did we still have to explicitly open our package for reflection? Remember that the --illegal-access flag affects only code on the classpath. In this example, jackson.databind is a module itself, reflecting on code in our module (not a platform module). There’s no code on the classpath involved.

Open Modules
In the previous section, you saw the use of open packages to give run-time-only access to a package. This is great to satisfy the reflection needs that many frameworks and libraries have. If we’re in the middle of a large migration, in a codebase that is not completely modularized yet, it may not be immediately obvious which packages need to be open. Ideally, we know exactly how the frameworks and libraries that we use access our code, but maybe we’re not intimately familiar with the codebase we’re working with. This could result in a tedious trial-and-error process, trying to figure out which packages need to be open. For these situations, we can use open modules, which is a less precise but more powerful tool:

open module books {
  requires jackson.databind;
}
An open module is a module that gives run-time access to all its packages. This does not grant compile-time access to packages, which is exactly what we want for our migrated code. If a package needs to be used at compile-time, it must be exported. Create an open module first to avoid reflection-related problems. This helps you focus on requires and compile-time usage (exports) first. Once the application is working again, we can fine-tune run-time access to the packages as well by removing the open keyword from the module, and be more specific about which packages should be open.

VM Arguments to Break Encapsulation
In some scenarios, adding export or opens to a module is not an option. Maybe we don’t have access to the code, or maybe access is required only during testing. In these scenarios, we can set up additional exports by using a VM argument. You have already seen this in action in “Compilation and Encapsulated APIs” for platform modules, and we can do the same for other modules, including our own.

Instead of adding an exports or opens clause to the books module descriptor, we can use a command-line flag to achieve the same:

--add-exports books/demo=jackson.databind
The complete command to run the application then is as follows:

java -cp lib/jackson-annotations-2.8.8.jar:lib/jackson-core-2.8.8.jar \
  --module-path out:mods \
  --add-exports books/demo=jackson.databind \
  -m books/demo.Main
This sets up a qualified export when starting the JVM. A similar flag exists to open packages: --add-opens. Although these flags are useful in special cases, they should be treated as a last resort. The same mechanism can also be used to gain access to internal, nonexported packages, as you saw in the previous chapter. Although this can be a temporary workaround until code is properly migrated, it should be used with great care. Breaking encapsulation should not be done lightly.

Automatic Modules and the Classpath
In the previous chapter, you saw the unnamed module. All code on the classpath is part of the unnamed module. In the Jackson example, you learned that code in a module you’re compiling can’t access code on the classpath. How is it then possible for the jackson.databind automatic module to work correctly while the Jackson Core and Jackson Annotations JARs, which it depends on, are still on the classpath? This works because these libraries are in the unnamed module. The unnamed module exports all code on the classpath and reads all other modules. There is a big restriction, however: the unnamed module itself is readable only from automatic modules!

Figure 8-3 illustrates the difference between automatic modules and explicit modules when it comes to reading the unnamed module. An explicit module can read only other explicit modules, and automatic modules. An automatic module reads all modules including the unnamed module.

Only automatic modules can read the classpath.
Figure 8-3. Only automatic modules can read the classpath
The readability to the unnamed module is only a mechanism to facilitate automatic modules in a mixed classpath/module path migration scenario. If we were to use a type from Jackson Core directly in our code (as opposed to from an automatic module), we would have to move Jackson Core to the module path as well. Let’s try this in Example 8-3.

Example 8-3. Main.java (➥ chapter8/readability_rules)
package demo;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.Versioned; 1

public class Demo {

  public static void main(String... args) throws Exception {
    Book modularityBook =
      new Book("Java 9 Modularity", "Modularize all the things!");

    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(modularityBook);
    System.out.println(json);

    Versioned versioned = (Versioned) mapper; 2
    System.out.println(versioned.version());

  }
}
1
Importing the Versioned type from Jackson Core

2
Using the Versioned type to print the library version

The Jackson Databind ObjectMapper type implements the interface Versioned from Jackson Core. Note that this wasn’t a problem until we started to use this type explicitly in our module. Any time we use an external type in a module, we should immediately think requires. Let’s demonstrate this by trying to compile the code, which will result in an error:

src/books/demo/Main.java:4: error:
package com.fasterxml.jackson.core does not exist

import com.fasterxml.jackson.core.Versioned;
                                 ^
src/books/demo/Main.java:16: error:
cannot find symbol
    Versioned versioned = (Versioned)mapper;
    ^
  symbol:   class Versioned
  location: class Main
src/books/demo/Main.java:16: error:
cannot find symbol
    Versioned versioned = (Versioned)mapper;
                           ^
  symbol:   class Versioned
  location: class Main
3 errors
Although the type exists in the unnamed module (the classpath), and the jackson.databind automatic module can access it, we can’t access it from our module. To fix this problem, we need to move Jackson Core to the module path as well, making it an automatic module. Let’s start with moving the JAR file to the mods directory, and removing it from the classpath, as we did for Jackson Databind:

javac -cp lib/jackson-annotations-2.8.8.jar \
  --module-path mods \
  -d out \
  --module-source-path src \
  -m books
This works! Taking a step back, however, why does it work? We’re clearly using a type from jackson.core, but we don’t have a requires for jackson.core in our module-info.java. Why didn’t the compilation fail? Remember that an automatic module requires transitive all other modules. This means that by requiring jackson.databind, we also read jackson.core transitively. Although this is convenient, it’s a big trade-off. We have an explicit code dependency on a module that we don’t explicitly require. If jackson.databind moves to become an explicit module, and the Jackson maintainers choose to not requires transitive jackson.core, our code would suddenly break.

Be aware that although automatic modules look like modules, they miss the metadata to provide reliable configuration. In this specific example, it is better to explicitly add a requires to jackson.core as well:

module books {
  requires jackson.databind;
  requires jackson.core;

  opens demo;
}
Now that we’re compiling happily again, we can also adapt the run command. We have to remove only the Jackson Core JAR file from the classpath, because the module path was already configured correctly:

java \
  -cp lib/jackson-annotations-2.8.8.jar \
  --module-path out:mods \
  -m books/demo.Main
If you are wondering why jackson.core was resolved in the first place (it’s not explicitly added to the module graph as a root module, and no explicit module depends on it directly), you have been paying attention! “Module Resolution and the Module Path” discussed in detail that the set of resolved modules is calculated based on a given set of root modules. In the case of automatic modules this would be confusing. Automatic modules don’t have explicit dependencies, so they would not cause their transitive dependencies that are also automatic modules to be resolved. Because it would be time-consuming to add dependencies manually with --add-modules, all automatic modules on the module path are automatically resolved when the application requires any one of them. This behavior can cause unused automatic modules to be resolved (taking unnecessary resources), so keep the module path clean.

The behavior is similar to what we’re used to with the classpath, however. Again, this illustrates how automatic modules are a feature squarely aimed at migrating from the classpath to modules.

Why isn’t the JVM smarter about this? It has access to all the code in an automatic module, after all, so why not analyze dependencies? To analyze whether any of this code calls into other modules, the JVM needs to perform bytecode analysis of all the code. Although this isn’t difficult to implement, it is an expensive operation potentially adding a lot of startup time in a large application. Moreover, such an analysis won’t find dependencies arising through reflection. Because of these limitations, the JVM doesn’t and probably never will do this. Instead, the JDK ships with another tool, jdeps, that does perform such bytecode analysis.

Using jdeps
In the preceding Jackson example, we’ve used kind of a trial-and-error approach to migrating the code. This gave you a good understanding of what happens, but is not efficient. jdeps is a tool shipped with the JDK that analyzes code and gives insights about module dependencies. We can use jdeps to optimize the process we’ve gone through to migrate the Jackson example.

We will start by using jdeps to analyze the classpath version of the demo, before we migrated to (automatic) modules. jdeps analyzes bytecode, not source files, so we’re interested only in the application’s output folder and JAR files. For reference, here’s the compiled version of the books example when using just the classpath, as in the beginning of this chapter:

├── lib
│   ├── jackson-annotations-2.8.8.jar
│   ├── jackson-core-2.8.8.jar
│   └── jackson-databind-2.8.8.jar
└── out
    └── demo
        ├── Book.class
        └── Main.class
We can analyze this application by using the following command:

$ jdeps -recursive -summary -cp lib/*.jar out

jackson-annotations-2.8.8.jar -> java.base
jackson-core-2.8.8.jar -> java.base
jackson-databind-2.8.8.jar -> lib/jackson-annotations-2.8.8.jar
jackson-databind-2.8.8.jar -> lib/jackson-core-2.8.8.jar
jackson-databind-2.8.8.jar -> java.base
jackson-databind-2.8.8.jar -> java.desktop
jackson-databind-2.8.8.jar -> java.logging
jackson-databind-2.8.8.jar -> java.sql
jackson-databind-2.8.8.jar -> java.xml
out -> lib/jackson-databind-2.8.8.jar
out -> java.base
The -recursive flag makes sure that transitive run-time dependencies are also analyzed. Without it, jackson-annotations-2.8.8.jar would not be analyzed, for example. The -summary flag, as the name suggests, summarizes the output. By default, jdeps outputs the complete list of dependencies for each package, which can be a long list. The summary shows module dependencies only, and hides the package details. The -cp argument is the classpath we want to use during the analysis, which should correspond to the run-time classpath. The out directory contains the application’s class files that must be analyzed.

We learn several things from the jdeps output:

Our own code (in directory out) has only a direct, compile-time dependency on jackson-databind-2.8.8.jar (and java.base, of course).

Jackson Databind has a dependency on Jackson Core and Jackson Annotations.

Jackson Databind has dependencies on several platform modules.

Based on this output, we can already conclude that to migrate our code to a module, we need to make jackson-databind an automatic module as well. We also see that jackson-databind depends on jackson-core and jackson-annotations, so they need to be provided either on the classpath or as automatic modules. If we want to know why a dependency exists, we can print more details by using jdeps. Omitting the -summary argument in the preceding command prints the full dependency graph, showing exactly which packages require which other packages:

$ jdeps -cp lib/*.jar out

com.fasterxml.jackson.databind.util (jackson-databind-2.8.8.jar)
      -> com.fasterxml.jackson.annotation jackson-annotations-2.8.8.jar
      -> com.fasterxml.jackson.core jackson-core-2.8.8.jar
      -> com.fasterxml.jackson.core.base jackson-core-2.8.8.jar

... Results truncated for readability
If this is still not enough detail, we can also instruct jdeps to print dependencies at the class level:

$ jdeps -verbose:class -cp lib/*.jar out

out -> java.base
   demo.Main (out)
      -> java.lang.Object
      -> java.lang.String
   demo.Main (out)
      -> com.fasterxml.jackson.databind.ObjectMapper jackson-databind-2.8.8.jar

... Results truncated for readability
So far, we have used jdeps on a classpath-based application. We can also use jdeps with modules. Let’s in this case try jdeps on the Jackson demo, where all Jackson JARs are available as automatic modules:

├── mods
│   ├── jackson-annotations-2.8.8.jar
│   ├── jackson-core-2.8.8.jar
│   └── jackson-databind-2.8.8.jar
├── out
│   └── books
│       ├── demo
│       │   ├── Book.class
│       │   └── Main.class
│       └── module-info.class
To invoke jdeps, we now have to pass in the module path that contains the application module and the automatic Jackson modules:

$ jdeps --module-path out:mods -m books
This prints the dependency graph as before. Looking at the (long) output, we can see the following:

module jackson.databind (automatic)
 requires java.base
   com.fasterxml.jackson.databind
     -> com.fasterxml.jackson.annotation jackson.annotations
   com.fasterxml.jackson.databind
     -> com.fasterxml.jackson.core jackson.core
   com.fasterxml.jackson.databind
     -> com.fasterxml.jackson.core.filter jackson.core

...

module books
 requires jackson.databind
 requires java.base
  demo -> com.fasterxml.jackson.databind jackson.databind
  demo -> java.io java.base
  demo -> java.lang java.base
We can see that jackson.databind has a dependency on both jackson.annotations and jackson.core. We also see that books has only a dependency on jackson.databind. The books code is not using jackson.core classes at compile-time, and transitive dependencies are not defined for automatic modules. Remember that the JVM won’t do this analysis at application startup, which means we have to take care of adding jackson.annotations and jackson.core to either the classpath or the module path ourselves. jdeps provides the information to set this up correctly.

TIP
jdeps can output dot files for module graphs using the -dotoutput flag. This is a useful format to represent graphs, and it’s easy to generate images from this format. Wikipedia has a nice introduction to the format.

Loading Code Dynamically
A situation that might require some special care when migrating to modules is the use of reflection to load code. A well-known example is loading JDBC drivers. You will see that in most cases loading a JDBC driver “just works,” but some corner cases give us better insights into the module system. Let’s start with an example that loads a JDBC driver that sits in the mods directory of the project as shown in Example 8-4. The HSQLDB driver JAR is not a module yet, so we can use it only as an automatic module.

Because the name of the class is just a string, the compiler will not know about the dependency, so compilation succeeds.

Example 8-4. Main.java (➥ chapter8/runtime_loading)
package demo;

public class Main {

  public static void main(String... args) throws Exception {
    Class<?> clazz = Class.forName("org.hsqldb.jdbcDriver");
    System.out.println(clazz.getName());
  }
}
The module descriptor (shown in Example 8-5) is empty; it does not require hsqldb, which is the driver we’re trying to load. Although this would normally be a reason to be suspicious, it could still work in theory, because the runtime will automatically create a readability relation when using reflection on code in another module.

Example 8-5. module-info.java (➥ chapter8/runtime_loading)
module runtime.loading.example {
}
Still, if we run the code, it fails with a ClassNotFoundException:

java --module-path mods:out -m runtime.loading.example/demo.Main

Exception in thread "main" java.lang.ClassNotFoundException:
  org.hsqldb.jdbcDriver
All observable automatic modules are resolved when the application uses at least one of them. Resolving happens at startup, so effectively our module is not causing the automatic module to load. If we had other automatic modules that are required directly, this would cause the hsqldb module to resolve as well as a side effect. In this case, we can add the automatic module ourselves with --add-modules hsqldb.

Now the driver loads but gives another error, because the driver depends on java.sql, which wasn’t resolved yet. Remember, automatic modules lack the metadata to specifically require other modules. When using JDBC in practice, we would need to require java.sql in our module to be able to use the JDBC API after loading the driver. This means adding it to the module descriptor as shown in Example 8-6.

Example 8-6. module-info.java (➥ chapter8/runtime_loading)
module runtime.loading.example {
   requires java.sql;
}
The code now runs successfully. With java.sql required by our module, we can see another interesting case of automatic module resolving. If we remove --add-modules hsqldb again, the application still runs! Why does requiring java.sql cause an automatic module to be loaded? It turns out that java.sql defines a java.sql.Driver service interface, and has a uses constraint for this service type as well. Our hsqldb JAR provides a service, which is registered via the “old” way of using a file in META-INF/services. Because of service binding, the JAR is automatically resolved from the module path. This goes into the subtleties of the module system but is good to understand.

Why didn’t we just put requires hsqldb in our module descriptor? Although typically we like to be as explicit as possible about dependencies by putting them in the module descriptor, this is a good example of where this rule of thumb does not apply. The JDBC driver to use often depends on the deployment environment of the application, where the exact driver name is configured in a configuration file. Application code should not be coupled to a specific database driver in that case (although in our example, it is). Instead we simply make sure that the driver is resolved by adding --add-modules. The module containing the driver will be in the resolved module graph, and the reflective instantiation establishes the readability relation to this module.

If the JDBC driver supports it (as HSQLDB does), it is even better to avoid reflective instantiation from application code altogether and use services instead. Services are discussed in detail in Chapter 4.

Split Packages
“Split Packages” explained the problem of split packages. Just as a refresher, a split package means two modules contain the same package. The Java module system doesn’t allow split packages.

When using automatic modules, we can run into split packages as well. In large applications, it is common to find split packages due to dependency mismanagement. Split packages are always a mistake, because they don’t work reliably on the classpath either. Unfortunately, when using build tools that resolve transitive dependencies, it’s easy to end up with multiple versions of the same library. The first class found on the classpath is loaded. When classes from two versions of a library mix, this often leads to hard-to-debug exceptions at run-time.

TIP
Modern build tools often have a setting to fail on duplicate dependencies. This makes dependency management issues clearer and forces you to deal with them early. It is highly recommended to use this.

The Java module system is much stricter about this issue than the classpath. When it detects that a package is exported from two modules on the module path, it will refuse to start. This fail-fast mechanism is much better than the unreliable situation we used to have with the classpath. Better to fail during development than in production, when some unlucky user hits a code path that is broken by an obscure classpath problem. But it also means we have to deal with these issues. Blindly moving all JARs from the classpath to the module path may result in split packages between the resulting automatic modules. These will then be rejected by the module system.

To make migration a little easier, an exception to this rule exists when it comes to automatic modules and the unnamed module. It acknowledges that a lot of classpaths are simply incorrect and contain split packages. When both a (automatic) module and the unnamed module contain the same package, the package from the module will be used. The package in the unnamed module will be ignored. This is also the case for packages that are part of the platform modules. It’s common to override platform packages by putting them on the classpath. This approach is no longer working in Java 9. You already saw this in the previous chapter, and learned that java.se.ee modules are not included in the java.se module for this reason.

If you run into split package issues while migrating to Java 9, there’s is no way around them. You must deal with them, even when your classpath-based application works correctly from a user’s perspective.

This chapter presented many techniques to make gradual migration to the Java module system possible. These techniques are extremely valuable, because it will take time before the Java ecosystem moves to the Java module system in its entirety. Automatic modules play an important role in migration scenarios; therefore, it’s important to fully understand how they work.