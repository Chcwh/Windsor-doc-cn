# 生命周期

## 简介

组件生命周期简化图：

![](images/lifecycle-simplified.png)

从广义上说，Windsor 是一个控制组件创建和销毁的工具。 总的来看，组件的生命周期包含三步：

* 创建 - 所有的事情都在 `container.Resolve` 或类似的方法内发生 (查看[组件是如何创建的](how-components-are-created.md)以了解详情)。
* 使用 - 在你的代码中使用组件完成工作。
* 销毁 - 所有的事情在`container.ReleaseComponent`之内/或之后或组件的生命期范围结束时发生。

Windsor 允许你插入到生命周期管道（创建和销毁）中，并调用一些额外的逻辑，无论是外部，还是内部的组件，通过使用lifecycle concerns （实现两个`ILifecycleConcern`派生接口的对象）。

## Creation - commission concerns

在组件创建期间执行的生命周期concerns被称为commission concerns。它们都实现了`ICommissionConcern`接口。它们在组件实例化和所有依赖关联完成后执行。有几种标准方式将一个commission concern挂接到组件。

### `OnCreate` 方法

当你使用[Fluent 注册 API](fluent-registration-api.md)注册组件时，你可以使用它的[`OnCreate`](fluent-registration-api.md#oncreate)方法挂接附加逻辑，其将会当做一个commission concern执行。

```csharp
container.Register(
   Component.For<IService>()
      .ImplementedBy<MyService>()
      .OnCreate((kernel, instance) => instance.Timestamp = DateTime.UtcNow)
   );
```

### `IInitializable` 接口

有一对接口在被你的组件实现时，会受到 Windsor 的特殊处理。其中一个是`Castle.Core.IInitializable`，它只有一个方法：

```csharp
void Initialize();
```

当 Windsor 实例化实现这个接口的组件时，它会在组件commission期间调用 `Initialize` 方法。

```csharp
public class InitializableComponent : IInitializable
{
   public DateTime Timestamp { get; private set; }

   public virtual void Initialize()
   {
      Timestamp = DateTime.UtcNow;
   }
}
```

### `ISupportInitialize` 接口

如果你不想在你的领域中引用 Castle 程序集，但是仍然想从 Windsor 的生命周期管理中获益，你可以实现另一个接口：

`System.ComponentModel.ISupportInitialize` 是 BCL 的一部分，它有两个方法：

```csharp
void BeginInit();
void EndInit();
```

当 Windsor 实例化实现这个接口的组件时，它会在组件commission期间调用 `BeginInit` 方法。

```csharp
public class InitializableComponent: ISupportInitialize
{
   public DateTime Timestamp {get; private set;}

   public virtual void BeginInit()
   {
      Timestamp = DateTime.UtcNow;
   }

   public virtual void EndInit()
   {
   }
}
```

## 销毁 - decommission concerns

在组件销毁期间执行的Lifecycle concerns被称为decommission concerns。它们都实现了`IDecommissionConcern`接口。它们在组件从容器释放时执行，即将会在通过`container.ReleaseComponent`方法释放时，容器销毁时，或组件的生命期范围结束时（比如Web请求）发生。

### `OnDestroy` 方法

这个方法类似于 `OnCreate` ，允许你指定临时 decommission concerns （代码将会在组件释放时执行）。

```csharp
container.Register(Component.For<MyClass>()
   .LifestyleTransient()
   .OnDestroy(myInstance => myInstance.ByeBye())
);
```

注意如果你的类实现了  `IDisposable`，在你自定义的销毁函数调用之前，将会自动在对象上调用 `Dispose`。

:warning: **实例跟踪和 `OnDestroy`:** 注意为了decommission对象，Windsor需要跟踪它。在管理组件实例的使用时，要注意确保在不需要的时候释放组件。 、

###  `IDisposable` 接口

`IDisposable`接口是.NET中的标准decommission方法，也受到 Windsor 的支持。Windsor 创建一个实现了 `IDisposable` 的组件时，它将会在释放组件时调用组件的 `Dispose` 方法。

:warning: **Windsor 跟踪组件:** 为了确保正确decommission组件，Windsor 持有它创建的每个组件的引用*。这就是为什么释放组件极其重要。否则你可能不得不面对内存消耗增加。

:information_source: **生命周期和释放策略:** 上述声明是不是100％准确。 [释放策略](release-policy.md)可以免除组件跟踪。然后，你失去了执行组件正确销毁的能力，通常不建议这样做。

## Custom lifecycle concerns

Windsor 的生命周期不限于上面提到的 concerns。组件的生命周期像 Windsor 里的所有东西一样都是可以扩展的，你可以用自己的concerns扩展它。在 Windsor [可启动设施](startable-facility.md) 里面使用组件生命周期concerns完成它的工作。

### Writing your own

生命周期concerns需要实现以下两个接口之一：`Castle.Core.ICommissionConcern` 或 `Castle.Core.IDecommissionConcern`。接口暴露了一个方法：

```csharp
void Apply(ComponentModel model, object component)
```

第一个参数时组件模型，第二个是它的实例。

### 附加生命周期 concerns

附加自定义的生命周期concern到你感兴趣的组件的`ComponentModel`，使用下面的代码：

```csharp
model.Lifecycle.Add(new MyCommissionConcern());
model.Lifecycle.Add(new MyDecommissionConcern());
```

:warning: **Use `ComponentModel` construction contributor:** 由于附加生命周期concerns是修改`ComponentModel`的操作，你应该在 [ComponentModel construction contributor](componentmodel-construction-contributors.md)内附加。
