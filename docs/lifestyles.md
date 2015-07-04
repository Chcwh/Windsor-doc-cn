# 生命期类型（Lifestyles）

假如你有两个类: `HomeViewModel` 和 `ApplicationSettingsViewModel`，都依赖于 `IUserService`，它们是应该拥有IUserService的同一个实例，还是各自拥有自己的实例？当你打开应用的设置窗口，关闭它，然后再次打开，你应该获得`ApplicationSettingsViewModel`的同一个实例还是一个新实例？当组件的一个实例不在需要，谁和如何清理它：如果需要的话进行回收，解除事件聚合器的事件订阅？

答案是 - 看情况。行为取决于组件的生命期类型。生命期类型决定什么情况下实例可重用，什么时候释放（这是必要的清理步骤，然后让GC销毁它）。 

## 标准生命期类型: 常见类型

Windsor 提供了一系列的生命期类型，涵盖了大多数现实场景。

### 单例（Singleton）

单例组件在一个容器中只有一个实例。实例将在第一次请求时被创建，随后每次需要时都进行重用。显式释放单例（通过调用`container.Release(mySingleton)`）什么都不会做。这个实例将会在它注册的容器销毁时释放。
注册单例组件的方式:

```csharp
Container.Register(Component.For<MySingletonComponent>().LifestyleSingleton());
```

:information_source: **默认:** 单例是默认的生命期类型，将会在未显式指定的时候使用。因此上面的注册可以改为: `Container.Register(Component.For<MySingletonComponent>());`

:information_source: **单例有什么好处:** 单例的特性使得它在某些情况下是一个好选择。 无状态组件是一个好的选择。 Components that have state that is valid throughout the lifetime of the application that all the other components may need to access.如果你的应用是多线程的，记得确保单例组件的状态转换是线程安全的。你应该考虑将组件作为单例，特别是对象很大时，这种情况下产生多个实例可能导致不必要的内存消耗。通常情况下，在一些实际应用中，这是个简单和快速的决定。

### Transient

Transient与单例相反。Transient组件不会绑定到任何作用域。每次需要Transient组件的实例时，容器都会产生一个新的实例，而不会重用它们。可以说Transient组件的作用域有它的使用者决定。因此当使用Transient组件的对象释放时，它们也被释放。
如果你的Transient组件是通过`container.Resolve<MyTransientComponent>()`解析的根对象（也就是说，使用它的对象不是由容器管理的），当你不再需要它的时候，应该通过调用`container.Release(myTransientInstance)`结束它的生命周期。

注册transient组件的方法:

```csharp
Container.Register(Component.For<MyTransientComponent>().LifestyleTransient());
```

:warning: **Transient 可能被容器跟踪: 仅`Release` 你 `Resolve`的:** 一些人，特别是过去使用过某些其它容器 ，有时会忘记 Windsor 可能会跟踪transient组件。他们`Resolve`组件，但从不`Release` 它们。为了确保正确的组件生命周期管理 Windsor 可能跟踪这些组件。 这意味着，除非你释放他们，垃圾收集器将无法收回他们，你将最终导致事实上的内存泄漏。所以请记住这个有用的经验法则：记住 `Release` 那些你显式 `Resolve`的。

:information_source: **transient有什么好处:** 当你想控制组件的生命期时，transient是一个好选择。 When you need new instance, with new state every time. 当然 transient 组件不需要是线程安全的，除非你将在多线程条件下使用。在大多数程序中你会发现你大多数的组件最后都是transient的。

### 每次Web请求（PerWebRequest）

组件的实例会在一个单一的Web请求范围内共享。该实例将在Web请求的范围第一次请求时创建。显示释放它是无效的。实例将在web请求结束时释放。

注册组件为每次Web请求:

```csharp
Container.Register(Component.For<MyPerWebRequestComponent>().LifestylePerWebRequest());
```

:warning: **注册 `PerWebRequestLifestyleModule`:** 为了每个Web请求的正常运行需要一个`IHttpModule` - `Castle.MicroKernel.Lifestyle.PerWebRequestLifestyleModule`注册到 web.config 文件中:

```xml
<httpModules>
   <add name="PerRequestLifestyle" type="Castle.MicroKernel.Lifestyle.PerWebRequestLifestyleModule, Castle.Windsor"/>
</httpModules>
```

如果是IIS7 ，可能需要在 `system.webServer/modules` 区段中注册。

```xml
<configuration>
   <system.webServer>
      <modules>
         <add name="PerRequestLifestyle" type="Castle.MicroKernel.Lifestyle.PerWebRequestLifestyleModule, Castle.Windsor" />
      </modules>
   </system.webServer>
</configuration>
```

## 标准生命期类型: 不常见类型

上面讨论的那些生命期类型满足大多数程序的需求。但是偶尔也需要更特定的生命期类型。

### 范围（Scoped）

Windsor 3 added option to specify arbitrary scope of component instance lifetime/reuse. Here's how you do it:
Windsor 3 增加了选项用于指定组件实例的生命期/重用范围。你可以这样做：

```csharp
Container.Register(Component.For<MyScopedComponent>().LifestyleScoped());
```

考虑下面这种情况:

```csharp
using Castle.MicroKernel.Lifestyle;

using (Container.BeginScope()) //extension method
{
	var one = Container.Resolve<MyScopedComponent>();
	var two = Container.Resolve<MyScopedComponent>();
	Assert.AreSame(one, two);

} // releases the instance
```

上面的代码使用using代码块限定重用的范围（范围内需要使用实例时，使用的是同一个实例）和生命期（using代码块结束的时候释放该对象）。

:information_source: **`CallContext` scope:** 对于更好奇的你，范围被绑定到[CallContext](http://msdn.microsoft.com/en-us/library/system.runtime.remoting.messaging.callcontext.aspx)。意思是，实例在当前线程上可用，但也可能传递到线程池和`Task`线程。在使用多线程时，要小心，以确保在using代码块结束之前子操作已经完成。

#### 自定义范围（Custom scopes）

在默认的 `CallContext` 范围之外，你可以将你的组件绑定到你选择的任意范围，通过实现 `Castle.MicroKernel.Lifestyle.Scoped.IScopeAccessor` 接口。(阅读 [实现自定义范围](implementing-custom-scope.md)).

在注册组件时可以指定范围访问器：

```csharp
Container.Register(Component.For<MyScopedComponent>().LifestyleScoped<MyCustomScopeAccessor>());
```

### Bound

请看下图。

![](images/graph-bound.png)

上图中有两个视图模型（view model），一个依赖于另一个，而且它们都依赖一些其他服务，比如说仓储（repository）。你可能会希望将仓储绑定到子图（[数学上的图](https://zh.wikipedia.org/wiki/%E5%9B%BE_%28%E6%95%B0%E5%AD%A6%29)）。换句话说，你可能会希望最外面的视图模型(`WelcomeScreenViewModel`)的整个子图都共享仓储的同一个实例，并且在该视图模型释放时自动释放仓储实例。

这就是bound生命期类型的用法：

```csharp
Container.Register(Component.For<Repository>().LifestyleBoundTo<ViewModelBase>());
```

:information_source: **绑定是在实现类上完成的:** 注意在指定要绑定到的类型时，我们使用`ViewModelBase`作为所有视图模型的基类。绑定不会查找容器暴露的服务，而是组件的实际实现类，并且检查其是否派生自指定的类型（基类）。

:information_source: **Bound生命期类型引入耦合（coupling）:** 这里有一个很重要的方面你需要考虑到，当选择bound生命期类型时。它假定仓储（在我们的例子中）总是被解析为一些视图模型的依赖。这有时不错，有时候却不。在选择这种生命期类型之前，请确保已考虑到这一点。

#### Bound to nearest

:information_source: 这是 Windsor 3.2 中的新功能

在某些情况下，与绑定到图上最远的匹配对象不同，你可能希望绑定到最近的对象。那就是`WelcomeScreenViewModel`将会与它的依赖（直接和间接的）共享`仓储`的实例。但是一旦一个依赖正好是另一个视图模型，该视图模型和它的依赖（直接和间接的）将会获得一个新的`仓储`的实例，除非某个依赖正好又是另一个依赖...... You get the idea。

这种情况下使用`BoundToNearest`注册 `仓储`。 

```csharp
Container.Register(Component.For<Repository>().LifestyleBoundToNearest<ViewModelBase>());
```

#### Bound: 自定义

如果默认选项不能满足你的需求，你可以提供自定义方式以选择要绑定的组件，使用下面`BoundTo`方法的重载：

```csharp
BoundTo(Func<IHandler[], IHandler> scopeRootBinder)
```

You can pass a custom delegate that from the collection of the `IHandler`s representing components in the subgraph (from the outermost to innermost) selects the one you want to bind your component to.

## 标准生命期类型: *极* 少使用的

上面描述的生命期类型，涵盖了99%的应用。读到这里就可以结束了，因为下面介绍的生命期方式你可能从不需要或从未见过。
对于某些极端的情况，Windsor 提供了另外的生命期类型。

### PerThread

组件的实例会在单一执行线程内共享。当在给定线程上第一次请求时创建组件的实例。显式释放组件没什么用。实例将会在其注册的容器销毁时释放。

:information_source: **使用该生命期类型时请三思:** Per thread生命期类型是一种特殊的生命期类型。在使用之前应该三思。它只应该在你的应用控制的线程上使用，绝不要在线程池线程（或`Task`参与时）上使用。如果有疑问，就不要使用。

### Pooled

实例的池将会创建，当请求时返回其中一个。Poolable生命期类型有两个影响其行为的参数 - `initialSize` 和 `maxSize`。

当组件被第一次请求时，将会创建一个实例池，其中包含`initialSize`指定的数量的实例，然后将其中一个在内部标记为*使用中*并返回。当更多的组件被请求时，池将会首先返回那些未处于*使用中*的实例。and if it runs of it will start creating new ones。释放组件时有两种可能。如果池中处于*使用中*的组件数量比`maxSize`还多，那么将直接释放组件。否则组件将会被重用（如果组件实现了`IRecyclable`）并返回池中标记为*允许使用*。

####  `IRecyclable` 接口

Windsor 为poolable组件提供特殊接口 - `Castle.Core.IRecyclable`。它只有一个方法:

```csharp
void Recycle();
```

该方法在组件返回到池中时被调用。组件可以使用它实现自定义初始化/清理逻辑。

## Custom lifestyles

允许你为组件设置自己的`ILifestyleManager`实现。 Also used be some facilities, like WCF Facility which provides two additional lifestyles - per WCF session and per WCF operation.

## Setting lifestyle

指定生命期类型最常用的方式是通过注册API。你可以为单个组件设置生命期类型，像上面的例子那样。在注册类型集合时也能使用同样的方法（method）。
这里有一个如何在注册时将你所有控制器注册为transient的例子：

```csharp
Container.Register(
   Classes.FromThisAssembly()
      .BasedOn<IController>()
      .LifestyleTransient());
```

### Via XML

可以在 [XML 配置](registering-components.md#component-with-lifestyle)中设置生命期类型。

### Via Attributes

Windsor 有一系列特性可以用于设置组件的建议生命期类型。

:information_source: **建议使用其他方式:** 生命期特性用于扩展容器自身的底层组件。对于你的领域服务，最好使用其他方式，这样你就不需要在你的领域中引用容器了。

那些特性在 `Castle.Core` 命名空间中，它们都继承与 `Castle.Core.LifestyleAttribute`。

There's an attribute for each lifestyle described above, each of them named after the lifestyle.
Here's how you mark a type as transient using `TransientAttribute` attribute.

```csharp
[Transient]
public class MyTransientComponent
{
   // something here...
}
```

:information_source: **特性是默认值:** 特性标识组件的建议生命期方式。你可以在Fluent API或者XML配置中显式指定其它的生命期类型。
