# Windsor 安装器（Installers）

## 简介

当使用容器工作时， [需要做的第一件事](three-calls-pattern.md#call-one-bootstrapper) 是注册所有 [组件](services-and-components.md)。 Windsor 使用安装器 (实现 `IWindsorInstaller` 接口的类型) 封装和分离你的注册逻辑，一些帮助类，比如 `Configuration` 和 `FromAssembly` 使得安装工作轻而易举。

## `IWindsorInstaller` 接口

安装器是实现了 `IWindsorInstaller` 接口的简单类型。 该接口只有一个 `Install` 方法。该方法获得容器的实例，然后就可以使用[Fluent 注册 API](fluent-registration-api.md)注册组件：

```csharp
public class RepositoriesInstaller : IWindsorInstaller
{
   public void Install(IWindsorContainer container, IConfigurationStore store)
   {
      container.Register(AllTypes.FromAssemblyNamed("Acme.Crm.Data")
                            .Where(type => type.Name.EndsWith("Repository"))
                            .WithService.DefaultInterfaces()
                            .Configure(c => c.LifeStyle.PerWebRequest));
   }
}
```

:information_source: **分离安装器:** 通常一个单独的安装器安装相关服务的连续闭集（比如仓储，控制器，等等），每个集合都有一个独立的安装器。这会帮助你保持安装器简洁易读，使得它们在测试中更容易使用，更容易找到特定组件的相关注册代码 - 常常被遗忘但是良好分离注册代码的重要作用。

:warning: **默认情况下安装器必须是公共的，并且有公共默认构造函数:** Windsor，在使用默认 `InstallerFactory` 时，**只扫描公共类型**，因此如果你的安装器不是公共的，Windsor 不会安装它们。 当安装器被 Windsor 实例时，它们必须有公共默认构造函数。否则会抛出异常。普通类也一样。

## 使用安装器

在创建安装器之后，你必须在[启动程序](three-calls-pattern.md#call-one-bootstrapper)中将它们安装到容器。为此，使用容器的 `Install` 方法：

```csharp
var container = new WindsorContainer();
container.Install(
   new ControllersInstaller(),
   new RepositoriesInstaller(),
   // and all your other installers
);
```

这可能有一点乏味，因为你的应用很可能有几个或更多的安装器。另外每次添加了一个新的安装器，你都需要记得回到启动程序来安装它。

为了远离这些乏味的手工工作，Windsor 有些自动处理这些的帮助类，也就是 `FromAssembly` 静态类， 和为了使用外部配置的 `Configuration` 类。

## `FromAssembly` 类

与其手工实例化安装器，你可以通过使用 `FromAssembly` 类，将事情留给 Windsor 完成。这个类有一些选择一个或多个程序集的方法，然后它会实例化那些程序集中的所有安装器类型。这样做的好处是，当你为这些程序集添加新的安装器时，它们会被 Windsor 自动选中，你不需要做额外的工作。

这个类型公开了几个有用的方法用于定位程序集。

```csharp
container.Install(
   FromAssembly.This(),
   FromAssembly.Named("Acme.Crm.Bootstrap"),
   FromAssembly.Containing<ServicesInstaller>(),
   FromAssembly.InDirectory(new AssemblyFilter("Extensions")),
   FromAssembly.Instance(this.GetPluginAssembly()));
```

:information_source: **安装器以不确定的顺序创建和安装:** 在使用 `FromAssembly` 时，你不应该依赖安装器将会被实例化或安装的顺序。它是不确定的，意味着你不会知道执行的顺序。如果你需要以指定顺序安装安装器，使用 `InstallerFactory`。

### `This`

从调用方法的程序集安装。即你的启动程序集。

### `Named`

通过指定程序集名称安装，使用标准.NET程序集定位机制。你也可以提供 .dll 或 .exe 文件的路径，当你的程序集在非标准位置时。

### `Containing`

从包含特定类型的程序集安装。这个方法通常作为 `FromAssembly.Named` 的string-less alternative (字符串替代？)。

### `InDirectory`

从指定文件夹安装。该方法需要一个 `AssemblyFilter` 对象，这个对象允许你做各种过滤，以缩小你感兴趣的程序集的范围，包括通过名称模式过滤，公钥标记或自定义过滤。

### `Instance`

从任意指定程序集安装。这个方法时是其他方法的后备，当你有一些定位你想安装的程序集的自定义代码时。

## `InstallerFactory` 类

上面所有的方法都有一个接收 `InstallerFactory` 实例的重载。大多数时候你不需要关心，things will just work。但是，如果你需要更严格的控制程序集的安装器（影响安装的顺序，改变实例化方式或只安装一部分，而不是全部），你可以从这个类派生，并提供你自己的实现去实现这些目标。

## `Configuration` 类

除了在代码中使用 [Windsor.Fluent-Registration-API|fluent registration API] 注册组件的安装器外，你可能有一些 [Windsor.XML-Registration-Reference|XML configuration]。你可以通过静态类 `Configuration` 公开的方法来安装。

```csharp
container.Install(
   Configuration.FromAppConfig(),
   Configuration.FromXmlFile("settings.xml"),
   Configuration.FromXml(new AssemblyResource("assembly://Acme.Crm.Data/Configuration/services.xml"))
);
```


你可以用它来访问AppDomain配置文件（的app.config或web.config文件）中的配置，或任意的XML文件。如本文最后一个例子所示，文件可以是嵌入到程序集中的（编译操作设置为嵌入的资源）。

`Configuration` 类的一个非常有用的用法，是使用XML配置文件消除一些额外组件（比如说你的应用的扩展）的编译时依赖。 你可以将这些程序集（或包含在它们内的特定安装器）在XML文件中列出，并让 Windsor 查找和安装它们。[Windsor.Registering-Installers|Read more here].

## 还可以看看

* [Fluent 注册 API](fluent-registration-api.md)
* [XML 配置引用](xml-registration-reference.md)
* [通过 XML 配置注册安装器](registering-installers.md)

## 外部资源

* [Blog post by Krzysztof Kozmic (Oct 10, 2010)](http://kozmic.pl/2010/08/10/ioc-patterns-ndash-partitioning-registration/)
