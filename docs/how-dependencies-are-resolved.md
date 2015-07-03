# 依赖是如何解析的

Windsor 中的组件很少是独立的。毕竟，Windsor 最主要的任务是查找和管理依赖。如果组件有一些依赖，Windsor 通过几个步骤来解析它们。

## 依赖解析器（Dependency Resolver）

Windsor 使用依赖解析器（实现 `IDependencyResolver` 接口的类型）解析组件的依赖。默认依赖解析器 （`DefaultDependencyResolver` 类）检查以下几个地方以解析依赖。

## 创建上下文（Creation Context）

首先依赖解析器为依赖调用 `CreationContext` 。 创建上下文首先尝试用名称解析依赖，然后通过使用内联提供的依赖的类型， 也就是说，它来自于以下任何一种情况:

1. 传递给 `Resolve` 的参数: `container.Resolve<IFoo>(new Arguments(new { inlineDependency = "its value" }));`
2. 通过 [Fluent 注册 API](fluent-registration-api.md) 的方法 [`DynamicParameters`](inline-dependencies#supplying-dynamic-dependencies) 传递的参数。

:information_source: **其他来源:** 注意 [强类型工厂设施](typed-factory-facility.md) 转发工厂方法传递的参数作为内联参数。

## 处理器

如果没有内联参数能够满足依赖，解析器询问处理器是否能够满足。处理器首先尝试通过名字解析依赖，然后通过类型。值来自 `ComponentModel` 的  `CustomDependencies` 集合，这通常是传递给DependsOn方法的参数。

```csharp
kernel.Register(Component.For<ClassWithArguments>()
    .DependsOn(
        Property.ForKey<string>().Eq("typed"),
        Property.ForKey<int>().Eq(2)
    )
);
```

## 子解析器（Subresolvers）

如果上面那些地方都不能解析依赖，解析器询问它的每一个子解析器[（实现(`ISubDependencyResolver`)）](resolvers.md)是否能够提供依赖。

## 核心（Kernel）

如果上面那些都不能解析依赖，容器尝试自己解析。根据依赖的类型，容器尝试下面的步骤：

* 对于参数依赖，容器尝试检查 `ComponentModel`的 `Parameters` 集合以匹配依赖。包括 XML 提供的参数，或通过 Fluent API 的 `Parameters` 方法传递的参数。
* 对于 [service override dependency](registering-components-one-by-one.md#supplying-the-component-for-a-dependency-to-use-service-override)，容器尝试通过指定关键字匹配组件。
* 对于服务依赖，容器将会尝试通过指定关键字匹配任意一个组件。

如果上面都不行，抛出`DependencyResolverException`异常。

## 还可以看看

* [组件是如何创建的](how-components-are-created.md)
* [构造函数是如何选择的](how-constructor-is-selected.md)