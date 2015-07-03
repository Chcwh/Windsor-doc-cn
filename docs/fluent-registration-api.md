# Fluent 注册 API

注册 API 提供注册和配置组件的Fluent方法。这是推荐的注册方式，比过细的XML注册更好。因为它是强类型的且容易被重构，它可以作为XML配置的替代方式或补充。和XML一起使用更为有利。

## 需求

`Castle.Core.dll` 和 `Castle.Windsor.dll` 都需要。为了使用这个API，你需要添加 引用 `Castle.MicroKernel.Registration` 。后面的示例假设你已经创建了Windsor 容器：

```csharp
IWindsorContainer container = new WindsorContainer();
```

我们将在整个API文档中使用这个实例。

## 一个一个的注册组件

Fluent API有一个一个注册组件的基础能力，可以显式的配置组件的所有方面([阅读更多](registering-components-one-by-one.md))

## 按照约定注册组件

注册组件的推荐方式是使用约定驱动注册，可以显著的减少配置应用程序的工作量和代码量。 ([查看更多](registering-components-by-conventions.md))

## 代理

[注册拦截器和代理选项](registering-interceptors-proxyoptions.md).

## 高级主题

如果对高级主题感兴趣, like pieces of the API targeted at extending the components 阅读 [高级主题](fluent-registration-api-extensions.md).

## 使用XML配置

如果你需要同时使用两种风格进行注册，这里有两个选择。如果XML配置可以先生效，先简单的使用需要Xml文件名的构造函数重载，然后使用Fluent API。

```csharp
IWindsorContainer container = new WindsorContainer("dependencies.config");

container.Register(
	Component.For....
);
```

如果 Fluent 注册必须先发生，在所有Fluent 调用之后加上下面的代码。

```csharp
container.Register(
	Component.For....
);

container.Install(
    Configuration.FromXmlFile("dependencies.config"));
```

## 还可以看看

* [一个一个注册组件](registering-components-one-by-one.md)
* [按约定注册组件](registering-components-by-conventions.md)
* [Windsor 安装器](installers.md)
* [XML 配置引用](XML-registration-reference.md)
