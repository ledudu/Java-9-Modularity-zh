# 第 8 章 迁移到模块

> Chapter 8. Migration to Modules

With all the module goodness from the previous chapters, you are hopefully excited to start using the Java module system. Writing new code based on modules is pretty straightforward now that you understand the basic concepts.

> 在了解了前几章所介绍的模块优点之后，也许你非常希望能够开始使用 Java 模块系统。只要理解了基本概念，编写基于模块的新代码还是比较简单的。

Back in the real world, there’s also a lot of existing code that we may want to migrate to modules. The previous chapter showed how to migrate existing code to Java 9, without transforming the codebase to modules. This is the first step in any migration scenario. With that taken care of, we can focus on migrating toward the Java module system in this chapter.

> 在现实世界中，存在许多需要迁移到模块中的现有代码。上一章介绍了如何将现有的代码迁移到 Java 9，而无须将代码转换为模块。这只是任何迁移方案中的第一步。掌握了相关内容之后，接下来可以把注意力放在如何迁移到 Java 模块系统上了。

NOTE

We’re not suggesting that every existing application should be migrated to use the Java module system. If an application is not actively developed anymore, it might not be worth the work. Similarly, small applications may not really benefit from structuring in modules. Migrate to improve maintainability, changeability, and reusability when it makes sense—not just for the sake of it.

> 在此，并不建议迁移每个现有的应用程序以使用 Java 模块系统。如果一个应用程序不再被积极开发，那么这么做是不值得的。此外，小型应用程序可能不会真正从模块结构中受益。在合理的情况下进行迁移，以提高可维护性、可变性和可重用性——而不仅仅是为了迁移而迁移。

The amount of work required for a migration largely depends on how well-structured a codebase is. But even for a well-structured codebase, migration to a modular run-time can be a challenging task. Most applications use third-party libraries, which are an important factor when migrating. These libraries aren’t necessarily modularized yet, nor do you want to take on that responsibility.

> 移植所需的工作量在很大程度上取决于代码库的结构如何，但即使对于结构良好的代码库，迁移到模块化运行时也是一项艰巨的任务。大多数应用程序都使用了第三方库，这是迁移时需要考虑的一个重要因素。这些库不一定是模块化的，当然你可能也不想承担这个责任。

Luckily, the Java module system is designed with backward compatibility and migration as a primary concern. Several constructs were introduced in the Java module system to make gradual migration of existing code possible. In this chapter, you will learn about these constructs to migrate your own code to modules. Migration from the perspective of a library maintainer is, of course, related but requires a slightly different process. Chapter 10 focuses on that perspective.

> 幸运的是，Java 模块系统将向后兼容性和可移植性作为主要的关注点。在 Java 模块系统中引入了几个结构，从而可以对现有代码进行逐步迁移。在本章中，将了解这些构造，从而将自己的代码迁移到模块。从库维护者的角度来看，迁移当然是需要的，但是需要一个略微不同的过程（第 10 章将着重介绍该内容）。

## 8.1 Migration Strategies 迁移策略

A typical application has application code (your code) and library code. The application code uses code from third-party libraries. Ideally, all the libraries used are already modules, and we can focus on modularizing our own code. For the first years after the release of Java 9, this is probably not a realistic scenario. Some libraries may not be available as modules yet, and perhaps never will be, because they are no longer maintained.

> 一个典型的应用程序包含应用程序代码以及库代码，而应用程序代码可能会使用来自第三方库的代码。在理想情况下，所有使用的库都已经是模块，此时只需专注于模块化自己的代码即可。在 Java 9 发布后的头几年，现实情况可能不是这样的，一些库还不能作为模块来使用，也许永远不能，因为它们不再被维护。

If we were to wait until the whole ecosystem moved toward modules, we might have to wait very long. Also, this would require updating to new versions of those libraries, potentially causing its own set of issues. We could also manually patch the libraries, adding a module descriptor and transforming them to a module. This is clearly a lot of work, and requires forking the library, which makes future updates more painful. It would be much better if we can focus on migrating our own code, while leaving libraries the way they are for the moment.

> 如果一直等待整个生态系统向模块方向发展，那么可能要等很长时间。此外，还需要更新到这些库的新版本，从而可能引起一系列问题。也可以手动修补库，添加一个模块描述符并将它们转换为一个模块。显然，这需要完成大量工作，并且需要对库进行拆分（fork），这使得未来的更新变得更加痛苦。如果能够专注于迁移自己的代码，而暂时保留库现有的工作方式，那就最好了。

## 8.2 A Simple Example 一个简单示例

You will look at several migration examples in this chapter to understand the various cases you can run into in practice. To start, we take a simple application that uses the Jackson library to convert a Java object to JSON. For this application, we need three JAR files from the Jackson project:

> 本章将介绍几个迁移示例，以了解实践中可能遇到的各种情况。首先看一个简单的应用程序，其使用 Jackson 库将 Java 对象转换为 JSON。对于这个应用程序，需要 Jackson 项目中的三个 JAR 文件：

- com.fasterxml.jackson.core
- com.fasterxml.jackson.databind
- com.fasterxml.jackson.annotations

The Jackson JAR files in the version used for this example (2.8.8) are not modules yet. They are plain JAR files without module descriptors.

> 本例所使用的版本（2.8.8）中的 Jackson JAR 文件并不是模块。它们是普通 JAR 文件，没有模块描述符。

The application consists of two classes, with the main class listed in Example 8-1. Not listed here is the Book class, a simple class with getters and setters representing a book. The Main class contains a main method using ObjectMapper from com.fasterxml.jackson.databind to convert a Book instance to JSON.

> 应用程序由两个类组成，其中示例 8-1 列出了主类。这里没有列出的 Book 类是一个表示书的简单类，包含 getter 和 setter。Main 类包含一个使用 com.fasterxml.jackson. databind 的 ObjectMapper 将一个 Book 实例转换为 JSON 的 main 方法。

Example 8-1. Main.java (➥ chapter8/jackson-classpath)

> 示例 8-1:Main.java（chapter8/jackson-classpath）

```java
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
```

The com.fasterxml.jackson.databind.ObjectMapper class in the example is part of jackson-databind-2.8.8.jar. This JAR file has a dependency on both jackson-core-2.8.8.jar and jackson-annotations-2.8.8.jar. However, this dependency information is implicit because the JAR files are not modules. The example project has the following file structure to start with:

> 示例中的 com.fasterxml.jackson.databind.ObjectMapper 类是 jackson-databind-2.8.8.jar 的一部分。这个 JAR 文件依赖于 jackson-core-2.8.8.jar 和 jackson-annotations-2.8.8.jar。但是，这种依赖关系是隐式的，因为 JAR 文件不是模块。示例项目开始时具有以下文件结构：

```
├── lib
│   ├── jackson-annotations-2.8.8.jar
│   ├── jackson-core-2.8.8.jar
│   └── jackson-databind-2.8.8.jar
└── src
    └── demo
        ├── Book.java
        └── Main.java
```

As you saw in the previous chapter, the classpath is still available in Java 9. Let’s start with building and running on the classpath, before we start migration to modules. We can build and run the application with the commands in Example 8-2.

> 正如在上一章中看到的那样，类路径在 Java 9 中仍然可用。在开始迁移到模块之前，首先在类路径上构建和运行。可以使用示例 8-2 中的命令来构建和运行应用程序。

Example 8-2. run.sh (➥ chapter8/jackson-classpath)

> 示例 8-2:run.sh（chapter8/jackson-classpath）

```
CP=lib/jackson-annotations-2.8.8.jar:
CP+=lib/jackson-core-2.8.8.jar:
CP+=lib/jackson-databind-2.8.8.jar

javac -cp $CP -d out -sourcepath src $(find src -name '*.java')

java -cp $CP:out demo.Main
```

This application compiles and runs with Java 9 without any changes.

> 该应用程序可以使用 Java 9 编译和运行，而无须进行任何更改。

The Jackson libraries are not directly under our control, but the Main and Book code is, so that’s what we focus on for migration. This is a common migration scenario. We want to move our own code toward modules, without worrying about libraries. The Java module system has some tricks up its sleeve to make gradual migration possible.

> 虽然 Jackson 库并不在我们的直接控制之下，但我们却可以控制 Main 和 Book 的代码，所以这些代码是迁移时所需要重点关注的。这是一个非常常见的迁移情况，将自己的代码转移到模块上，而不用担心库。Java 模块系统提供了一些技巧来实现逐步迁移。

## 8.3 Mixing Classpath and Module Path 混合类路径和模块路径

To make gradual migration possible, we can mix the usage of classpath and module path. This is not an ideal situation, since we only partially benefit from the advantages of the Java module system. However, it is extremely helpful for migrating in small steps.

> 为了使逐步迁移成为可能，可以混合使用类路径和模块路径。但这不是一种理想的情况，因为只能部分受益于 Java 模块系统的优点。但是，“小步”迁移是非常有帮助的。

Because the Jackson libraries are not our own source code, ideally we would not change them at all. Instead we start the migration top-down, by migrating our own code first. Let’s put this code in a module named books. You will see in a moment that this isn’t sufficient, but let’s start by creating a simple module-info.java for our module:

> 因为 Jackson 库不是我们自己的源代码，所以理想情况下根本不会更改它们。相反，首先通过迁移自己的代码开始自上而下的迁移。接下来将代码放到一个名为 books 的模块中。虽然这样做是远远不够的，但却可以通过为模块创建一个简单的 module-info.java 开始迁移之旅：

```java
module books {

}
```

Note that the module doesn’t have any requires statements yet. This is suspicious because we clearly do have a dependency on classes from the jackson-databind-2.8.8.jar JAR file. Because we now have a real module, we can compile our code by using the --module-source-path flag. The Jackson libraries are not modules, so they stay on the classpath for now:

> 请注意，该模块还没有任何 requires 语句。这非常可疑，因为显而易见其需要依赖于 jackson-databind-2.8.8.jar 文件中的类。因为已经有了一个真正的模块，所以可以使用--module-source-path 标志来编译代码。Jackson 库不是模块，所以暂时留在类路径上：

```log
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
```

The compiler is clearly not happy! Although jackson-databind-2.8.8.jar is still on the classpath, the compiler tells us that it is not usable in our module. Modules cannot read the classpath, so our module can’t access types on the classpath, as illustrated in Figure 8-1.

> 此时，编译器显然不开心！尽管 jackson-databind-2.8.8.jar 仍然在类路径中，但编译器却告知无法在模块中使用。模块不能读取类路径，所以模块不能访问类路径上的类型，如图 8-1 所示。

Modules don't read the classpath.
Figure 8-1. Modules don’t read the classpath

Not being able to read from the classpath is a good thing, even when it requires some work during migration, because we want to be explicit about our dependencies. When modules can read the classpath, besides their explicit dependencies, all bets are off.

> 即使在迁移过程中需要做一些工作，而无法从类路径中读取却是一件好事，因为需要明确依赖关系。如果模块可以读取类路径，除了便于获取显式依赖关系之外，其他的优势就不复存在了。

Even so, our application is not compiling now, so let’s try to fix that first. If we can’t rely on the classpath, the only way forward is to make the code that our module relies on available as a module as well. This requires us to turn jackson-databind-2.8.8.jar into a module.

> 即便如此，应用程序现在还无法编译，所以先试着解决这个问题。如果不能依赖类路径，那么唯一的方法就是将模块依赖的代码作为模块来使用。这就要求将 jackson-databind-2.8.8.jar 变成一个模块。

## 8.4 Automatic Modules 自动模块

The source code of the Jackson libraries is open source, so we could patch the code to turn it into a module ourselves. In a large application that uses a long list of (transitive) dependencies, patching all of them isn’t appealing. Moreover, we probably don’t have enough knowledge of the libraries to properly modularize them.

> Jackson 库的源代码是开源的，所以可以修补代码，将其变为一个模块。在使用长列表（可传递）依赖关系的大型应用程序中，并不需要修补所有的依赖关系。此外，我们可能还没有掌握正确模块化库所需的知识。

The Java module system has a useful feature to deal with code that isn’t a module yet: automatic modules. An automatic module can be created by moving an existing JAR file from the classpath to the module path, without changing its contents. This turns the JAR into a module, with a module descriptor generated on the fly by the module system. In contrast, explicit modules always have a user-defined module descriptor. All modules we’ve seen so far, including platform modules, are explicit modules. Automatic modules behave differently than explicit modules. An automatic module has the following characteristics:

> Java 模块系统提供了一个有用的功能来处理非模块的代码：自动模块。只需将现有的 JAR 文件从类路径移动到模块路径，而不改变其内容，就可以创建一个自动模块。这样一来，JAR 就转换为一个模块，同时模块系统动态生成模块描述符。相比之下，显式模块始终有一个用户自定义的模块描述符。到目前为止，所看到的所有模块（包括平台模块）都是显式模块。自动模块的行为不同于显式模块。自动模块具有以下特征：

- It does not contain module-info.class.
- It has a module name specified in META-INF/MANIFEST.MF or derived from its filename.
- It requires transitive all other resolved modules.
- It exports all its packages.
- It reads the classpath (or more precisely, the unnamed module as discussed later).
- It cannot have split packages with other modules.

---

> - 不包含 module-info.class。
> - 它有一个在 META-INF/MANIFEST.MF 中指定或者来自其文件名的模块名称。
> - 通过 requires transitive 请求所有其他已解析模块。
> - 导出所有包。
> - 读取路径（或者更准确地讲，读取前面所讨论的未命名模块）。
> - 它不能与其他模块拆分包。

This makes the automatic module immediately usable for other modules. Mind you, it’s not a well-designed module. Requiring all modules and exporting all packages doesn’t sound like proper modularization, but at least it’s usable.

> 上述特征使得自动模块可以立即用于其他模块。请注意，自动模块并不是一个设计良好的模块。虽然请求所有的模块并导出所有的包听起来不像是正确的模块化，但至少是可用的。

What does it mean to require all other resolved modules? An automatic module requires every module in the already resolved module graph. Remember, there still is no explicit information in an automatic module telling the module system which other modules it really needs. This means the JVM can’t warn at startup when dependencies of automatic modules are missing. We, as developers, are responsible for making sure the module path (or classpath) contains all required dependencies. This is not very different from working with the classpath.

> 请求所有其他已解析模块是什么意思呢？自动模块需要已解析模块图中的每个模块。请记住，自动模块中仍然没有明确的信息来告诉模块系统真正需要哪些模块，这意味着 JVM 在启动时不会警告自动模块的依赖项丢失。作为开发人员，负责确保模块路径（或类路径）包含所有必需的依赖项。这与使用类路径没有太大区别。

All modules in the module graph are required transitive by automatic modules. This effectively means that if you require one automatic module, you get implied readability to all other modules “for free.” This is a trade-off, which we will discuss in more detail soon.

> 模块图中的所有模块都需要通过自动模块传递。这实际上意味着，如果请求一个自动模块，那么就可以“免费”获得所有其他模块的隐式可读性。这是一种权衡，将在稍后详细讨论。

Let’s take the jackson-databind-2.8.8.jar JAR file and turn it into an automatic module, by moving it to the module path. First we move the JAR file to a new directory that we name mods in this example:

> 请将 jackson-databind-2.8.8.jar 文件移动到模块路径，从而将其转换成一个自动模块。首先把 JAR 文件移动到一个新的目录，在本示例中将其命名为 mods：

```
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
```

Next we have to modify the module-info.java in our books module to require jackson.databind:

> 接下来必须修改 books 模块中的 module-info.java，以便请求 jackson.databind：

```java
module books {
  requires jackson.databind;
}
```

The books module requires jackson.databind as if it were a normal module. But where did the module name come from? The name of an automatic module can be specified in the newly introduced Automatic-Module-Name field of a META-INF/MANIFEST.MF file. This provides a way for library maintainers to choose a module name even before they fully migrate the library to be a module. See “Choosing a Library Module Name” for more details about naming modules this way.

> books 模块请求 jackson.databind，就好像它是一个正常的模块。但模块名称来自哪里呢？自动模块的名称可以在 META-INF/MANIFEST.MF 文件的新引入的 Automatic-Module-Name 字段中指定。这样一来，即使在将库完全迁移到模块之前，库维护人员也可以选择模块名称。有关以这种方式命名模块的更多详细信息，请参阅 10.2 节。

If no name is specified, the module name is derived from the JAR’s filename. The naming algorithm is roughly the following:

> 如果没有指定名称，则模块名称是从 JAR 的文件名派生的。命名算法大致如下：

- Dashes (-) are replaced by dots (.).
- Version numbers are omitted.

---

> - 使用点（.）替换破折号（-）。
> - 忽略版本号。

In the Jackson example, the module name is based on the filename.

> 在 Jackson 示例中，模块名称是基于文件名称的。

We can now successfully compile the program by using the following command:

> 现在，可以通过运行下面的命令成功地编译程序：

```
CP=lib/jackson-annotations-2.8.8.jar:
CP+=lib/jackson-core-2.8.8.jar

javac -cp $CP --module-path mods -d out --module-source-path src -m books
```

The jackson-databind-2.8.8.jar JAR file is removed from the classpath, and a module path is now configured, pointing to the mods directory. Figure 8-2 provides an overview of where all the code lives.

> 将 jackson-databind-2.8.8.jar 文件从类路径中移除，并配置了一个模块路径，指向 mods 目录。图 8-2 提供了所有代码的概述。

Non-modular JARs on the module path become automatic modules. The classpath becomes the unnamed module.
Figure 8-2. Nonmodular JARs on the module path become automatic modules. The classpath becomes the unnamed module.

To run the program, we also have to update the java invocation:

> 为了运行程序，还需要更新 Java 调用：

```
java -cp $CP --module-path mods:out -m books/demo.Main
```

We made the following change to the java command:

> 对 Java 命令完成如下修改：

- Move the out directory to the module path.
- Move jackson-databind-2.8.8.jar from the classpath (lib) to the module path (mods).
- Start the application by using the -m flag to specify our module.

---

> - 将 out 目录移至模块路径中。
> - 将 jackson-databind-2.8.8.jar 文件从类路径（lib）移至模块路径（mods）中。
> - 通过使用-m 标志来指定模块，从而启动应用程序。

TIP

Instead of moving just jackson-databind to the module path, we could have moved all the JARs to the module path. This makes the process a little easier, but makes it harder to see what happens. Feel free to move all JARs to the module path when migrating your own applications.

> 可以将所有 JAR 移动到模块路径，而不仅仅是移动（jackson-databind）。虽然这样做使得移动过程变得容易一些，但是很难看到究竟发生了什么。当迁移自己的应用程序时，可以随意将所有 JAR 移动到模块路径。

We’re close to our first migrated application, but unfortunately we still get an exception when starting the application:

> 现在已经很接近第一个迁移的应用程序了，但不幸的是在启动应用程序时仍然遇到了异常：

```log
Exception in thread "main" java.lang.reflect.InaccessibleObjectException:
  Unable to make public java.lang.String demo.Book.getTitle() accessible:
  module books does not "exports demo" to module jackson.databind
  ...
```

This is a problem specific to Jackson Databind, but not an uncommon scenario. We use Jackson Databind to marshal the Book class, which is part of the books module. Jackson Databind uses reflection to look at the fields of a class to be able to serialize it. Therefore, Jackson Databind needs to access the Book class; otherwise, it can’t use reflection to look at its fields. For this to be possible, the package containing the class must be exported or opened by its containing module (books, in this example). Exporting the package restricts Jackson Databind to reflect only over public elements, whereas opening the package allows for deep reflection as well. In our example, reflecting over public elements is enough.

> 虽然这是 Jackson Databind 所特有的问题，但并不罕见。此时使用 Jackson Databind 来编组 Book 类（该类是 books 模块的一部分）。Jackson Databind 使用反射来查看类的字段以便进行序列化。因此，Jackson Databind 需要访问 Book 类；否则，就无法使用反射来查看它的字段。为此，包含该类的包必须通过其包含模块（本例中的 books 模块）被导出或开放。导出包限制了 Jackson Databind 只能反射公共元素，而开放包则还允许进行深度反射。在本示例中，反射公共元素就足够了。

This puts us in a difficult position. We don’t necessarily want to export the package containing Book to other modules, just because Jackson needs it. If we did, we would give up on encapsulation, and that was one of the primary reasons to move to modules! There are multiple ways to work around this problem, each with its own trade-offs. The first way is to use a qualified export. Using a qualified export, we can export the package to just jackson.databind, so we don’t lose encapsulation with respect to other modules:

> 此时陷入了一个困境。没有必要仅因为 Jackson 需要 Book 类而将包含 Book 的包导出到其他模块。如果这样做，就意味着放弃封装，而封装性是将现有应用程序转换为模块的主要原因之一！解决这个问题有多种方法，每种方法都有自己的权衡考虑。第一种方法是使用限制导出。通过使用限制导出，可以将包仅导出到 jackson.databind，同时又不会失去其他模块的封装：

```java
module books {
  requires jackson.databind;

  exports demo to jackson.databind;
}
```

After recompiling, we can now successfully run the application! We have another option besides exporting that may better fit our needs when it comes to reflection, and we will explore this in the next section.

> 重新编译后，现在可以成功运行应用程序了！除了使用导出方法以外，还有另外一种方法可以更好地适应需求，该方法将在下一节中探讨。

#### WARNINGS WHEN USING AUTOMATIC MODULES 使用自动模块时出现的警告

Although automatic modules are essential to migration, they should be used with some caution. Whenever you write a requires on an automatic module, make a mental note to come back to this later. If the library is released as an explicit module, you want to use that instead.

> 尽管自动模块对迁移至关重要，但应谨慎使用。每当在自动模块上写入一个 requires 时，请记住，稍后再回来。如果库作为一个显式模块发布，则应该使用该模块。

Two warnings were added to the compiler to help with this as well. Note that it is only a recommendation that Java compilers support these warnings, so different compiler implementations may have different results. The first warning is opt out (enabled by default), and will give a warning for every requires transitive on an automatic module. The warning can be disabled with the -Xlint:-requires-transitive-automatic flag. Notice the dash (-) after the colon. The second warning is opt in (disabled by default), and will give a warning for every requires on an automatic module. This warning can be enabled with the -Xlint:requires-automatic (no dash after the colon) flag. The reason that the first warning is enabled by default is that it is a more dangerous scenario. You expose a (possibly volatile) automatic module to consumers of your module through implied readability.

> 编译器中增加了两个警告，以帮助解决这个问题。请注意，Java 编译器支持这些警告只是一个建议，所以不同的编译器实现可能会有不同的结果。第一个警告是选择退出（默认情况下启用），并针对自动模块上的每个 requires transitive 发出警告。该警告可以通过-Xlint:-requires-transitive-automatic 标志禁用。请注意冒号后的短划级（-）。第二个警告是选择进入（默认情况下禁用），并针对自动模块上的每个 requires 发出警告。该警告可以通过-Xlint:requires-automatic（冒号后没有短划线）标志来启用。第一个警告之所以是默认启用的，原因在于这是一种更危险的情况。通过隐式可读性，会将一个（可能不稳定的）自动模块公开给模块的使用者。

Replace automatic modules with explicit modules when available, and ask the library maintainer for one if not available yet. Also remember that such a module may have a more restricted API, because the default of having all packages exported is likely not what the library maintainer intended. This can lead to extra work when switching from an automatic module to an explicit module, where the library maintainer has created a module descriptor.

> 当显式模块可用时，可以用显式模块替代自动模块，如果显式模块尚不可用，可以要求库维护者提供。另外请记住，这样的模块可能提供了受更多限制的 API，因为库维护者并不希望默认情况下导出所有包。当从自动模块切换到显式模块时会产生额外的工作，库维护者需要创建一个模块描述符。

## 8.5 Open Packages 开放式包

Using exports in the context of reflection has some caveats. First of all, it’s arguably strange that we need to give compile-time readability to a package, while we only expect run-time (reflection) usage. Frameworks often use reflection to work on application code, but they don’t need compile-time readability. Also, we might not always know which module requires readability up front, so a qualified export isn’t possible.

> 在反射的上下文中使用 exports 时有一些需要注意的地方。首先，需要将编译时可读性赋予一个包，这一点看上去非常奇怪，因为我们只期望运行时（反射）使用。虽然框架经常使用反射来处理应用程序代码，但是它们不需要编译时可读性。另外，不可能总是事先知道哪个模块需要可读性，所以限制导出是不可能的。

Using the Java Persistence API (JPA) is an example of such a scenario. When using JPA, you typically program to the standardized API. At run-time, you would use an implementation of this API such as Hibernate or EclipseLink. The API and the implementation live in separate modules. In the end, the implementation needs accessibility to your classes. If we put an exports com.mypackage to hibernate.core or something like that in our module, we’d be suddenly coupled to the implementation. Changing JPA implementations would require us to change the module descriptor of our code, which would be a clear sign of leaking implementation details.

> 使用 Java Persistence API（JPA）就是这种情况的一个例子。当使用 JPA 时，通常编程为标准化的 API。而在运行时，会使用该 API 的实现，比如 Hibernate 或者 EclipseLink。API 和实现在不同的模块中。最终，实现需要访问类。如果在模块中将 exports com. mypackage 放到 hibernate.core 或类似的包中，就会连接到实现。如果想要更改 JPA 实现，就需要更改代码的模块描述符，而这恰恰是泄露实现细节的迹象。

As discussed in detail in “Open Modules and Packages”, another problem arises when it comes to reflection. Exporting a package exports only the public types in the package. Protected or package-private classes, and nonpublic methods and fields in exported classes, are not accessible. Deep reflection, using the setAccessible method, will not work even when a package is exported. To allow deep reflection (which many frameworks need), a package must be open.

> 如 6.1.2 节中所讨论的那样，当涉及反射时会出现另一个问题：导出包只导出了包中的公共类型，受保护的或者包专用的类以及导出类中的非公共方法和字段都是不可访问的。即使导出了包，深度反射（使用 setAccessible 方法）也不起作用。如果想要进行深度反射（许多框架需要该功能），一个包必须是开放的。

Going back to our Jackson example, we can use the opens keyword instead of the qualified export to jackson.databind:

> 返回到 Jackson 示例，此时可以使用 opens 关键字，而不是对 jsckson.databind 进行限制导出：

```java
module books {
  requires jackson.databind;

  opens demo;
}
```

An open package gives run-time access (including deep reflection) to its types to any module, but no compile-time access. This avoids others using your implementation code accidentally at compile-time, while frameworks can do their magic without problems at run-time. When only run-time access is required, opens is a good choice in most cases. Remember that an open package is not truly encapsulated. Another module can always access the package by using reflection. But at least we’re protected against accidental usage during development, and it clearly documents that the package isn’t meant to be used directly by other modules.

> 一个开放式包允许任何模块对其类型进行运行时访问（包括深度反射），但编译时访问却是禁止的。这避免了其他人在编译时意外地使用了实现代码，而框架可以在运行时毫无问题地施展它们的“魔力”。当仅需要运行时访问时，在大多数情况下 opens 是一个不错的选择。请记住，一个开放式包并没有真正地封装，其他模块始终可以使用反射来访问这个包。但至少在开发过程中受到了保护，避免了意外使用，并且清楚地表明这个软件包不能被其他模块直接使用。

Like the exports keyword, the opens keyword can be qualified. This way, a package can be opened just to a limited set of modules:

> 与 exports 关键字一样，opens 关键字也可以被限制，从而向一组有限的模块开放包：

```java
module books {
  requires jackson.databind;

  opens demo to jackson.databind;
}
```

Now that you have seen two ways to work around the run-time accessibility problem, one question remains: why did we find out about this problem only when running the application, and not during compilation? Let’s revisit the rules of readability again to understand this scenario better. For a class to able to read another class from another module, the following needs be true:

> 现在已经看到了解决运行时可访问性问题的两种方法，但仍然存在一个问题：为什么只有在运行应用程序时才发现此问题，而不是在编译时呢？为了更好地理解这种情况，需要重新审视可读性规则。对于一个能够读取来自另一个模块中另一个类的类来说，需要满足以下条件：

- The class needs to be public (ignoring the case of deep reflection).
- The package in the other module must be exported, or opened in the case of deep reflection.
- The consuming module must have a readability relation to the other module (requires).

---

> - 该类必须是公共的（忽略深度反射的情况）。
> - 另一个模块中的包必须被导出，或者在进行深度反射时是开放的。
> - 消费模块与其他模块之间必须具有可读性关系（requires）。

Usually, this can all be checked at compile-time. Jackson Databind doesn’t have a compile-time dependency on our code, however. It learns about our Book class only because we pass it in as an argument to ObjectMapper. This means the compiler can’t help us. When doing reflection, the runtime takes care of automatically setting up a readability relation (requires), so this step is taken care of. Next it will find out that the class is not exported nor opened (and therefore not accessible) at run-time, and this is not automatically “fixed” by the runtime.

> 通常，可以在编译时对上述条件进行检查。然而，Jackson Databind 对所编写的代码并不存在编译时依赖。它之所以知道 Book 类，因为它将作为参数传入 ObjectMapper。这意味着编译器并不能帮助我们。在进行反射时，运行时会自动设置一个可读性关系（requires），所以该步骤要额外小心。接下来，它会发现该类在运行时没有导出或开放（因此也不可访问），并且不会由运行时自动“修复”。

If the runtime is smart enough to add a readability relation automatically, why doesn’t it also take care of opening the package? This is about intentions and module ownership. When code uses reflection to access code in another module, the intention from that module’s perspective is clearly to read the other module. It would be unnecessary extra boilerplate to be (even more) explicit about this. The same is not true for exports/opens. The module owner should decide which packages are exported or opened. Only the module itself should define this intention, so it can’t be automatically inferred by the behavior of some other module.

> 既然运行时足够聪明，可以自动添加可读性关系，那么为什么不能开放包呢？这涉及意图和模块所有权的问题。当代码使用反射访问另一个模块中的代码时，从该模块的角度来看，其目的显然是读取其他模块，所以无须过多地明确这一点。但对于 exports/opens 来说却不是这样的。模块所有者应决定导出或开放哪些包。只有模块本身应该定义这个意图，所以该意图不能被其他模块的行为自动推断出来。

Many frameworks use reflection in a similar way, so it’s always important to test well after migration.

> 许多框架以类似的方式使用反射，所以在迁移之前进行测试通常是非常重要的。

TIP

In “Libraries, Strong Encapsulation, and the JDK 9 Classpath”, you learned that by default Java 9 runs with --illegal-acces=permit. Why did we still have to explicitly open our package for reflection? Remember that the --illegal-access flag affects only code on the classpath. In this example, jackson.databind is a module itself, reflecting on code in our module (not a platform module). There’s no code on the classpath involved.

> 在 7.2 节中已经讲过，在默认情况下，Java 9 使用--illegal-access =permit 运行。那么为什么仍然为了进行反射而显式地开放包呢？请记住，--illegal-access 标志只影响类路径上的代码。在本示例中，jackson. databind 本身是一个模块，反映了我们所编写模块（不是平台模块）中的代码。而在涉及的类路径中没有代码。

## 8.6 Open Modules 开放式模块

In the previous section, you saw the use of open packages to give run-time-only access to a package. This is great to satisfy the reflection needs that many frameworks and libraries have. If we’re in the middle of a large migration, in a codebase that is not completely modularized yet, it may not be immediately obvious which packages need to be open. Ideally, we know exactly how the frameworks and libraries that we use access our code, but maybe we’re not intimately familiar with the codebase we’re working with. This could result in a tedious trial-and-error process, trying to figure out which packages need to be open. For these situations, we can use open modules, which is a less precise but more powerful tool:

> 在前面的章节中，学习了如何使用开放式包提供对包的运行时访问，这可以满足许多框架和库的反射需求。如果正在进行大规模的迁移，那么在一个尚未完全模块化的代码库中，哪些包需要开放可能并不是那么明显。理想情况下或许可以确切地知道所使用的框架和库是如何访问代码的，但我们可能对正在使用的代码库并不熟悉。这样一来，就会导致一个单调乏味的试错过程，试图找出哪些包需要开放。针对这些情况，可以使用开放式模块，这是一个不太精确但功能更强大的工具：

```java
open module books {
  requires jackson.databind;
}
```

An open module is a module that gives run-time access to all its packages. This does not grant compile-time access to packages, which is exactly what we want for our migrated code. If a package needs to be used at compile-time, it must be exported. Create an open module first to avoid reflection-related problems. This helps you focus on requires and compile-time usage (exports) first. Once the application is working again, we can fine-tune run-time access to the packages as well by removing the open keyword from the module, and be more specific about which packages should be open.

> 开放式模块提供了对其所有包的运行时访问，但并不会授予对包的编译时访问权限，而这正是迁移代码想要的。如果需要在编译时使用包，则必须将其导出。首先创建一个开放式模块以避免与反射有关的问题，这样做有助于首先关注需求（requires）和编译时使用（exports）。一旦应用程序再次运行，可以通过从模块中删除 open 关键字来更好地调整对包的运行时访问权限，同时更具体地说明哪些包应该开放。

## 8.7 VM Arguments to Break Encapsulation 破坏封装的 VM 参数

In some scenarios, adding export or opens to a module is not an option. Maybe we don’t have access to the code, or maybe access is required only during testing. In these scenarios, we can set up additional exports by using a VM argument. You have already seen this in action in “Compilation and Encapsulated APIs” for platform modules, and we can do the same for other modules, including our own.

> 在某些情况下，向模块添加 exports 或者 opens 并不是一个选项，可能是无法访问代码，或者只有在测试期间才需要访问。在这些情况下，可以使用 VM 参数来设置更多的导出。对于平台模块，已经在 7.3 节中看到了这一点，也可以对其他模块（包括自己的模块）执行相同的操作。

Instead of adding an exports or opens clause to the books module descriptor, we can use a command-line flag to achieve the same:

> 可以使用命令行标志来实现相同的功能，而不是向 books 模块描述符添加 exports 或 opens 子句：

```
--add-exports books/demo=jackson.databind
```

The complete command to run the application then is as follows:

> 运行应用程序的完整命令如下所示：

```
java -cp lib/jackson-annotations-2.8.8.jar:lib/jackson-core-2.8.8.jar \
  --module-path out:mods \
  --add-exports books/demo=jackson.databind \
  -m books/demo.Main
```

This sets up a qualified export when starting the JVM. A similar flag exists to open packages: --add-opens. Although these flags are useful in special cases, they should be treated as a last resort. The same mechanism can also be used to gain access to internal, nonexported packages, as you saw in the previous chapter. Although this can be a temporary workaround until code is properly migrated, it should be used with great care. Breaking encapsulation should not be done lightly.

> 上述命令在启动 JVM 时设置了一个限制导出。可以使用一个类似的标志来打开包：--add-opens。虽然这些标志在特殊情况下是有用的，但它们应被视为最后的手段。如前一章所示，可以同样的机制访问内部的非导出包。虽然在代码正确迁移之前这可能是一个临时的解决方法，但应该非常谨慎地使用。不应该轻易地破坏封装。

## 8.8 Automatic Modules and the Classpath 自动模块和类路径

In the previous chapter, you saw the unnamed module. All code on the classpath is part of the unnamed module. In the Jackson example, you learned that code in a module you’re compiling can’t access code on the classpath. How is it then possible for the jackson.databind automatic module to work correctly while the Jackson Core and Jackson Annotations JARs, which it depends on, are still on the classpath? This works because these libraries are in the unnamed module. The unnamed module exports all code on the classpath and reads all other modules. There is a big restriction, however: the unnamed module itself is readable only from automatic modules!

> 在前一章中已经看到过未命名模块，类路径上的所有代码都是未命名模块的一部分。在 Jackson 示例中已经讲过，正在编译的模块中的代码不能访问类路径上的代码。那么当 jackson.databind 自动模块所依赖的 Jackson Core 和 Jackson Annotations JAR 仍然在类路径上时该模块是如何正常工作的？该模块能正常工作是因为这些库位于未命名模块中。未命名模块导出类路径中的所有代码并读取所有其他模块。然而这存在一个很大的限制：未命名模块本身只能通过自动模块读取！

Figure 8-3 illustrates the difference between automatic modules and explicit modules when it comes to reading the unnamed module. An explicit module can read only other explicit modules, and automatic modules. An automatic module reads all modules including the unnamed module.

> 图 8-3 显示了当读取未命名模块时自动模块和显式模块之间的区别。显式模块只能读取其他显式模块和自动模块。而自动模块可读取所有模块，包括未命名模块。

Only automatic modules can read the classpath.
Figure 8-3. Only automatic modules can read the classpath

The readability to the unnamed module is only a mechanism to facilitate automatic modules in a mixed classpath/module path migration scenario. If we were to use a type from Jackson Core directly in our code (as opposed to from an automatic module), we would have to move Jackson Core to the module path as well. Let’s try this in Example 8-3.

> 未命名模块的可读性只是一种在混合类路径/模块路径迁移方案中有助于自动模块的机制。如果想要在代码中直接使用来自 Jackson Core 的类型（而不是来自自动模块），那么必须将 JacksonCore 移动到模块路径中。如示例 8-3 所示。

Example 8-3. Main.java (➥ chapter8/readability_rules)

> 示例 8-3:Main.java（chapter8/readability_rules）

```java
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
```

1. Importing the Versioned type from Jackson Core
2. Using the Versioned type to print the library version

---

> 1. 从 Jackson Core 导出 Versioned 类型。
> 2. 使用 Versioned 类型打印库版本。

The Jackson Databind ObjectMapper type implements the interface Versioned from Jackson Core. Note that this wasn’t a problem until we started to use this type explicitly in our module. Any time we use an external type in a module, we should immediately think requires. Let’s demonstrate this by trying to compile the code, which will result in an error:

> Jackson Databind 的 ObjectMapper 类型实现了 Jackson Core 的 Versioned 接口。请注意，自在模块中显式地使用该类型之前都没有什么问题。当需要在一个模块中使用外部类型时，都应该立即考虑 requires。接下来通过编译该代码来证明这一点，此时将产生一个错误：

```log
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
```

Although the type exists in the unnamed module (the classpath), and the jackson.databind automatic module can access it, we can’t access it from our module. To fix this problem, we need to move Jackson Core to the module path as well, making it an automatic module. Let’s start with moving the JAR file to the mods directory, and removing it from the classpath, as we did for Jackson Databind:

> 尽管在未命名模块（类路径）中存在类型，并且 jackson.databind 自动模块可以访问它，但我们却无法从自己的模块中访问它。为了解决这个问题，需要将 Jackson Core 移动到模块路径中，使其成为一个自动模块。接下来将 JAR 文件移动到 mods 目录，并从类路径中移除它，就像为 Jackson Databind 所做的那样：

```
javac -cp lib/jackson-annotations-2.8.8.jar \
  --module-path mods \
  -d out \
  --module-source-path src \
  -m books
```

This works! Taking a step back, however, why does it work? We’re clearly using a type from jackson.core, but we don’t have a requires for jackson.core in our module-info.java. Why didn’t the compilation fail? Remember that an automatic module requires transitive all other modules. This means that by requiring jackson.databind, we also read jackson.core transitively. Although this is convenient, it’s a big trade-off. We have an explicit code dependency on a module that we don’t explicitly require. If jackson.databind moves to become an explicit module, and the Jackson maintainers choose to not requires transitive jackson.core, our code would suddenly break.

> 此时一切正常！后退一步想一下，为什么没有出现错误呢？显而易见，代码使用了 jackson.core 中的一个类型，但是 module-info.java 中并没有对 jackson.core 的 requires。编译为什么没有失败呢？请记住，自动模块对所有其他模块都会进行 requires transitive。这意味着通过请求 jackson.databind，也可以以传递的方式读取 jackson.core。虽然这样做很方便，但却需要一定的权衡。我们对一个没有明确需要的模块存在显式代码依赖。如果 jackson.databind 移动成为一个显式模块，并且 Jackson 维护者没有选择 requires transitive jackson.core，那么代码将突然中断。

Be aware that although automatic modules look like modules, they miss the metadata to provide reliable configuration. In this specific example, it is better to explicitly add a requires to jackson.core as well:

> 请注意，虽然自动模块看起来像模块，但它们却缺少可以提供可靠配置的元数据。在这个特定的例子中，最好显式地向 jackson.core 添加一个 requires：

```java
module books {
  requires jackson.databind;
  requires jackson.core;

  opens demo;
}
```

Now that we’re compiling happily again, we can also adapt the run command. We have to remove only the Jackson Core JAR file from the classpath, because the module path was already configured correctly:

> 现在可以再次进行编译，也可以调整运行命令。必须从类路径中只删除 Jackson Core JAR 文件，因为已经正确配置了模块路径：

```
java \
  -cp lib/jackson-annotations-2.8.8.jar \
  --module-path out:mods \
  -m books/demo.Main
```

If you are wondering why jackson.core was resolved in the first place (it’s not explicitly added to the module graph as a root module, and no explicit module depends on it directly), you have been paying attention! “Module Resolution and the Module Path” discussed in detail that the set of resolved modules is calculated based on a given set of root modules. In the case of automatic modules this would be confusing. Automatic modules don’t have explicit dependencies, so they would not cause their transitive dependencies that are also automatic modules to be resolved. Because it would be time-consuming to add dependencies manually with --add-modules, all automatic modules on the module path are automatically resolved when the application requires any one of them. This behavior can cause unused automatic modules to be resolved (taking unnecessary resources), so keep the module path clean.

> 如果想要知道为什么 jackson.core 最先被解析（它并没有作为根模块显式添加到模块图中，也没有显式模块直接依赖它），那么可以回顾一下“模块解析和模块路径”一节所讨论的内容，即解析模块组是根据一组给定的根模块计算出来的。在自动模块的情况下，这将会产生混乱。自动模块没有显式依赖项，所以不会导致其传递依赖项（也是自动模块）也被解析。由于使用--add-modules 手动添加依赖项是非常耗时的，因此当应用程序 requires 模块路径上任何一个自动模块时，所有自动模块都会被自动解析。这样一来，就可能导致未使用的自动模块也被解析（占用不必要的资源），所以请保持模块路径尽可能“干净”。

The behavior is similar to what we’re used to with the classpath, however. Again, this illustrates how automatic modules are a feature squarely aimed at migrating from the classpath to modules.

> 但这种行为与使用类路径时所产生的行为相类似，从而再次说明了自动模块的主要功能是实现从类路径到模块的迁移。

Why isn’t the JVM smarter about this? It has access to all the code in an automatic module, after all, so why not analyze dependencies? To analyze whether any of this code calls into other modules, the JVM needs to perform bytecode analysis of all the code. Although this isn’t difficult to implement, it is an expensive operation potentially adding a lot of startup time in a large application. Moreover, such an analysis won’t find dependencies arising through reflection. Because of these limitations, the JVM doesn’t and probably never will do this. Instead, the JDK ships with another tool, jdeps, that does perform such bytecode analysis.

> JVM 为什么没有那么“聪明”呢？它有权访问自动模块中的所有代码，那么为什么不分析依赖关系呢？如果想要分析这些代码是否调用其他模块，JVM 需要对所有代码执行字节码分析。虽然这不难实现，但却是一个昂贵的操作，可能会大量增加大型应用程序的启动时间。而且，这样的分析不会发现通过反射产生的依赖关系。由于存在这些限制，JVM 不可能也永远不会这样做。相反，JDK 附带了另一个工具 jdeps，它可以执行字节码分析。

## 8.9 Using jdeps 使用 jdeps

In the preceding Jackson example, we’ve used kind of a trial-and-error approach to migrating the code. This gave you a good understanding of what happens, but is not efficient. jdeps is a tool shipped with the JDK that analyzes code and gives insights about module dependencies. We can use jdeps to optimize the process we’ve gone through to migrate the Jackson example.

> 在前面的 Jackson 示例中，使用了一种反复试验的方法来迁移代码。虽然这种方法很好地解释了所发生的事情，但效率不高。jdeps 是 JDK 附带的一个工具，用于分析代码并提供关于模块依赖关系的了解。可以使用 jdeps 来优化迁移 Jackson 示例的过程。

We will start by using jdeps to analyze the classpath version of the demo, before we migrated to (automatic) modules. jdeps analyzes bytecode, not source files, so we’re interested only in the application’s output folder and JAR files. For reference, here’s the compiled version of the books example when using just the classpath, as in the beginning of this chapter:

> 在迁移到（自动）模块之前，首先使用 jdeps 来分析示例的类路径版本。jdeps 分析的是字节码，而不是源文件，所以只对应用程序的输出文件夹和 JAR 文件感兴趣。为了便于参考，如本章开头所示，在使用类路径时使用了 books 示例的编译版本，如下所示：

```
├── lib
│   ├── jackson-annotations-2.8.8.jar
│   ├── jackson-core-2.8.8.jar
│   └── jackson-databind-2.8.8.jar
└── out
    └── demo
        ├── Book.class
        └── Main.class
```

We can analyze this application by using the following command:

> 通过使用下面的命令，可以分析应用程序：

```
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
```

The -recursive flag makes sure that transitive run-time dependencies are also analyzed. Without it, jackson-annotations-2.8.8.jar would not be analyzed, for example. The -summary flag, as the name suggests, summarizes the output. By default, jdeps outputs the complete list of dependencies for each package, which can be a long list. The summary shows module dependencies only, and hides the package details. The -cp argument is the classpath we want to use during the analysis, which should correspond to the run-time classpath. The out directory contains the application’s class files that must be analyzed.

> -recursive 标志确保可传递的运行时依赖项也被分析。如果没有它，jackson-annotations-2.8.8.jar 将不会被分析。顾名思义，-summary 标志总结了输出。在默认情况下，jdeps 输出每个包的依赖项的完整列表，这可能是一个非常长的列表。摘要仅显示模块相关性，并隐藏了包的详细信息。-cp 参数是希望在分析过程中使用的类路径，它应该与运行时类路径相对应。out 目录包含必须分析的应用程序的类文件。

We learn several things from the jdeps output:

> 从 jdeps 输出中可以学到以下内容：

- Our own code (in directory out) has only a direct, compile-time dependency on jackson-databind-2.8.8.jar (and java.base, of course).
- Jackson Databind has a dependency on Jackson Core and Jackson Annotations.
- Jackson Databind has dependencies on several platform modules.

---

> - 代码对 jackson-databind-2.8.8.jar（当然也包括 java.base）只有一个直接、编译时依赖。
> - Jackson Databind 依赖 Jackson Core 和 Jackson Annotation。
> - Jackson Databind 依赖多个平台模块。

Based on this output, we can already conclude that to migrate our code to a module, we need to make jackson-databind an automatic module as well. We also see that jackson-databind depends on jackson-core and jackson-annotations, so they need to be provided either on the classpath or as automatic modules. If we want to know why a dependency exists, we can print more details by using jdeps. Omitting the -summary argument in the preceding command prints the full dependency graph, showing exactly which packages require which other packages:

> 根据上面的输出已经可以得出结论，为了将代码迁移到一个模块，还需要使 jackson-databind 成为一个自动模块。同时也看到，jackson-databind 依赖于 jackson-core 和 jackson-annotations，所以需要在类路径中提供它们或者以自动模块的形式提供。如果想知道为什么存在依赖关系，可以使用 jdeps 打印更多的细节。省略上述命令中的-summary 参数，可以打印完整的依赖关系图，以及准确显示哪些包需要哪些其他包：

```
$ jdeps -cp lib/*.jar out

com.fasterxml.jackson.databind.util (jackson-databind-2.8.8.jar)
      -> com.fasterxml.jackson.annotation jackson-annotations-2.8.8.jar
      -> com.fasterxml.jackson.core jackson-core-2.8.8.jar
      -> com.fasterxml.jackson.core.base jackson-core-2.8.8.jar

... Results truncated for readability
```

If this is still not enough detail, we can also instruct jdeps to print dependencies at the class level:

> 如果以上信息还不够详细，还可以指示 jdeps 以类级别打印依赖项：

```
$ jdeps -verbose:class -cp lib/*.jar out

out -> java.base
   demo.Main (out)
      -> java.lang.Object
      -> java.lang.String
   demo.Main (out)
      -> com.fasterxml.jackson.databind.ObjectMapper jackson-databind-2.8.8.jar

... Results truncated for readability
```

So far, we have used jdeps on a classpath-based application. We can also use jdeps with modules. Let’s in this case try jdeps on the Jackson demo, where all Jackson JARs are available as automatic modules:

> 到目前为止，已经在基于类路径的应用程序上使用了 jdeps。此外，其也可以用于模块。接下来在 Jackson 示例上使用 jdeps，其中所有的 Jackson JAR 都可以作为自动模块使用：

```
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
```

To invoke jdeps, we now have to pass in the module path that contains the application module and the automatic Jackson modules:

> 为了调用 jdeps，现在必须传入包含应用程序模块和自动 Jackson 模块的模块路径：

```
$ jdeps --module-path out:mods -m books
```

This prints the dependency graph as before. Looking at the (long) output, we can see the following:

> 如前面一样，该命令打印了依赖关系图，从中可以看到以下内容：

```log
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
```

We can see that jackson.databind has a dependency on both jackson.annotations and jackson.core. We also see that books has only a dependency on jackson.databind. The books code is not using jackson.core classes at compile-time, and transitive dependencies are not defined for automatic modules. Remember that the JVM won’t do this analysis at application startup, which means we have to take care of adding jackson.annotations and jackson.core to either the classpath or the module path ourselves. jdeps provides the information to set this up correctly.

> 可以看到 jackson.databind 对 jackson.annotations 和 jackson.core 都有依赖。同时，books 仅依赖于 jackson.databind。books 代码在编译时没有使用 jackson.core 类，同时没有为自动模块定义可传递依赖关系。请记住，JVM 在应用程序启动时不会执行此分析，这意味着必须亲自将 jackson.annotations 和 jackson. core 添加到类路径或模块路径中。jdeps 提供了正确设置所需的信息。

TIP

jdeps can output dot files for module graphs using the -dotoutput flag. This is a useful format to represent graphs, and it’s easy to generate images from this format. Wikipedia has a nice introduction to the format.

> jdeps 可以使用-dotoutput 标志输出模块图的点文件。该文件是表示图形的有用格式，可以很容易地通过该格式生成图像。关于该格式，Wikipedia 给出了非常详细的介绍（http://en.wikipedia.org/wiki/DOT_(graph_description_language)）。

## 8.10 Loading Code Dynamically 动态加载代码

A situation that might require some special care when migrating to modules is the use of reflection to load code. A well-known example is loading JDBC drivers. You will see that in most cases loading a JDBC driver “just works,” but some corner cases give us better insights into the module system. Let’s start with an example that loads a JDBC driver that sits in the mods directory of the project as shown in Example 8-4. The HSQLDB driver JAR is not a module yet, so we can use it only as an automatic module.

> 在迁移到模块时可能需要特别小心的情况是使用反射来加载代码。一个众所周知的例子是加载 JDBC 驱动程序。在大多数情况下看到的只是加载 JDBC 驱动程序能“正常工作”，但了解一些极端情况有助于更好地了解模块系统。首先从一个示例开始，加载位于项目 mods 目录下的 JDBC 驱动程序，如示例 8-4 所示。HSQLDB 驱动程序 JAR 还不是一个模块，所以只能把它作为一个自动模块来使用。

Because the name of the class is just a string, the compiler will not know about the dependency, so compilation succeeds.

> 因为类的名称只是一个字符串，所以编译器不会知道该依赖项，因此编译成功。

Example 8-4. Main.java (➥ chapter8/runtime_loading)

> 示例 8-4:Main.java（chapter8/runtime_loading）

```java
package demo;

public class Main {

  public static void main(String... args) throws Exception {
    Class<?> clazz = Class.forName("org.hsqldb.jdbcDriver");
    System.out.println(clazz.getName());
  }
}
```

The module descriptor (shown in Example 8-5) is empty; it does not require hsqldb, which is the driver we’re trying to load. Although this would normally be a reason to be suspicious, it could still work in theory, because the runtime will automatically create a readability relation when using reflection on code in another module.

> 此时模块描述符（如示例 8-5 所示）为空；它没有请求 hsqldb（这是正在尝试加载的驱动程序）。虽然这是一个值得可疑的理由，但在理论上仍然成立，因为当在其他模块中对代码使用反射时，运行时会自动创建可读性关系。

Example 8-5. module-info.java (➥ chapter8/runtime_loading)

> 示例 8-5:module-info.java（chapter8/runtime_loading）

```java
module runtime.loading.example {
}
```

Still, if we run the code, it fails with a ClassNotFoundException:

> 如果运行代码，仍然会失败，并生成 ClassNotFoundException：

```
java --module-path mods:out -m runtime.loading.example/demo.Main

Exception in thread "main" java.lang.ClassNotFoundException:
  org.hsqldb.jdbcDriver
```

All observable automatic modules are resolved when the application uses at least one of them. Resolving happens at startup, so effectively our module is not causing the automatic module to load. If we had other automatic modules that are required directly, this would cause the hsqldb module to resolve as well as a side effect. In this case, we can add the automatic module ourselves with --add-modules hsqldb.

> 当应用程序至少使用其中一个时，所有可观察的自动模块都将被解析。解析发生在启动时，所以所创建的模块不会导致自动模块加载。如果有其他直接需要的自动模块，则会解析 hsqldb 模块并产生一些副作用。在这种情况下，可以使用--add-modules hsqldb 自己添加自动模块。

Now the driver loads but gives another error, because the driver depends on java.sql, which wasn’t resolved yet. Remember, automatic modules lack the metadata to specifically require other modules. When using JDBC in practice, we would need to require java.sql in our module to be able to use the JDBC API after loading the driver. This means adding it to the module descriptor as shown in Example 8-6.

> 现在驱动程序可以加载但又出现了另一个错误，因为驱动程序依赖于尚未解析的 java. sql。请记住，自动模块缺少元数据来具体要求其他模块。在实践中使用 JDBC 时，需要在模块中请求 java.sql，以便在加载驱动程序后能够使用 JDBC API。这意味着将其添加到模块描述符中，如示例 8-6 所示。

Example 8-6. module-info.java (➥ chapter8/runtime_loading)

> 示例 8-6:module-info.java（chapter8/runtime_loading）

```java
module runtime.loading.example {
   requires java.sql;
}
```

The code now runs successfully. With java.sql required by our module, we can see another interesting case of automatic module resolving. If we remove --add-modules hsqldb again, the application still runs! Why does requiring java.sql cause an automatic module to be loaded? It turns out that java.sql defines a java.sql.Driver service interface, and has a uses constraint for this service type as well. Our hsqldb JAR provides a service, which is registered via the “old” way of using a file in META-INF/services. Because of service binding, the JAR is automatically resolved from the module path. This goes into the subtleties of the module system but is good to understand.

> 代码现在可以成功运行。有了模块所需的 java.sql，就可以看到另一个有趣的自动模块解析案例。如果再次删除--add-modules hsqldb，应用程序仍然可以运行！那么请求 java.sql 为什么会导致自动模块被加载呢？事实证明，java.sql 定义了一个 java.sql.Driver 服务接口，并且对这个服务类型也有一个 uses 约束。hsqldb JAR 提供了一个服务，它是通过在 META-INF/services 中使用文件的“旧”方式进行注册的。由于服务绑定，JAR 将从模块路径中自动解析。虽然这涉及了模块系统的细微之处，但却很好理解。

Why didn’t we just put requires hsqldb in our module descriptor? Although typically we like to be as explicit as possible about dependencies by putting them in the module descriptor, this is a good example of where this rule of thumb does not apply. The JDBC driver to use often depends on the deployment environment of the application, where the exact driver name is configured in a configuration file. Application code should not be coupled to a specific database driver in that case (although in our example, it is). Instead we simply make sure that the driver is resolved by adding --add-modules. The module containing the driver will be in the resolved module graph, and the reflective instantiation establishes the readability relation to this module.

> 为什么不将 requires hsqldb 放到模块描述符中呢？虽然一般来说都希望将 requires 子句放置到模块描述符中，从而尽可能明确地表示依赖关系，但此时并不适用这种经验法则。要使用的 JDBC 驱动程序通常取决于应用程序的部署环境，其中确切的驱动程序名称在配置文件中配置。在这种情况下，应用程序代码不应该耦合到特定的数据库驱动程序（尽管在本示例中出现了耦合现象）。相反，只需确保通过添加--add-modules 来解析驱动程序。包含驱动程序的模块将位于已解析的模块图中，反射实例化建立了与此模块的可读性关系。

If the JDBC driver supports it (as HSQLDB does), it is even better to avoid reflective instantiation from application code altogether and use services instead. Services are discussed in detail in Chapter 4.

> 如果 JDBC 驱动程序支持它（就像 HSQLDB 那样），那么最好避免应用程序代码的反射实例化，而使用服务。服务在第 4 章中已详细讨论。

## 8.11 Split Packages 拆分包

“Split Packages” explained the problem of split packages. Just as a refresher, a split package means two modules contain the same package. The Java module system doesn’t allow split packages.

> 5.5.1 节已经介绍了拆分包所涉及的相关问题。现在复习一下，拆分包意味着两个模块包含相同的包。Java 模块系统不允许拆分包。

When using automatic modules, we can run into split packages as well. In large applications, it is common to find split packages due to dependency mismanagement. Split packages are always a mistake, because they don’t work reliably on the classpath either. Unfortunately, when using build tools that resolve transitive dependencies, it’s easy to end up with multiple versions of the same library. The first class found on the classpath is loaded. When classes from two versions of a library mix, this often leads to hard-to-debug exceptions at run-time.

> 当使用自动模块时，也会遇到拆分包。在大型应用程序中，由于依赖关系管理不善，通常会发现拆分包。拆分包始终是一个错误，因为它们在类路径上不能可靠地工作。不幸的是，当使用解析可传递依赖项的构建工具时，很容易得到同一个库的多个版本。在类路径中找到的第一个类将被加载。当混合使用来自两个版本库的类时，往往会导致在运行时出现难以调试的异常。

TIP

Modern build tools often have a setting to fail on duplicate dependencies. This makes dependency management issues clearer and forces you to deal with them early. It is highly recommended to use this.

> 现代构建工具通常有一个“在重复依赖项上失败”的设置，这使得依赖关系管理问题变得更加清晰，从而迫使尽早解决这些问题。强烈建议使用这个设置。

The Java module system is much stricter about this issue than the classpath. When it detects that a package is exported from two modules on the module path, it will refuse to start. This fail-fast mechanism is much better than the unreliable situation we used to have with the classpath. Better to fail during development than in production, when some unlucky user hits a code path that is broken by an obscure classpath problem. But it also means we have to deal with these issues. Blindly moving all JARs from the classpath to the module path may result in split packages between the resulting automatic modules. These will then be rejected by the module system.

> 关于该问题，Java 模块系统比类路径要严格得多。当它检测到一个包从模块路径上的两个模块导出时，就会拒绝启动。相比于以前使用类路径时所遇到的不可靠情况，这种快速失败（fail-fast）机制要好得多。在开发过程中失败好过在生产过程中失败，尤其是当一些不幸的用户碰到一个由于模糊的类路径问题而被破坏的代码路径时。但这也意味着我们必须处理这些问题。盲目地将所有 JAR 从类路径移至模块路径可能导致在生成的自动模块之间出现拆分包。而这些拆分包将被模块系统所拒绝。

To make migration a little easier, an exception to this rule exists when it comes to automatic modules and the unnamed module. It acknowledges that a lot of classpaths are simply incorrect and contain split packages. When both a (automatic) module and the unnamed module contain the same package, the package from the module will be used. The package in the unnamed module will be ignored. This is also the case for packages that are part of the platform modules. It’s common to override platform packages by putting them on the classpath. This approach is no longer working in Java 9. You already saw this in the previous chapter, and learned that java.se.ee modules are not included in the java.se module for this reason.

> 为了使迁移变得容易一些，当涉及自动模块和未命名模块时，上述规则存在一个例外，即承认很多类路径是不正确的，并且包含拆分包。当（自动）模块和未命名模块都包含相同的包时，将使用来自（自动）模块的包，而未命名模块中的包被忽略。对于作为平台模块一部分的包来说也是如此。通过在类路径上放置相关包来覆盖平台包是很常见的，但这种方法已经不再适用于 Java 9 了，从前面的章节中可以看到这一点。基于此原因，java.se.ee 模块不再包含在 java.se 模块中。

If you run into split package issues while migrating to Java 9, there’s is no way around them. You must deal with them, even when your classpath-based application works correctly from a user’s perspective.

> 如果在迁移到 Java 9 时遇到了拆分包问题，那么是无法绕过的。即使从用户角度来看基于类路径的应用程序可以正确工作，你也必须处理这些问题。

This chapter presented many techniques to make gradual migration to the Java module system possible. These techniques are extremely valuable, because it will take time before the Java ecosystem moves to the Java module system in its entirety. Automatic modules play an important role in migration scenarios; therefore, it’s important to fully understand how they work.

> 本章介绍了许多逐步迁移到 Java 模块系统所需的技术。这些技术都非常有价值，因为在 Java 生态系统完全转移到 Java 模块系统之前需要花费一定的时间。自动模块在迁移场景中扮演着重要的角色，因此，充分了解它们的工作方式非常重要。
