# 传递参数

尽管应用中的大部分组件都依赖其他组件，但并不总是如此。同时 Windsor 用来查找满足依赖的正确组件的默认规则有时不得不进行调整。

如何做到这一点基于值的来源和获取值的地方。

## Composition root - `container.Resolve`

`container.Resolve` 方法有几个重载允许传递 `IDictionary` 作为参数（在这种情况下，建议使用 `[Arguments]` 类），或者匿名对象作为命名参数。

:warning: **不要直接引用容器：** 建议抵抗这种到处传递容器的诱惑。仅在解析根组件时，使用容器（见[三步调用](three-calls-pattern.md)）。在其它情况下，尽量使用其它方式。

:information_source: **不会传播内联依赖：** 无论给 `Resolve` 方法传递任何参数，都只会在解析根组件，和它的[拦截器](interceptors.md)时生效。所有之后的组件（根的依赖，以及依赖的依赖等等）都不能访问它们。

这种方法对仅在composition root内生效的参数非常有用，比如 `Program.Main` 方法。虽然一开始看起来很简单，通常在事实上被 Windsor 新手使用，但是它的适用性非常有限，通常另外两种方式使用得更多。

### 示例 - 控制台应用中的命令行参数

```csharp
// 为简洁起见，其余代码被省略
public void Main(string[] args)
{
   var serverAddress = GetServerAddress(args);
   var container = BootstrapContainer();

   var serverMonitor = container.Resolve<IServerMonitor>(new Arguments(new { serverAddress }));

   DoSomethingUseful(serverMonitor);
}
```

## 注册时 - `DependsOn` 和 `DynamicParameters`

在依赖的值可以在注册的时候知道的时候，或在注册时已经有了所有在以后的调用时所需要的信息，你可以直接插入注册 API ，并在那里提供值，不需要调用时显式信息。查看[这里]获取更多信息(inline-dependencies.md) 。这种方法可以为非服务（原始的）依赖，以及指定非默认组件以满足服务依赖。在[这里](inline-dependencies.md)获取详细信息和其他方法（该 API 比 appSettings 使用得更多）的描述。

### 示例 - 来自配置文件的依赖值

```csharp
// 在 Installer 类里面注册 IServiceMonitor
public void Install(IWindsorContainer container, IConfigurationStore store)
{
   container.Register(Component.For<IServerMonitor>()
      .ImplementedBy<PingServerMonitor>()
      .LifestyleTransient()
      // 'serverAddress' 依赖的值来自 .config 文件的名为 'serverAddress' 的 appSettings 值。
      .DependsOn(Dependency.OnAppSettingsValue("serverAddress"))
}
```

## 分解（Resolution）时 - 强类型工厂

在其他两个不可用时，只能使用第三个选项了。即在前期不知道值。比如（使用服务监控示例）当要监控的服务地址由用户在应用 UI 上键入时。在这种情况下，使用一个[强类型工厂](typed-factory-facility.md) pull 一个新的服务监控，通过工厂传递值。

### 示例 - 强类型工厂

```csharp
//工厂接口
public interface IServerMonitorFactory
{
   IServerMonitor Open(string serverAddress);
   void Close(IServerMonitor monitor);
}
//在另一个组件中，比如 ServerMonitorController。剩余代码已被省略；
IServerMonitorFactory factory;

void StartMonitoring(string addressToMonitor)
{
   var monitor = factory.Open(addressToMonitor);
   DoSomethingUseful(monitor);
}
```

## 还可以看看

* [依赖是如何解析的](how-dependencies-are-resolved.md)
* [内联依赖（通过注册 API）](inline-dependencies.md)
* [XML 内联依赖](xml-inline-parameters.md)
* [参数](arguments.md)
* [强类型工厂](typed-factory-facility.md)
