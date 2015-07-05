# 释放策略（Release Policy）

为了正确的管理组件的[生命周期](lifecycle.md)，Windsor 使用释放策略，就是负责保持对 Windsor 创建的组件的跟踪并且在需要的时候释放它们。

其约定由`IReleasePolicy`接口定义，并能够通过`IKernel`的`ReleasePolicy`属性访问。

```csharp
var policy = container.Kernel.ReleasePolicy;
container.Kernel.ReleasePolicy = someOtherPolicy;
```

:warning: **不要改变工作中容器的释放策略:** 尽管 Windsor 允许你改变其释放策略，你也绝不应该在已有组件被解析之后去改变。如果你这样做 Windsor 将不能正确的释放组件。如果你打算改变策略，将其作为容器的第一个操作，在解析任何组件之前。

## 默认策略

默认情况下 Windsor 使用 `LifecycledComponentsReleasePolicy`，其将会保持跟踪创建的所有组件，并在释放它们的时候，调用它们的退役生命周期步骤。

:information_source: **总是释放组件:** 当释放策略跟踪你的组件的时候，GC不能回收它们。这就是为什么总是释放你解析(不管通过调用 `Resolve`/`ResolveAll` 还是通过 [强类型工厂](typed-factory-facility.md))的对象至关重要的原因，特别是那些没有被自动释放的。对于 [transient](lifestyles.md#transient) 组件尤其是如此，因为除非你释放它们，所有它们的实例将会保留，直到你回收容器。

## `NoTrackingReleasePolicy`

如果你不希望 Windsor 跟踪你的组件，可以使用 `NoTrackingReleasePolicy`。它不会跟踪创建的组件，也不会处理组件退役。通常不建议这样使用，其针对的是整合遗留系统或外部框架等不允许你正确释放组件的有限场景。

## 还可以看看

* [生命周期](lifecycle.md)

## 外部资源

* [(Feb 19, 2010) Blog post by Davy Brion discussing hybrid release policy for integration with external framework](http://davybrion.com/blog/2010/02/avoiding-memory-leaks-with-nservicebus-and-your-own-castle-windsor-instance/)
