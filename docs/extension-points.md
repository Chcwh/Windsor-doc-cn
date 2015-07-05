# 扩展点（Extension Points）

Windsor 不打算支持所有情况和不确定的功能。相反，它公开了一组丰富的扩展点，你可以插入你自己的逻辑去扩展或修改 Windsor 的行为：

* [子系统（SubSystems）](subsystems.md) - Windsor 最核心的扩展点。很少被扩展或包装。
* [设施（Facilities）](facilities.md) - 主要的扩展点。通常包含一个或多个下面列出的扩展点。
* [ComponentModel construction contributors](componentmodel-construction-contributors.md) - 检查或修改 [ComponentModel](componentmodel.md).
* [处理器选择器（Handler Selectors）](handler-selectors.md) - 自定义逻辑将会替代组件选择的逻辑。通常在多租户（tenant）应用中使用。
* [模型拦截器选择器（Model Interceptors Selectors）](model-interceptors-selectors.md) - 自定义逻辑为给定组件动态的选择拦截器。
* [延迟组件加载器（Lazy Component Loaders）](lazy-component-loaders.md) - just in time component registration. Especially targeted at pulling component information from other frameworks, like MEF or WCF or resolving un-pre-registered concrete types.
* [Lifecycle concerns](lifecycle.md) - 在组件实例被创建或退役时执行的逻辑。
* [生命期类型管理器（Lifestyle managers）](lifestyles.md) - 控制组件什么时候应该创建/重用，什么时候结束生命周期。
* [组件激活器（Component Activators）](component-activators.md) - 控制组件应当如何实例化。
* [释放策略（Release Policy）](release-policy.md) - 处理组件的跟踪和释放。
* [解析器（Resolvers）](resolvers.md) - 重写组件解析逻辑。
* [容器事件（Container Events）](container-events.md) - 容器事件通知。
* 更多
