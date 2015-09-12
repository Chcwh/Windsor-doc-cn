# 注册拦截器（Interceptors）和代理选项（ProxyOptions）

Windsor 使用 Castle DynamicProxy 公开了丰富的 AOP 能力。

## 拦截器

拦截器是围绕方法的调用注入代码的手段。

使用 Fluent API 可以为指定组件指定拦截器：
```csharp
container.Register(
    Component.For<ICalcService>()
        .Interceptors(InterceptorReference.ForType<ReturnDefaultInterceptor>()).Last,
    Component.For<ReturnDefaultInterceptor>()
);
```

这里我们为一个接口注册服务，但是未提供实现。在这种情况下，拦截器提供 `ICalcService` 的方法的实现。我们使用`InterceptorReference`明确指定拦截器。通过这种方式我们可以指定拦截器在拦截管道上的位置。如果不需要的话，可以使用其他重载：

## 对拦截器进行排序

就像上面提到的那样，你可以控制拦截器应用到组件的顺序，在定义拦截器引用的时候，使用 `.Last`， `.First`或 `.AtIndex()`。如果不关心拦截器排序，使用`.Anywhere`。

## 为指定方法选择指定拦截器

默认情况下，所有拦截器将会作用于每一个可拦截的方法。为了控制哪些拦截器作用于方法，可以使用`InterceptorSelector` （实现 `IInterceptorSelector` 接口的类），如下所示：

```csharp
container.Register(
    Component.For<ICatalog>()
        .ImplementedBy<SimpleCatalog>()
        .Interceptors(InterceptorReference.ForType<DummyInterceptor>())
            .SelectedWith(new FooInterceptorSelector()).Anywhere,
    Component.For<DummyInterceptor>()
);
```

现在每个方法第一次调用的时候，将会首先调用 `FooInterceptorSelector` 以决定哪些拦截器可以生效。

## 代理选项

（在指定拦截器和拦截器选择器的时候，有一些其余的代理相关的选项可供设置）
In addition to specifying interceptors and interceptor selector, there's a number of other proxy-related options you can specify which are exposed via Proxy property. For example, you can use it to specify objects you want to mix in with your service:

```csharp
container.Register(
    Component.For<ICalcService>()
        .ImplementedBy<CalculatorService>()
        .Proxy.MixIns(new SimpleMixIn())
);
```

:information_source: **了解更多关于 *Castle DynamicProxy* :** Castle 中代理和 AOP 是比这里表现出来的那些宽泛得多的主题。学习更多关于支撑 DynamicProxy 的内容，阅读[下面的教程](http://kozmic.net/dynamic-proxy-tutorial/).

## See also

* [Castle DynamicProxy](https://github.com/castleproject/Core/blob/master/docs/dynamicproxy.md)
* [拦截器](interceptors.md)
