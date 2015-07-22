# Windsor 教程 - ASP.NET MVC 3 应用 （To be Seen）

这是一个入门教程，帮助你加快在一个简单 Web 应用中使用 Windsor。该应用叫做 *To be seen* 帮助用户收集有关即将上映的电影、光盘、书籍、活动等信息，并在到来时提醒他们。

本教程假定事先不熟悉 Windsor，和其他容器或相关概念。但是良好的 C# 知识和 ASP.NET 经验是必须的。

可以在 [github](https://github.com/kkozmic/ToBeSeen) 上找到完成的完整代码。

## 教程

到目前为止以下部分已发布：

### 介绍

* [第一部分 - 获取 Windsor](mvc-tutorial-part-1-getting-windsor.md) 讨论下载 Windsor 程序集及添加到项目。
* [第二部分 - 插入 Windsor](mvc-tutorial-part-2-plugging-windsor-in.md) 讨论自定义控制器工厂的创建
* [第三部分 - 编写第一个安装器](mvc-tutorial-part-3-writing-your-first-installer.md) 讨论编写 Windsor 安装器
  * [第三部分 (a) - 测试第一个安装器](mvc-tutorial-part-3a-testing-your-first-installer.md) 讨论测试 Windsor 安装器 （验证约定）
* [第四部分 - 综合](mvc-tutorial-part-4-putting-it-all-together.md) 讨论使用以前的所有元素来获得工作程序。

## 构建应用

* [第五部分 - 添加日志支持](mvc-tutorial-part-5-adding-logging-support.md) 讨论使用日志设施添加日志支持到应用
* [第六部分 - 持久层](mvc-tutorial-part-6-persistence-layer.md) 讨论创建自定义设施，并注册外部创建的对象，比如配置 NHibernate
* [第七部分 - 生命期类型](mvc-tutorial-part-7-lifestyles.md) 讨论使用不同的生命期类型，特别是 per-web-request
* [第八部分 - 满足依赖](mvc-tutorial-part-8-satisfying-dependencies.md) 讨论依赖是如何解析的，指定依赖的方法
* [第九部分 - 诊断依赖丢失问题](mvc-tutorial-part-9-diagnosing-missing-dependency-issues.md) 讨论如何处理容器抛出的异常
