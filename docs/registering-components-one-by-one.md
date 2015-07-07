# 一个一个注册组件

## 基础注册示例

在容器中注册任何东西的起点是容器的 `Register` 方法，使用一个或多个 `IRegistration` 对象作为参数。创建那些对象最简单的方式是使用静态类 `Castle.MicroKernel.Registration.Component` 。它的 `For` 方法返回 `ComponentRegistration`，可以用于配置组件注册。

:information_source: **分离你的注册代码:** 分离你的注册代码到实现 [`IWindsorInstaller`](installers.md) 的独立类是一个推荐实践。

:information_source: **先安装框架组件:** 一些组件可能需要设施或其他容器核心的扩展正确的注册。因此推荐你总是在注册你的组件之前，先注册[设施，自定义子系统，组件模型构造支持器等等](extension-points.md) 。

## 在容器中注册类型

```csharp
container.Register(
    Component.For<MyServiceImpl>()
);
```

这将会注册类型 `MyServiceImpl` 作为服务 `MyServiceImpl` ，使用默认生命期类型 (Singleton).

## 注册类型为非默认服务

```csharp
container.Register(
    Component.For<IMyService>()
        .ImplementedBy<MyServiceImpl>()
);
```

注意 `For` 和 `ImplementedBy` 也有泛型重载。

```csharp
// Same result as example above.
container.Register(
    Component.For(typeof(IMyService)
        .ImplementedBy(typeof(MyServiceImpl)
);
```

:information_source: **服务和组件:** 你可以在 [这里] 找到更多关于服务和组件的信息(services-and-components.md)。

## 注册泛型

加入你有一个 `IRepository<TEntity>` 接口， `NHRepository<TEntity>` 是它的实现。

你可以为每一个实体类注册仓储，但这是不必要的。

```csharp
// Registering a repository for each entity is not needed.
container.Register(
    Component.For<IRepository<Customer>>()
        .ImplementedBy<NHRepository<Customer>>(),
    Component.For<IRepository<Order>>()
        .ImplementedBy<NHRepository<Order>>(),
//    and so on...
);
```

一个 `IRepository<>` (也叫开放泛型类) 注册，不需要指定实体，就够了。

```csharp
// Does not work (compiler won't allow it):
container.Register(
    Component.For<IRepository<>>()
        .ImplementedBy<NHRepository<>>()
);
```

这样做是非法的，上面的代码不能编译。你必须使用 `typeof()`

```csharp
// Use typeof() and do not specify the entity:
container.Register(
    Component.For(typeof(IRepository<>)
        .ImplementedBy(typeof(NHRepository<>)
);
```

## 配置组件的生命期方式

```csharp
container.Register(
   Component.For<IMyService>()
      .ImplementedBy<MyServiceImpl>()
      .LifeStyle.Transient
);
```

如果没有显式的指定 [生命期类型](lifestyles.md) ，将会使用默认的 Singleton 生命期类型。

## 为服务注册多个组件

你可以为同一个服务使用多次注册来实现。

```csharp
container.Register(
    Component.For<IMyService>().ImplementedBy<MyServiceImpl>(),
    Component.For<IMyService>().ImplementedBy<OtherServiceImpl>()
);
```

当一个组件有一个 `IMyService` 依赖，它将会默认获得第一个注册的 `IMyService` （这里是 `MyServiceImpl`）。

:information_source: **在 Windsor 里面第一个胜利:** 在 Castle 里面，服务的默认实现是第一个注册的实现。这与 AutoFac 不同，AutoFac 的默认实现是最后一个注册的 ([http://code.google.com/p/autofac/wiki/ComponentCreation](http://code.google.com/p/autofac/wiki/ComponentCreation))。

你可以通过 `IsDefault` 方法强制后面注册的组件成为默认组件。

```csharp
container.Register(
    Component.For<IMyService>().ImplementedBy<MyServiceImpl>(),
    Component.For<IMyService>().Named("OtherServiceImpl").ImplementedBy<OtherServiceImpl>().IsDefault()
);
```

在上面的例子中，任何拥有 `IMyService` 依赖的组件，都会默认获得 `OtherServiceImpl` 的实例，即时它是后面注册的。

Of course, you can override which implementation is used by a component that needs it. This is done with service overrides.

当你显式调用 `container.Resolve<IMyService>()` （不指定名称）时，容器也会返回为 `IMyService` 注册的第一个组件(上面的例子中是 `MyServiceImpl`)。

:information_source: **为duplicated组件提供唯一名称:** 如果你想多次注册同一个实现，确保为注册的组件提供不同的名称。

## 注册存在的实例

可以将存在的对象注册为服务。

```csharp
var customer = new CustomerImpl();
container.Register(
    Component.For<ICustomer>().Instance(customer)
);
```

:warning: **注册实例无视生命期类型:** 当你注册一个已存在的实例，即时你指定了生命期类型，也会忽略。Also registering instance, will set the implementation type for you, so if you try to do it manually, an exception will be thrown.

## 使用委托作为组件工厂

你可以使用委托作为组件的轻量工厂：

```csharp
container
   .Register(
      Component.For<IMyService>()
         .UsingFactoryMethod(
            () => MyLegacyServiceFactory.CreateMyService())
);
```

`UsingFactoryMethod` 方法还有两个重载，可以让你访问核心（kernel）和构造上下文，如果需要的话。

`UsingFactoryMethod` 使用核心重载的例子 (Converter<IKernel, IMyService>)

```csharp
container.Register(
    Component.For<IMyFactory>().ImplementedBy<MyFactory>(),
    Component.For<IMyService>()
         .UsingFactoryMethod(kernel => kernel.Resolve<IMyFactory>().Create())
);
```

`UsingFactoryMethod` 方法之外，还有一个 `UsingFactory` 方法。(没有 "method" 后缀 :-) )。可以看做 `UsingFactoryMethod` 方法的特殊版本，从容器中解析一个存在的工厂，并且让你使用它创建服务的实例。

```csharp
container.Register(
    Component.For<User>().Instance(user),
    Component.For<AbstractCarProviderFactory>(),
    Component.For<ICarProvider>()
        .UsingFactory((AbstractCarProviderFactory f) => f.Create(container.Resolve<User>()))
);
```

:warning: **避免 UsingFactory:** 建议使用 `UsingFactoryMethod`，避免使用 `UsingFactory` 当你通过工厂创建服务时。`UsingFactory` 将是 obsoleted/removed 在未来的版本。

## OnCreate

有时需要检查或修改创建的实例，在它被使用之前。你可以使用 `OnCreate` 方法来做这件事。

```csharp
container.Register(
    Component.For<IService>()
        .ImplementedBy<MyService>()
        .OnCreate((kernel, instance) => instance.Name += "a")
);
```

这个方法有两个重载。一个方法的参数是接收一个 `IKernel` 和新创建实例作为参数的委托，另一个只有新创建的实例。

:information_source: **`OnCreate` 只对容器创建的组件有效:** This method is not called for components where instance is provided externally (like when using Instance method). It is called only for components created by the container. This also includes components created via certain facilities ([Remoting Facility](remoting-facility.md), [Factory Support Facility](factory-support-facility.md))

## 为组件指定名称

注册的组件的默认名称是实现类的全名。你可以使用 Named() 方法指定一个不同的名称。 

```csharp
container.Register(
    Component.For<IMyService>()
        .ImplementedBy<MyServiceImpl>()
        .Named("myservice.default")
);
```

## 使用 (Service override) 为组件提供依赖

如果一个组件需要或想要其他组件，这叫做依赖。当注册的时候，使用Service override可以显式设置要使用的组件。
If a component needs or wants an other component to function, this is called a dependency. 

```csharp
container.Register(
    Component.For<IMyService>()
        .ImplementedBy<MyServiceImpl>()
        .Named("myservice.default"),
    Component.For<IMyService>()
        .ImplementedBy<OtherServiceImpl>()
        .Named("myservice.alternative"),

    Component.For<ProductController>()
        .ServiceOverrides(ServiceOverride.ForKey("myService").Eq("myservice.alternative"))
);

public class ProductController
{
    // Will get a OtherServiceImpl for myService.
    // MyServiceImpl would be given without the service override.
    public ProductController(IMyService myService)
    {
    }
}
```

## 为复数服务注册组件

可以将单个组件用作多个服务。比如你有一个类 `FooBar`，实现了 `IFoo` 和 `IBar` 接口。你可以配置容器，在 `IFoo` 和 `IBar` 被请求时，返回同一个服务。该功能被称为 type forwarding。

## Type forwarding

指定[type forwarding](forwarded-types.md)最简单的办法是使用 `Component.For` 方法的多泛型参数重载。

```csharp
container.Register(
    Component.For<IUserRepository, IRepository>()
        .ImplementedBy<MyRepository>()
);
```

There are overloads for up to four forwarded services, which should always be enough. 如果你发现你需要更多，你很可能违反了单一职责原则（SRP）。你可能需要将你的巨大的组件拆分为多个，每个只做一件事情。

还有一个非泛型重载，需要 `IEnumerable<Type>` 或 `params Type[]` 参数。在你需要开放泛型（open generics）支持或因为各种原因不能使用泛型版本时使用。

此外，你可以使用 `Forward` 方法，它为 `For` 方法公开了相同的行为和重载。

```csharp
container.Register(
    Component.For<IUserRepository>()
        .Forward<IRepository, IRepository<User>>()
            .ImplementedBy<MyRepository>()
);
```

## 提供内联依赖

不是所有的东西都必须是 Windsor 的组件。有些组件需要的参数，如连接字符串，缓冲区大小等等。你可以提供这些参数为[内联依赖](inline-dependencies.md)。

## 还可以看看

* [条件组件注册](conditional-component-registration.md)
* [通过约定注册组件](registering-components-by-conventions.md)
