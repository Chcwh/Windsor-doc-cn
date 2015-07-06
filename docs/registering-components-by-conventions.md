# 按照约定注册组件

## 一次注册多个类型

[一个一个注册组件](registering-components-one-by-one.md) 是非常机械的工作。 同时要记住注册每个添加的新类型令人沮丧。幸运的是，在大多数情况下，你不需要，也不应该这样做。  通过使用 `Classes` 或 `Types` 入口类可以根据您指定的某些特定的特性进行组注册。你将发现这是使用 Windsor 编写应用程序时，使用得最多的方式。

## 三个步骤

通常使用下面的方式进行多个类型注册:

```csharp
container.Register(Classes.FromThisAssembly()
    .InSameNamespaceAs<RootComponent>()
    .WithService.DefaultInterfaces()
    .LifestyleTransient());
```

你可以看到在注册调用中有三个不同的步骤。

### 选择程序集

第一步是为Windsor指出它可以扫描的程序集。你可以这样做:

```csharp
Classes.FromThisAssembly()...
```

或者它的一个重载。

:information_source: **我应该使用 `Classes` 还是 `Types`？:** 有两种方式可以开始按约定注册. 一是使用 Classes 静态类，像上面的例子那样。二是使用 Types 静态类。它们都公开了相同的方法。它们之间的区别是， Types 允许你从给定的程序集注册所有 (准确的说，默认设置，所有公共) 类型，包括类，接口，结构体，委托和枚举。 Classes 会预先过滤类型，以便只考虑非抽象类。 你大多数时候应该使用 Classes , 但是 Types 在某些高级场景里非常有用，比如[基于接口的强类型工厂](typed-factory-facility-interface-based.md)的注册。

:warning: ** `AllTypes` 呢？:** Both `Classes` 和 `Types` 都是在 Windsor 3 才有的. 以前的版本只有一个类做这项工作 - `AllTypes`。在 Windsor 3, 不建议使用 `AllTypes` 了, 因为它的名字有歧义。看起来它应该像 `Types`一样工作, 实际上它和 `Classes` 一样, 进行了预先过滤以便只有非抽象类. 为了避免混淆，我们使用了两个新的类型。

### 基于类型/约定选择

一旦你选择了一个程序集你第一件事是过滤出你想要注册的类型。通过以下的方式之一，可以缩小对象的范围：

1. 通过基类/实现的接口，例如:
  * `Classes.FromThisAssembly().BasedOn<IMessage>()`
1. 通过命名空间, 例如:
  * :information_source: T这个方法 (和它的重载) 在 Windsor 3 中加入
  * `Classes.FromAssemblyInDirectory(new AssemblyFilter("bin")).InNamespace("Acme.Crm.Extensions")`
1. 通过约定, 例如:
  * `Classes.FromAssemblyContaining<MyController>().Where( t=> Attribute.IsDefined(t, typeof(CacheAttribute)))`
1. 没有限制的方式:
  * `Classes.FromAssemblyNamed("Acme.Crm.Services").Pick()`

### 额外的过滤和配置

一旦你选择了源类型和基本条件，你还可以配置类型或额外过滤掉一部分。API的所有细节将在下面讨论。

:warning: **`BasedOn`, `Where` 和 `Pick` 进行逻辑 `或` 运算:** 在多次使用 `BasedOn`, `Where` 和 `Pick` 时，应谨慎一些. 如下所示:

```csharp
container.Register(
    Classes.FromThisAssembly()
        .BasedOn<IMessage>()
        .BasedOn(typeof(IMessageHandler<>)).WithService.Base()
        .Where(Component.IsInNamespace("Acme.Crm.MessageDTOs"))
);
```

将会注册所有 messages 与 所有 message handlers 与 所有 message DTOs。这通常不是你想要的行为，为了避免混淆，在 Windsor 3 中此调用链将会给你编译警告. 应在3个独立的调用中注册这3个组件集合。

#### 注册给定类型的所有派生类 (例如 MVC 应用中的所有控制器)

这里有一个 Monorail 配置示例:

```csharp
container.Register(
    Classes.FromThisAssembly()
        .BasedOn<SmartDispatcherController>()
        .Configure(c => c.Lifestyle.Transient)
);
```

我们从执行中的程序集中注册了所有实现了 `SmartDispatcherController` 的类型，这是注册你的所有控制器的快捷方式。 当你在应用中添加新控制器时，将会自动注册。

#### 默认服务

请记住 `Of` 和 `Pick` 以及其他过滤方法一样都只会缩小我们想要注册的类型集合。 **They do not specify the service that these types provide** and unless you specify one the default will be used, that is the implementation type itself. 换句话说，上面的注册将会注册所有派生自 `SmartDispatcherController` 的类型作为它们自身类型的服务, 而不是 `SmartDispatcherController`， 因此这样调用的话：

```csharp
var controller = container.Resolve<SmartDispatcherController>();
```

将会抛出异常。

这个例子中，类型只能通过它们的实现类型请求(默认服务)。

```csharp
var controller = container.Resolve<MyHomeController>();
```

## 为组件选择服务

默认情况下，组件的服务是它们自身。有时这还不够。Windsor 允许你显式指定服务。

### `Base()`

```csharp
container.Register(
    Classes.FromThisAssembly()
        .BasedOn(typeof(ICommand<>)).WithService.Base(),
    Classes.FromThisAssembly()
        .BasedOn(typeof(IValidator<>)).WithService.Base()
);
```

这里我们注册所有实现 ICommand<> 和 IValidator<> 相应版本的类型，分别的 (现在，再读一遍，慢慢的)。举个例子 - 当我们调用 ICommand<AddCustomer> 时， 我们将会获得一个实现了 ICommand<AddCustomer> 的实例， most likely named something like `AddCustomerCommand`。

这个约定最重要的概念, 也是最容易混淆的, 是 `WithService.Base()` 的存在。 这告诉注册策略选择接口规范的最近版本作为服务类型。在这个例子中，将会选择类似  `IValidator<Customer>` 的。没有这个，类型将会被注册为接口的开放版本，并且你可能不会获得需要的解析精确度。（待定）

### `DefaultInterfaces()`

:information_source: `DefaultInterface` 在 Windsor 3 被改成了这个名字，以 表明它可以为一个组件匹配多个服务

```csharp
container.Register(
    Classes.FromThisAssembly()
        .InNamespace("Acme.Crm.Services")
        .WithService.DefaultInterfaces()
);
```

这个方法根据类型名称和接口名称进行匹配。通常你的接口和实现成对出现，就像这样: `ICustomerRepository`/`CustomerRepository`, `IMessageSender`/`SmsMessageSender`, `INotificationService`/`DefaultNotificationService`。这种情况下你可能希望使用 `DefaultInterfaces` 方法来匹配你的服务。 它将会检查选定类型的实现的所有接口，并将那些名称匹配的接口作为类型的服务。名称匹配，意味着实现类的名称包含接口的名称 (没有前面的 `I`)。

### `FromInterface()`

另一种常见的情况是需要注册实现了某接口的所有类型， but are otherwise unrelated.

```csharp
container.Register(
    Classes.FromThisAssembly()
        .BasedOn<IService>().WithService.FromInterface()
);
```

这里我们从执行程序集注册服务类。在这个例子中,  `IService` 接口可能是一个指定组件在系统中的角色的标记接口。不像 `WithService.Base()`, 这个注册选择的服务类型是扩展了 `IService` 接口的类型。下面是一个例子帮助说明这一点。

比方说你有一个标记接口 `IService` 标记您程序集的所有服务。

```csharp
public interface IService {}

public interface ICalculatorService : IService
{
     float Add(float op1, float op2);
}

public class CalculatorService : ICalculatorService
{
     public float Add(float op1, float op2)
     {
         return op1 + op2;
     }
}
```

上面的注册等同于

```
container.Register(
    Component.For<ICalculatorService>().ImplementedBy<CalculatorService>()
);
```

正如你所看到的，实际的服务接口不是 `IService`，而是派生自 `IService` 的接口, 这里是 `ICalculatorService`。

### `AllInterfaces()`

当一个组件实现多个接口并且你想将其作为这些接口的服务，使用 `WithService.AllInterfaces()` 方法。

### `Self()`

要明确注册该组件实现类型作为服务，使用 `WithService.Self()`。

### `Select()`

如果上面的办法都不适合你，你可以提供自己的选择逻辑作为委托，并将其传递给 `WithService.Select()` 方法。

:information_source: **服务是累计的:** 多次调用 `WithService.Something()` 是允许的，这样它们就是累计的。意思是如果你这样调用:

```csharp
Classes.FromThisAssembly()
   .BasedOn<IFoo>()
   .WithService.Self()
   .WithService.Base()
```

匹配的类型将会注册为 IFoo，和它们自身。  换句话说上面的用法与对每个实现了 `IFoo` 的类型调用如下方法是等价的

```csharp
Component.For<IFoo, FooImpl>().ImplementedBy<FooImpl>();
```

### 注册非公开类型

默认情况下只有在程序集外面能够访问的方法会被注册。如果你需要包括非公开类型，你需要首先指定程序集，然后调用 `IncludeNonPublicTypes` 方法

```csharp
container.Register(
    Classes.FromThisAssembly()
        .IncludeNonPublicTypes()
        .BasedOn<NonPublicComponent>()
);
```

:warning: **不要暴露非公开类型:** 通过容器暴露那些不可用类型不是一个好主意。通常它们因为某些原因不公开。在使用该选项时请三思。

### 配置注册

当你注册多个组件时，你可能会为每个组件的相同属性设置值。对此，你使用Configure()方法。最常见的例子是设置组件的生命期模式（lifestyle），而不是使用默认的单例模式。

```csharp
container.Register(
    Classes.FromAssembly(Assembly.GetExecutingAssembly())
        .BasedOn<ICommon>()
        .Configure(component => component.LifestyleTransient())
);
```

这种用法是如此常见以至于 Windsor 3 为它提供了简易方式。

:information_source: 下面的方法和它的重载是 Windsor 3 新增的

```csharp
container.Register(
    Classes.FromAssembly(Assembly.GetExecutingAssembly())
        .BasedOn<ICommon>()
        .LifestyleTransient()
);
```

除了指定生命期类型，你可以设置其他的配置选项:

```csharp
container.Register(
    Classes.FromAssembly(Assembly.GetExecutingAssembly())
        .BasedOn<ICommon>()
        .LifestyleTransient()
        .Configure(component => component.Named(component.Implementation.FullName + "XYZ"))
);
```

在这里我们注册实现了 `ICommon` 的类, 并设置它们的生命期模式和名称。

你可以做一些更细的配置，为你的组件的子集设置一些其他属性：

```csharp
container.Register(
    Classes.FromThisAssembly()
        .BasedOn<ICommon>()
        .LifestyleTransient()
        .Configure(
            component => component.Named(component.Implementation.FullName + "XYZ")
        )
        .ConfigureFor<CommonImpl1>(
            component => component.DependsOn(Property.ForKey("key1").Eq(1))
        )
        .ConfigureFor<CommonImpl2>(
            component => component.DependsOn(Property.ForKey("key2").Eq(2))
        )
);
```

在这里, 我们做了和上面一样的事情, 但是对两个实现了其他接口的类型设置了额外的内联依赖。

:information_source: See the "Conditional component registration" section below for a discussion on better filtering the types you register.
