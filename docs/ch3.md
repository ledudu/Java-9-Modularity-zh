# 第 3 章 使用模块

> Chapter 3. Working with Modules

In this chapter, we take the first steps towards modular development using Java 9. Instead of looking at existing modules in the JDK, it’s time to get your hands dirty by writing your first module. To start things off easily, we turn our attention to the simplest possible module. Let’s call it our Modular Hello World. Armed with this experience, we’ll then be ready to take on a more ambitious example with multiple modules. At that point, we’ll introduce the running example to be used throughout this book, called EasyText. It’s designed to gradually grow with you, as you learn more about the module system.

> 在本章，将迈出使用Java 9进行模块开发的第一步，即动手编写自己的第一个模块，而不仅仅是查看JDK中现有的模块。为了轻松地走好第一步，首先创建一个最简单的模块，可以将其称为Modular Hello World。有了相关的经验之后，就可以开始创建带有多个模块的更复杂的示例了，此时将会介绍贯穿本书的运行示例EasyText。随着对模块系统的进一步了解，该示例的设计也将逐步完善。

## 3.1 Your First Module 第一个模块
You’ve seen examples of module descriptors in the previous chapter. A module is generally more than just a descriptor, though. The Modular Hello World therefore transcends the level of a single source file: we need to examine it in context. We’ll start by compiling, packaging, and running a single module to get acquainted with the new tooling options for modules.

> 在前一章，已经给出了模块描述符的示例。然而，一个模块通常不只是一个描述符。因此，Modular Hello World超越了单个源文件的级别：需要在上下文中来研究该示例。我们将从编译、打包和运行单个模块开始，以了解模块的新工具选项。

### 3.1.1 Anatomy of a Module 剖析模块
Our goal for this first example is to compile the following class into a module and run it (Example 3-1). We start out with a single class in a package, leading to a single module. Modules may contain only types that are inside packages, so a package definition is required.

> 第一个示例的目的是将下面所示的类编译成一个模块并运行（见示例3-1）。首先从一个包的单一类开始，编译成单一模块。模块可能仅包含包中的类型，所以需要一个包定义。

Example 3-1. HelloWorld.java (➥ chapter3/helloworld)

> 示例3-1:HelloWorld.java（chapter3/helloworld）
```java
package com.javamodularity.helloworld;

public class HelloWorld {

   public static void main(String... args) {
      System.out.println("Hello Modular World!");
   }

}
```
The layout of the sources on the filesystem looks as follows:

> 文件系统中源文件的布局如下所示：
```
src
└── helloworld 1
    ├── com
    │   └── javamodularity
    │       └── helloworld
    │           └── HelloWorld.java
    └── module-info.java 2
```
1 Module directory
2 Module descriptor

---

> 1. 模块目录。
> 2. 模块描述符。

Compared to the traditional layout of Java source files, there are two major differences. First, there is an extra level of indirection: below src we introduce another directory, helloworld. The directory is named after the name of the module we’re creating. Second, inside this module directory we find both the source file (nested in its package structure as usual) and a module descriptor. A module descriptor lives in module-info.java and is the key ingredient to Java modules. Its presence signals to the Java compiler that we are working with a module rather than plain Java sources. Compiler behavior is quite different when working with modules as compared to plain Java source files, as you will see in the remainder of this chapter. The module descriptor must be present in the root of the module directory. It is compiled along with the other source files into a binary class-file called module-info.class.

> 相比于Java源文件的传统布局，上述布局存在两个主要区别。首先，有一个额外的间接层：在src的下面引入了另一个目录helloworld，该目录以所创建的模块名称命名。其次，在模块目录中找到了源文件（像往常一样嵌套在包结构中）和一个模块描述符。模块描述符位于module-info.java文件中，是Java模块的关键组成部分。它的存在相当于告诉Java编译器正在使用的是一个模块，而不是普通的Java源代码。如本章的后续内容所述，与普通的Java源文件相比，当使用模块时，编译器的行为是完全不同的。模块描述符必须存在于模块目录的根目录中。它与其他源文件一起编译成一个名为module-info.class的二进制类文件。

So what’s inside the module descriptor? Our Modular Hello World example is quite minimalistic:

> 模块描述符的内部是什么呢？Modular Hello World示例非常简单：
```java
module helloworld {

}
```
We declare a module by using the new module keyword, followed by the module name. The name must match the name of the directory containing the module descriptor. Otherwise, the compiler refuses to compile and reports the mismatch.

> 此时，使用新关键字module并紧跟模块名称声明了一个模块。该名称必须与包含模块描述符的目录名称相匹配。否则，编译器将拒绝编译并报告匹配错误。

NOTE

This name-matching requirement is true only when running the compiler in multimodule mode, which is a common scenario. For the single-module scenario discussed in “Compilation”, the directory name does not matter. In any case, it’s still a good idea to use the name of the module as the directory name.

> 仅在多模块模式下（这种情况是非常常见的）运行编译器时，才需要满足名称匹配要求。对于3.1.3节中所讨论的单模块方案，目录名称无关紧要。但在任何情况下，使用模块名称作为目录名称不失为一个好主意。

Because of the empty module declaration body, nothing from the helloworld module is exported to other modules. By default, all packages are strongly encapsulated. Even though there’s no dependency information (yet) in this declaration, remember that this module implicitly depends on the java.base platform module.

> 由于模块声明体是空的，因此不会从helloworld模块中导出任何内容到其他模块。默认情况下，所有包都是强封装的。即使目前在这个声明中没有任何依赖关系信息，但是请记住，该模块隐式地依赖java.base平台模块。

You may be wondering whether adding a new keyword to the language breaks existing code that uses module as an identifier. Fortunately, that’s not the case. You can still use identifiers called module in your other source files, because the module keyword is a restricted keyword. It is treated as a keyword only inside module-info.java. The same holds for the requires keyword and other new keywords you’ve seen so far in module descriptors.

> 此时，你可能会问，向语言中添加新的关键字是否会破坏使用module作为标识符的现有代码。幸运的是，情况并非如此。仍然可以在其他源文件中使用名为module的标识符，因为module关键字是限制关键字（restricted keyword），它仅在module-info.java中被视为关键字。对于目前在模块描述符中所看到的requires关键字和其他新关键字来说，也是一样的。

#### THE MODULE-INFO NAME 名称module-info
Normally, the name of a Java source file corresponds to the (public) type it contains. For example, our file containing the HelloWorld class must be in a file named HelloWorld.java. The module-info name breaks this correspondence. Moreover, module-info is not even a legal Java identifier, because it contains a dash. This is done on purpose, to prevent non-module-aware tools from blindly processing module-info.java or module-info.class as if it were a normal Java class.

> 通常，Java源文件的名称与其所包含的（公共）类型相对应。例如，包含HelloWorld类的文件名必须为HelloWorld.java。但名称module-info打破了这种对应关系。此外，module-info甚至不是一个合法的Java标识符，因为它包含了破折号。这样做的目的是防止非模块感知工具盲目地将module-info.java或module-info.class作为普通的Java类加以处理。

Reserving a name for special source files is not unprecedented in the Java language. Before module-info.java, there was package-info.java. Although it is relatively unknown, it’s been around since Java 5. In package-info.java, you can add documentation and annotations to a package declaration. Like module-info.java, it is compiled to a class file by the Java compiler.

> 为特殊源文件保留名称在Java语言中并不是没有出现过。在module-info.java之前，就已经有了package-info.java。该名称虽然可能相对比较陌生，但自Java 5之后它就出现了。在package-info.java中，可以向包声明中添加文档和注释。与module-info.java一样，它也是由Java编译器编译成一个类文件。

We now have a module descriptor containing nothing but a module declaration, and a single source file. Enough to compile our first module!

> 现在，已经拥有了一个仅包含模块声明的模块描述符以及一个源文件。接下来可以编译第一个模块了！

### 3.1.2 Naming Modules 命名模块
Naming things, while hard, is important. Especially for modules, since they will convey the high-level structure of your application.

> 命名事物虽然很难，但却很重要。尤其是对于模块而言更是如此，因为它们将传递应用程序的高级结构。

Module names live in a global namespace separate from other namespaces in Java. So, theoretically, you could give a module the same name as a class, interface, or package. In practice, this might lead to confusion.

> 模块名称所在的全局命名空间与Java中其他命名空间是分开的。因此，从理论上讲，模块的名称可以与类、接口或包的名称相同。但实际上，这样做可能会导致混乱。

Module names must be unique: an application can have only a single module for any given name. In Java it’s customary to make package names globally unique by using reverse DNS notation. You could apply the same reasoning to modules. For example, you could rename the helloworld module to com.javamodularity.helloworld. However, this leads to long and somewhat clunky module names.

> 模块名称必须是唯一的：应用程序只能具有一个给定名称的模块。在Java中，通常使用反向DNS符号来确保包名称全局唯一。可以对模块应用相同的方法。例如，可以将helloworld模块重命名为com.javamodularity.helloworld，但这样做会导致模块名称过长且有些笨重。

Is it really necessary for module names in applications to be globally unique? Certainly, when your module is a published library, used in many applications, it makes sense to choose a globally unique module name. In “Choosing a Library Module Name”, we discuss this notion further. For application modules, there’s no shame in choosing shorter and more memorable names.

> 应用程序中的模块名称是否需要全局唯一呢？答案是肯定的，当模块是已发布的库并且在许多应用程序中使用时，选择全局唯一的模块名称就显得非常有意义了。在10.2节中，将会进一步讨论这个概念。对于应用程序模块来说，要尽量选择更短且更令人难忘的名称。

In this book we prefer shorter module names to increase the readability of examples.

> 为了增加示例的可读性，本书选择使用更简短的模块名称。

### 3.1.3 Compilation 编译
Having a module in source format is one thing, but we can’t run it without compiling first. Prior to Java 9, the Java compiler is invoked with a destination directory and a set of sources to compile:

> 有一个源文件格式的模块是一回事，但要运行该模块，首先必须进行编译。在Java 9之前，Java编译器使用目标目录以及一组源文件进行编译：
```sh
javac -d out src/com/foo/Class1.java src/com/foo/Class2.java
```
In practice, this is often done under the hood by build tools such as Maven or Gradle, but the principle remains the same. Classes are emitted in the target directory (out in this case) with nested folders representing the input (package) structure. Following the same pattern, we can compile our Modular Hello World example:

> 在实践中，通常是通过构建工具（如Maven或Gradle）来完成的，但原理是一样的。在目标目录中输出类（此时使用了out），其中包含代表输入（包）结构的嵌套文件夹。按照同样的模式，可以编译Modular Hello World示例：

```sh
javac -d out/helloworld \
      src/helloworld/com/javamodularity/helloworld/HelloWorld.java \
      src/helloworld/module-info.java
```
There are two notable differences:

> 存在两个显著的区别：

1. We output into a helloworld directory, reflecting the module name.
2. We add module-info.java as an additional source file to compile.

---

> 1. 输出到反映了模块名称的helloworld目录。
> 2. 将module-info.java作为额外源文件进行编译。

The presence of module-info.java in the set of files to be compiled triggers the module-aware mode of javac. Running the compilation results in the following output, also known as the exploded module format:

> 在要编译的文件集中出现module-info.java源文件会触发javac的模块感知模式。下面显示的输出是编译后的结果，也被称为分解模块（exploded module）格式：

```
out
└── helloworld
    ├── com
    │   └── javamodularity
    │       └── helloworld
    │           └── HelloWorld.class
    └── module-info.class
```
It’s best to name the directory containing the exploded module after the module, but not required. Ultimately, the module system takes the name of the module from the descriptor, not from the directory name. In “Running Modules”, we’ll take this exploded module and run it.

> 最好在模块之后命名包含分解模块的目录，但不是必需的。最终，模块系统是从描述符中获取模块名称，而不是从目录名称中。在3.1.5节中，将会创建并运行这个分解模块。

#### COMPILING MULTIPLE MODULES 编译多个模块

What you’ve seen so far is the so-called single-module mode of the Java compiler. Typically, the project you want to compile consists of multiple modules. These modules may or may not refer to each other. Or the project might be a single module but uses other (already compiled) modules. For these cases, additional compiler flags have been introduced: --module-source-path and --module-path. These are the module-aware counterparts of the -sourcepath and -classpath flags that have been part of javac for a long time. Their semantics are explained when we start looking at multimodule examples in “A Tale of Two Modules”. Keep in mind, the name of the module source directories must match the name declared in module-info.java in this multimodule mode.

> 到目前为止所看到的都是所谓的Java编译器单模块模式（single-module mode）。通常，需要编译的项目由多个模块组成，这些模块还可能会相互引用。又或者项目是单个模块，但却使用了其他（已经编译）的模块。为了处理这些情况，引入了额外的编译器标志：--module-source-path和--module-path。这些标志都是-sourcepath和-classpath标志（长期以来，这些标志一直是javac的一部分）的模块感应对应项。在学习3.2.2节中的多模块示例时，将会解释其语义。请记住，模块源目录的名称必须与在多模块模式下module-info.java中声明的名称相匹配。

#### BUILD TOOLS 构建工具

It is not common practice to use the Java compiler directly from the command line, manipulating its flags and listing all source files manually. More often, build tools such as Maven or Gradle are used to abstract away these details. For this reason, we won’t cover all the nitty-gritty details of every new option added to the Java compiler and runtime in this book. You can find comprehensive coverage of these in the official documentation. Of course, build tools need to adapt to the new modular reality as well. In Chapter 11, we show how some of the most popular build tools can be used with Java 9.

直接通过命令行使用Java编译器、操作其标志以及手动列出所有源文件并不是常见的做法，更常见的做法是使用Maven或Gradle等构建工具来抽取这些细节信息。因此，本书不会详细介绍添加到Java编译器和运行时的每个新选项，可以在官方文档中找到相关详细信息（http://bitly/tools-comm-ref）。当然，构建工具也需要适应新的模块化现实。在第11章中，将介绍一些可以与Java 9一起使用的最流行的构建工具。

### 3.1.4 Packaging 打包
So far, we’ve created a single module and compiled it into the exploded module format. In the next section, we show you how to run such an exploded module as is. This works in a development situation, but in production scenarios you want to distribute your module in a more convenient format. To this end, modules can be packaged and used in JAR files. This results in modular JAR files. A modular JAR file is similar to a regular JAR file, except it also contains a module-info.class.

> 到目前为止，已经创建了单个模块并将其编译为分解模块格式，在下一节将会讨论如何运行分解模块。这种格式在开发环境下是可行的，但在生产环境中则需要以更方便的格式分发模块。为此，可以将模块打包成JAR文件并使用，从而产生了模块化JAR文件。模块化JAR文件类似于普通的JAR文件，但它还包含了module-info.class。

The JAR tool has been updated to work with modules in Java 9. To package up the Modular Hello World example, execute the following command:

> JAR工具已经进行了更新，从而可以使用Java 9中的模块。为了打包Modular Hello World示例，请运行下面的命令：

```sh
jar -cfe mods/helloworld.jar com.javamodularity.helloworld.HelloWorld \
    -C out/helloworld .
```
With this command, we’re creating a new archive (-cf) called helloworld.jar in the mods directory (make sure the directory exists, though). Furthermore, we want the entry point (-e) for this module to be the HelloWorld class; whenever the module is started without specifying another main class to run, this is the default. We provide the fully qualified classname as an argument for the entry point. Finally, we instruct the jar tool to change (-C) to the out/helloworld directory and put all compiled files from this directory in the JAR file. The contents of the JAR are now similar to the exploded module, with the addition of a MANIFEST.MF file:

> 通过上述命令，在mods目录（请确保该目录存在）中创建了一个新存档文件（-cf）helloworld.jar。此外，还希望这个模块的入口点（-e）是HelloWorld类；每当模块启动并且没有指定另一个要运行的主类时，这是默认入口点。此时提供了完全限定的类名称作为入口点的参数。最后，指示jar工具更改（-C）为out/helloworld目录，并将此目录中的所有已编译文件放在JAR文件中。现在，JAR的内容类似于分解模块，同时还额外添加了一个MANIFEST.MF文件：
```
helloworld.jar
├── META-INF
|   └── MANIFEST.MF
├── com
│   └── javamodularity
│       └── helloworld
│           └── HelloWorld.class
└── module-info.class
```
The filename of the modular JAR file is not significant, unlike the situation with the module directory name during compilation. You can use any filename you like, since the module is identified by the name declared in the bundled module-info.class.

> 与编译期间的模块目录名称不同，模块化JAR文件的名称并不重要。可以使用任意喜欢的文件名，因为模块由绑定的module-info.class中声明的名称所标识。

### 3.1.5 Running Modules 运行模块
Let’s recap what we did so far. We started our Modular Hello World example by creating a helloworld module with a single HelloWorld.java source file and a module descriptor. Then we compiled the module into the exploded module format. Finally, we took the exploded module and packaged it as a modular JAR file. This JAR file contains our compiled class and the module descriptor, and knows about the main class to execute.

> 现在，回顾一下目前所完成的事情。首先从Modular Hello World示例开始，创建了一个带有单个HelloWorld.java源文件和模块描述符的helloworld模块。然后，将模块编译成分解模块格式。最后，将分解模块打包成一个模块化JAR文件。这个JAR文件包含了已编译的类和模块描述符，并且知道要执行的主类。

Now let’s try to run the module. Both the exploded module format and modular JAR file can be run. The exploded module format can be started with the following command:

> 尝试运行模块，分解模块格式和模块化JAR文件都可以运行。可以使用下面的命令运行分解模块格式：
```sh
$ java --module-path out \
       --module helloworld/com.javamodularity.helloworld.HelloWorld
Hello Modular World!
```
TIP

You can also use the short-form -p flag instead of --module-path. The --module flag can be shortened to -m.

> 还可以使用--module-path的缩写格式-p。标志--module可以缩写为-m。

The java command has gained new flags to work with modules in addition to classpath-based applications. Notice we put the out directory (containing the exploded helloworld module) on the module path. The module path is the module-aware counterpart of the original classpath.

> 除了基于类路径的应用程序之外，Java命令还获得了新的标志来处理模块。请注意，此时将out目录（包含分解的helloworld模块）放在模块路径上。模块路径是原始类路径的模块感知对应项。

Next, we provide the module to be run with the --module flag. In this case, it consists of the module name followed by a slash and then the class to be run. On the other hand, if we run our modular JAR, providing just the module name is enough:

> 接下来使用--module标志提供所运行的模块。此时，模块名称后跟斜杠，然后是要运行的类。另一方面，如果运行模块化JAR，则只需提供模块名称即可：
```sh
$ java --module-path mods --module helloworld
Hello Modular World!
```
This makes sense, because the modular JAR knows the class to execute from its meta-data. We explicitly set the entry point to com.javamodularity.helloworld.HelloWorld when constructing the modular JAR.

> 这么做是有道理的，因为模块化JAR知道要从其元数据执行的类。在构建模块化JAR时已经显式地将入口点设置为com.javamodularity.helloworld.Hello World。

TIP

The --module or -m flag with corresponding module name (and optional main class) must always come last. Any subsequent arguments are passed to the main class being started from the given module.

> 带有相应模块名称（和可选主类）的--module或-m标志必须始终在最后。任何后续参数都将传递至从给定模块启动的主类。

Launching in either of these two ways makes helloworld the root module for execution. The JVM starts from this root module, and resolves any other modules necessary to run the root module from the module path. Resolving modules is a recursive process: if a newly resolved module requires other modules, the module system automatically takes this into account, as discussed earlier in “Module Resolution and the Module Path”.

> 以这两种方式中的任何一种启动都会使helloworld成为执行的根模块。JVM从这个根模块开始，解析从模块路径运行根模块所需的任何其他模块。如前面的2.7节所述，解析模块是一个递归过程：如果新解析的模块需要其他模块，那么模块系统会自动考虑到这一点。

In our simple helloworld example, there is not too much to resolve. You can trace the actions taken by the module system by adding --show-module-resolution to the java command:

> 在简单的HelloWorld例子中，没有执行太多的解析。可以向java命令中添加--show-module-resolution，从而跟踪模块系统所采取的操作：

```sh
$ java --show-module-resolution --limit-modules java.base \
       --module-path mods --module helloworld
root helloworld file:///chapter3/helloworld/mods/helloworld.jar
Hello Modular World!
```
(The --limit-modules java.base flag is added to prevent other platform modules from being resolved through service binding. Service binding is discussed in the next chapter.)

> （通过添加java.base标志--limit-modules，可以阻止通过服务绑定解析其他平台模块。下一章将会详细介绍服务绑定。）

In this case, no other modules are required (besides the implicitly required platform module java.base) to run helloworld. Only the root module, helloworld, is shown in the module resolution output. This means running the Modular Hello World example involves just two modules at run-time: helloworld and java.base. Other platform modules, or modules on the module path, are not resolved. During classloading, no resources are wasted searching through classes that aren’t relevant to the application.

> 此时，除了隐式需要的平台模块java.base之外，不再需要其他模块来运行helloworld。在模块解析输出中仅显示了根模块helloworld。这意味着运行Modular Hello World示例仅涉及运行时的两个模块，即helloworld和java.base，其他平台模块或者模块路径上的模块都没有解析。在类加载期间，没有任何资源被浪费在搜索与应用程序无关的类上。

TIP

Even more diagnostic information about module resolution can be shown with -Xlog:module=debug. Options starting with -X are nonstandard, and may not be supported on Java implementations that are not based on OpenJDK.

> 通过使用-Xlog:module=debug，可以显示更多关于模块解析的诊断信息。以-X开头的选项都是非标准的，那些不是基于OpenJDK的Java实现可能不支持这些选项。

An error would be encountered at startup if another module were necessary to run helloworld and it’s not present on the module path (or part of the JDK platform modules). This form of reliable configuration is a huge improvement over the old classpath situation. Before the module system, a missing dependency is noticed only when the JVM tries to load a nonexistent class at run-time. Using the explicit dependency information of module descriptors, module resolution ensures a working configuration of modules before running any code.

> 如果需要另一个模块来运行helloworld，并且该模块不存在于模块路径上（或不是JDK平台模块的一部分），那么在启动时就会遇到错误。这种形式的可靠配置解决了使用类路径所面临的问题。在模块系统出现之前，只有当JVM在运行时尝试加载不存在的类时才会注意到缺少的依赖项。通过使用模块描述符的显式依赖信息，模块解析可以确保在运行任何代码之前对模块进行工作配置。

### 3.1.6 Module Path 模块路径
Even though module path sounds quite similar to classpath, they behave differently. The module path is a list of paths to individual modules and directories containing modules. Each directory on the module path can contain zero or more module definitions, where a module definition can be an exploded module or a modular JAR file. An example module path containing all three options looks like this: out/:myexplodedmodule/:mypackagedmodule.jar. All modules inside the out directory are on the module path, in conjunction with the module myexplodedmodule (a directory) and mypackagedmodule (a modular JAR file).

> 虽然模块路径听起来与类路径类似，但两者的行为却完全不同。模块路径是各个模块以及包含模块的目录的路径列表。模块路径上的每个目录都可以包含零个或多个模块定义，其中模块定义可以是分解模块或模块化JAR文件。包含三个选项的示例模块路径如下所示：out/:myexplodedmodule/:mypackagedmodule.jar。out目录中的所有模块都在模块路径上，并与模块myexplodedmodule（目录）以及mypackagedmodule（模块化JAR文件）相结合。

TIP

Entries on the module path are separated by the default platform separator. On Linux/macOS, that’s a colon (java -p dir1:dir2); on Windows, use a semicolon (java -p dir1;dir2). The -p flag is short for --module-path.

> 模块路径上的条目由默认的平台分隔符分隔。在Linux/macOS上，分隔符是一个冒号（java -p dir1:dir2）；而在Windows上，则使用分号（java -p dir1; dir2）。标志-p是--module-path的缩写。

Most important, all artifacts on the module path have module descriptors (possibly synthesized on the fly, as we will learn in “Automatic Modules”). The resolver relies on this information to find the right modules on the module path. When multiple modules with the same name are in the same directory on the module path, the resolver shows an error and won’t start the application. Again, this prevents scenarios with conflicting JAR files that were previously possible on the classpath.

> 最重要的是，模块路径上的所有工件都有模块描述符（可能是在运行中合成，8.4节将会详细讨论相关内容），解析器依赖此信息找到模块路径上的正确模块。当模块路径上相同目录中具有相同名称的多个模块时，解析器就会显示错误，并且不会启动应用程序。这样一来，就可以防止以前在类路径上可能发生的JAR文件冲突的问题。

WARNING

When multiple modules with the same name are in different directories on the module path, no error is produced. Instead, the first module is selected and subsequent modules with the same name are ignored.

> 当具有相同名称的多个模块位于模块路径上的不同目录中时，则不会产生错误，而是选择第一个模块，并忽略具有相同名称的后续模块。

### 3.1.7 Linking Modules 链接模块
In the previous section, you saw that the module system resolved only two modules: helloworld and java.base. Wouldn’t it be great if we could take advantage of this up-front knowledge by creating a special distribution of the Java runtime containing the bare minimum to run our application? That’s exactly what you can do in Java 9 with custom runtime images.

> 前一节所示的模块系统仅解析了两个模块：helloworld和java.base。如果可以利用前面所学到的知识创建一个Java运行时的特殊分布，其中包含运行应用程序所需的最少模块，岂不是很好？而这正是在Java 9中使用自定义运行时映像（custom runtime image）所完成的事情。

An optional linking phase is introduced with Java 9, between the compilation and run-time phases. With a new tool called jlink, you can create a runtime image containing only the necessary modules to run an application. Using the following command, we create a new runtime image with the helloworld module as root:

> 在编译和运行时阶段之间，Java 9引入了一个可选的链接阶段。通过使用一个名为jlink的新工具，可以创建仅包含运行应用程序所需的模块的运行时映像。使用以下命令，创建一个以helloworld为根模块的运行时映像：
```sh
$ jlink --module-path mods/:$JAVA_HOME/jmods \
        --add-modules helloworld \
        --launcher hello=helloworld \
        --output helloworld-image
```
TIP

The jlink tool lives in the bin directory of the JDK installation. It is not added to the system path by default, so in order to use it as in the preceding example, you must add it to the path first.

> jlink工具位于JDK安装目录下的bin目录。在默认情况下并没有将它添加到系统路径中，所以想要在示例中使用该工具，必须首先将其添加到路径中。

The first option constructs a module path containing the mods directory (where helloworld lives) and the directory of the JDK installation containing the platform modules we want to link into the image. Unlike with javac and java, you have to explicitly add platform modules to the jlink module path. Then, --add-modules indicates helloworld is the root module that needs to be runnable in the runtime image. With --launcher, we define an entry point to directly run the module in the image. Last, --output indicates a directory name for the runtime image.

> 第一个选项是构造一个模块路径，其中包含mods目录（helloworld所在的位置）以及要链接到映像中的平台模块的JDK安装目录。与javac和java不同，必须将平台模块显式添加到jlink模块路径中。随后，--add-modules表示helloworld是需要在运行时映像中运行的根模块。--launcher定义了一个入口点来直接运行映像中的模块。最后，--output表示运行时映像的目录名称。

The result of running this command is a new directory containing essentially a Java runtime completely tailored to running helloworld:

> 运行上述命令的结果是生成一个新目录，包含了一个完全适合运行helloworld的Java运行时：
```
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
```
1 An executable script directly launching the helloworld module
2 The Java runtime, capable of resolving only helloworld and its dependencies

---

> 1. 直接启动helloworld模块的可执行脚本。
> 2. 仅能解析helloworld及其依赖项的Java运行时。

Since the resolver knows that only java.base is necessary in addition to helloworld, nothing more is added to the runtime image. Therefore, the resulting runtime image is many times smaller than a full JDK. A custom runtime image can be used on resource-constrained devices, or serve as the basis for a container image for running an application in the cloud. Linking is optional, but can substantially reduce the footprint of your application. In Chapter 13, the advantages of custom runtime images and the use of jlink are discussed in greater detail.

> 由于解析器知道除了helloworld之外只需要使用java.base，因此无须向运行时映像中添加更多的内容。因此，生成的运行时映像比完整的JDK小许多。可以在资源受限的设备上使用自定义运行时映像，或者将其作为在云中运行应用程序的容器映像的基础。虽然链接是可选的，却可以大大减少应用程序的占用空间。在第13章中，将会更详细地讨论自定义运行时映像的优点以及如何使用jlink。

## 3.2 No Module Is an Island 任何模块都不是一座孤岛
So far, we’ve purposely kept things small and simple in order to understand the mechanics of module creation and the associated tooling. However, the real magic happens when you compose multiple modules. Only then the advantages of the module system become apparent.

> 到目前为止，为了便于理解模块的创建机制和相关工具，特意将事情进行了简单化。然而，模块系统的真正魅力在于组合使用多个模块。也只有这样，模块系统的优势才会显现出来。

It would be rather boring to extend our Modular Hello World example. Therefore, we continue with a more interesting example application called EasyText. Starting from a single monolithic module, we gradually create a multimodule application. EasyText may not be as big as your typical enterprise application (fortunately), but it touches enough real-world concerns to serve as a learning vehicle.

> 扩展Modular Hello World示例将是一件相当无聊的事情，因此，这里使用一个更有趣的示例应用程序EasyText。首先从单个模块开始，然后逐渐创建一个多模块应用程序。虽然EasyText示例可能没有典型的企业应用程序那么大，但在实践中可以作为一种学习工具来使用。

### 3.2.1 Introducing the EasyText Example EasyText示例介绍
EasyText is an application for analyzing text complexity. It turns out there are some quite interesting algorithms you can apply to text to determine its complexity. Read “Text Complexity in a Nutshell” if you’re interested in the details.

> EasyText是一个分析文本复杂性的应用程序。事实证明，可以将一些非常有趣的算法应用到文本以确定它的复杂性。如果对细节感兴趣，请阅读随后的“文本的复杂性”分析。

Of course, our focus is not on the text analysis algorithms, but rather on the composability of the modules making up EasyText. The goal is to use Java modules to create a flexible and maintainable application. Here are the requirements we want to fulfill through a modular implementation of EasyText:

> 当然，我们关注的不是文本分析算法，而是构成EasyText应用程序的模块组合，主要目标是使用Java模块来创建灵活且可维护的应用程序。以下是EasyText模块化实现所需满足的要求：

1. It must have the ability to add new analysis algorithms without modifying or recompiling existing modules.
2. Different frontends (for example, GUI and command line) must be able to reuse the same analysis logic.
1. It must support different configurations, without recompilation and without deploying all code for each configuration.

---

> 1. 必须能够在不修改或不重新编译现有模块的情况下添加新的分析算法。
> 2. 不同的前端（如GUI和命令行）必须能够重用相同的分析逻辑。
> 1. 必须支持不同的配置，同时无须重新编译，也无须部署每个配置的所有代码。


Granted, all of these requirements can be met without modules. It’s not an easy job, though. Using the Java module system helps us to meet these requirements.

> 当然，即使不使用模块也可以满足所有这些需求。不过，这不是一件容易的工作。使用Java模块系统有助于更容易地满足这些要求。

#### TEXT COMPLEXITY IN A NUTSHELL 文本的复杂性
Even though the focus of the EasyText example is on the structure of the solution, it never hurts to learn something new along the way. Text analysis is a field with a long history. The EasyText application applies readability formulas to texts. One of the most popular readability formulas is the Flesch-Kincaid score:

> 虽然EasyText示例的重点是介绍解决方案的结构，但仍然可以学习其他新内容。文本分析是一个有着悠久历史的领域。EasyText应用程序将可读性公式（readability formula）应用于文本。其中最受欢迎的可读性公式是Flesch-Kincaid评分：

complexityflesch_kincaid=206.835−1.015*totalwords/totalsentences−84.6*totalsyllables/totalwords
Given some relatively easily derivable metrics from a text, a score is calculated. If a text scores between 90 and 100, it is easily understood by an average 11-year-old student. Texts scoring in the range of 0 to 30, on the other hand, are best suited to graduate-level students.

> 通过使用文本中一些相对容易的可推导性指标，可以计算得分。如果一个文本得分在90到100之间，那么普通11岁的学生可以很容易理解该文本，而得分在0到30范围内的文本则最适合研究生阶段的学生。

There are numerous other readability formulas, such as Coleman-Liau and Fry readability, not to mention the many localized formulas. Each formula has its own scope, and there is no single best one. Of course, this is one of the reasons to make EasyText as flexible as possible.

> 当然，还有很多其他可读性公式，如Coleman-Liau和Fry可读性公式，另外还有许多局部化公式。每个公式都有自己的范围，不存在最好的公式。当然，这也是一个让EasyText尽可能灵活的原因。

Throughout this chapter and the subsequent chapters, each of these requirements is addressed. From a functional perspective, analyzing a text comprises several steps:

> 本章及其后续章节将会满足前面的所有要求。从功能角度来看，文本分析由以下步骤组成：

1. Read the input text (either from a file, GUI, or otherwise).
2. Split the text into sentences and words (since many readability formulas work with sentence- or word-level metrics).
1. Run one or more analyses on the text.
1. Show the result to the user.

---

> 1）读取输入文本（从文件、GUI或者其他地方获取）。
> 2）将文本拆分成句子和单词（因为许多可读性公式需要使用句子或单词）。
> 3）对文本进行一次或者多次分析。
> 4）向用户显示结果。

Initially, our implementation consists of a single module, easytext. With this starting point, there is no separation of concerns. There’s just a single package inside a module, which by now we are familiar with, as shown in Example 3-2.

> 刚开始，其实现由单个模块easytext构成。从这一点上看，不存在关注点分离的问题。该模块内只有一个包，如示例3-2所示。

Example 3-2. EasyText as single module (➥ chapter3/easytext-singlemodule)

> 示例3-2：包含单个模块的EasyText（chapter3/easytext-singlemodule）
```
src
└── easytext
    ├── javamodularity
    │   └── easytext
    │       └── Main.java
    └── module-info.java
```
The module descriptor is an empty one. The Main class reads a file, applies a single readability formula (Flesch-Kincaid) and prints the results to the console. After compiling and packaging the module, it works like this:

> 模块描述符是空的。Main类首先读取了一个文件，然后应用了一个可读性公式（Flesch-Kincaid），最后向控制台打印结果。在对包进行编译和打包之后，工作过程如下所示：
```
$ java --module-path mods -m easytext input.txt
Reading input.txt
Flesh-Kincaid: 83.42468299865723
```
Obviously, the single-module setup ticks none of the boxes as far as our requirements are concerned. It’s time to add more modules into the mix.

> 显而易见，单个模块无法满足上述要求，接下来是时候添加更多的模块了。

### 3.2.2 A Tale of Two Modules 两个模块
As a first step, we separate the text-analysis algorithm and the main program into two modules. This opens up the possibility to reuse the analysis module with different frontend modules later. The main module uses the analysis module, as shown in Figure 3-1.

> 第一步，需要将文本分析算法和主程序分离成两个模块。这样一来，就可以在不同的前端模块上重复使用分析模块。主模块使用分析模块，如图3-1所示。

EasyText in two modules
Figure 3-1. EasyText in two modules

The easytext.cli module contains the command-line handling logic and file-parsing code. The easytext.analysis module contains the implementation of the Flesch-Kincaid algorithm. During the split of the single easytext module, we create two new modules with two different packages, as shown in Example 3-3.

> easytext.cli模块包含了命令行处理逻辑以及文件解析代码。easytext. analysis模块包含了Flesch-Kincaid算法的实现过程。在分离单个easytext模块的过程中，在两个不同的包中创建了两个新模块，如示例3-3所示。


Example 3-3. EasyText as two modules (➥ chapter3/easytext-twomodules)

> 示例3-3：包含两个模块的EasyText（chapter3/easytext-twomodules）
```
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
```
The difference is that the Main class now delegates the algorithmic analysis to the FleschKincaid class. Since we have two interdependent modules, we need to compile them using the multimodule mode of javac:

> 所不同的是，现在Main类将算法分析委托给FleschKincaid类。因为有两个相互独立的模块，所以需要使用javac的多模块模式进行编译。
```
javac -d out --module-source-path src -m easytext.cli
```
From this point onward, we assume all modules of our examples are always compiled together. Instead of listing all source files as input to the compiler, we specify the actual modules to be compiled with -m. In this case, providing the easytext.cli module is enough. The compiler knows through the module descriptor that easytext.cli also needs easytext.analysis, which it then compiles from the module source path as well. Just providing a list of all source files1 (not using -m), as we saw in the Hello World example, also works.

> 此后，假设示例的所有模块总是被编译在一起的。只需使用-m指定要编译的实际模块即可，而无须列出所有源文件作为编译器的输入。在这种情况下，提供easytext. cli模块就足够了。编译器通过模块描述符知道，easytext.cli也需要easytext. analysis（也是通过模块源路径进行编译）。当然，也可以像示例Hello World那样只提供所有源文件列表（不使用-m）。

The --module-source-path flag tells javac where to look for other modules in source format during compilation. It is mandatory to provide a destination directory with -d when compiling in multimodule mode. After compilation, the destination directory contains the compiled modules in exploded module format. This output directory can then be used as an element on the module path when running modules.

> --module-source-path标志告诉javac在编译期间去哪里查找源格式的其他模块。在多模块模式下进行编译时，必须使用-d提供目标目录。编译之后，目标目录包含了分解模块格式的编译模块。此输出目录还可以用作运行模块时模块路径上的一个元素。

In this example, javac looks up FleschKincaid.java on the module source path when compiling Main.java. But how does the compiler know to look in the easytext.analysis module for this class? In the old classpath situation, it might have been in any JAR that is put on the compilation classpath. Remember, the classpath is a flat list of types. Not so for the module path; it deals only with modules. Of course, the missing piece of the puzzle is in the content of the module descriptors. They provide the necessary information for locating the right module exporting a given package. No more aimless scanning of all available classes wherever they live.

> 在本示例中，当编译Main.java时，javac在模块源中查找FleschKincaid.java。但编译器是如何知道在easytext.analysis模块中查找该类呢？如果使用类路径，那么该类可以位于编译类路径上的任何JAR中。请记住，类路径是一个类型的平面列表。但模块路径却不是这样，它仅处理模块。当然，所缺少的部分位于模块描述符的内容中。这些内容提供了必要的信息以找到正确的模块并导出指定包。这样一来，不管所需的类在哪里，都无须漫无目的地扫描所有可用类。

In order for the example to work, we need to express the dependencies we’ve already seen in Figure 3-1. The analysis module needs to export the package containing the FleschKincaid class:

> 为了让示例正常运行，还需要表示出如图3-1所示的依赖关系。分析模块需要导出包含FleschKincaid类的包：

```java
module easytext.analysis {
   exports javamodularity.easytext.analysis;
}
```
With the exports keyword, packages in the module are exposed for use by other modules. By declaring that package javamodularity.easytext.analysis is exported, all its public types can now be used by other modules. A module can export multiple packages. In this case, only the FleschKincaid class is exported to other modules. Conversely, every package inside a module that is not exported is private to the module.

> 通过使用关键字exports，可以将模块中的包公开以供其他模块使用。通过声明导出包javamodularity.easytext.analysis，其所有的公共类型都可以被其他模块使用。一个模块可以导出多个包。在本示例中，仅将FleschKincaid类导出。反之，模块中未导出的包都是模块私有的。

You’ve seen how the analysis module exports the package containing the FleschKincaid class. The module descriptor for easytext.cli, on the other hand, needs to express its dependency on the analysis module:


> 前面已经介绍了分析模块如何导出包含FleschKincaid类的包，而easytext.cli的模块描述符需要表达对分析模块的依赖：

```java
module easytext.cli {
   requires easytext.analysis;
}
```
We require the module easytext.analysis because the Main class imports the FleschKincaid class, originating from that module. With both these module descriptors in place, the code compiles and can be run.

> 之所以需要easytext.analysis模块，是因为Main类导入了来自该模块的FleschKincaid类。完成了上述的模块描述符之后，就可以编译和运行代码了。

What happens if we omit the requires statement from the module descriptor? In that case, the compiler produces the following error:

> 如果在模块描述符中省略requires语句，会发生什么事情呢？此时，编译器将产生如下所示的错误：

```log
src/easytext.cli/javamodularity/easytext/cli/Main.java:11:
  error: package javamodularity.easytext.analysis is not visible
import javamodularity.easytext.analysis.FleschKincaid;
                              ^
  (package javamodularity.easytext.analysis is declared in module
   easytext.analysis, but module easytext.cli does not read it)
```
Even though the FleschKincaid.java source file is still available to the compiler (assuming we compile with -m easytext.analysis,easytext.cli to compensate for the missing requires easytext.analysis), it throws this error. A similar error is produced when we omit the exports statement from the analysis module’s descriptor. Here we see the major advantage of making dependencies explicit in every step of the software development process. A module can use only what it requires, and the compiler enforces this. At run-time, the same information is used by the resolver to ensure that all modules are present before starting the application. No more accidental compile-time dependencies on libraries, only to find out at run-time this library isn’t available on the classpath.

> 虽然FleschKincaid.java源文件对编译器可用（假设使用-m easytext.analysis, easytext.cli进行编译，以弥补所缺少的requires easytext.analysis），但仍然会抛出一个错误。如果在分析模块的描述符中省略exports语句，也会产生类似的错误。此时，可以看到在软件开发过程的每个步骤中明确依赖关系的主要优势。模块只能使用它所需要的内容，编译器会强制执行此操作。在运行时，解析器使用相同的信息，以确保在启动应用程序之前所有模块都已存在。在编译时不会出现对库的意外依赖，只有在运行时才能发现这个库在类路径上不可用。

Another check the module system enforces is for cyclic dependencies. In the previous chapter, you learned that readability relations between modules must be acyclic at compile-time. Within modules, you can still create cyclic relations between classes, as has always been the case. It’s debatable whether you really want to do so from a software engineering perspective, but you can. However, at the module level, there is no choice. Dependencies between modules must form an acyclic, directed graph. By extension, there can never be cyclic dependencies between classes in different modules. If you do introduce a cyclic dependency, the compiler won’t accept it. Adding requires easytext.cli to the analysis module descriptor introduces a cycle, as shown in Figure 3-2.

> 模块系统执行的另一个检查是循环依赖。在上一章已经讲过，在编译时，模块之间的可读性关系必须是非循环的。而在模块中，仍然可以在类之间创建循环关系，过去一直是这么做的。从软件工程的角度来看，是否真的需要这么做存在争议，但只要愿意，也是可以的。但是，在模块级别将别无选择。模块之间的依赖关系必须形成非循环的有向图。推而广之，不同模块中的类之间也不能存在循环依赖。如果引入了循环依赖关系，编译器就不会接受。向分析模块添加requireseasytext.cli，引入一个循环，如图3-2所示。

EasyText modules with illegal cyclic dependency
Figure 3-2. EasyText modules with an illegal cyclic dependency

If you try to compile this, you run into the following error:

> 如果尝试编译该模块，会出现如下所示的错误：
```log
src/easytext.analysis/module-info.java:3:
   error: cyclic dependence involving easytext.cli
   requires easytext.cli;
                    ^
```
Note that cycles can be indirect as well, as illustrated in Figure 3-3. These cases are less obvious in practice, but are treated the same as direct cycles: they result in an error from the Java module system.

> 请注意，循环也可以是间接的，如图3-3所示。虽然在实践中以下情况不太常见，但被视为与直接循环相同：它们导致Java模块系统发生错误。

Cycles can be indirect as well.
Figure 3-3. Cycles can be indirect as well

Many real-world applications do have cyclic dependencies between their components. In “Breaking Cycles”, we discuss how to prevent and break cycles in your application’s module graph.

> 许多实际应用程序的组件之间存在循环依赖关系。在5.5.2节，将会讨论如何防止和打破应用程序模块图中的循环。

## 3.3 Working with Platform Modules 使用平台模块
Platform modules come with the Java runtime and provide functionality including XML parsers, GUI toolkits, and other functionality that you expect to see in a standard library. In Figure 2-1, you already saw a subset of the platform modules. From a developer’s perspective, they behave the same as application modules. Platform modules encapsulate certain code, possibly export packages, and can have dependencies on other (platform) modules. Having a modular JDK means you need to be aware of what platform modules you’re using in application modules.

> 平台模块伴随着Java运行时，并提供了包括XML解析器、GUI工具包在内的功能以及期望在标准库中看到的其他功能。图2-1显示了平台模块的一个子集。从开发人员的角度来看，它们的行为与应用程序模块相同。平台模块封装某些代码，可能导出包，并且可以依赖其他（平台）模块。使用模块化JDK意味着需要了解应用程序模块中所使用的平台模块。

In this section, we extend the EasyText application with a new module. It’s going to use platform modules, unlike the modules we’ve created thus far. Technically we did use a platform module already: the java.base module. However, this is an implicit dependency. The new module we are going to create has explicit dependencies on other platform modules.

> 在本节，将使用新的模块扩展EasyText应用程序。扩展后的应用程序使用平台模块，而不是前面所创建的模块。从技术上讲，我们已经使用了一个平台模块：java.base模块。然而，这只是一个隐式依赖关系，接下来所创建的新模块与其他平台模块之间具有显式依赖关系。

### 3.3.1 Finding the Right Platform Module 找到正确的平台模块
If you need to be aware of the platform modules you use, how do you find out which platform modules exist? You can depend on a (platform) module only if you know its name. When you run java --list-modules, the runtime outputs all available platform modules:

> 如果需要了解所使用的平台模块，那么如何知道存在哪些平台模块呢？毕竟只有知道了名称，才能依赖（平台）模块。当运行java --list-modules时，运行时将输出所有可用的平台模块：
```
$ java --list-modules
java.base@9
java.xml@9
javafx.base@9
jdk.compiler@9
jdk.management@9
```
This abbreviated output shows there are several types of platform modules. Platform modules prefixed with java. are part of the Java SE specification. They export APIs as standardized through the Java Community Process for Java SE. The JavaFX APIs are distributed in modules sharing the javafx. prefix. Modules starting with jdk. contain JDK-specific code, which may be different across JDK implementations.

> 上面简短的输出显示了几种类型的平台模块。以java．为前缀的平台模式是Java SE规范的一部分。它们通过Java SE的JCP（Java Community Process）导出标准化的API。JavaFX API分布在共享javafx．前缀的模块中。以jdk．开头的模块包含了JDK特定的代码，在不同的JDK实现中可能会有所不同。

Even though the --list-modules functionality is a good starting point for discovering platform modules, you need more. Whenever you import from a package that’s not exported by java.base, you need to know which platform module provides this package. That module must be added to module-info.java with a requires clause. So let’s return to our example application to find out what working with platform modules entails.

> 尽管--list-modules功能是发现平台模块的良好起点，但远远不够。如果导入一个不是由java.base导出的包时，则需要知道哪个平台模块提供了这个包，此时必须使用requires子句将该模块添加到module-info.java中。因此，让我们返回到示例应用程序，了解哪些模块与平台模块一起工作。

### 3.3.2 Creating a GUI Module 创建GUI模块
EasyText so far has two application modules working together. The command-line main application has been separated from the analysis logic. In the requirements, we stated that we want to support multiple frontends on top of the same analysis logic. So let’s try to create a GUI frontend in addition to the command-line version. Obviously, it should reuse the analysis module that is already in place.

> 到目前为止，EasyText使用了两个应用程序模块，命令行主应用程序已经与分析模块分离开来。按照需求，希望在相同的分析逻辑之上支持多个前端。所以，接下来尝试创建除命令行版本之外的GUI前端。显然，该前端应该重用已经存在的分析模块。

We’ll use JavaFX to create a modest GUI for EasyText. As of Java 8, the JavaFX GUI framework has been part of the Java platform and is intended to replace the older Swing framework. The GUI looks like Figure 3-4.

> 使用JavaFX为EasyText创建一个合适的GUI。从Java 8起，JavaFX GUI框架就已经成为Java平台的一部分，旨在取代旧的Swing框架。此GUI如图3-4所示。

A simple GUI for EasyText
Figure 3-4. A simple GUI for EasyText


When you click Calculate, the analysis logic is run on the text from the text field and the resulting value is shown in the GUI. Currently, we have only a single analysis algorithm that can be selected in the drop-down, but that will change later, given our extensibility requirements. For now, we’ll keep it simple and assume that the Flesch-Kincaid analysis is the only available algorithm. The code for the GUI Main class is quite straightforward, as shown in Example 3-4.

> 当点击Calculate按钮时，分析逻辑在文本字段的文本上运行，并在GUI上显示结果值。虽然在下拉列表中只能选择一种分析算法，但稍后随着需求的扩展，会添加更多算法。目前暂时保持示例的简单性，假设FleschKincaid分析是唯一可用的算法。GUI Main类的代码非常简单，如示例3-4所示。

Example 3-4. EasyText GUI implementation (➥ chapter3/easytext-threemodules)

> 示例3-4:EasyText GUI实现过程（chapter3/easytext-threemodules）
```java
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
```
There are imports from eight JavaFX packages in the Main class. How do we know which platform modules to require in module-info.java? One way to find out in which module a package lives is through JavaDoc. For Java 9, JavaDoc has been updated to include the module name that each type is part of.

> 在Main类中导入了8个JavaFX包。如何知道module-info.java中需要哪些平台模块呢？一种方法是使用JavaDoc找出包位于哪个模块中。对于Java 9来说，JavaDoc已经进行了更新——包含了模块名称（类型是模块名称的一部分）。

Another approach is to inspect the JavaFX modules available by using java --list-modules. After running this command, we see eight modules containing javafx in the name:

> 另一种方法是使用java --list-modules检查可用的JavaFX模块。在运行完该命令后，可以看到名称中包含javafx的8个模块：
```
javafx.base@9
javafx.controls@9
javafx.deploy@9
javafx.fxml@9
javafx.graphics@9
javafx.media@9
javafx.swing@9
javafx.web@9
```
Because there is not always a one-to-one correspondence between the module name and the packages it contains, choosing the right module is somewhat of a guessing game from this list. You can inspect the module declarations of platform modules with --describe-module to verify assumptions. If, for example, we think javafx.controls might contain the javafx.scene.control package, we can verify that with the following:

> 因为模块名称与其所包含的包之间并不总是一一对应的，所以选择正确的模块有点像根据上述列表所进行的一个猜测游戏。可以使用--describe-module检查平台模块的模块声明以验证猜测是否正确。例如，如果认为javafx.controls可能包含javafx.scene.control包，那么可以使用下面的代码加以验证：

```sh
$ java --describe-module javafx.controls
javafx.controls@9
exports javafx.scene.chart
exports javafx.scene.control 1
exports javafx.scene.control.cell
exports javafx.scene.control.skin
requires javafx.base transitive
requires javafx.graphics transitive
```
1 Module javafx.controls exports the javafx.scene.control package.

> 1. 模块javafx.controls导出了javafx.scene.control包。

Indeed, the package we want is contained in this package. This process of manually finding the right platform module this way is a bit tedious. It’s expected that IDEs will support the developer with this task after Java 9 support is in place. For the EasyText GUI, it turns out we need to require two JavaFX platform modules:

> 事实上，所需要的包就包含在这个包中。以上述方法手动地找到正确平台模块的过程是相当单调乏味的。预计在Java 9提供了相应的支持之后，IDE将会帮助开发人员完成此任务。对于EasyText GUI来说，只需要两个JavaFX平台模块：
```java
module easytext.gui {
   requires javafx.graphics;
   requires javafx.controls;
   requires easytext.analysis;
}
```
Given this module descriptor, the GUI module compiles correctly. However, when trying to run it, the following curious error comes up:

> 根据上面的模块描述符，GUI模块顺利编译。但是当尝试运行时，会出现下面的奇怪错误：

```log
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
```
TIP

Another change in Java 9 is that stacktraces now also show which module a class comes from. The name before the slash (/) is the module containing the class given after the slash.

> Java 9中另一个变化是，堆栈跟踪也会显示一个类来自哪个模块。斜杠（/）之前的名称是包含斜杠之后指定类的模块。

What is going on here? The root cause is an IllegalAccessException because class Main cannot be loaded. Main extends javafx.application.Application (which lives in the javafx.graphics module) and calls Application::launch from the main method. That’s a typical way to bootstrap a JavaFX application, delegating the UI creation to the JavaFX framework. JavaFX then uses reflection to instantiate Main, subsequently invoking the start method. That means the javafx.graphics module must have access to the Main class in easytext.gui. As you learned in “Accessibility”, accessibility to a class in another module calls for two things: readability to the target module, and the target module must export the given class.

> 到底发生了什么事情呢？根本原因是由于无法加载Main类而产生了IllegalAccess-Exception。Main类扩展了javafx.application.Application（它位于javafx. graphics模块中），并从main方法调用了Application::launch。这是启动JavaFX应用程序的一种典型方式，即将UI创建委托给JavaFX框架。然后JavaFX使用反射实例化Main，随后调用start方法。这意味着javafx.graphics模块必须能够访问easytext.gui中的Main类。正如2.4节中所讲到的，访问另一个模块中的类需要满足两个条件：对目标模块的可读性以及目标模块必须导出给定的类。

In this case, javafx.graphics must have a readability relation to easytext.gui. Fortunately, the module system is smart enough to dynamically establish a readability relation to the GUI module. This happens transparently whenever reflection is used to load a class from another module. The problem is, the package containing Main is never exposed from the GUI module. Main is not accessible for the javafx.graphics module because it is not exported. This is exactly what the preceding error message tells us.

> 在这种情况下，javafx.graphics必须具有与easytext.gui的可读性关系。幸运的是，模块系统非常智能，可以动态地建立与GUI模块的可读性关系（显然，只要使用反射加载另一个模块中的类）。可问题是，包含Main类的包并不是从GUI模块中公开的。对于javafx.graphics模块来说，Main是不可访问的，因为它没有被导出，而这也正是前面的错误信息所告诉我们的内容。

One solution would be adding an exports clause for the javamodularity.easytext.gui package to the module descriptor. Only that would expose the Main class to any module requiring the GUI module. Is that really what we want? Is the Main class really part of a public API we want to support? Not really. The only reason we need it to be accessible is that JavaFX needs to instantiate it. This is a perfect use case for qualified exports:

> 一种解决方案是在模块描述符中为javamodularity.easytext.gui包添加一个exports子句。只有这样才能将Main类暴露给任何需要GUI模块的模块。这真的是我们想要的结果吗？Main类是否真的成为需要支持的公共API的一部分呢？答案是否定的。需要访问该类的唯一原因是JavaFX需要实例化它。而这恰恰是使用限制导出的时候：
```java
module easytext.gui {

   exports javamodularity.easytext.gui to javafx.graphics;

   requires javafx.graphics;
   requires javafx.controls;
   requires easytext.analysis;
}
```
TIP

During compilation, the target modules of a qualified export must be on the module path or be compiled at the same time. Obviously this is not an issue for platform modules, but it is something to be aware of when using qualified exports to nonplatform modules.

> 在编译期间，限制导出的目标模块必须在模块路径上或者同时编译。显然对平台模块来说这不是问题，但是当对非平台模块进行限制导出时，这是一个需要注意的问题。

Through the qualified exports, only javafx.graphics is able to access our Main class. Now we can run the application, and JavaFX is able to instantiate Main. In “Open Modules and Packages”, you’ll learn about another way to deal with reflective access to module internals at run-time.

> 通过限制导出，只有javafx.graphics可以访问Main类。现在运行应用程序，JavaFX能够实例化Main类。在6.1.2节，将会学习另一种方法来处理在运行时对模块内部的反射访问。

An interesting situation arises at run-time. As discussed, the javax.graphics module dynamically establishes a readability relation with easytext.gui at run-time (depicted by the bold edge in Figure 3-5).

> 在运行时出现了一个有趣的情况。如上所述，javax.graphics模块在运行时动态地建立了与easytext.gui的可读性关系（如图3-5中粗箭头所示）。

Readability edges at run-time
Figure 3-5. Readability edges at run-time

But that means there is a cycle in the readability graph! How can this be? Cycles were supposed to be impossible. They are, at compile-time. In this case, we compile easytext.gui with a dependency on (and thus readability relation to) javafx.graphics. At run-time, javax.graphics automatically establishes a readability relation to easytext.gui when it reflectively instantiates Main. Readability relations can be cyclic at run-time. Because the export is qualified, only javafx.graphics can access our Main class. Any other module establishing a readability relation with easytext.gui won’t be able to access the javamodularity.easytext.gui package.

> 这意味着可读性图中存在一个循环！这怎么可能？循环通常被认为是不可能的。在编译时可能存在循环。在本示例中，使用与javafx.graphics的依赖关系（也因此产生可读性关系）编译easytext.gui。在运行时，当它以反射方式实例化Main时，javax.graphics自动建立与easytext.gui的可读性关系。在运行时，可读性关系可以是循环的。因为导出是受限的，所以只有javafx.graphics可以访问Main类。与easytext.gui建立可读性关系的任何其他模块将无法访问javamodularity.easytext.gui包。

## 3.4 The Limits of Encapsulation 封装的限制
Looking back, we have come a long way in this chapter. You learned how to create modules, run them, and use them in conjunction with platform modules. Our example application, EasyText, has grown from a mini-monolith to a multimodule Java application. Meanwhile, it achieves two of the stated requirements: it supports multiple frontends while reusing the same analysis module, and we can create different configurations of our modules targeting the command line or a GUI.

> 回顾一下本章前面所学的内容，主要学习了如何创建和运行模块，以及如何将它们与平台模块结合使用。示例应用程序EasyText已从一个小型的整体应用程序发展成一个多模块Java应用程序。同时，还实现了两个规定要求：在重复使用相同的分析模块前提下支持多个前端，以及创建针对命令行或GUI的模块的不同配置。

Looking at the other requirements, however, there’s still a lot to be desired. As things stand, both frontend modules instantiate a specific implementation class (FleschKincaid) from the analysis module to do their work. Even though the code lives in separate modules, tight coupling is going on here. What if we want to extend the application with different analyses? Should every frontend module be changed to know about new implementation classes? That sounds like poor encapsulation. Should the frontend modules be updated with dependencies on newly introduced analysis modules? That sounds distinctly nonmodular. It also runs counter to our requirement of adding new analysis algorithms without modifying or recompiling existing modules. Figure 3-6 already shows how messy this gets with two frontends and two analyses. (Coleman-Liau is another well-known complexity metric.)

> 然而，看一下其他需求，会发现还有很多需要改进的地方。从目前情况来看，前端模块都会从分析模块中实例化一个特定的实现类（FleschKincaid）来完成自己的工作。虽然代码存在于单独的模块中，但此时却出现了紧耦合。如果想要使用不同的分析来扩展应用程序，那么应该怎么办呢？是否应该对每个前端模块进行修改以了解新的实现类呢？这听起来像是很差的封装。前端模块是否应该根据新引入的分析模块进行更新呢？这听起来很明显是非模块化的，也违反了在不修改或重新编译现有模块的情况下添加新的分析算法的要求。图3-6显示由两个前端和两个分析所带来的混乱（Coleman-Liau是另一种众所周知的复杂性算法）。

Every front-end module needs to depend on every analysis module and instantiate its exported implementation class.
Figure 3-6. Every frontend module needs to depend on all analysis modules in order to instantiate their exported implementation classes

In summary, we have two issues to address:

> 简而言之，有两个问题需要解决：

- The frontends need to be decoupled from concrete analysis implementation types and modules. Analysis modules should not export these types either to avoid tight coupling.
- Frontends should be able to discover new analysis implementations in new modules without any code changes.

---

> - 前端需要与具体分析实施类型和模块相分离。分析模块不应导出这些类型，以避免出现紧耦合。
> - 前端应该能够发现新模块中的新分析实现，而不需要更改任何代码。

By solving these two problems, we satisfy the requirement that new analyses can be introduced by adding them on the module path, without touching the frontends.

> 一旦解决了这两个问题，则只需将新的分析模块添加到模块路径上从而引入新的分析算法，而无须接触前端。

#### Interfaces and Instantiation 接口和实例化
Ideally, we’d abstract away the different analyses behind an interface. After all, we’re just passing in sentences and getting back a score for each algorithm:

> 在理想情况下，可以通过一个接口抽象出不同的分析。毕竟只是传递句子，并为每种算法返回一个分数：
```java
public interface Analyzer {

   String getName();

   double analyze(List<List<String>> text);

}
```
As long as we can find out the name of an algorithm (for display purposes) and get it to calculate the complexity, we’re good. This type of abstraction is what interfaces were made for. The Analyzer interface is stable and can live in its own module—say, easytext.analysis.api. That’s what the frontend modules should know and care about. The analysis implementation modules then require this API module as well and implement the Analyzer interface. So far, so good.

> 只要能找到一种算法的名称（为了显示的需要）并计算复杂度就可以了，这种抽象类型恰恰是接口可以实现的。Analyzer接口比较稳定，并且可以在自己的模块中使用，比如easytext.analysis.api。该接口是前端模块应该知道和关注的。分析实现模块也需要这个API模块，并实现Analyzer接口。到目前为止，一切都比较顺利。

However, there’s a problem. Even though the frontend modules care only about calling the analyze method through the Analyzer interface, they still need a concrete instance to call this method on:

> 然而，存在一个问题。即使前端模块仅关注如何通过Analyzer接口调用analyze方法，但它们仍然需要一个具体实例来调用该方法：
```java
Analyzer analyzer = ???
```
How can we get ahold of an instance that implements Analyzer without relying on a specific implementation class? You could say this:

> 如何才能找到一个实现了Analyzer接口却又不依赖特定实现类的实例呢？可以参考下面的代码：
```java
Analyzer analyzer = new FleschKincaid();
```
Unfortunately, the FleschKincaid class still needs to be exported for this to work, bringing us right back to square one. Instead, we need a way to obtain instances without referring to concrete implementation classes.

> 不幸的是，如果想要上面的代码正常工作，仍然需要公开FleschKincaid类，这样一来又回到了原点。此时需要一种方法来获取不参考具体实现类的实例。

As with all problems in computer science, we can try to solve this by adding a new layer of indirection. We will look at solutions for this problem in the next chapter, which details the factory pattern and how it leads to services.

> 与计算机科学中的所有问题一样，可以尝试通过添加一个新的间接层来解决这个问题。在下一章中将会讨论这个问题的解决方案，其详细介绍了工厂模式及其如何启动服务。

1 On a Linux/macOS system, you can easily provide $(find . -name '*.java') as the last argument to the compiler to achieve this.