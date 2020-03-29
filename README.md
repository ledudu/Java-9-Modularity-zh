# Java-9-Modularity-zh

《Java 9 模块化开发》中文翻译

在线阅读：[http://gdut_yy.gitee.io/doc-java9/](http://gdut_yy.gitee.io/doc-java9/)

<img src="./docs/cover.jpg" width=24% />

## 目录

- [第一部分 Java 模块系统介绍](docs/part1.md)
- [第 1 章 模块化概述](docs/ch1.md)
- [第 2 章 模块化和模块化 JDK](docs/ch2.md)
- [第 3 章 使用模块](docs/ch3.md)
- [第 4 章 服务](docs/ch4.md)
- [第 5 章 模块化模式](docs/ch5.md)
- [第 6 章 高级模块化模式](docs/ch6.md)

- [第二部分 迁移](docs/part3.md)
- [第 7 章 没有模块的迁移](docs/ch7.md)
- [第 8 章 迁移到模块](docs/ch8.md)
- [第 9 章 迁移案例研究：Spring 和 Hibernate](docs/ch9.md)
- [第 10 章 库迁移](docs/ch10.md)

- [第三部分 模块化开发工具](docs/part4.md)
- [第 11 章 构建工具和 IDE](docs/ch11.md)
- [第 12 章 测试模块](docs/c12.md)
- [第 13 章 使用自定义运行时映像进行缩减](docs/ch13.md)
- [第 14 章 模块化的未来](docs/ch14.md)

## 本地开发 & 阅读

本项目基于 vuepress 进行开发，以提供比 github mardown 更佳的阅读体验

依赖于 `node.js`、`yarn`、`vuepress` 等环境

```sh
# vuepress
yarn global add vuepress

# 本地开发
git clone https://github.com/gdut-yy/Java-9-Modularity-zh.git
cd Java-9-Modularity-zh/
yarn docs:dev
```

## License

[MIT](./LICENSE)
