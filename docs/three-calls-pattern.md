# 三个调用模式

IoC 框架与其他框架不同之处在于你不会在代码中看见许多对框架的调用。实际上，在大多数应用中（忽略它们的大小和复杂性）你仅在三个地方直接调用容器。这是最常用的使用模式，Windsor完全支持。

## 三个容器调用模式

该模式被称为 Three Calls。有时被称为 RRR - 注册，解析，释放（Register, Resolve, Release） - 在 [Mark Seemann's book about containers](http://www.manning.com/seemann/)使用的名称。顾名思义，在应用中应用容器只需要在三个地方调用容器。更精确的说，在入口项目。

:information_source: **什么是入口项目？** 入口项目指的是`.exe`中的`Program.Main` 方法，web应用中的 `Application_Start` 和 `Application_End` 或 其他类型应用的对应地方。

:information_source: **是的，这也意味着在 *企业级* 应用中:** IoC 新手很难相信，你可以使用容器构建一个“企业级”“大”，“真实”应用并且几乎没有明确的使用它。是的，绝对可以。它已经做过多次并且运行良好。就像第一段提到的那样，那就是IoC的全部。

这也暗示了使用容器的一个非常重要的方面 - 在应用中有一个应用实例。你只需要一个单独的，容器的根容器实例。在一些非常罕见的高级场景，你可能会创建“子”容器，但是它们都与那个单一的无处不在的根容器栓在一起，作为子容器。

现在来看看都有哪三步。

### Call one - 引导程序（bootstrapper）

你在引导程序中创建和配置容器。它通常只是一个看起来有点像这样的方法：

```csharp
public IWindsorContainer BootstrapContainer()
{
    return new WindsorContainer()
        .Install(
            Configuration.FromAppConfig(),
            FromAssembly.This()
            //perhaps pass other installers here if needed
        );
}
```

在引导程序你做这些事情：

* 创建将要使用的容器。
* 需要的话自定义容器。 大多数时候你不会这样做 (通常是从不) ，因为容器的默认行为和配置应该能满足95％的应用程序。 通过自定义，我的意思是更换容器的 `HandlerFactory`, `ReleasePolicy`, `DependencyResolver`, 子系统。容器内部使用来完成其工作。你可能还需要一些扩展添加到容器，例如 需要在任何组件注册之前的[设施](facilities.md)。
* 注册所有容器将要管理的所有[组件](services-and-components.md)。即上面代码中 `Install` 方法的调用。这里你传递 封装有关应用程序的特定组件的所有信息的[安装器](installers.md) 。这就是你将要看到的大部分工作。

:information_source: **建议仅调用一次 `Install` :** 建议在`Install`的单个调用中安装所有安装器。虽然目前容器也能正常工作，即使多次调用`Install` ，或在这个方法以外配置组件。Windsor 对此进行了优化，但它会处理得更好。 在将来的版本中将会对单次 `Install` 调用进行额外的优化。

### Call two - `Resolve`

在第一步，我们完成了配置容器，现在我们可以使用它了。最重要的是我们只使用它一次（记住 *IoC* 部分）。每个应用有一个根组件。在 MonoRail 或 ASP.NET MVC 应用中它是控制器，在Silverlight, WPF 或 WinForms应用中是主窗口，在WCF服务中是服务，等等。你可能会有多个根组件，比如在控制台应用中你可能有两个 - 一个用于解析命令行参数，另一个处理实际的工作。how ever there will always be very few of them and they will be the root of your component graph.

这些是你显式从容器`解析`的全部组件。容器然后构造它们的依赖和依赖的依赖等等的整个图，容器做了所有工作，因此就可以像下面这样简单的编写代码：

```csharp
var container = BootstrapContainer();
var shell = container.Resolve<IShell>();
shell.Display();
```

:information_source: **通过类型解析 与 通过名称解析:**  `Resolve` 方法有一些重载，可以分为两组：使用名称和不使用名称。如果没有提供名称，Windsor 使用类型去定位服务（就像上面的例子）。 If name is provided the name will be used and type will be used as a convenience for you (so that you don't have to cast from `object` or as a hint to the container, when you're resolving open generic component, how Windsor should close the open generic type). **除非有足够的理由去使用名称解析，使用类型**

到上面代码的第二行执行完毕的时候，你将会拥有一个完全配置的`IShell`服务的实例，然后你可以在后面的代码中控制它了。如果你刚刚接触容器，那可能是真正令人印象深刻的。容器为你创建整个的复杂的对象图，在一行代码里面。这是深刻的，相信我。

:information_source: **What about components I can't/don't want to obtain at root resolution time/place?** For cases where you need to pull some components from the container at some later point in time, not when resolving root component(s) use [typed factories](typed-factory-facility.md).

:information_source: **What about components I want to instantiate at root resolution time/place but I don't really want to do anything with them, like background tasks?** For cases where you have components that are not dependencies of your root, but need to start as the application starts (like background tasks) use [Startable Facility](startable-facility.md).

### Call three - `Dispose`

这一步许多人（特别是那些坚持container does "dependency injection"）都忘记了。容器管理组件的整个生命期，并且在关闭应用之前我们需要关闭容器，这将反过来停用它管理的所有组件（比如回收它们）。这就是为什么在关闭应用之前调用`container.Dispose()`是如此重要。

## 其他资源

* [Windsor 安装器](installers.md)
* [Fluent 注册 API](fluent-registration-api.md)
* [强类型工厂设施（Typed Factory Facility）](typed-factory-facility.md)
* [可启动设施（Startable Facility）](startable-facility.md)
* [生命周期](lifecycle.md)
* [Extensibility Sample App - EventBrokerFacility](sample-eventbrokerfacility.md)
* [Silvertlight Sample App Customers Contact Manager](sample-silverlight-customer-contact-manager.md)
