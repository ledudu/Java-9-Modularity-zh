# 前言

> Preface

Java 9 introduces a module system to the platform. This is a major leap, marking the start of a new era for modular software development on the Java platform. We’re very excited about these changes, and we hope you are too after reading this book. You’ll be ready to make the best use of the module system before you know it.

> Java 9 向 Java 平台引入了模块系统，这是一个重大的飞跃，标志着 Java 平台上模块化软件开发的一个新时代的开始。看到这些变化让人感到非常兴奋，希望读者看完本书后也会感到兴奋。在深入了解模块系统之前需要做好充分利用该系统的准备。

## Who Should Read This Book 本书读者

This book is for Java developers who want to improve the design and structure of their applications. The Java module system improves the way we can design and build Java applications. Even if you’re not going to use modules right away, understanding the modularization of the JDK itself is an important first step. After you acquaint yourself with modules in the first part of the book, we expect you to also really appreciate the migration chapters that follow. Moving existing code to Java 9 and the module system will become an increasingly common task.

> 本书为那些想要提高应用程序的设计和结构的 Java 开发者而编写。Java 模块系统改进了设计和构建 Java 应用程序的方法。即使你不打算马上使用模块，了解 JDK 模块化本身也是非常重要的一步。在熟悉了本书第一部分所介绍的模块之后，希望你也能真正理解后续关于迁移的相关章节。将现有代码移至 Java 9 和模块系统将成为一项越来越常见的任务。

This book is by no means a general introduction to Java. We assume you have experience writing relatively large Java applications in a team setting. That’s where modularity becomes more and more important. As an experienced Java developer, you will recognize the problems caused by the classpath, helping you appreciate the module system and its features.

> 本书绝不是对 Java 的一般性介绍。我们假设你拥有在一个团队中编写过较大 Java 应用程序的经验，在较大的 Java 应用程序中模块变得越来越重要。作为一名经验丰富的 Java 开发人员，应该认识到类路径所带来的问题，从而有助于理解模块系统及其功能。

There are many other changes in Java 9 besides the module system. This book, however, focuses on the module system and related features. Where appropriate, other Java 9 features are discussed in the context of the module system.

> 除了模块系统之外，Java 9 中还有许多其他变化。然而，本书主要关注模块系统及其相关功能。当然，在适当的情况下，在模块系统的上下文中也会讨论其他 Java 9 功能。

## Why We Wrote This Book 编写本书的原因

We have been Java users since the early days of Java, when applets still were hot stuff. We’ve used and enjoyed many other platforms and languages over the years, but Java still remains our primary tool. When it comes to building maintainable software, modularity is a key principle. Pursuing modular application development has become somewhat of a passion for us, after spending a lot of energy building modular software over the years. We’ve used technology such as OSGi extensively to achieve this, without support in the Java platform itself. We’ve also learned from tools outside the Java space, such as module systems for JavaScript. When it became clear that Java 9 would feature the long-awaited module system, we decided we didn’t want to just use this feature, but also help with onboarding other developers.

> 很多读者从 Java 早期开始就是 Java 用户，当时 Applet 还非常流行。多年来，我们使用和喜欢过许多其他平台和语言，但 Java 仍然是主要工具。在构建可维护的软件方面，模块化是一个关键原则。多年来人们花费了大量精力来构建模块化软件，并逐渐热衷于开发模块化应用程序。曾经广泛使用诸如 OSGi 之类的技术来实现模块化，但 Java 平台本身并不支持这些技术。此外，还可通过 Java 之外的其他工具学习模块化，比如 JavaScript 的模块系统。当 Java 9 推出了期待已久的模块系统时，我们认为并不能只是使用该功能，还应该帮助其刚入职的开发人员了解模块系统。

Maybe you have heard of Project Jigsaw at some point in the past decade. Project Jigsaw prototyped many possible implementations of a Java module system over the course of many years. A module system for Java has been on and off the table several times. Both Java 7 and 8 were originally going to include the results of Project Jigsaw.

> 也许在过去 10 年的某个时候你曾经听说过 Jigsaw 项目。经过多年的发展，Jigsaw 项目具备了 Java 模块系统许多功能的原型。Java 的模块系统发展断断续续。Java 7 和 Java 8 最初计划包含 Jigsaw 项目的发展结果。

With Java 9, this long period of experimentation culminates into an official module system implementation. Many changes have occurred in the scope and functionality of the various module system prototypes over the years. Even when you’ve been following this process closely, it’s difficult to see what the final Java 9 module system really entails. Through this book, we want to provide a definitive overview of the module system. And, more important, what it can do for the design and architecture of your applications.

> 随着 Java 9 的出现，长期的模块化尝试最终完成了正式模块系统的实现。多年来，各种模块系统原型的范围和功能发生了许多变化。即使你一直在密切关注该过程，也很难弄清楚最终 Java 9 模块系统真正包含什么。本书将会给出模块系统的明确概述，更重要的是将介绍模块系统能够为应用程序的设计和架构做些什么。

## Navigating This Book 本书内容

The book is split into three parts:

> 本书共分为三个部分：

1. Introduction to the Java Module System
2. Migration
3. Modular Development Tooling

---

> 1）Java 模块系统介绍。
> 2）迁移。
> 3）模块化开发工具。

The first part teaches you how to use the module system. Starting with the modular JDK itself, it then goes into creating your own modules. Next we discuss services, which enable decoupling of modules. The first part ends with a discussion of modularity patterns, and how you use modules in a way to maximize maintainability and extensibility.

> 第一部分主要介绍如何使用模块系统。首先从介绍模块化 JDK 本身开始，然后学习创建自己的模块，随后讨论可以解耦模块的服务，最后探讨模块化模式以及如何以最大限度地提高可维护性和可扩展性的方式使用模块。

The second part of the book is about migration. You most likely have existing Java code, probably using Java libraries that were not designed for the module system. In this part of the book, you will learn how to migrate existing code to modules, and how to use existing libraries that are not modules yet. If you are the author or maintainer of a library, there is a chapter specifically about adding module support to libraries.

> 第二部分主要介绍迁移。有可能读者现在所拥有的 Java 代码不是使用专为模块系统而设计的 Java 库。该部分介绍如何将现有代码迁移到模块中，以及如何使用尚未模块化的现有库。如果你是一名库的编写者或者维护者，那么这部分中有一章专门介绍了如何向库添加模块支持。

The third and last part of the book is about tooling. In this part, you will learn about the current state of IDEs and build tools. You will also learn how to test modules, because modules give some new challenges but also opportunities when it comes to (unit) testing. Finally, you will also learn about linking, another exciting feature of the module system. It enables the creation of highly optimized custom runtime images, changing the way you can ship Java applications by virtue of modules.

> 第三部分（也是最后一部分）主要介绍工具。该部分介绍了 IDE 的现状以及构建工具。此外还会学习如何测试模块，因为模块给（单元）测试带来了一些新的挑战，也带来了机会。最后学习链接（linking）——模块系统另一个引人注目的功能。

The book is designed to be read from cover to cover, but we kept in mind that this is not an option for every reader. We recommend to at least go over the first four chapters in detail. This will set you up with the basic knowledge to make good use of the rest of the book. If you are really short on time and have existing code to migrate, you can skip to the second part of the book after that. Once you’re ready for it, you should be able to come back to the more advanced chapters.

> 虽然建议从头到尾按顺序阅读本书，但是请记住并不是所有的读者都必须这样阅读。建议至少详细阅读前四章，从而具备基本知识，以便更好地阅读本书的其他章节。如果时间有限并且有现有的代码需要迁移，那么可以在阅读完前四章后跳到本书的第二部分。一旦完成了迁移，就可以回到“更高级”的章节。

## Using Code Examples 使用代码示例

The book contains many code examples. All code examples are available on GitHub at https://github.com/java9-modularity/examples. In this repository, the code examples are organized by chapter. Throughout the book we refer to specific code examples as follows: ➥ chapter3/helloworld. This means the example can be found in https://github.com/java9-modularity/examples/chapter3/helloworld.

> 本书包含了许多代码示例。所有代码示例都可以在 GitHub（https://github.com/java9-modularity/examples）上找到。在该存储库中，代码示例是按照章节组织的。在本书中，使用下面的方法引用具体的代码示例：chapter3/helloworld，其含义是可以在“https://github.com/java9-modularity/examples/chapter3/helloworld”中找到示例。

We highly recommend having the code available when going through the book, because longer code sections just read better in a code editor. We also recommend playing with the code yourself—for example, to reproduce errors that we discuss in the book. Learning by doing beats just reading the words.

> 强烈建议在阅读本书时使用相关的代码，因为在代码编辑器中可以更好地阅读较长的代码段。此外还建议亲自动手改写代码，如重现书中所讨论的错误。动手实践胜过读书。
