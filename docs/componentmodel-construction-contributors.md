# ComponentModel construction contributors 组件模型构造支持器

[组件模型](componentmodel.md) 构造支持器是实现了 `IContributeComponentModelConstruction` 接口的对象。顾名思义，在[`组件模型`](componentmodel.md)创建之后，它们其将正确的构造到最终状态。 

:warning: **不要在支持器的外面修改 `ComponentModel`:** 不鼓励在构造支持器的外面修改组建模型。在 `组建模型` 被它的构造支持器处理完成之后，它应该是只读的。 在以后的任何时候修改都可能导致并发（concurrency）问题或其他难以追查的问题。

##  `IContributeComponentModelConstruction` 接口

组件模型构造支持器需要实现这个方法：

```csharp
void ProcessModel(IKernel kernel, ComponentModel model);
```

根据来自其他支持器，核心，模型的配置或状态的信息，支持器检查或修改 `模型` 参数。Windsor 使用内置支持器设置代理（proxying），参数（parameters），生命期方式（lifestyles），生命周期步骤（lifecycle steps,），依赖等。

### 自己的实现

编写自己的支持器是扩展/自定义 Windsor 最常用的方式。比方说我们希望所有组件的所有类型为 `ILogger` 的属性是必须的（Windsor 中的属性依赖默认情况下是可选的）。为此我们可以写一个支持器，像下面那样：

```csharp
public class RequireLoggerProperties : IContributeComponentModelConstruction
{
    public void ProcessModel(IKernel kernel, ComponentModel model)
    {
        model.Properties
            .Where(p => p.Dependency.TargetType == typeof(ILogger))
            .All(p => p.Dependency.IsOptional = false);
    }
}
```

支持器扫描每个组件的所有属性依赖，寻找类型为 `ILogger` 的，并将其标记为必须的。

### 插入支持器

在创建支持器的时候，需要将其添加到容器的支持器集合 `ComponentModelBuilder` ：

```csharp
container.Kernel.ComponentModelBuilder.AddContributor(new RequireLoggerProperties());
```

### 外部资源

* [Blog post by Mark Seemann (Apr 26, 2010)](http://blog.ploeh.dk/2010/04/26/ChangingWindsorLifestylesAfterTheFact.aspx)
* [Blog post by Ayende (Mar 11, 2007)](http://ayende.com/Blog/archive/2007/03/11/AOP-With-Windsor-Adding-Caching-to-IRepositoryT-based-on-Ts.aspx)
