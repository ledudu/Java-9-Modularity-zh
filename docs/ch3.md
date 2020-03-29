# 第 3 章 使用模块

> Chapter 3. Working with Modules

In this chapter, we take the first steps towards modular development using Java 9. Instead of looking at existing modules in the JDK, it’s time to get your hands dirty by writing your first module. To start things off easily, we turn our attention to the simplest possible module. Let’s call it our Modular Hello World. Armed with this experience, we’ll then be ready to take on a more ambitious example with multiple modules. At that point, we’ll introduce the running example to be used throughout this book, called EasyText. It’s designed to gradually grow with you, as you learn more about the module system.

Your First Module
You’ve seen examples of module descriptors in the previous chapter. A module is generally more than just a descriptor, though. The Modular Hello World therefore transcends the level of a single source file: we need to examine it in context. We’ll start by compiling, packaging, and running a single module to get acquainted with the new tooling options for modules.

Anatomy of a Module
Our goal for this first example is to compile the following class into a module and run it (Example 3-1). We start out with a single class in a package, leading to a single module. Modules may contain only types that are inside packages, so a package definition is required.

Example 3-1. HelloWorld.java (➥ chapter3/helloworld)
package com.javamodularity.helloworld;

public class HelloWorld {

   public static void main(String... args) {
      System.out.println("Hello Modular World!");
   }

}
The layout of the sources on the filesystem looks as follows:

src
└── helloworld 1
    ├── com
    │   └── javamodularity
    │       └── helloworld
    │           └── HelloWorld.java
    └── module-info.java 2
1
Module directory

2
Module descriptor

Compared to the traditional layout of Java source files, there are two major differences. First, there is an extra level of indirection: below src we introduce another directory, helloworld. The directory is named after the name of the module we’re creating. Second, inside this module directory we find both the source file (nested in its package structure as usual) and a module descriptor. A module descriptor lives in module-info.java and is the key ingredient to Java modules. Its presence signals to the Java compiler that we are working with a module rather than plain Java sources. Compiler behavior is quite different when working with modules as compared to plain Java source files, as you will see in the remainder of this chapter. The module descriptor must be present in the root of the module directory. It is compiled along with the other source files into a binary class-file called module-info.class.

So what’s inside the module descriptor? Our Modular Hello World example is quite minimalistic:

module helloworld {

}
We declare a module by using the new module keyword, followed by the module name. The name must match the name of the directory containing the module descriptor. Otherwise, the compiler refuses to compile and reports the mismatch.

NOTE
This name-matching requirement is true only when running the compiler in multimodule mode, which is a common scenario. For the single-module scenario discussed in “Compilation”, the directory name does not matter. In any case, it’s still a good idea to use the name of the module as the directory name.

Because of the empty module declaration body, nothing from the helloworld module is exported to other modules. By default, all packages are strongly encapsulated. Even though there’s no dependency information (yet) in this declaration, remember that this module implicitly depends on the java.base platform module.

You may be wondering whether adding a new keyword to the language breaks existing code that uses module as an identifier. Fortunately, that’s not the case. You can still use identifiers called module in your other source files, because the module keyword is a restricted keyword. It is treated as a keyword only inside module-info.java. The same holds for the requires keyword and other new keywords you’ve seen so far in module descriptors.

THE MODULE-INFO NAME
Normally, the name of a Java source file corresponds to the (public) type it contains. For example, our file containing the HelloWorld class must be in a file named HelloWorld.java. The module-info name breaks this correspondence. Moreover, module-info is not even a legal Java identifier, because it contains a dash. This is done on purpose, to prevent non-module-aware tools from blindly processing module-info.java or module-info.class as if it were a normal Java class.

Reserving a name for special source files is not unprecedented in the Java language. Before module-info.java, there was package-info.java. Although it is relatively unknown, it’s been around since Java 5. In package-info.java, you can add documentation and annotations to a package declaration. Like module-info.java, it is compiled to a class file by the Java compiler.

We now have a module descriptor containing nothing but a module declaration, and a single source file. Enough to compile our first module!

Naming Modules
Naming things, while hard, is important. Especially for modules, since they will convey the high-level structure of your application.

Module names live in a global namespace separate from other namespaces in Java. So, theoretically, you could give a module the same name as a class, interface, or package. In practice, this might lead to confusion.

Module names must be unique: an application can have only a single module for any given name. In Java it’s customary to make package names globally unique by using reverse DNS notation. You could apply the same reasoning to modules. For example, you could rename the helloworld module to com.javamodularity.helloworld. However, this leads to long and somewhat clunky module names.

Is it really necessary for module names in applications to be globally unique? Certainly, when your module is a published library, used in many applications, it makes sense to choose a globally unique module name. In “Choosing a Library Module Name”, we discuss this notion further. For application modules, there’s no shame in choosing shorter and more memorable names.

In this book we prefer shorter module names to increase the readability of examples.

Compilation
Having a module in source format is one thing, but we can’t run it without compiling first. Prior to Java 9, the Java compiler is invoked with a destination directory and a set of sources to compile:

javac -d out src/com/foo/Class1.java src/com/foo/Class2.java
In practice, this is often done under the hood by build tools such as Maven or Gradle, but the principle remains the same. Classes are emitted in the target directory (out in this case) with nested folders representing the input (package) structure. Following the same pattern, we can compile our Modular Hello World example:

javac -d out/helloworld \
      src/helloworld/com/javamodularity/helloworld/HelloWorld.java \
      src/helloworld/module-info.java
There are two notable differences:

We output into a helloworld directory, reflecting the module name.

We add module-info.java as an additional source file to compile.

The presence of module-info.java in the set of files to be compiled triggers the module-aware mode of javac. Running the compilation results in the following output, also known as the exploded module format:

out
└── helloworld
    ├── com
    │   └── javamodularity
    │       └── helloworld
    │           └── HelloWorld.class
    └── module-info.class
It’s best to name the directory containing the exploded module after the module, but not required. Ultimately, the module system takes the name of the module from the descriptor, not from the directory name. In “Running Modules”, we’ll take this exploded module and run it.

COMPILING MULTIPLE MODULES
What you’ve seen so far is the so-called single-module mode of the Java compiler. Typically, the project you want to compile consists of multiple modules. These modules may or may not refer to each other. Or the project might be a single module but uses other (already compiled) modules. For these cases, additional compiler flags have been introduced: --module-source-path and --module-path. These are the module-aware counterparts of the -sourcepath and -classpath flags that have been part of javac for a long time. Their semantics are explained when we start looking at multimodule examples in “A Tale of Two Modules”. Keep in mind, the name of the module source directories must match the name declared in module-info.java in this multimodule mode.

BUILD TOOLS
It is not common practice to use the Java compiler directly from the command line, manipulating its flags and listing all source files manually. More often, build tools such as Maven or Gradle are used to abstract away these details. For this reason, we won’t cover all the nitty-gritty details of every new option added to the Java compiler and runtime in this book. You can find comprehensive coverage of these in the official documentation. Of course, build tools need to adapt to the new modular reality as well. In Chapter 11, we show how some of the most popular build tools can be used with Java 9.

Packaging
So far, we’ve created a single module and compiled it into the exploded module format. In the next section, we show you how to run such an exploded module as is. This works in a development situation, but in production scenarios you want to distribute your module in a more convenient format. To this end, modules can be packaged and used in JAR files. This results in modular JAR files. A modular JAR file is similar to a regular JAR file, except it also contains a module-info.class.

The JAR tool has been updated to work with modules in Java 9. To package up the Modular Hello World example, execute the following command:

jar -cfe mods/helloworld.jar com.javamodularity.helloworld.HelloWorld \
    -C out/helloworld .
With this command, we’re creating a new archive (-cf) called helloworld.jar in the mods directory (make sure the directory exists, though). Furthermore, we want the entry point (-e) for this module to be the HelloWorld class; whenever the module is started without specifying another main class to run, this is the default. We provide the fully qualified classname as an argument for the entry point. Finally, we instruct the jar tool to change (-C) to the out/helloworld directory and put all compiled files from this directory in the JAR file. The contents of the JAR are now similar to the exploded module, with the addition of a MANIFEST.MF file:

helloworld.jar
├── META-INF
|   └── MANIFEST.MF
├── com
│   └── javamodularity
│       └── helloworld
│           └── HelloWorld.class
└── module-info.class
The filename of the modular JAR file is not significant, unlike the situation with the module directory name during compilation. You can use any filename you like, since the module is identified by the name declared in the bundled module-info.class.

Running Modules
Let’s recap what we did so far. We started our Modular Hello World example by creating a helloworld module with a single HelloWorld.java source file and a module descriptor. Then we compiled the module into the exploded module format. Finally, we took the exploded module and packaged it as a modular JAR file. This JAR file contains our compiled class and the module descriptor, and knows about the main class to execute.

Now let’s try to run the module. Both the exploded module format and modular JAR file can be run. The exploded module format can be started with the following command:

$ java --module-path out \
       --module helloworld/com.javamodularity.helloworld.HelloWorld
Hello Modular World!
TIP
You can also use the short-form -p flag instead of --module-path. The --module flag can be shortened to -m.

The java command has gained new flags to work with modules in addition to classpath-based applications. Notice we put the out directory (containing the exploded helloworld module) on the module path. The module path is the module-aware counterpart of the original classpath.

Next, we provide the module to be run with the --module flag. In this case, it consists of the module name followed by a slash and then the class to be run. On the other hand, if we run our modular JAR, providing just the module name is enough:

$ java --module-path mods --module helloworld
Hello Modular World!
This makes sense, because the modular JAR knows the class to execute from its meta-data. We explicitly set the entry point to com.javamodularity.helloworld.HelloWorld when constructing the modular JAR.

TIP
The --module or -m flag with corresponding module name (and optional main class) must always come last. Any subsequent arguments are passed to the main class being started from the given module.

Launching in either of these two ways makes helloworld the root module for execution. The JVM starts from this root module, and resolves any other modules necessary to run the root module from the module path. Resolving modules is a recursive process: if a newly resolved module requires other modules, the module system automatically takes this into account, as discussed earlier in “Module Resolution and the Module Path”.

In our simple helloworld example, there is not too much to resolve. You can trace the actions taken by the module system by adding --show-module-resolution to the java command:

$ java --show-module-resolution --limit-modules java.base \
       --module-path mods --module helloworld
root helloworld file:///chapter3/helloworld/mods/helloworld.jar
Hello Modular World!
(The --limit-modules java.base flag is added to prevent other platform modules from being resolved through service binding. Service binding is discussed in the next chapter.)

In this case, no other modules are required (besides the implicitly required platform module java.base) to run helloworld. Only the root module, helloworld, is shown in the module resolution output. This means running the Modular Hello World example involves just two modules at run-time: helloworld and java.base. Other platform modules, or modules on the module path, are not resolved. During classloading, no resources are wasted searching through classes that aren’t relevant to the application.

TIP
Even more diagnostic information about module resolution can be shown with -Xlog:module=debug. Options starting with -X are nonstandard, and may not be supported on Java implementations that are not based on OpenJDK.

An error would be encountered at startup if another module were necessary to run helloworld and it’s not present on the module path (or part of the JDK platform modules). This form of reliable configuration is a huge improvement over the old classpath situation. Before the module system, a missing dependency is noticed only when the JVM tries to load a nonexistent class at run-time. Using the explicit dependency information of module descriptors, module resolution ensures a working configuration of modules before running any code.

Module Path
Even though module path sounds quite similar to classpath, they behave differently. The module path is a list of paths to individual modules and directories containing modules. Each directory on the module path can contain zero or more module definitions, where a module definition can be an exploded module or a modular JAR file. An example module path containing all three options looks like this: out/:myexplodedmodule/:mypackagedmodule.jar. All modules inside the out directory are on the module path, in conjunction with the module myexplodedmodule (a directory) and mypackagedmodule (a modular JAR file).

TIP
Entries on the module path are separated by the default platform separator. On Linux/macOS, that’s a colon (java -p dir1:dir2); on Windows, use a semicolon (java -p dir1;dir2). The -p flag is short for --module-path.

Most important, all artifacts on the module path have module descriptors (possibly synthesized on the fly, as we will learn in “Automatic Modules”). The resolver relies on this information to find the right modules on the module path. When multiple modules with the same name are in the same directory on the module path, the resolver shows an error and won’t start the application. Again, this prevents scenarios with conflicting JAR files that were previously possible on the classpath.

WARNING
When multiple modules with the same name are in different directories on the module path, no error is produced. Instead, the first module is selected and subsequent modules with the same name are ignored.

Linking Modules
In the previous section, you saw that the module system resolved only two modules: helloworld and java.base. Wouldn’t it be great if we could take advantage of this up-front knowledge by creating a special distribution of the Java runtime containing the bare minimum to run our application? That’s exactly what you can do in Java 9 with custom runtime images.

An optional linking phase is introduced with Java 9, between the compilation and run-time phases. With a new tool called jlink, you can create a runtime image containing only the necessary modules to run an application. Using the following command, we create a new runtime image with the helloworld module as root:

$ jlink --module-path mods/:$JAVA_HOME/jmods \
        --add-modules helloworld \
        --launcher hello=helloworld \
        --output helloworld-image
TIP
The jlink tool lives in the bin directory of the JDK installation. It is not added to the system path by default, so in order to use it as in the preceding example, you must add it to the path first.

The first option constructs a module path containing the mods directory (where helloworld lives) and the directory of the JDK installation containing the platform modules we want to link into the image. Unlike with javac and java, you have to explicitly add platform modules to the jlink module path. Then, --add-modules indicates helloworld is the root module that needs to be runnable in the runtime image. With --launcher, we define an entry point to directly run the module in the image. Last, --output indicates a directory name for the runtime image.

The result of running this command is a new directory containing essentially a Java runtime completely tailored to running helloworld:

helloworld-image
├── bin
│   ├── hello 1
│   ├── java 2
│   └── keytool
├── conf
│   └── ...
├── include
│   └── ...
├── legal
│   └── ...
├── lib
│   └── ...
└── release
1
An executable script directly launching the helloworld module

2
The Java runtime, capable of resolving only helloworld and its dependencies

Since the resolver knows that only java.base is necessary in addition to helloworld, nothing more is added to the runtime image. Therefore, the resulting runtime image is many times smaller than a full JDK. A custom runtime image can be used on resource-constrained devices, or serve as the basis for a container image for running an application in the cloud. Linking is optional, but can substantially reduce the footprint of your application. In Chapter 13, the advantages of custom runtime images and the use of jlink are discussed in greater detail.

No Module Is an Island
So far, we’ve purposely kept things small and simple in order to understand the mechanics of module creation and the associated tooling. However, the real magic happens when you compose multiple modules. Only then the advantages of the module system become apparent.

It would be rather boring to extend our Modular Hello World example. Therefore, we continue with a more interesting example application called EasyText. Starting from a single monolithic module, we gradually create a multimodule application. EasyText may not be as big as your typical enterprise application (fortunately), but it touches enough real-world concerns to serve as a learning vehicle.

Introducing the EasyText Example
EasyText is an application for analyzing text complexity. It turns out there are some quite interesting algorithms you can apply to text to determine its complexity. Read “Text Complexity in a Nutshell” if you’re interested in the details.

Of course, our focus is not on the text analysis algorithms, but rather on the composability of the modules making up EasyText. The goal is to use Java modules to create a flexible and maintainable application. Here are the requirements we want to fulfill through a modular implementation of EasyText:

It must have the ability to add new analysis algorithms without modifying or recompiling existing modules.

Different frontends (for example, GUI and command line) must be able to reuse the same analysis logic.

It must support different configurations, without recompilation and without deploying all code for each configuration.

Granted, all of these requirements can be met without modules. It’s not an easy job, though. Using the Java module system helps us to meet these requirements.

TEXT COMPLEXITY IN A NUTSHELL
Even though the focus of the EasyText example is on the structure of the solution, it never hurts to learn something new along the way. Text analysis is a field with a long history. The EasyText application applies readability formulas to texts. One of the most popular readability formulas is the Flesch-Kincaid score:

complexityflesch_kincaid=206.835−1.015totalwordstotalsentences−84.6totalsyllablestotalwords
Given some relatively easily derivable metrics from a text, a score is calculated. If a text scores between 90 and 100, it is easily understood by an average 11-year-old student. Texts scoring in the range of 0 to 30, on the other hand, are best suited to graduate-level students.

There are numerous other readability formulas, such as Coleman-Liau and Fry readability, not to mention the many localized formulas. Each formula has its own scope, and there is no single best one. Of course, this is one of the reasons to make EasyText as flexible as possible.

Throughout this chapter and the subsequent chapters, each of these requirements is addressed. From a functional perspective, analyzing a text comprises several steps:

Read the input text (either from a file, GUI, or otherwise).

Split the text into sentences and words (since many readability formulas work with sentence- or word-level metrics).

Run one or more analyses on the text.

Show the result to the user.

Initially, our implementation consists of a single module, easytext. With this starting point, there is no separation of concerns. There’s just a single package inside a module, which by now we are familiar with, as shown in Example 3-2.

Example 3-2. EasyText as single module (➥ chapter3/easytext-singlemodule)
src
└── easytext
    ├── javamodularity
    │   └── easytext
    │       └── Main.java
    └── module-info.java
The module descriptor is an empty one. The Main class reads a file, applies a single readability formula (Flesch-Kincaid) and prints the results to the console. After compiling and packaging the module, it works like this:

$ java --module-path mods -m easytext input.txt
Reading input.txt
Flesh-Kincaid: 83.42468299865723
Obviously, the single-module setup ticks none of the boxes as far as our requirements are concerned. It’s time to add more modules into the mix.

A Tale of Two Modules
As a first step, we separate the text-analysis algorithm and the main program into two modules. This opens up the possibility to reuse the analysis module with different frontend modules later. The main module uses the analysis module, as shown in Figure 3-1.

EasyText in two modules
Figure 3-1. EasyText in two modules
The easytext.cli module contains the command-line handling logic and file-parsing code. The easytext.analysis module contains the implementation of the Flesch-Kincaid algorithm. During the split of the single easytext module, we create two new modules with two different packages, as shown in Example 3-3.

Example 3-3. EasyText as two modules (➥ chapter3/easytext-twomodules)
src
├── easytext.analysis
│   ├── javamodularity
│   │   └── easytext
│   │       └── analysis
│   │           └── FleschKincaid.java
│   └── module-info.java
└── easytext.cli
    ├── javamodularity
    │   └── easytext
    │       └── cli
    │           └── Main.java
    └── module-info.java
The difference is that the Main class now delegates the algorithmic analysis to the FleschKincaid class. Since we have two interdependent modules, we need to compile them using the multimodule mode of javac:

javac -d out --module-source-path src -m easytext.cli
From this point onward, we assume all modules of our examples are always compiled together. Instead of listing all source files as input to the compiler, we specify the actual modules to be compiled with -m. In this case, providing the easytext.cli module is enough. The compiler knows through the module descriptor that easytext.cli also needs easytext.analysis, which it then compiles from the module source path as well. Just providing a list of all source files1 (not using -m), as we saw in the Hello World example, also works.

The --module-source-path flag tells javac where to look for other modules in source format during compilation. It is mandatory to provide a destination directory with -d when compiling in multimodule mode. After compilation, the destination directory contains the compiled modules in exploded module format. This output directory can then be used as an element on the module path when running modules.

In this example, javac looks up FleschKincaid.java on the module source path when compiling Main.java. But how does the compiler know to look in the easytext.analysis module for this class? In the old classpath situation, it might have been in any JAR that is put on the compilation classpath. Remember, the classpath is a flat list of types. Not so for the module path; it deals only with modules. Of course, the missing piece of the puzzle is in the content of the module descriptors. They provide the necessary information for locating the right module exporting a given package. No more aimless scanning of all available classes wherever they live.

In order for the example to work, we need to express the dependencies we’ve already seen in Figure 3-1. The analysis module needs to export the package containing the FleschKincaid class:

module easytext.analysis {
   exports javamodularity.easytext.analysis;
}
With the exports keyword, packages in the module are exposed for use by other modules. By declaring that package javamodularity.easytext.analysis is exported, all its public types can now be used by other modules. A module can export multiple packages. In this case, only the FleschKincaid class is exported to other modules. Conversely, every package inside a module that is not exported is private to the module.

You’ve seen how the analysis module exports the package containing the FleschKincaid class. The module descriptor for easytext.cli, on the other hand, needs to express its dependency on the analysis module:

module easytext.cli {
   requires easytext.analysis;
}
We require the module easytext.analysis because the Main class imports the FleschKincaid class, originating from that module. With both these module descriptors in place, the code compiles and can be run.

What happens if we omit the requires statement from the module descriptor? In that case, the compiler produces the following error:

src/easytext.cli/javamodularity/easytext/cli/Main.java:11:
  error: package javamodularity.easytext.analysis is not visible
import javamodularity.easytext.analysis.FleschKincaid;
                              ^
  (package javamodularity.easytext.analysis is declared in module
   easytext.analysis, but module easytext.cli does not read it)
Even though the FleschKincaid.java source file is still available to the compiler (assuming we compile with -m easytext.analysis,easytext.cli to compensate for the missing requires easytext.analysis), it throws this error. A similar error is produced when we omit the exports statement from the analysis module’s descriptor. Here we see the major advantage of making dependencies explicit in every step of the software development process. A module can use only what it requires, and the compiler enforces this. At run-time, the same information is used by the resolver to ensure that all modules are present before starting the application. No more accidental compile-time dependencies on libraries, only to find out at run-time this library isn’t available on the classpath.

Another check the module system enforces is for cyclic dependencies. In the previous chapter, you learned that readability relations between modules must be acyclic at compile-time. Within modules, you can still create cyclic relations between classes, as has always been the case. It’s debatable whether you really want to do so from a software engineering perspective, but you can. However, at the module level, there is no choice. Dependencies between modules must form an acyclic, directed graph. By extension, there can never be cyclic dependencies between classes in different modules. If you do introduce a cyclic dependency, the compiler won’t accept it. Adding requires easytext.cli to the analysis module descriptor introduces a cycle, as shown in Figure 3-2.

EasyText modules with illegal cyclic dependency
Figure 3-2. EasyText modules with an illegal cyclic dependency
If you try to compile this, you run into the following error:

src/easytext.analysis/module-info.java:3:
   error: cyclic dependence involving easytext.cli
   requires easytext.cli;
                    ^
Note that cycles can be indirect as well, as illustrated in Figure 3-3. These cases are less obvious in practice, but are treated the same as direct cycles: they result in an error from the Java module system.

Cycles can be indirect as well.
Figure 3-3. Cycles can be indirect as well
Many real-world applications do have cyclic dependencies between their components. In “Breaking Cycles”, we discuss how to prevent and break cycles in your application’s module graph.

Working with Platform Modules
Platform modules come with the Java runtime and provide functionality including XML parsers, GUI toolkits, and other functionality that you expect to see in a standard library. In Figure 2-1, you already saw a subset of the platform modules. From a developer’s perspective, they behave the same as application modules. Platform modules encapsulate certain code, possibly export packages, and can have dependencies on other (platform) modules. Having a modular JDK means you need to be aware of what platform modules you’re using in application modules.

In this section, we extend the EasyText application with a new module. It’s going to use platform modules, unlike the modules we’ve created thus far. Technically we did use a platform module already: the java.base module. However, this is an implicit dependency. The new module we are going to create has explicit dependencies on other platform modules.

Finding the Right Platform Module
If you need to be aware of the platform modules you use, how do you find out which platform modules exist? You can depend on a (platform) module only if you know its name. When you run java --list-modules, the runtime outputs all available platform modules:

$ java --list-modules
java.base@9
java.xml@9
javafx.base@9
jdk.compiler@9
jdk.management@9
This abbreviated output shows there are several types of platform modules. Platform modules prefixed with java. are part of the Java SE specification. They export APIs as standardized through the Java Community Process for Java SE. The JavaFX APIs are distributed in modules sharing the javafx. prefix. Modules starting with jdk. contain JDK-specific code, which may be different across JDK implementations.

Even though the --list-modules functionality is a good starting point for discovering platform modules, you need more. Whenever you import from a package that’s not exported by java.base, you need to know which platform module provides this package. That module must be added to module-info.java with a requires clause. So let’s return to our example application to find out what working with platform modules entails.

Creating a GUI Module
EasyText so far has two application modules working together. The command-line main application has been separated from the analysis logic. In the requirements, we stated that we want to support multiple frontends on top of the same analysis logic. So let’s try to create a GUI frontend in addition to the command-line version. Obviously, it should reuse the analysis module that is already in place.

We’ll use JavaFX to create a modest GUI for EasyText. As of Java 8, the JavaFX GUI framework has been part of the Java platform and is intended to replace the older Swing framework. The GUI looks like Figure 3-4.

A simple GUI for EasyText
Figure 3-4. A simple GUI for EasyText
When you click Calculate, the analysis logic is run on the text from the text field and the resulting value is shown in the GUI. Currently, we have only a single analysis algorithm that can be selected in the drop-down, but that will change later, given our extensibility requirements. For now, we’ll keep it simple and assume that the Flesch-Kincaid analysis is the only available algorithm. The code for the GUI Main class is quite straightforward, as shown in Example 3-4.

Example 3-4. EasyText GUI implementation (➥ chapter3/easytext-threemodules)
package javamodularity.easytext.gui;

import java.util.ArrayList;
import java.util.List;

import javafx.application.Application;
import javafx.event.*;
import javafx.geometry.*;
import javafx.scene.*;
import javafx.scene.control.*;
import javafx.scene.layout.*;
import javafx.scene.text.Text;
import javafx.stage.Stage;

import javamodularity.easytext.analysis.FleschKincaid;

public class Main extends Application {

    private static ComboBox<String> algorithm;
    private static TextArea input;
    private static Text output;

    public static void main(String[] args) {
        Application.launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        primaryStage.setTitle("EasyText");
        Button btn = new Button();
        btn.setText("Calculate");
        btn.setOnAction(event ->
          output.setText(analyze(input.getText(), (String) algorithm.getValue()))
        );

        VBox vbox = new VBox();
        vbox.setPadding(new Insets(3));
        vbox.setSpacing(3);
        Text title = new Text("Choose an algorithm:");
        algorithm = new ComboBox<>();
        algorithm.getItems().add("Flesch-Kincaid");

        vbox.getChildren().add(title);
        vbox.getChildren().add(algorithm);
        vbox.getChildren().add(btn);

        input = new TextArea();
        output = new Text();
        BorderPane pane = new BorderPane();
        pane.setRight(vbox);
        pane.setCenter(input);
        pane.setBottom(output);
        primaryStage.setScene(new Scene(pane, 300, 250));
        primaryStage.show();
    }

    private String analyze(String input, String algorithm) {
        List<List<String>> sentences = toSentences(input);

        return "Flesch-Kincaid: " + new FleschKincaid().analyze(sentences);
    }

    // implementation of toSentences() omitted for brevity
}
There are imports from eight JavaFX packages in the Main class. How do we know which platform modules to require in module-info.java? One way to find out in which module a package lives is through JavaDoc. For Java 9, JavaDoc has been updated to include the module name that each type is part of.

Another approach is to inspect the JavaFX modules available by using java --list-modules. After running this command, we see eight modules containing javafx in the name:

javafx.base@9
javafx.controls@9
javafx.deploy@9
javafx.fxml@9
javafx.graphics@9
javafx.media@9
javafx.swing@9
javafx.web@9
Because there is not always a one-to-one correspondence between the module name and the packages it contains, choosing the right module is somewhat of a guessing game from this list. You can inspect the module declarations of platform modules with --describe-module to verify assumptions. If, for example, we think javafx.controls might contain the javafx.scene.control package, we can verify that with the following:

$ java --describe-module javafx.controls
javafx.controls@9
exports javafx.scene.chart
exports javafx.scene.control 1
exports javafx.scene.control.cell
exports javafx.scene.control.skin
requires javafx.base transitive
requires javafx.graphics transitive
...
1
Module javafx.controls exports the javafx.scene.control package.

Indeed, the package we want is contained in this package. This process of manually finding the right platform module this way is a bit tedious. It’s expected that IDEs will support the developer with this task after Java 9 support is in place. For the EasyText GUI, it turns out we need to require two JavaFX platform modules:

module easytext.gui {
   requires javafx.graphics;
   requires javafx.controls;
   requires easytext.analysis;
}
Given this module descriptor, the GUI module compiles correctly. However, when trying to run it, the following curious error comes up:

Exception in Application constructor
Exception in thread "main" java.lang.reflect.InvocationTargetException
        ...
Caused by: java.lang.RuntimeException: Unable to construct Application instance:
           class javamodularity.easytext.gui.Main
  at javafx.graphics/..LauncherImpl.launchApplication1(LauncherImpl.java:963)
  at javafx.graphics/..LauncherImpl.lambda$launchApplication$2(LauncherImpl.java)
  at java.base/java.lang.Thread.run(Thread.java:844)
Caused by: java.lang.IllegalAccessException: class ..application.LauncherImpl
           (in module javafx.graphics) cannot access class
                                       javamodularity.easytext.gui.Main
           (in module easytext.gui) because module easytext.gui does not export
           javamodularity.easytext.gui to module javafx.graphics
  at java.base/..Reflection.newIllegalAccessException(Reflection.java:361)
  at java.base/..AccessibleObject.checkAccess(AccessibleObject.java:589)
TIP
Another change in Java 9 is that stacktraces now also show which module a class comes from. The name before the slash (/) is the module containing the class given after the slash.

What is going on here? The root cause is an IllegalAccessException because class Main cannot be loaded. Main extends javafx.application.Application (which lives in the javafx.graphics module) and calls Application::launch from the main method. That’s a typical way to bootstrap a JavaFX application, delegating the UI creation to the JavaFX framework. JavaFX then uses reflection to instantiate Main, subsequently invoking the start method. That means the javafx.graphics module must have access to the Main class in easytext.gui. As you learned in “Accessibility”, accessibility to a class in another module calls for two things: readability to the target module, and the target module must export the given class.

In this case, javafx.graphics must have a readability relation to easytext.gui. Fortunately, the module system is smart enough to dynamically establish a readability relation to the GUI module. This happens transparently whenever reflection is used to load a class from another module. The problem is, the package containing Main is never exposed from the GUI module. Main is not accessible for the javafx.graphics module because it is not exported. This is exactly what the preceding error message tells us.

One solution would be adding an exports clause for the javamodularity.easytext.gui package to the module descriptor. Only that would expose the Main class to any module requiring the GUI module. Is that really what we want? Is the Main class really part of a public API we want to support? Not really. The only reason we need it to be accessible is that JavaFX needs to instantiate it. This is a perfect use case for qualified exports:

module easytext.gui {

   exports javamodularity.easytext.gui to javafx.graphics;

   requires javafx.graphics;
   requires javafx.controls;
   requires easytext.analysis;
}
TIP
During compilation, the target modules of a qualified export must be on the module path or be compiled at the same time. Obviously this is not an issue for platform modules, but it is something to be aware of when using qualified exports to nonplatform modules.

Through the qualified exports, only javafx.graphics is able to access our Main class. Now we can run the application, and JavaFX is able to instantiate Main. In “Open Modules and Packages”, you’ll learn about another way to deal with reflective access to module internals at run-time.

An interesting situation arises at run-time. As discussed, the javax.graphics module dynamically establishes a readability relation with easytext.gui at run-time (depicted by the bold edge in Figure 3-5).

Readability edges at run-time
Figure 3-5. Readability edges at run-time
But that means there is a cycle in the readability graph! How can this be? Cycles were supposed to be impossible. They are, at compile-time. In this case, we compile easytext.gui with a dependency on (and thus readability relation to) javafx.graphics. At run-time, javax.graphics automatically establishes a readability relation to easytext.gui when it reflectively instantiates Main. Readability relations can be cyclic at run-time. Because the export is qualified, only javafx.graphics can access our Main class. Any other module establishing a readability relation with easytext.gui won’t be able to access the javamodularity.easytext.gui package.

The Limits of Encapsulation
Looking back, we have come a long way in this chapter. You learned how to create modules, run them, and use them in conjunction with platform modules. Our example application, EasyText, has grown from a mini-monolith to a multimodule Java application. Meanwhile, it achieves two of the stated requirements: it supports multiple frontends while reusing the same analysis module, and we can create different configurations of our modules targeting the command line or a GUI.

Looking at the other requirements, however, there’s still a lot to be desired. As things stand, both frontend modules instantiate a specific implementation class (FleschKincaid) from the analysis module to do their work. Even though the code lives in separate modules, tight coupling is going on here. What if we want to extend the application with different analyses? Should every frontend module be changed to know about new implementation classes? That sounds like poor encapsulation. Should the frontend modules be updated with dependencies on newly introduced analysis modules? That sounds distinctly nonmodular. It also runs counter to our requirement of adding new analysis algorithms without modifying or recompiling existing modules. Figure 3-6 already shows how messy this gets with two frontends and two analyses. (Coleman-Liau is another well-known complexity metric.)

Every front-end module needs to depend on every analysis module and instantiate its exported implementation class.
Figure 3-6. Every frontend module needs to depend on all analysis modules in order to instantiate their exported implementation classes
In summary, we have two issues to address:

The frontends need to be decoupled from concrete analysis implementation types and modules. Analysis modules should not export these types either to avoid tight coupling.

Frontends should be able to discover new analysis implementations in new modules without any code changes.

By solving these two problems, we satisfy the requirement that new analyses can be introduced by adding them on the module path, without touching the frontends.

Interfaces and Instantiation
Ideally, we’d abstract away the different analyses behind an interface. After all, we’re just passing in sentences and getting back a score for each algorithm:

public interface Analyzer {

   String getName();

   double analyze(List<List<String>> text);

}
As long as we can find out the name of an algorithm (for display purposes) and get it to calculate the complexity, we’re good. This type of abstraction is what interfaces were made for. The Analyzer interface is stable and can live in its own module—say, easytext.analysis.api. That’s what the frontend modules should know and care about. The analysis implementation modules then require this API module as well and implement the Analyzer interface. So far, so good.

However, there’s a problem. Even though the frontend modules care only about calling the analyze method through the Analyzer interface, they still need a concrete instance to call this method on:

Analyzer analyzer = ???
How can we get ahold of an instance that implements Analyzer without relying on a specific implementation class? You could say this:

Analyzer analyzer = new FleschKincaid();
Unfortunately, the FleschKincaid class still needs to be exported for this to work, bringing us right back to square one. Instead, we need a way to obtain instances without referring to concrete implementation classes.

As with all problems in computer science, we can try to solve this by adding a new layer of indirection. We will look at solutions for this problem in the next chapter, which details the factory pattern and how it leads to services.

1 On a Linux/macOS system, you can easily provide $(find . -name '*.java') as the last argument to the compiler to achieve this.