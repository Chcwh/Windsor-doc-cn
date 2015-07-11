# 常见问题

## 我怎样和容器交互。如何调用？在哪里调用？

Windsor 是 IoC 容器，也就是你一般不需要调用它，你的应用应该无视或不知道它的存在。与容器的交互（即调用容器的任何方法）应该限制在应用生命期的三个地方：

* 在应用启动的时候 (`Main`, `Application_Start` 等等) y创建容器，并调用容器的 `Install` 方法。一次。注意你应该只有一个容器的实例。
* 只有一个地方(在 `Main`， `ControllerFactory` 等中) 可以调用 `Resolve`。If you need to callback to the container to pull some additional dependencies later on, use [typed factories](typed-factory-facility.md)。
* 在应用结束的时候，调用容器的 `Dispose` 方法，让容器清理和释放所有组件。

### 还可以看看

* [关于与容器交互的深入讨论](three-calls-pattern.md)
* [IoC](ioc.md)
* [Windsor 安装器](installers.md)
* [Fluent 注册 API](fluent-registration-api.md)
* [强类型工厂](typed-factory-facility.md)

### 外部资源

* [Blog post by Nicholas Blumhardt explaining thought process behind this usage (Dec 26, 2008)](http://blogs.msdn.com/b/nblumhardt/archive/2008/12/27/container-managed-application-design-prelude-where-does-the-container-belong.aspx)
* [Blog post by Krzysztof Koźmic introducing container usage (in push scenarios) (Jun 20, 2010](http://kozmic.pl/2010/06/20/how-i-use-inversion-of-control-containers/)
* [Blog post by Krzysztof Koźmic explaining container usage (in pull scenarios) (Jun 22, 2010](http://kozmic.pl/2010/06/22/how-i-use-inversion-of-control-containers-ndash-pulling-from/)

## 为什么 Windsor 不注入它自己（`IWindsorContainer`）到我的组件中？

因为你的组件不应该调用 Windsor。这违反了 IoC 的根本原则。或者从实用的角度 - 将会导致你痛苦，是你会后悔的决定。

### 那么我应该如何做

看第一个问题。

## 为什么 Windsor 保持对临时组件（transient）的跟踪？

Windsor，默认情况下跟踪所有的组件以确保正确的[生命周期](lifecycle.md)管理，特别是确保所有 `IDisposable`  组件和它们的依赖正确被正确回收。你可以让 Windsor 停止跟踪组件，通过设置它的[释放策略](release-policy.md)为 `NoTrackingReleasePolicy`，但是要注意的是这是不推荐的，这样做的话你放弃了正确的生命周期管理。

## 为什么 Windsor 不能注入组件的数组或列表？

Windsor，在默认情况下当你有一个`IFoo[]`， `IEnumerable<IFoo>` 或 `IList<IFoo>` 这样的依赖时，会检查是否有一个相同类型的组件被注册（`IFoo` 的数组或列表），而不是是否为 `IFoo` 注册了任何组件（组件的数组，与数组组件是不同的）。你可以将行为改变为 *“当你看见 `IFoo` 的数组或列表时，就给我所有你能找到的 `IFoo` t”*，使用 `CollectionResolver`。查看 [解析器](resolvers.md)。

## 是否可以将组件注册为多个服务？

可以。该功能叫做 [forwarded types](forwarded-types.md).

## 是否可以为任意指定服务注册多个组件？

可以。但你需要给组件唯一的名称。

## 为什么 Windsor 不能解析具体（concrete）类型，在先前没有注册它们时？

因为比起解决的问题，它将导致更多的问题。关于应该配置多少组件没有良好的定论。同时这样做可能掩盖你的注册问题 - 你可能想要注册一个类型，但是忘了，然后得到了错误配置的对象。

话虽如此 - 有一种方法可以在不注册的情况下解析组件，通过使用 [延迟组件加载器](lazy-component-loaders.md)，Windsor （可以确定的设施，比如 [WCF 集成设施](wcf-facility.md) 和 [强类型工厂设施](typed-factory-facility.md)） 从中获益。你也可以，但那是你制定了对象应该如何配置的规则，而不是容器。

## Windsor可以为已存在的对象注入属性吗？

不可以。

## 依赖的有效值可以是 `null` 吗？

不可以。 `null` 意味着 *没有值* 并被 Windsor忽略。 明确指出 - 如果该值是可选的，需要提供不包含该值的构造函数。或者在构造函数的签名中将 `null` 指定为默认值。这是 Windsor 允许传递 `null` 的唯一方案。
