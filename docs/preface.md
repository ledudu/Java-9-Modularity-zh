Preface
Java 9 introduces a module system to the platform. This is a major leap, marking the start of a new era for modular software development on the Java platform. We’re very excited about these changes, and we hope you are too after reading this book. You’ll be ready to make the best use of the module system before you know it.

Who Should Read This Book
This book is for Java developers who want to improve the design and structure of their applications. The Java module system improves the way we can design and build Java applications. Even if you’re not going to use modules right away, understanding the modularization of the JDK itself is an important first step. After you acquaint yourself with modules in the first part of the book, we expect you to also really appreciate the migration chapters that follow. Moving existing code to Java 9 and the module system will become an increasingly common task.

This book is by no means a general introduction to Java. We assume you have experience writing relatively large Java applications in a team setting. That’s where modularity becomes more and more important. As an experienced Java developer, you will recognize the problems caused by the classpath, helping you appreciate the module system and its features.

There are many other changes in Java 9 besides the module system. This book, however, focuses on the module system and related features. Where appropriate, other Java 9 features are discussed in the context of the module system.

Why We Wrote This Book
We have been Java users since the early days of Java, when applets still were hot stuff. We’ve used and enjoyed many other platforms and languages over the years, but Java still remains our primary tool. When it comes to building maintainable software, modularity is a key principle. Pursuing modular application development has become somewhat of a passion for us, after spending a lot of energy building modular software over the years. We’ve used technology such as OSGi extensively to achieve this, without support in the Java platform itself. We’ve also learned from tools outside the Java space, such as module systems for JavaScript. When it became clear that Java 9 would feature the long-awaited module system, we decided we didn’t want to just use this feature, but also help with onboarding other developers.

Maybe you have heard of Project Jigsaw at some point in the past decade. Project Jigsaw prototyped many possible implementations of a Java module system over the course of many years. A module system for Java has been on and off the table several times. Both Java 7 and 8 were originally going to include the results of Project Jigsaw.

With Java 9, this long period of experimentation culminates into an official module system implementation. Many changes have occurred in the scope and functionality of the various module system prototypes over the years. Even when you’ve been following this process closely, it’s difficult to see what the final Java 9 module system really entails. Through this book, we want to provide a definitive overview of the module system. And, more important, what it can do for the design and architecture of your applications.

Navigating This Book
The book is split into three parts:

Introduction to the Java Module System

Migration

Modular Development Tooling

The first part teaches you how to use the module system. Starting with the modular JDK itself, it then goes into creating your own modules. Next we discuss services, which enable decoupling of modules. The first part ends with a discussion of modularity patterns, and how you use modules in a way to maximize maintainability and extensibility.

The second part of the book is about migration. You most likely have existing Java code, probably using Java libraries that were not designed for the module system. In this part of the book, you will learn how to migrate existing code to modules, and how to use existing libraries that are not modules yet. If you are the author or maintainer of a library, there is a chapter specifically about adding module support to libraries.

The third and last part of the book is about tooling. In this part, you will learn about the current state of IDEs and build tools. You will also learn how to test modules, because modules give some new challenges but also opportunities when it comes to (unit) testing. Finally, you will also learn about linking, another exciting feature of the module system. It enables the creation of highly optimized custom runtime images, changing the way you can ship Java applications by virtue of modules.

The book is designed to be read from cover to cover, but we kept in mind that this is not an option for every reader. We recommend to at least go over the first four chapters in detail. This will set you up with the basic knowledge to make good use of the rest of the book. If you are really short on time and have existing code to migrate, you can skip to the second part of the book after that. Once you’re ready for it, you should be able to come back to the more advanced chapters.

Using Code Examples
The book contains many code examples. All code examples are available on GitHub at https://github.com/java9-modularity/examples. In this repository, the code examples are organized by chapter. Throughout the book we refer to specific code examples as follows: ➥ chapter3/helloworld. This means the example can be found in https://github.com/java9-modularity/examples/chapter3/helloworld.

We highly recommend having the code available when going through the book, because longer code sections just read better in a code editor. We also recommend playing with the code yourself—for example, to reproduce errors that we discuss in the book. Learning by doing beats just reading the words.

Conventions Used in This Book
The following typographical conventions are used in this book:

Italic
Indicates new terms, URLs, email addresses, filenames, and file extensions.

Constant width
Used for program listings, as well as within paragraphs to refer to program elements such as variable or function names, databases, data types, environment variables, statements, and keywords.

Constant width bold
Shows commands or other text that should be typed literally by the user.

Constant width italic
Shows text that should be replaced with user-supplied values or by values determined by context.

NOTE
This icon signifies a general note.

TIP
This icon signifies a tip or suggestion.

WARNING
This icon indicates a warning or caution.

O’Reilly Safari
Safari (formerly Safari Books Online) is a membership-based training and reference platform for enterprise, government, educators, and individuals.

Members have access to thousands of books, training videos, Learning Paths, interactive tutorials, and curated playlists from over 250 publishers, including O’Reilly Media, Harvard Business Review, Prentice Hall Professional, Addison-Wesley Professional, Microsoft Press, Sams, Que, Peachpit Press, Adobe, Focal Press, Cisco Press, John Wiley & Sons, Syngress, Morgan Kaufmann, IBM Redbooks, Packt, Adobe Press, FT Press, Apress, Manning, New Riders, McGraw-Hill, Jones & Bartlett, and Course Technology, among others.

For more information, please visit http://oreilly.com/safari.

How to Contact Us
You can follow a Twitter account accompanying this book to keep up with developments around modular development in Java:

@javamodularity: http://twitter.com/javamodularity

You can also visit https://javamodularity.com

You can also contact the authors directly:

Sander Mak
@sander_mak

sandermak@gmail.com

Paul Bakker
@pbakker

paul.bakker.nl@gmail.com

Please address comments and questions concerning this book to the publisher:

O’Reilly Media, Inc.
1005 Gravenstein Highway North
Sebastopol, CA 95472
800-998-9938 (in the United States or Canada)
707-829-0515 (international or local)
707-829-0104 (fax)
We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at http://bit.ly/java-9-modularity.

To comment or ask technical questions about this book, send email to bookquestions@oreilly.com.

For more information about our books, courses, conferences, and news, see our website at http://www.oreilly.com.

Find us on Facebook: http://facebook.com/oreilly

Follow us on Twitter: http://twitter.com/oreillymedia

Watch us on YouTube: http://www.youtube.com/oreillymedia

Acknowledgments
The idea for this book originated during a conversation with Brian Foster from O’Reilly at JavaOne back in 2015. Thank you for entrusting us with this project. Since that moment, many people have helped us create Java 9 Modularity.

This book would not have been what it is today without the great technical reviews from Alex Buckley, Alan Bateman, and Simon Maple. Many thanks to them, as they contributed many improvements to the book. We’re also grateful for the support of O’Reilly’s editorial team. Nan Barber and Heather Scherer made sure all organizational details were taken care of.

Writing this book would not have been possible without the unwavering support of my wife, Suzanne. She and our three boys had to miss me on many evenings and weekends. Thank you for sticking with me to the end! I also want to thank Luminis for graciously providing support to write this book. I’m glad to be part of a company that lives and breathes the mantra “Knowledge is the only treasure that increases on sharing.”

Sander Mak

I would also like to thank my wife, Qiushi, for supporting me while writing my second book, even while we were moving to the other side of the world. Also thanks to both Netflix and Luminis, for giving me the time and opportunity to work on this book.

Paul Bakker

The comics in Chapters 1, 7, 13, and 14 were created by Oliver Widder and are licensed under Creative Commons Attribution 3.0 Unported (CC BY 3.0). The authors changed the comics to be horizontal and grayscale.