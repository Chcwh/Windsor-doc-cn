# 拦截器

Windsor 可以充分利用 [Castle DynamicProxy](https://github.com/castleproject/Core) 的潜在力量来提供有趣的功能。

:information_source: **了解关于 DynamicProxy 的更多内容：** 对 DynamicProxy 是如何工作的和它的限制有一个充分的了解，在使用这里描述的特征的时候，是非常有用的。查看 [DynamicProxy 文档](https://github.com/castleproject/Core/blob/master/docs/dynamicproxy.md)。

## 如何创建代理

不需要显式指定希望一个组件成为代理。Windsor 将会自动创建代理，如果下面的任何一个条件被满足：

* 有为该组件指定的 [拦截器](https://github.com/castleproject/Core/blob/master/docs/dynamicproxy.md)（即包含通过[组件拦截器选择器](model-interceptors-selectors.md)动态选择的拦截器）。
* 有为该组件指定的 mixins
* 有为该组件指定的其他接口

:information_source: **其他情况：** 也有这些条件间接变为真的其他情况。比如[强类型工厂](typed-factory-facility.md)的工作原理是隐式创建代理。其他设施也可以随意修改组件，使它们被代理。

## 指定拦截器

你可以用这些方式指定拦截器和其他代理选项：

* 使用 [Fluent 注册 API](registering-interceptors-and-proxyoptions.md)
* 使用 [XML 配置](registering-components.md)
* 使用 Attributes (just for interceptors)

## `InterceptorAttribute`

如果想显式将拦截器关联到组件，可以通过 `InterceptorAttribute` 实现。

```csharp
public interface IOrderRepository
{
   Order GetOrder(Guid id);
}

[Interceptor("cache")]
[Interceptor(typeof(LoggingInterceptor))]
public class OrderRepository: IOrderRepository
{
   Order GetOrder(Guid id)
   {
      // some implementation
   }
}
```

关于特性的一些注意事项：

* 将特性放在组件类上，而不是接口。
* 可以通过类上面的多个特性指定多个拦截器。
* 可以通过类型或名称指定拦截器。

:warning: **`InterceptorAttribute` 和 类：** `InterceptorAttribute` 可以放在类上，也可以放在其他允许自定义属性的目标上。但是 Windsor 将会忽略特性类，除非特性是在组件的实现类上。这种许可行为允许用户通过创建自定义扩展，为其他目标增加支持。

## 使用拦截器

为了使用拦截器，需要像其他服务一样注册到容器。

```csharp
container.Register(
   Component.For<LoggingInterceptor>().Lifestyle.Transient,
   Component.For<CacheInterceptor>().Lifestyle.Transient.Named("cache"),
   Component.For<IOrderRepository>().ImplementedBy<OrderRepository>());
```

当你解析 `IOrderRepository` 时，Windsor 将会首先创建一个代理，然后使用这两个拦截器去拦截它的成员的调用。

:information_source: **拦截器应该是临时的：** 强烈建议总是让拦截器是临时的。因为拦截器可以拦截多个使用不同生命期方式的组件，最好是让拦截器自身的寿命不超过它们拦截的组件。因此除非有充足的理由不这么做，总是让拦截器是临时的。

## `IOnBehalfAware` 接口

有时为了正确工作，拦截器不仅需要来自 `IInvocation` 的信息，也需要来自它拦截的组件的 `ComponentModel` 的信息。For example it wants to cache some information about the component upfront before intercepting any call, or it needs some information from extended properties of the component in order to do its job.

在这种情况下。拦截器需要实现 `IOnBehalfAware` 接口，它只有一个方法。

```csharp
void SetInterceptedComponentModel(ComponentModel target);
```

在实例化拦截器并将其连接到组件的时候，Windsor 将会检查是否有选定组件的任何拦截器，实现了 `IOnBehalfAware` 接口，并为那些拦截器调用 `SetInterceptedComponentModel` 传递正在创建的组件的 `ComponentModel` 。

## Mixins

可以通过 Fluent API 或 XML 配置指定mixins。

:information_source: **匹配生命期类型:** When mixing components keep in mind their lifestyle. Same rules apply as when building direct dependencies between components. That's why it's best if lifestyles of mixed components match.

## 其他接口

可以通过 Fluent API 或 XML 配置指定其他接口。
