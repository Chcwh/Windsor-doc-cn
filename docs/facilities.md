# 设施（Facilities）

设施是扩展容器的主要方式。使用设施可以将外部框架与容器集成，比如 WCF 或 NHibernate，为容器添加新的功能，比如事件连接（event wiring），事务支持（transaction support）……，或为组件（synchronization, startable semantics...）。

## 如何使用设施

在使用设施前需要将其注册到容器，通过代码（如下所示）或者[XML 配置](facilities-xml-configuration.md)。作为用户，通常这就是你需要做的全部工作。某些设施，特别是 WCF 设施可能有额外的 API，从容器对象分离，但这是具体到某一设施的东西。Some facilities, most notably WCF facility may have also additional API, detached from the container object but that's something specific to a given facility.

```csharp
container.AddFacility<TypedFactoryFacility>();
```

一些设施可能需要配置，这时使用下面的重载方法：

```csharp
container.AddFacility<StartableFacility>(f => f.DeferredTryStart());
```

:warning: **在启动时添加设施：** 为了正常工作，大多数设施都需要在它们影响的组件之前注册。在编写注册代码时牢记这一点，因为忘记这样做可能导致难以发现的问题（比如可启动组件没有启动）。

## 标准设施

在容器的程序集 `Castle.Windsor.dll` 中你可以找到如下设施。注意许多设施都由它们自己的程序集提供，但是依然是 Castle 项目的一部分。许多外部项目都提供了与 Windsor 集成的设施。

* [强类型工厂设施（Typed Factory Facility）](typed-factory-facility.md) - 为工厂类提供自动实现，你可以在代码中使用来创建对象，而不需要引入容器依赖。Provides automatic implementation for factory classes, that you can use in your code to create objects on demand without introducing dependency on the container.
* [可启动设施（Startable Facility）](startable-facility.md) - 提供'开始' 和 '停止'对象的能力。对像 WCF 服务等你希望在应用启动时尽早启动的服务非常有用。

## 其它设施

除了上述作为 Castle 项目一部分的之外，还提供了一些其它设施，多数用于集成 Windsor 和其他框架。它们也有一些非常强大的功能，可以显著简化你的工作。

* [WCF 集成设施](wcf-facility.md) - 提供与 WCF 的集成。它大大的简化了 WCF 服务的配置，让你轻松扩展它们，使用非默认构造函数，在不必求助于代码生成的情况下调用异步服务等等。
* [日志设施](logging-facility.md) - 大多数应用使用日志。该设施允许你轻松的将记录器注入到组件中。它提供了最流行的第三方日志框架的集成，如[NLog](http://nlog-project.org/) 和 [log4net](http://logging.apache.org/log4net/)。
* [工厂支持设施](factory-support-facility.md) - 提供由工厂创建组件的能力。你可以使用它在容器中注册类似 `HttpContext` 的东西。
  * :information_source: **推荐 `UsingFactoryMethod`:** 该设施通常用于向后兼容（XML 配置），不推荐在新应用中使用。建议使用 [`UsingFactoryMethod`](registering-components-one-by-one.md#using-a-delegate-as-component-factory)。将来的版本可能废弃（obsolete）工厂支持设施。
* [事件连接设施](event-wiring-facility.md) - 提供连接公开事件类与消费事件类的能力。
* [远程（Remoting）设施](remoting-facility.md) - 提供使用.NET Remoting 公开或消费其他 `AppDomain` 的组件的能力。
* [活动记录（ActiveRecord）集成设施](activerecord-integration-facility.md) - 活动记录集成设施维护 [Castle ActiveRecord](https://github.com/castleproject/ActiveRecord) 的配置和启动，并增加了声明式事务支持的集成（declarative transaction support integration）。
* [NHibernate 集成设施](nhibernate-facility.md) - 如果你使用 NHibernate, 而不是 [Castle ActiveRecord](https://github.com/castleproject/ActiveRecord)，该设施提供了两个组件之间的良好集成。
* [同步（synchronize）设施](synchronize-facility.md) - 集成 .NET 框架的同步元素（如 `ISynchronizeInvoke` 接口，`SynchronizationContext`），确保继承自 `Control` 的组件在 UI 线程上创建。
* [自动事物管理设施](atm-facility.md) - 该组件管理事务的创建及相关的提交或回滚，基于是否有异常抛出。事务是逻辑事务。It is up the other integration to be transaction aware and enlist its resources on it.
* [MonoRail 集成设施](https://github.com/castleproject/MonoRail/blob/master/MR2/docs/windsor-integration.md) - 提供与 MonoRail 控制器和内部服务的集成。

## 第三方设施

:information_source: **更多设施:** 设施是与 Windsor 容器集成的主要方式。Multiple other projects, like MVC Contrib, OpenRasta, NServiceBus to name just a few offer their own ready to use facilities that help you use these frameworks with Windsor.

这里是其他项目为 Windsor 提供的设施的部分列表。

:information_source: 如果你知道其它项目提供了设施，将它们添加到下表。 

* [ASP.NET MVC](http://mvccontrib.codeplex.com/)
* [WCF Session Facility](http://www.sharparchitecture.net/) - Part of [Sharp Architecture project](http://www.sharparchitecture.net/), this facility may be registered within your web application to automatically look for and close WCF connections.  This eliminates all the redundant code for closing the connection and aborting if any appropriate exceptions are encountered.
* [Workflow Foundation](http://using.castleproject.org/display/Contrib/Castle.Facilities.WorkflowIntegration)
* [NServiceBus](http://ayende.com/Blog/archive/2008/07/13/Putting-the-container-to-work-Refactoring-NServiceBus-configuration.aspx)
* [MassTransit](http://code.google.com/p/masstransit/source/browse/tags/0.5/Containers/MassTransit.WindsorIntegration/)
* [re-motion](https://www.re-motion.org/blogs/mix/archive/2009/01/21/we-have-a-facility.aspx)
* [Rhino Service Bus](http://hibernatingrhinos.com/open-source/rhino-service-bus/config)
* [Rhino Security](http://bartreyserhove.blogspot.com/2008/08/rhinosecurity-in-practice.html)
* [Quartz.Net](http://github.com/castleprojectcontrib/QuartzNetIntegration) - Provides integration with [Quartz.Net](http://quartznet.sourceforge.net/) jobs and listeners.
* [SolrNet](http://code.google.com/p/solrnet/wiki/Initialization)
* [SolrSharp](http://bugsquash.blogspot.com/2008/07/castle-facility-for-solrsharp.html)
* [Windows Fax Services](http://bugsquash.blogspot.com/2008/01/castle-facility-for-windows-fax.html)
* [App Config Facility](https://github.com/adamconnelly/WindsorAppConfigFacility)

### 还可以看看

* [组件模型构造支持器](componentmodel-construction-contributors.md)
