# 第 9 章 迁移案例研究：Spring 和 Hibernate

> Chapter 9. Migration Case Study: Spring and Hibernate
Chapter 8 presented all the available tools for migrating applications to modules. This chapter brings everything together in a case study. We’ll migrate a fully functional application that uses Spring and Hibernate to modules. Note that we’re deliberately using an example of “traditional” Spring/Hibernate development, instead of using the most modern way of doing things. We’re using pre-Java 9 versions, to create an interesting case study. Many applications have been written this way, which makes it extra interesting to see how these applications can be migrated toward the future. Newer versions of frameworks will support Java 9 better out of the box, and migrations based on these versions will probably be even easier. If you are not familiar with these frameworks, don’t worry. You don’t have to understand all the code and configuration to learn about common problems you may face during a migration toward modules.

You will get a lot more value out of this chapter if you check out the code repository and try to migrate the code while you’re reading the chapter. In the code repository, we have provided three versions:

chapter9/spring-hibernate-starter
The classpath version of the application before migration.

chapter9/spring-hibernate
The migrated application.

chapter9/spring-hibernate-refactored
The migrated application after additional modularization.

We recommend opening the spring-hibernate-starter project in an editor, and applying each step described in this chapter on the code. You should end up with roughly the same result as the finished spring-hibernate example.

Getting Familiar with the Application
The application represents a bookstore. Books are stored in a database by using Hibernate. Spring is used to bootstrap Hibernate, including transaction management and dependency injection. The Spring configuration uses a mix of XML and annotation-based configuration.

Before migration, the application code, direct dependencies, and transitive dependencies are on the classpath, as shown in Figure 9-1.

Migration starting point (➥ chapter9/spring-hibernate-starter)
Figure 9-1. Migration starting point (➥ chapter9/spring-hibernate-starter)
The end result of the migration is a codebase with a single module, using automatic modules for dependencies where necessary. Figure 9-2 shows the end result.

Application migrated (➥ chapter9/spring-hibernate)
Figure 9-2. Application migrated (➥ chapter9/spring-hibernate)
At the end of the chapter, we will also look at refactoring the application code itself to be more modular.

Before we even start thinking about splitting our code in modules, we should get over some technical issues coming from our dependencies. Remember that we’re dealing with pre-Java 9 libraries, which were not designed to work with the Java module system. Specifically, we are using the following framework versions:

Spring 4.3.2

Hibernate 5.0.1

Also note that great progress has been made within these frameworks when it comes to module support. At the time of writing, the first Release Candidate of Spring 5 was released, with explicit support for usage as automatic modules. We do not use these updated versions because that wouldn’t make a realistic migration example. Even without special support from frameworks and libraries, migration to modules is possible.

The focus of this section is to create a single module for our code. This means defining requires for dependencies, and moving libraries to automatic modules. We will also have to deal with exports and opens to make our code accessible to the frameworks. Once this migration is complete, we can take a good look at the design of our code, and potentially split the code into smaller modules. Because we already solved all the technical issues, this becomes a design exercise.

Let’s start by looking at the most important parts of the code to get an idea of the application. For optimal readability of the code, we strongly advise opening the code in your favorite editor.

The Book class (shown in Example 9-1) is a JPA entity that can be stored in a database by using Hibernate (or another JPA implementation). It has annotations such as @Entity and @Id to configure the mapping to the database.

Example 9-1. Book.java (➥ chapter9/spring-hibernate)
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
The HibernateBooksService is a Spring Repository. This is a service that automatically takes care of transaction management, which is required to successfully store something in the database. It implements our service interface BooksService, and uses the Hibernate API (such as SessionFactory) to store and retrieve books from the database.

The BookstoreService is a simple interface, whose implementation in BookStoreServiceImpl, shown in Example 9-2, can calculate the total price of a given list of books. It is annotated with Spring’s @Component annotation so that it becomes available for dependency injection.

Example 9-2. BookstoreServiceImpl.java (➥ chapter9/spring-hibernate)
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
Finally, we have a main class that bootstraps Spring, and stores and retrieves some books, as shown in Example 9-3.

Example 9-3. Main.java (➥ chapter9/spring-hibernate)
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
Spring is bootstrapped by using ClassPathXmlApplicationContext, which requires an XML configuration. In this configuration, shown in Example 9-4, we set up component scanning, which automatically registers @Component and @Repository annotated classes as Spring beans. We also set up transaction management and Hibernate.

Example 9-4. main.xml (➥ chapter9/spring-hibernate)
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
The directory structure of the project is currently as follows:

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
The src directory contains the configuration files and the source code packages. The lib directory contains the JAR files of Spring, Hibernate, and transitive dependencies of both, which is a long list of 31 JAR files in total. To build and run the application, we can use the following commands:

javac -cp [list of JARs in lib] -d out -sourcepath src $(find src -name '*.java')

cp $(find src -name '*.xml') out

java -cp [list of JARs in lib]:out main.Main
Running on the Classpath with Java 9
The first step toward any migration to modules should start with compiling and running the code with Java 9, while still using the classpath. This also shows the first problem to solve. Hibernate relies on some JAXB classes. In “Using JAXB and Other Java EE APIs”, you learned that JAXB is part of the java.se.ee subgraph, but not of the default java.se module subgraph. Without modification, running Main results in java.lang.ClassNotFoundException: javax.xml.bind.​JAXBException. We need to add JAXB to our application by using the --add-modules flag:

java -cp [list of JARs in lib]:out --add-modules java.xml.bind main.Main
We’re now seeing another, rather obscure, warning:

WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by javassist.util.proxy.SecurityActions
(file:.../lib/javassist-3.20.0-GA.jar)
to method java.lang.ClassLoader.defineClass(...)
WARNING: Please consider reporting this to the maintainers
  of javassist.util.proxy.SecurityActions
WARNING: Use --illegal-access=warn to enable warnings of further illegal
  reflective access operations
WARNING: All illegal access operations will be denied in a future release
We discussed exactly this issue in “Libraries, Strong Encapsulation, and the JDK 9 Classpath”. The javassist library tries to use deep reflection on JDK types, which is by default allowed, but generates a warning. If we were to run the application with --illegal-access=deny, this would even turn into an error. Remember, in a future Java release, this will be the default. We don’t want to fix this issue right now by updating to a possibly fixed version of javassist. Still, we can get rid of the warning by using --add-opens:

java -cp [list of JARs in lib]:out \
--add-modules java.xml.bind \
--add-opens java.base/java.lang=ALL-UNNAMED main.Main
It’s up to you whether you want to silence the warning by adding --add-opens. Doing so prepares you for the next Java releases, where illegal access from the classpath isn’t treated as friendly as in Java 9. The problematic behavior of javassist is still there. Raising such issues with library maintainers is the right course of action.

Setting Up for Modules
With these problems out of the way, we can start migrating toward modules. First, we migrate the code to a single module. Keeping the code’s internal structure unchanged first is a good strategy when migrating to modules. Although this structure might not be the final structure you want, it can be easier to first focus on the technical problems.

The first step is to change -sourcepath to --module-source-path. To do this, we need to slightly change the structure of the project. The src directory should not contain packages directly, but a module directory first. The module directory should also contain module-info.java:

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
We modify the compile/run script to use --module-source-path and to start the main class from our module:

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
We didn’t move any libraries to the module path yet, nor did we put anything in module-info.java, so the preceding commands are obviously going to fail.

Using Automatic Modules
To be able to compile our module, we need to add requires statements to module-info.java for any compile-time dependencies. This also implies that we need to move some of the JAR files from the classpath to the module path to make them automatic modules. To figure out which compile-time dependencies we have exactly, we can look at import statements in our code or use jdeps. For our example application, we can come up with the following list of requires statements, based on direct compile-time dependencies:

requires spring.context;
requires spring.tx;

requires javax.inject;

requires hibernate.core;
requires hibernate.jpa;
All are pretty self-explanatory; packages from these modules are used directly from the sample code. To be able to require these libraries, we move their corresponding JAR files to the module path to make them automatic modules. Transitive dependencies can stay on the classpath, as shown in Figure 9-3.

TIP
Remember “Using jdeps”? jdeps is useful when you have to figure out what modules to require during migration. For example, we could have found our requires by running the following on our classpath version of the application:

jdeps -summary -cp lib/*.jar out

out -> lib/hibernate-core-5.2.2.Final.jar
out -> lib/hibernate-jpa-2.1-api-1.0.0.Final.jar
out -> java.base
out -> lib/javax.inject-1.jar
out -> lib/spring-context-4.3.2.RELEASE.jar
out -> lib/spring-tx-4.3.2.RELEASE.jar
Migration with automatic modules.
Figure 9-3. Migration with automatic modules
Besides these compile-time dependencies, we have both an additional compile-time and run-time requirement that we can add by using --add-modules. Types from the java.naming platform module are used in the Hibernate API, requiring it to be available at compile-time, even when we don’t use this type explicitly in our code. Without adding the module explicitly, we would see this error:

src/bookapp/books/impl/service/HibernateBooksService.java:19:
    error: cannot access Referenceable
    return sessionFactory.getCurrentSession().get(BookEntity.class, id);
                         ^
  class file for javax.naming.Referenceable not found
1 error
Because Hibernate is used as an automatic module, it does not cause extra platform modules to resolve. Our code does have an indirect compile-time dependency on it, though, because Hibernate uses a type from java.naming in its API (the SessionFactory interface extends Referenceable). This means we have a compile-time dependency, without having a requires, which doesn’t work. If Hibernate were an explicit module, it should have requires transitive java.naming in its module-info.java to set up implied readability and prevent this problem.

Until then, we can work around the issue by adding --add-modules java.naming to javac. Alternatively, we could have added another requires for java.naming in our module descriptor. As discussed previously, we prefer to avoid requires statements for indirect dependencies, hence the choice for --add-modules instead.

The application compiles successfully now. Running it still results in some errors.

Java Platform Dependencies and Automatic Modules
A java.lang.NoClassDefFoundError tells us we need to add java.sql to --add-modules for the java command. Why wasn’t java.sql resolved without our manual intervention? Hibernate depends on java.sql internally. Because Hibernate is used as an automatic module, it doesn’t have a module descriptor to require other (platform) modules. This problem is similar to the preceding problem with java.naming, but manifests itself differently. The java.naming example was a compile-time error because the Hibernate API that our code uses references a type from java.naming. In this case, Hibernate itself uses java.sql internally, but its types are not part of the Hibernate API that we compile against. Therefore, the error shows only at run-time.

With an extra --add-modules java.sql, we can move on to the next step.

Opening Packages for Reflection
We’re getting close to successfully running the application, but rerunning the application still results in an error. This time the error is pretty straightforward:

Caused by: java.lang.IllegalAccessException:
class org.springframework.beans.BeanUtils cannot access class
  books.impl.service.HibernateBooksService (in module bookapp) because module
  bookapp does not export books.impl.service to unnamed module @5c45d770
Spring relies on reflection to instantiate classes. For this to work, we have to open implementation packages containing classes that Spring needs to instantiate. Hibernate, by the same token, also uses reflection to manipulate entity classes. Hibernate needs access to the Book interface and to the BookEntity implementation class.

For the API packages in the application, it makes sense to export them. Later, when splitting into more modules, these packages are most probably used by other modules as well. For implementation packages, we use opens instead. This way, the frameworks can do their reflection magic, while we still ensure encapsulation at build-time:

exports books.api.service;
exports books.api.entities;

opens books.impl.entities;
opens books.impl.service;
opens bookstore.impl.service;
In a larger application, it could be a good choice to use an open module at first instead of specifying individual packages to be open. After setting up our package opens/exports, we see another familiar error.

java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException
One of our libraries is using JAXB (not our code), and in “Using JAXB and Other Java EE APIs”, you learned that java.xml.bind is not resolved by default. Simply adding the module to our --add-modules, like we did when running this example on the classpath, will help us out of this situation.

Almost done!

Unfortunately, the javassist library gives us a last obscure error when trying to run the application:

Caused by: java.lang.IllegalAccessError: superinterface check failed:
  class books.impl.entities.BookEntity_$$_jvstced_0 (in module bookapp)
  cannot access class javassist.util.proxy.ProxyObject
  (in unnamed module @0x546621c4) because module bookapp
  does not read unnamed module @0x546621c4
Fixing Illegal Access
Hibernate uses the javassist library to dynamically create subclasses of our entity classes. At run-time, our application code uses these subclasses instead of the original classes. Because our code runs from a module, the generated classes end up being part of the same bookapp module. The generated classes implement an interface (ProxyObject) from javassist. However, javassist is still on the classpath, which is unreachable from explicit modules. Therefore, the generated classes implement an interface that’s not accessible for them at run-time. Although this is an obscure and hard-to-understand error, the fix is easy: move javassist from the classpath to the module path, so that it becomes an automatic module and can be accessed from other modules.

However, turning javassist into an automatic module introduces a new problem. Earlier you saw that javassist uses illegal deep reflection on JDK types. On the classpath, with its lenient --illegal-access=permit default, this was only giving us a warning. Because javassist is now an automatic module, the --illegal-access mechanism doesn’t apply anymore. It affects only code on the classpath. This means we are now getting an error, which is essentially the same error that we would have seen if we had run the classpath example with --illegal-access=deny:

Caused by: java.lang.reflect.InaccessibleObjectException:
  Unable to make protected final java.lang.Class
  java.lang.ClassLoader.defineClass(...)
  throws java.lang.ClassFormatError accessible:
  module java.base does not "opens java.lang" to module javassist
We already know that we can work around this issue by adding --add-opens java.base/java.lang=javassist to the java command. Our final script to compile and run the application is shown in Example 9-5.

Example 9-5. run.sh (➥ chapter9/spring-hibernate)
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
We’ve migrated the application by moving only the libraries to automatic modules that we really need. Alternatively, you could start migration by copying all JAR files to the module path. Although this generally makes it easier to quickly get an application running, it makes it harder to come up with a reasonable module descriptor for your application module. Because automatic modules set up implied readability to all other modules, it will hide missing requires in your own module. When upgrading automatic modules to explicit modules, things may break. It’s important to get your module dependencies as well-defined as possible, so you should spend the time investigating this.

Refactor to Multiple Modules
Now that we have a working application, it would be nice to also split the codebase into smaller modules and embrace modularity in the design of the application. That’s beyond the scope of this chapter, but the GitHub repository contains an implementation with multiple modules (➥ chapter9/spring-hibernate-refactored). Figure 9-4 provides a reasonable design for such an improved structure.

This design has trade-offs, and you should be aware of them. For example, you have a choice of whether to create a separate API module or to export the API from a module that also contains the implementation. We already discussed many of these choices in Chapter 5.

Design proposal for modularised application
Figure 9-4. Application refactored
With this case study, you’ve seen all the tools and processes you need to migrate an existing classpath-based application to modules. Use jdeps to analyze your existing code and dependencies. Move libraries to the module path to turn them into automatic modules, allowing you to create a module descriptor for your application. When your application uses libraries that involve reflection, such as dependency injection, object-relational mapping, or serialization libraries, open packages and modules are the way to go.

Migrating an application to modules can expose violations of strong encapsulation, either in the application or its libraries. As we’ve seen, this can lead to sometimes baffling errors, although they can all be explained with enough knowledge of the module system. In this chapter, you’ve learned how to mitigate these issues. Life would be a lot better, though, if libraries already were proper Java 9 modules with explicit module descriptors. The next chapter shows how library maintainers can work toward Java 9 support.