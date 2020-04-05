# 第 9 章 迁移案例研究：Spring 和 Hibernate

> Chapter 9. Migration Case Study: Spring and Hibernate

Chapter 8 presented all the available tools for migrating applications to modules. This chapter brings everything together in a case study. We’ll migrate a fully functional application that uses Spring and Hibernate to modules. Note that we’re deliberately using an example of “traditional” Spring/Hibernate development, instead of using the most modern way of doing things. We’re using pre-Java 9 versions, to create an interesting case study. Many applications have been written this way, which makes it extra interesting to see how these applications can be migrated toward the future. Newer versions of frameworks will support Java 9 better out of the box, and migrations based on these versions will probably be even easier. If you are not familiar with these frameworks, don’t worry. You don’t have to understand all the code and configuration to learn about common problems you may face during a migration toward modules.

> 第8章介绍了将应用程序迁移到模块时可用的所有工具，本章结合前面所学的内容进行一个案例研究，将一个使用Spring和Hibernate的全功能应用程序迁移到模块。请注意，此时故意使用“传统的”Spring / Hibernate开发的例子，而没有使用最现代的方式，并且使用Java 9之前的版本来创建一个有趣的案例研究。许多应用程序都是用这种方式编写的，这使得迁移这些应用程序变得更加有趣。这些框架的较新版本可以更好地支持Java 9，基于这些版本的迁移也会变得更加容易。如果对这些框架不熟悉，也不要担心，因为没有必要为了了解在模块迁移时可能遇到的常见问题而熟悉所有代码和配置。

You will get a lot more value out of this chapter if you check out the code repository and try to migrate the code while you’re reading the chapter. In the code repository, we have provided three versions:

> 如果在阅读本章时仔细检查代码库并尝试迁移代码，那么就可以从中获得更多有价值的信息。在代码库中共提供了三个版本：

- chapter9/spring-hibernate-starter: The classpath version of the application before migration.
- chapter9/spring-hibernate: The migrated application.
- chapter9/spring-hibernate-refactored: The migrated application after additional modularization.

---


> - Chapter9/spring-hibernate-starter：迁移之前应用程序的类路径版本。
> - Chapter9/spring-hibernate：迁移后的应用程序。
> - Chapter9/spring-hibernate-refactored：在进行了额外的模块化之后的迁移后的应用程序。

We recommend opening the spring-hibernate-starter project in an editor, and applying each step described in this chapter on the code. You should end up with roughly the same result as the finished spring-hibernate example.

> 建议在编辑器中打开spring-hibernate-starter项目，并应用本章所介绍的每一步来处理代码。这样一来，可以得到与完成后的spring-hibernate示例大致相同的结果。

## 9.1 Getting Familiar with the Application 熟悉应用程序
The application represents a bookstore. Books are stored in a database by using Hibernate. Spring is used to bootstrap Hibernate, including transaction management and dependency injection. The Spring configuration uses a mix of XML and annotation-based configuration.

> 该应用程序表示了一个书店。书籍通过使用Hibernate存储在数据库中。Spring用来启动Hibernate，包括事务管理和依赖注入。Spring配置混合使用了XML和基于注释的配置。

Before migration, the application code, direct dependencies, and transitive dependencies are on the classpath, as shown in Figure 9-1.

> 在迁移之前，应用程序代码、直接依赖项和可传递依赖项都在类路径上，如图9-1所示。

Migration starting point (➥ chapter9/spring-hibernate-starter)
Figure 9-1. Migration starting point (➥ chapter9/spring-hibernate-starter)

The end result of the migration is a codebase with a single module, using automatic modules for dependencies where necessary. Figure 9-2 shows the end result.

> 迁移的最终结果是一个带有单个模块的代码库，并在必要时针对依赖项使用自动模块。图9-2显示了最终的结果。


Application migrated (➥ chapter9/spring-hibernate)
Figure 9-2. Application migrated (➥ chapter9/spring-hibernate)

At the end of the chapter, we will also look at refactoring the application code itself to be more modular.

> 在本章的最后，还将着重于对应用程序代码本身进行重构，以便更加模块化。


Before we even start thinking about splitting our code in modules, we should get over some technical issues coming from our dependencies. Remember that we’re dealing with pre-Java 9 libraries, which were not designed to work with the Java module system. Specifically, we are using the following framework versions:

> 在开始考虑在模块中拆分代码之前，应该解决依赖关系中所存在的一些技术问题。请记住，正在处理的是Java 9之前的库，这些库不是为了与Java模块系统一起工作而设计的。具体来说，正在使用以下框架版本：

- Spring 4.3.2
- Hibernate 5.0.1

Also note that great progress has been made within these frameworks when it comes to module support. At the time of writing, the first Release Candidate of Spring 5 was released, with explicit support for usage as automatic modules. We do not use these updated versions because that wouldn’t make a realistic migration example. Even without special support from frameworks and libraries, migration to modules is possible.

> 还要注意，在模块支持方面，这些框架已经取得了很大的进展。在编写本书的时候，Spring 5的第一个RC版本已经发布了，并且明确支持用作自动模块。但本章示例中并不使用这些更新后的版本，因为这样无法做出一个现实的迁移示例。即使没有框架和库的特殊支持，也可以迁移到模块。

The focus of this section is to create a single module for our code. This means defining requires for dependencies, and moving libraries to automatic modules. We will also have to deal with exports and opens to make our code accessible to the frameworks. Once this migration is complete, we can take a good look at the design of our code, and potentially split the code into smaller modules. Because we already solved all the technical issues, this becomes a design exercise.

> 本节的重点是为代码创建一个模块。这意味着需要为依赖关系定义requires，并将库移动到自动模块。此外，还必须处理exports和opens，以便代码可以被框架访问。一旦完成迁移，就可以仔细看看代码的设计，并将代码拆分成更小的模块。因为前面已经介绍过所有的技术问题，所以本示例成为一个很好的设计练习。

Let’s start by looking at the most important parts of the code to get an idea of the application. For optimal readability of the code, we strongly advise opening the code in your favorite editor.

> 首先看看代码中最重要的部分，以便了解应用程序。为了获得最佳的代码可读性，强烈建议在你最喜欢的编辑器中打开代码。

The Book class (shown in Example 9-1) is a JPA entity that can be stored in a database by using Hibernate (or another JPA implementation). It has annotations such as @Entity and @Id to configure the mapping to the database.

> Book类（如示例9-1所示）是一个JPA实体，可以使用Hibernate（或另一个JPA实现）将其存储在数据库中。它有诸如@Entity和@Id之类注释，以便将映射配置到数据库。

Example 9-1. Book.java (➥ chapter9/spring-hibernate)

> 示例9-1:Book.java（chapter9/spring-hibernate）
```java
package books.impl.entities;

import books.api.entities.Book;
import javax.persistence.*;

@Entity
public class BookEntity implements Book {
  @Id @GeneratedValue
  private int id;
  private String title;
  private double price;


  //Getters and setters omitted for brevity

}
```
The HibernateBooksService is a Spring Repository. This is a service that automatically takes care of transaction management, which is required to successfully store something in the database. It implements our service interface BooksService, and uses the Hibernate API (such as SessionFactory) to store and retrieve books from the database.

> HibernateBooksService是一个Spring Repository。这是一个自动处理事务管理的服务，用来将某些内容成功地存储到数据库中。它实现了服务接口BooksService，并使用HibernateAPI（如SessionFactory）存储和检索数据库中的书籍。

The BookstoreService is a simple interface, whose implementation in BookStoreServiceImpl, shown in Example 9-2, can calculate the total price of a given list of books. It is annotated with Spring’s @Component annotation so that it becomes available for dependency injection.

> BookstoreService是一个简单的接口，它在BookstoreServiceImpl中的实现（如示例9-2所示）可以计算给定书籍列表的总价格，并且使用了Spring的@Component进行注释，以便用于依赖注入。

Example 9-2. BookstoreServiceImpl.java (➥ chapter9/spring-hibernate)

> 示例9-2:BookstoreServiceImpl.java（chapter9/spring-hibernate）

```java
package bookstore.impl.service;

import java.util.Arrays;
import books.api.entities.Book;
import books.api.service.BooksService;
import bookstore.api.service.BookstoreService;
import org.springframework.stereotype.Component;

@Component
public class BookstoreServiceImpl implements BookstoreService {

  private static double TAX = 1.21d;

  private BooksService booksService;

  public BookstoreServiceImpl(BooksService booksService) {
    this.booksService = booksService;
  }

  public double calculatePrice(int... bookIds) {
    double total = Arrays
      .stream(bookIds)
      .mapToDouble(id -> booksService.getBook(id).getPrice())
      .sum();

    return total * TAX;
  }

}
```
Finally, we have a main class that bootstraps Spring, and stores and retrieves some books, as shown in Example 9-3.

> 最后，有一个用来启动Spring并存储和检索书籍的主类，如示例9-3所示。

Example 9-3. Main.java (➥ chapter9/spring-hibernate)

> 示例9-3:Main.java（chapter9/spring-hibernate）
```java
package main;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import books.api.service.BooksService;
import books.api.entities.Book;
import bookstore.api.service.BookstoreService;

public class Main {

  public void start() {
    System.out.println("Starting...");

    ApplicationContext context =
        new ClassPathXmlApplicationContext(new String[] {"classpath:/main.xml"});

    BooksService booksService = context.getBean(BooksService.class);
    BookstoreService store = context.getBean(BookstoreService.class);

      // Create some books
      int id1 = booksService.createBook("Java 9 Modularity", 45.0d);
      int id2 = booksService.createBook("Modular Cloud Apps with OSGi", 40.0d);
      printf("Created books with id [%d, %d]", id1, id2);

      // Retrieve them again
      Book book1 = booksService.getBook(id1);
      Book book2 = booksService.getBook(id2);
      printf("Retrieved books:\n  %d: %s [%.2f]\n  %d: %s [%.2f]",
        id1, book1.getTitle(), book1.getPrice(),
        id2, book2.getTitle(), book2.getPrice());

      // Use the other service to calculate a total
      double total = store.calculatePrice(id1, id2);
      printf("Total price (with tax): %.2f", total);

  }

  public static void main(String[] args) {
    new Main().start();
  }

  private void printf(String msg, Object... args) {
      System.out.println(String.format(msg + "\n", args));
  }
}
```
Spring is bootstrapped by using ClassPathXmlApplicationContext, which requires an XML configuration. In this configuration, shown in Example 9-4, we set up component scanning, which automatically registers @Component and @Repository annotated classes as Spring beans. We also set up transaction management and Hibernate.

> Spring通过使用需要XML配置的ClassPathXmlApplicationContext进行启动。在如示例9-4所示的配置中，设置了组件扫描，自动将@Component和@Repository注释类注册为Springbean，同时还设置了事务管理和Hibernate。

Example 9-4. main.xml (➥ chapter9/spring-hibernate)

> 示例9-4:main.xml（chapter9/spring-hibernate）
```xml
<context:component-scan base-package="books.impl.service"/>
<context:component-scan base-package="bookstore.impl.service"/>

<bean id="myDataSource"
class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="org.hsqldb.jdbcDriver"/>
    <property name="url" value="jdbc:hsqldb:mem:testdb"/>
    <property name="username" value="sa"/>
    <property name="password" value=""/>
</bean>

<bean id="mySessionFactory"
  class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="myDataSource"/>
    <property name="annotatedClasses">
  <list>
    <value>books.impl.entities.BookEntity</value>
  </list>
</property>

<property name="hibernateProperties">
  <props>
    <prop key="hibernate.hbm2ddl.auto">create</prop>
  </props>
</property>
</bean>

 <bean id="transactionManager"
    class="org.springframework.orm.hibernate5.HibernateTransactionManager">
    <property name="sessionFactory" ref="mySessionFactory"/>
</bean>

<tx:annotation-driven/>
```
The directory structure of the project is currently as follows:

> 项目的目录结构如下所示：

```
├── lib
├── run.sh
└── src
    ├── books
    │   ├── api
    │   │   ├── entities
    │   │   │   └── Book.java
    │   │   └── service
    │   │       └── BooksService.java
    │   └── impl
    │       ├── entities
    │       │   └── BookEntity.java
    │       └── service
    │           └── HibernateBooksService.java
    ├── bookstore
    │   ├── api
    │   │   └── service
    │   │       └── BookstoreService.java
    │   └── impl
    │       └── service
    │           └── BookstoreServiceImpl.java
    ├── log4j2.xml
    ├── main
    │   └── Main.java
    └── main.xml
```
The src directory contains the configuration files and the source code packages. The lib directory contains the JAR files of Spring, Hibernate, and transitive dependencies of both, which is a long list of 31 JAR files in total. To build and run the application, we can use the following commands:

> src目录包含配置文件和源代码包。lib目录包含Spring、Hibernate以及两者的可传递依赖项的JAR文件，这是一个包含31个JAR文件的长列表。为了构建和运行应用程序，可以使用以下命令：
```
javac -cp [list of JARs in lib] -d out -sourcepath src $(find src -name '*.java')

cp $(find src -name '*.xml') out

java -cp [list of JARs in lib]:out main.Main
```
## 9.2 Running on the Classpath with Java 9 使用Java 9在类路径上运行
The first step toward any migration to modules should start with compiling and running the code with Java 9, while still using the classpath. This also shows the first problem to solve. Hibernate relies on some JAXB classes. In “Using JAXB and Other Java EE APIs”, you learned that JAXB is part of the java.se.ee subgraph, but not of the default java.se module subgraph. Without modification, running Main results in java.lang.ClassNotFoundException: javax.xml.bind.​JAXBException. We need to add JAXB to our application by using the --add-modules flag:

> 向模块迁移的第一步应该从使用Java 9编译和运行代码开始，同时仍然使用类路径。这也显示了要解决的第一个问题。Hibernate依赖于一些JAXB类。在7.5节中已经讲过，JAXB是java.se.ee子图的一部分，但不是默认java.se模块子图的一部分。在不进行任何修改的情况下，运行Main会导致java.lang.ClassNotFoundException:javax. xml.bind。此时，需要使用--add-modules标志将JAXB添加到应用程序中：
```
java -cp [list of JARs in lib]:out --add-modules java.xml.bind main.Main
```
We’re now seeing another, rather obscure, warning:

> 接下来看到了另一个模糊的警告：
```log
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by javassist.util.proxy.SecurityActions
(file:.../lib/javassist-3.20.0-GA.jar)
to method java.lang.ClassLoader.defineClass(...)
WARNING: Please consider reporting this to the maintainers
  of javassist.util.proxy.SecurityActions
WARNING: Use --illegal-access=warn to enable warnings of further illegal
  reflective access operations
WARNING: All illegal access operations will be denied in a future release
```
We discussed exactly this issue in “Libraries, Strong Encapsulation, and the JDK 9 Classpath”. The javassist library tries to use deep reflection on JDK types, which is by default allowed, but generates a warning. If we were to run the application with --illegal-access=deny, this would even turn into an error. Remember, in a future Java release, this will be the default. We don’t want to fix this issue right now by updating to a possibly fixed version of javassist. Still, we can get rid of the warning by using --add-opens:

> 在7.2节中已经讨论过该问题。javassist库尝试对JDK类型使用深度反射，默认情况下这是允许的，但会生成警告。如果用--illegal-access = deny来运行应用程序，甚至会变成错误。请记住，在未来的Java版本中，这将是默认设置。此时并不需要通过更新到javassist可能的固定版本来解决这个问题。不过，可以通过使用--add-opens来消除警告：
```
java -cp [list of JARs in lib]:out \
--add-modules java.xml.bind \
--add-opens java.base/java.lang=ALL-UNNAMED main.Main
```
It’s up to you whether you want to silence the warning by adding --add-opens. Doing so prepares you for the next Java releases, where illegal access from the classpath isn’t treated as friendly as in Java 9. The problematic behavior of javassist is still there. Raising such issues with library maintainers is the right course of action.

> 是否使用--add-opens来消除警告完全取决于你的决定。这样做可以为下一个Java版本做好准备，其对从类路径的非法访问的处理不会再像Java 9那样友好了。但javassist的问题行为仍然存在，所以向库护人员提出这些问题是正确的行为。

## 9.3 Setting Up for Modules 设置模块
With these problems out of the way, we can start migrating toward modules. First, we migrate the code to a single module. Keeping the code’s internal structure unchanged first is a good strategy when migrating to modules. Although this structure might not be the final structure you want, it can be easier to first focus on the technical problems.

> 解决了这些问题之后，就可以开始向模块迁移了。首先，当迁移到模块时，先保持代码内部结构不变是一个很好的策略。尽管此时的结构可能并不是你想要的最终结构，但把重点放在技术问题上可能更容易一些。

The first step is to change -sourcepath to --module-source-path. To do this, we need to slightly change the structure of the project. The src directory should not contain packages directly, but a module directory first. The module directory should also contain module-info.java:

> 第一步是将-sourcepath更改为--module-source-path。为此，需要稍微改变项目的结构。src目录不应该直接包含包，而应该先包含一个模块目录。模块目录也应该包含module-info.java：
```
├── lib
├── mods
├── run.sh
└── src
    └── bookapp
        ├── books
        │   ├── api
        │   │   ├── entities
        │   │   │   └── Book.java
        │   │   └── service
        │   │       └── BooksService.java
        │   └── impl
        │       ├── entities
        │       │   └── BookEntity.java
        │       └── service
        │           └── HibernateBooksService.java
        ├── bookstore
        │   ├── api
        │   │   └── service
        │   │       └── BookstoreService.java
        │   └── impl
        │       └── service
        │           └── BookstoreServiceImpl.java
        ├── log4j2.xml
        ├── main
        │   └── Main.java
        ├── main.xml
        └── module-info.java
```
We modify the compile/run script to use --module-source-path and to start the main class from our module:

> 修改编译/运行脚本，从而使用--module-source-path并从模块启动主类：
```
javac -cp [list of JARs in lib] \
  --module-path mods \
  -d out \
  --module-source-path src \
  -m bookapp

cp $(find src -name '*.xml') out/bookapp

java -cp out:[list of JARs in lib] \
  --module-path mods:out \
  --add-modules java.xml.bind \
  -m bookapp/main.Main
```
We didn’t move any libraries to the module path yet, nor did we put anything in module-info.java, so the preceding commands are obviously going to fail.

> 此时没有将任何库移动到模块路径，也没有将任何东西放在module-info.java中，所以前面的命令显然会失败。

## 9.4 Using Automatic Modules 使用自动模块
To be able to compile our module, we need to add requires statements to module-info.java for any compile-time dependencies. This also implies that we need to move some of the JAR files from the classpath to the module path to make them automatic modules. To figure out which compile-time dependencies we have exactly, we can look at import statements in our code or use jdeps. For our example application, we can come up with the following list of requires statements, based on direct compile-time dependencies:

> 为了能够顺利编译模块，还需要在module-info.java中针对任何编译时依赖项添加requires语句。这也意味着需要将一些JAR文件从类路径移动到模块路径，以使它们成为自动模块。为了弄清楚有哪些编译时依赖项，可以在代码中查看import语句，或者使用jdeps。对于示例应用程序，可以根据直接的编译时依赖项提出以下requires语句列表：
```java
requires spring.context;
requires spring.tx;

requires javax.inject;

requires hibernate.core;
requires hibernate.jpa;
```
All are pretty self-explanatory; packages from these modules are used directly from the sample code. To be able to require these libraries, we move their corresponding JAR files to the module path to make them automatic modules. Transitive dependencies can stay on the classpath, as shown in Figure 9-3.

> 所有这些requires语句的含义都是不言自明的，来自这些模块的包可直接在示例代码中使用。为了能够请求这些库，可以将它们相应的JAR文件移动到模块路径，从而使它们成为自动模块。而可传递依赖项可以留在类路径上，如图9-3所示。

TIP

Remember “Using jdeps”? jdeps is useful when you have to figure out what modules to require during migration. For example, we could have found our requires by running the following on our classpath version of the application:

> 还记得8.9节吗？当需要在迁移过程中找出所需的模块时，jdeps是非常有用的。例如，可以通过在应用程序的类路径版本上运行以下代码来找到requires：
```
jdeps -summary -cp lib/*.jar out

out -> lib/hibernate-core-5.2.2.Final.jar
out -> lib/hibernate-jpa-2.1-api-1.0.0.Final.jar
out -> java.base
out -> lib/javax.inject-1.jar
out -> lib/spring-context-4.3.2.RELEASE.jar
out -> lib/spring-tx-4.3.2.RELEASE.jar
```
Migration with automatic modules.
Figure 9-3. Migration with automatic modules

Besides these compile-time dependencies, we have both an additional compile-time and run-time requirement that we can add by using --add-modules. Types from the java.naming platform module are used in the Hibernate API, requiring it to be available at compile-time, even when we don’t use this type explicitly in our code. Without adding the module explicitly, we would see this error:

> 除了这些编译时依赖项之外，还有其他的编译时和运行时需求，可以使用--add-modules进行添加。如果要在Hibernate API中使用java.naming平台模块中的类型，就需要其在编译时可用，即使在代码中没有明确地使用该类型。如果没有显式地添加该模块，就会看到如下错误：
```log
src/bookapp/books/impl/service/HibernateBooksService.java:19:
    error: cannot access Referenceable
    return sessionFactory.getCurrentSession().get(BookEntity.class, id);
                         ^
  class file for javax.naming.Referenceable not found
1 error
```
Because Hibernate is used as an automatic module, it does not cause extra platform modules to resolve. Our code does have an indirect compile-time dependency on it, though, because Hibernate uses a type from java.naming in its API (the SessionFactory interface extends Referenceable). This means we have a compile-time dependency, without having a requires, which doesn’t work. If Hibernate were an explicit module, it should have requires transitive java.naming in its module-info.java to set up implied readability and prevent this problem.

> 由于Hibernate被用作自动模块，因此不会导致额外的平台模块解析。不过，代码对其存在间接的编译时依赖关系，因为Hibernate在其API中使用了来自java.naming的类型（SessionFactory接口扩展了Referenceable）。这意味着存在编译时依赖关系，如果没有使用requires语句是行不通的。如果Hibernate是一个显式模块，那么它应该在module-info.java中包含requires transitive java.naming，从而设置隐式可读性并防止上述问题出现。

Until then, we can work around the issue by adding --add-modules java.naming to javac. Alternatively, we could have added another requires for java.naming in our module descriptor. As discussed previously, we prefer to avoid requires statements for indirect dependencies, hence the choice for --add-modules instead.

> 在此之前，可以通过将--add-modules java.naming添加到javac来解决此问题。或者，可以在模块描述符中针对java.naming添加另一个requires。正如前面所讨论的那样，应该避免针对间接依赖关系使用requires语句，因此选择使用--add-modules。

The application compiles successfully now. Running it still results in some errors.

> 应用程序现在编译成功，但运行时仍然会导致一些错误。

## 9.5 Java Platform Dependencies and Automatic Modules Java平台依赖项和自动模块
A java.lang.NoClassDefFoundError tells us we need to add java.sql to --add-modules for the java command. Why wasn’t java.sql resolved without our manual intervention? Hibernate depends on java.sql internally. Because Hibernate is used as an automatic module, it doesn’t have a module descriptor to require other (platform) modules. This problem is similar to the preceding problem with java.naming, but manifests itself differently. The java.naming example was a compile-time error because the Hibernate API that our code uses references a type from java.naming. In this case, Hibernate itself uses java.sql internally, but its types are not part of the Hibernate API that we compile against. Therefore, the error shows only at run-time.


> java.lang.NoClassDefFoundError告诉我们，需要将java.sql添加到Java命令的--add-modules中。为什么在没有人工干预的情况下就不会解析java.sql呢？Hibernate在内部依赖于java.sql。因为Hibernate被用作自动模块，所以它没有模块描述符来请求其他（平台）模块。这个问题与前面的java.naming问题类似，只不过是以不同的方式表现出来。java.naming示例是一个编译时错误，因为代码所使用的Hibernate API引用了来自java.naming的类型。在这种情况下，Hibernate本身在内部使用了java.sql，但它的类型不是编译的Hibernate API的一部分。因此，错误仅在运行时显示。

With an extra --add-modules java.sql, we can move on to the next step.

> 在添加了额外的--add-modules java.sql之后，可以继续下一步。

## 9.6 Opening Packages for Reflection 开放用于反射的包
We’re getting close to successfully running the application, but rerunning the application still results in an error. This time the error is pretty straightforward:

> 现在离成功运行应用程序又近了一步，但重新运行应用程序仍然会导致错误。这一次的错误是非常简单的：
```log
Caused by: java.lang.IllegalAccessException:
class org.springframework.beans.BeanUtils cannot access class
  books.impl.service.HibernateBooksService (in module bookapp) because module
  bookapp does not export books.impl.service to unnamed module @5c45d770
```
Spring relies on reflection to instantiate classes. For this to work, we have to open implementation packages containing classes that Spring needs to instantiate. Hibernate, by the same token, also uses reflection to manipulate entity classes. Hibernate needs access to the Book interface and to the BookEntity implementation class.

> Spring依靠反射来实例化类。为此，必须开放包含Spring实例化所需类的实现包。同样的道理，Hibernate也使用反射来操纵实体类。Hibernate需要访问Book接口和BookEntity实现类。

For the API packages in the application, it makes sense to export them. Later, when splitting into more modules, these packages are most probably used by other modules as well. For implementation packages, we use opens instead. This way, the frameworks can do their reflection magic, while we still ensure encapsulation at build-time:

> 对于应用程序中的API包，导出它们是有意义的。稍后当拆分成更多的模块时，这些包也极有可能被其他模块所使用。对于实现包，则使用opens。这样一来，框架可以完成自己的反射“魔术”，而我们在构建时仍然可以保持良好的封装：
```java
exports books.api.service;
exports books.api.entities;

opens books.impl.entities;
opens books.impl.service;
opens bookstore.impl.service;
```
In a larger application, it could be a good choice to use an open module at first instead of specifying individual packages to be open. After setting up our package opens/exports, we see another familiar error.

> 在更大型的应用程序中，一个比较好的选择是首先使用开放式模块，而不是指定单个要开放的包。使用opens/exports设置包后，会看到另一个熟悉的错误。
```log
java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException
```
One of our libraries is using JAXB (not our code), and in “Using JAXB and Other Java EE APIs”, you learned that java.xml.bind is not resolved by default. Simply adding the module to our --add-modules, like we did when running this example on the classpath, will help us out of this situation.

其中一个库正在使用JAXB（而不是我们所编写的代码），在7.5节中讲过，默认情况下不会解析java.xml.bind。就像在类路径上运行该示例时所做的那样，将模块添加到--add-modules有助于摆脱这种情况。

Almost done!

> 就快大功告成了！

Unfortunately, the javassist library gives us a last obscure error when trying to run the application:

> 不幸的是，当试图运行应用程序时，javassist库给了一个最难以理解的错误：
```log
Caused by: java.lang.IllegalAccessError: superinterface check failed:
  class books.impl.entities.BookEntity_$$_jvstced_0 (in module bookapp)
  cannot access class javassist.util.proxy.ProxyObject
  (in unnamed module @0x546621c4) because module bookapp
  does not read unnamed module @0x546621c4
```
## 9.7 Fixing Illegal Access 解决非法访问问题
Hibernate uses the javassist library to dynamically create subclasses of our entity classes. At run-time, our application code uses these subclasses instead of the original classes. Because our code runs from a module, the generated classes end up being part of the same bookapp module. The generated classes implement an interface (ProxyObject) from javassist. However, javassist is still on the classpath, which is unreachable from explicit modules. Therefore, the generated classes implement an interface that’s not accessible for them at run-time. Although this is an obscure and hard-to-understand error, the fix is easy: move javassist from the classpath to the module path, so that it becomes an automatic module and can be accessed from other modules.

> Hibernate使用javassist库来动态创建实体类的子类。在运行时，应用程序代码使用的是这些子类而不是原来的类。因为代码是从一个模块运行的，所以生成的类最终成为同一个bookapp模块的一部分。所生成的类实现了javassist的一个接口（ProxyObject）。但是，javassist仍然在类路径中，这是显式模块无法访问的。因此，生成的类实现了一个在运行时无法访问的接口。虽然这是一个难以理解的错误，但却非常容易解决：将javassist从类路径移动到模块路径，使其成为自动模块，从而可以从其他模块访问。

However, turning javassist into an automatic module introduces a new problem. Earlier you saw that javassist uses illegal deep reflection on JDK types. On the classpath, with its lenient --illegal-access=permit default, this was only giving us a warning. Because javassist is now an automatic module, the --illegal-access mechanism doesn’t apply anymore. It affects only code on the classpath. This means we are now getting an error, which is essentially the same error that we would have seen if we had run the classpath example with --illegal-access=deny:

> 但是，将javassist转换为自动模块引入了一个新问题。前面已经看到，javassist在JDK类型上使用了非法的深度反射。在类路径中，由于默认值--illegal-access= permit较为宽容，因此只给出了一个警告。因为javassist现在是一个自动模块，所以--illegal-access机制不再适用，它只影响类路径上的代码。这意味着现在会收到一个错误，从本质上讲，该错误与使用--illegal-access=deny运行类路径示例时所看到的错误是一样的：
```log
Caused by: java.lang.reflect.InaccessibleObjectException:
  Unable to make protected final java.lang.Class
  java.lang.ClassLoader.defineClass(...)
  throws java.lang.ClassFormatError accessible:
  module java.base does not "opens java.lang" to module javassist
```
We already know that we can work around this issue by adding --add-opens java.base/java.lang=javassist to the java command. Our final script to compile and run the application is shown in Example 9-5.

> 现在已经知道可以通过向Java命令添加--add-opens java.base/java.lang=javassist来解决上述问题。示例9-5给出了用来编译和运行应用程序的最终脚本。

Example 9-5. run.sh (➥ chapter9/spring-hibernate)

> 示例9-5:run.sh（chapter9/spring-hibernate）
```
CP=[list of JARs in lib]

javac -cp $CP \
      --module-path mods \
      --add-modules java.naming \
      -d out         \
      --module-source-path src \
      -m bookapp

cp $(find src -name '*.xml') out/bookapp

java -cp $CP \
     --module-path mods:out       \
     --add-modules java.xml.bind,java.sql \
     --add-opens java.base/java.lang=javassist \
     -m bookapp/main.Main
```
We’ve migrated the application by moving only the libraries to automatic modules that we really need. Alternatively, you could start migration by copying all JAR files to the module path. Although this generally makes it easier to quickly get an application running, it makes it harder to come up with a reasonable module descriptor for your application module. Because automatic modules set up implied readability to all other modules, it will hide missing requires in your own module. When upgrading automatic modules to explicit modules, things may break. It’s important to get your module dependencies as well-defined as possible, so you should spend the time investigating this.

> 通过将真正需要的库转换为自动模块来实现应用程序的迁移。或者，也可以通过将所有JAR文件复制到模块路径来开始迁移。虽然这样做通常可以更容易地使应用程序快速地运行，但是却难以为应用程序模块提供合理的模块描述符。由于自动模块为所有其他模块设置了隐式可读性，因此它会在模块中隐藏所缺少的requires。当将自动模块升级为显式模块时，事情可能会中断。尽可能明确地定义模块依赖关系是很重要的，所以应该在这方面多花些时间来研究。

## 9.8 Refactor to Multiple Modules 重构到多个模块
Now that we have a working application, it would be nice to also split the codebase into smaller modules and embrace modularity in the design of the application. That’s beyond the scope of this chapter, but the GitHub repository contains an implementation with multiple modules (➥ chapter9/spring-hibernate-refactored). Figure 9-4 provides a reasonable design for such an improved structure.

> 现在已经有了一个可以工作的应用程序，如果可以将代码拆分成更小的模块，并且在应用程序的设计中实现模块化，那就更好了。虽然这些内容超出了本章的讨论范围，但是GitHub资料库包含了一个带有多个模块的实现（chapter9/spring-hibernate-refactored）。图9-4为这种改进的结构提供了合理的设计。

This design has trade-offs, and you should be aware of them. For example, you have a choice of whether to create a separate API module or to export the API from a module that also contains the implementation. We already discussed many of these choices in Chapter 5.

> 应该可以看到，该设计进行了一定的权衡。例如，可以选择是创建一个单独的API模块还是从同样包含实现的模块中导出API。第5章已经详细讨论过许多类似的选择。

Design proposal for modularised application
Figure 9-4. Application refactored

With this case study, you’ve seen all the tools and processes you need to migrate an existing classpath-based application to modules. Use jdeps to analyze your existing code and dependencies. Move libraries to the module path to turn them into automatic modules, allowing you to create a module descriptor for your application. When your application uses libraries that involve reflection, such as dependency injection, object-relational mapping, or serialization libraries, open packages and modules are the way to go.

> 通过这个案例研究，可以了解将现有的基于类路径的应用程序迁移到模块所需的所有工具和流程。使用jdeps分析现有的代码和依赖关系。将库移动到模块路径以使其转换为自动模块，从而允许为应用程序创建模块描述符。当应用程序使用涉及反射的库时（如依赖注入、对象关系映射或序列化库），则需要开放包和模块。

Migrating an application to modules can expose violations of strong encapsulation, either in the application or its libraries. As we’ve seen, this can lead to sometimes baffling errors, although they can all be explained with enough knowledge of the module system. In this chapter, you’ve learned how to mitigate these issues. Life would be a lot better, though, if libraries already were proper Java 9 modules with explicit module descriptors. The next chapter shows how library maintainers can work toward Java 9 support.

> 无论是在应用程序还是其库中，将应用程序迁移到模块可能会破坏强封装。正如前面所看到的，有时这可能会导致一些莫名其妙的错误，尽管可以使用对模块系统的了解来解释相关错误。在本章中，已经学会了如何解决这些问题。但是，如果库已经是具有显式模块描述符的正确Java 9模块了，那么一切就会更好了。下一章将介绍库维护人员如何支持Java 9。