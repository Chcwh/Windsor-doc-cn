# 组件是怎样创建的

<img align="right" src="images/creation-flow.png">

当从Windsor容器请求组件时，容器需要经过几个步骤来提供实例。右边的图片描述了这些步骤的更重要的方面。我们将在这里讨论它们的更多细节。

:information_source: **关于实例生命周期（lifecycle）和生命期方式（lifestyle）:** 该页面提供了双重目的。首先它解释了当[组件](services-and-components.md)的一个实例被以**忽略组件的 [生命期方式](lifestyles.md)**请求时，Windsor所做的工作。除此之外，它描述了组件的生命周期的第一个部分，这是它的开始和出生（或用技术术语，什么导致实例被创建，以及它如何被创建）。请千万记住这不是一个实例的整个生命周期，这只是第一步。要了解的整个生命周期，一直到组件的死亡，请查看[实体生命周期](lifecycle.md)页面。

## 定位处理器（Locating handler）

当组件被请求时由容器执行的第一步，是检查所请求的组件是否被注册到容器中。容器从其[命名子系统](subsystems.md)中查找组件，如果找不到该组件，容器会尝试[延迟-组件-加载器|延迟注册]，如果没有成功，一个`ComponentNotFoundException`将被抛出。

假设可以发现找到正确的组件，容器将轮询其[处理器](handlers.md)，并要求解析组件实例。

## 处理器都做了什么

处理器做了几件事情:

* 它调用与它相关联的所有`ComponentResolvingDelegate`，让他们有机会在实际开始之前来影响它的决定。这里有个例子， 什么时候[委托传递到Fluent注册API的DynamicParameters方法](inline-dependencies.md#supplying-dynamic-dependencies)。（[Fluent注册API](fluent-registration-api.md)）
* 如果未提供内嵌参数，它会检查该组件及其所有强制依赖是否能够解析。如果不能，抛出`HandlerException`异常。
* 最后，处理器要求[生命期方式管理器](lifestyles.md)解析该组件。

## 生命期方式管理器都做了什么

生命期管理器具有相对简单的角色。如果它有一个它可以重复使用的组件实例，它获得并直接返回该实例返回到处理器。如果没有，它要求它的[组件激活器](component-activators)创建一个。

## 组件激活器都做了什么

:information_source: **组件激活器:** 
组件激活器负责创建组件的实例。各种激活器有各自的实现。当您通过`UsingFactoryMethod`创建组件时，您提供的委托将被调用以创建实例。[工厂支持设施](factory-support-facility.md)或[远程设施](remoting-facility.md)有它们自己的一套激活器，用于执行组件的自定义初始化。

多数时候你应该使用 `DefaultComponentActivator` ，其工作流程如下:

* 调用组件的构造函数实例化组件。 :information_source: **构造函数是怎样选择的:** 了解默认组件激活器如何选择构造函数，点击[这里](how-constructor-is-selected.md).
* 当实例被创建，就会解析组件的属性依赖。 :information_source: **属性时如何注入的:** 了解默认组件激活器如何将依赖注入到属性，点击[这里](how-properties-are-injected.md).
* 当组件创建完成后， 调用组件的所有[commission concerns](lifecycle.md)。
* 在核心（kernel）触发`ComponentCreated` 事件。
* 返回实例到生命期方式管理器。

## 处理器，发布政策和容器都做了什么

生命期方式管理器将会在需要的时候将实例保存到上下文缓存中，之后就可以重用，并将实例传递给处理器。如果允许和需要，处理器调用[发布政策](release-policy.md)以跟踪组件，然后将组件传递给容器，容器将组件返回给用户。

## 还可以看看

* [依赖是怎样解析的](how-dependencies-are-resolved.md)