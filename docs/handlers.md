# 处理器（Handlers）

## 什么是处理器

处理器是实现 `IHandler` 接口的类型. Windsor 使用处理器为特定服务解析组件，之后释放处理器。处理器能够访问[`ComponentModel`](componentmodel.md) ，这允许开发人员通过编程检查组件。

## See also

* [服务和组件](services-and-components.md)
* [ComponentModel](componentmodel.md)
* [组件是怎样创建的](how-components-are-created.md)