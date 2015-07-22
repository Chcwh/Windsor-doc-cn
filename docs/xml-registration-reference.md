# 使用 XML 配置

大多是时候使用 [Fluent 注册 API](fluent-registration-api.md) 来注册和配置组件。但这不是唯一的方法，Windsor 具有全面支持 XML 配置来完成一些容器相关的任务。

:information_source: **在哪里进行配置：** 可以将 Windsor 的配置放在 app.config/web.config 文件中，如果需要的话，可以放在自定义的专用文件或分布到多个文件中。此外，文件可以在磁盘上，如果不想暴露给用户，也可以嵌入到程序集中。

## XML 配置可以做什么

XML 配置可以用于完成以下目标：
* [为你的组件提供配置时属性](xml-configuration-properties.md)（连接字符串，服务地址，管理邮箱地址等）
* [注册安装器](registering-installers.md)
* [注册和配置组件](registering-components.md) （如果可以的话，建议在代码中进行）
* [注册和配置设施](facilities-xml-configuration.md) （如果可以的话，建议在代码中进行）

:information_source: **在 XML 注册组件：** 在 XML 中注册组件的功能是 [Fluent 注册 API](fluent-registration-api.md) 诞生之前的用法。没有在代码中注册那么好用，而且很多任务只能通过代码完成。

为了让 XML 配置更容易，可以将配置[分布到多个文件](xml-configuration-includes.md)中，如果需要分离的话。

:information_source: **XML 架构：** 本文档只讨论默认元素，provided out of the box。Windsor 的架构不是刚性的，各种扩展，比如设施，可能（经常这样）提供扩展默认集合的其他元素。

## XML 配置一览

:information_source: 本节仅集中讨论格式，不讨论使用或扩展的代码。

:information_source: **XML 中的引用类型：** Windsor 允许你在 XML 中引用类型的时候省略程序集限定名称部分。阅读 [XML 中的引用类型](referencing-types-in-xml.md) 了解更多。

下面的内容演示了容器默认使用的所有节点和属性。上一节包含了前往更详细内容的链接。

```xml
<configuration>
  <!--允许你在引用该程序集中的类型时，只指定它们的名称，不需要指定完全限定名-->
  <using assembly="Acme.Crm.Services, Version=1.0.0.0, Culture=neutral, PublicKeyToken=1987352536523" />

  <include uri="file://Configurations/services.xml" />
  <include uri="assembly://Acme.Crm.Data/Configuration/DataConfiguration.xml" />

  <installers>
    <install type="Acme.Crm.Infrastructure.ServicesInstaller, Acme.Crm.Infrastructure"/>
    <install assembly="Acme.Crm.Infrastructure"/>
  </installers>

  <properties>
    <connection_string>这里填入值</connection_string>
  </properties>

  <facilities>
    <facility id="uniqueId" type="Acme.Common.Windsor.AcmeFacility, Acme.Common" />
  </facilities>

  <components>
    <component
      id="uniqueId"
      service="Acme.Crm.Services.INotificationService, Acme.Crm"
      type="Acme.Crm.Services.EmailNotificationService, Acme.Crm"
      inspectionBehavior="all|declaredonly|none"
      lifestyle="singleton|thread|transient|pooled|custom"
      customLifestyleType="type that implements ILifestyleManager"
      componentActivatorType="type that implements IComponentActivator"
      initialPoolSize="2" maxPoolSize="6">

      <forwardedTypes>
        <add service="Acme.Crm.Services.IEmailSender, Acme.Crm" />
      </forwardedTypes>

      <additionalInterfaces>
        <add interface="Acme.Crm.Services.IMetadataService, Acme.Crm" />
      </additionalInterfaces>

      <parameters>
        <paramtername>value</paramtername>
        <otherparameter>#{connection_string}</otherparameter>
      </parameters>

      <interceptors selector="${interceptorsSelector.id}" hook="${generationHook.id}">
        <interceptor>${interceptor.id}</interceptor>
      </interceptors>

      <mixins>
        <mixin>${mixin.id}</mixin>
      </mixins>

    </component>
  </components>
</configuration>
```

## 加载 XML 配置

有两种方式来向容器中安装 XML 配置：

### 使用静态类 `Configuration`

可以从 XML 安装配置，就像其它通过 `Configuration` 类安装的其它安装器一样。（[了解更多](installers.md#configuration-class)）

### 使用构造函数

使用 `WindsorContainer` 的构造函数：

```csharp
public WindsorContainer(IConfigurationInterpreter interpreter)
```

你可以在创建容器的时候，传递 XML 配置文件的引用。

```csharp
IResource resource = new AssemblyResource("assembly://Acme.Crm.Data/Configuration/services.xml");
container = new WindsorContainer(new XmlInterpreter(resource));
```

在这个例子中，使用嵌入在 `Acme.Crm.Data` 程序集中的 XML 文件。

也可以使用 `XmlInterpreter` 的无参构造函数，这种情况下，将使用 `AppDomain` 配置文件作为配置的来源：

```csharp
container = new WindsorContainer(new XmlInterpreter());
```

:information_source: **建议使用 `Configuration` 类：** 建议使用上面提到的其他方法。不仅更加灵活，而且 Windsor 将来的版本可能为那个用法进行优化。 
