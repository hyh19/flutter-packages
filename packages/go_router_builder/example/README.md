# GoRouter Builder 示例索引

本目录包含了 `go_router_builder` 包的所有示例代码，按照学习难度和知识点相关性进行了分组和排序。每个示例都演示了特定的功能特性，帮助您循序渐进地学习如何使用 `go_router_builder`。

## 目录

- [GoRouter Builder 示例索引](#gorouter-builder-示例索引)
  - [目录](#目录)
  - [第一组：基础入门](#第一组基础入门)
    - [simple\_example.dart](#simple_exampledart)
    - [main.dart](#maindart)
  - [第二组：路由参数基础](#第二组路由参数基础)
    - [all\_types.dart](#all_typesdart)
    - [all\_extension\_types.dart](#all_extension_typesdart)
  - [第三组：复杂参数类型](#第三组复杂参数类型)
    - [json\_example.dart](#json_exampledart)
    - [json\_nested\_example.dart](#json_nested_exampledart)
  - [第四组：导航特性](#第四组导航特性)
    - [go\_relative.dart](#go_relativedart)
    - [extra\_example.dart](#extra_exampledart)
    - [on\_exit\_example.dart](#on_exit_exampledart)
  - [第五组：高级配置](#第五组高级配置)
    - [case\_sensitive\_example.dart](#case_sensitive_exampledart)
    - [custom\_encoder\_example.dart](#custom_encoder_exampledart)
  - [第六组：Shell Route](#第六组shell-route)
    - [shell\_route\_example.dart](#shell_route_exampledart)
    - [shell\_route\_with\_keys\_example.dart](#shell_route_with_keys_exampledart)
    - [shell\_route\_with\_observers\_example.dart](#shell_route_with_observers_exampledart)
    - [stateful\_shell\_route\_example.dart](#stateful_shell_route_exampledart)
    - [stateful\_shell\_route\_initial\_location\_example.dart](#stateful_shell_route_initial_location_exampledart)
  - [第七组：其他](#第七组其他)
    - [readme\_excerpts.dart](#readme_excerptsdart)
    - [separate\_file\_route.dart](#separate_file_routedart)
  - [学习路径建议](#学习路径建议)
    - [初学者路径](#初学者路径)
    - [进阶路径](#进阶路径)
    - [参考资源](#参考资源)

## 第一组：基础入门

### simple_example.dart

**文件路径**：`lib/simple_example.dart`

**核心知识点**：

- `@TypedGoRoute` 注解的基本用法
- 基本路由定义和路径配置
- 路径参数（path parameters）的使用
- `go()` 导航方法
- 路由名称（name）的配置

**简要说明**：

这是最简单的路由示例，演示了如何使用 `@TypedGoRoute` 注解定义路由，以及如何通过路径参数传递数据。适合初学者作为第一个学习的示例。

**相关示例**：

- [main.dart](#maindart) - 更完整的应用示例

### main.dart

**文件路径**：`lib/main.dart`

**核心知识点**：

- 嵌套路由（nested routes）的定义
- 路由重定向（redirect）的使用
- `refreshListenable` 与状态管理集成
- 登录状态管理和路由保护
- `push()` 方法的返回值处理
- `buildPage()` 自定义页面构建
- `$extra` 参数的使用
- 多层级路由嵌套

**简要说明**：

这是一个完整的应用示例，展示了实际项目中常见的路由场景，包括登录验证、路由重定向、嵌套路由等。适合在掌握基础路由后深入学习。

**相关示例**：

- [simple_example.dart](#simple_exampledart) - 基础路由示例
- [extra_example.dart](#extra_exampledart) - $extra 参数详解

## 第二组：路由参数基础

### all_types.dart

**文件路径**：`lib/all_types.dart`

**核心知识点**：

- 基本类型参数：`String`、`int`、`double`、`bool`、`BigInt`、`DateTime`、`Uri`、`num`
- 枚举类型（enum）作为路由参数
- 增强枚举（enhanced enum）的使用
- 集合类型：`Iterable`、`List`、`Set` 及其泛型支持
- 必需参数（required）与可选参数（optional）的区别
- 查询参数（query parameters）的使用
- 查询参数默认值的设置

**简要说明**：

该示例全面展示了 `go_router_builder` 支持的所有数据类型，包括基本类型、枚举和集合类型。通过这个示例可以了解如何在路由中使用不同类型的参数。

**相关示例**：

- [all_extension_types.dart](#all_extension_typesdart) - Extension Types 版本
- [json_example.dart](#json_exampledart) - 自定义类参数

### all_extension_types.dart

**文件路径**：`lib/all_extension_types.dart`

**核心知识点**：

- Dart 3.5+ Extension Types 特性
- 使用 Extension Types 进行类型安全封装
- Extension Types 作为路由参数的使用方法
- 与基本类型示例的对比

**简要说明**：

该示例演示了如何使用 Dart 3.5+ 引入的 Extension Types 特性来增强类型安全。通过 Extension Types 可以创建轻量级的类型包装器，同时保持零运行时开销。

**相关示例**：

- [all_types.dart](#all_typesdart) - 基本类型版本

## 第三组：复杂参数类型

### json_example.dart

**文件路径**：`lib/json_example.dart`

**核心知识点**：

- 自定义类作为路由参数
- `fromJson` 和 `toJson` 序列化方法
- 复杂对象在 URL 中的传递
- JSON 编码/解码机制

**简要说明**：

该示例展示了如何将自定义类对象作为路由参数传递。通过实现 `fromJson` 和 `toJson` 方法，可以将复杂对象序列化到 URL 中，并在路由解析时自动反序列化。

**相关示例**：

- [json_nested_example.dart](#json_nested_exampledart) - 嵌套对象示例
- [all_types.dart](#all_typesdart) - 基本类型参数

### json_nested_example.dart

**文件路径**：`lib/json_nested_example.dart`

**核心知识点**：

- 泛型嵌套对象作为路由参数
- 嵌套 JSON 结构的序列化
- 复杂数据结构的传递

**简要说明**：

该示例演示了如何处理嵌套的泛型对象结构。当需要传递包含嵌套对象的复杂数据结构时，可以参考此示例的实现方式。

**相关示例**：

- [json_example.dart](#json_exampledart) - 简单对象示例

## 第四组：导航特性

### go_relative.dart

**文件路径**：`lib/go_relative.dart`

**核心知识点**：

- `TypedRelativeGoRoute` 相对路由定义
- `RelativeGoRouteData` 相对路由数据类
- `goRelative()` 相对导航方法
- 路由复用和相对路径的优势

**简要说明**：

该示例展示了相对路由的使用方法。相对路由可以在不同的父路由下复用相同的路由定义，特别适合构建可复用的路由组件。

**相关示例**：

- [simple_example.dart](#simple_exampledart) - 绝对路由示例

### extra_example.dart

**文件路径**：`lib/extra_example.dart`

**核心知识点**：

- `$extra` 特殊参数的使用
- 必需和可选的 `$extra` 参数
- 非 URL 参数传递机制
- 通过 `$extra` 传递复杂对象

**简要说明**：

该示例演示了如何使用 `$extra` 参数传递不在 URL 中显示的数据。`$extra` 参数适合传递敏感信息或不需要在 URL 中暴露的复杂对象。

**相关示例**：

- [main.dart](#maindart) - 在完整应用中的使用
- [json_example.dart](#json_exampledart) - URL 参数传递对比

### on_exit_example.dart

**文件路径**：`lib/on_exit_example.dart`

**核心知识点**：

- `onExit()` 方法实现路由退出拦截
- 路由切换确认对话框
- 异步拦截处理
- 阻止或允许路由切换

**简要说明**：

该示例展示了如何在用户离开当前路由时进行拦截，例如显示确认对话框。这对于需要保存数据或确认操作的场景非常有用。

**相关示例**：

- [main.dart](#maindart) - 路由重定向示例

## 第五组：高级配置

### case_sensitive_example.dart

**文件路径**：`lib/case_sensitive_example.dart`

**核心知识点**：

- `caseSensitive` 参数配置路由大小写敏感性
- 路径匹配规则的控制
- 大小写敏感与不敏感路由的对比

**简要说明**：

该示例演示了如何控制路由路径的大小写敏感性。默认情况下路由是大小写敏感的，但可以通过 `caseSensitive: false` 来禁用大小写检查。

**相关示例**：

- [simple_example.dart](#simple_exampledart) - 基础路由配置

### custom_encoder_example.dart

**文件路径**：`lib/custom_encoder_example.dart`

**核心知识点**：

- `@CustomParameterCodec` 注解的使用
- 自定义编码/解码函数
- Base64 编码示例
- 参数值的自定义序列化

**简要说明**：

该示例展示了如何为路由参数定义自定义的编码和解码逻辑。当默认的序列化方式不满足需求时（例如需要 Base64 编码），可以使用自定义编码器。

**相关示例**：

- [json_example.dart](#json_exampledart) - JSON 序列化示例

## 第六组：Shell Route

### shell_route_example.dart

**文件路径**：`lib/shell_route_example.dart`

**核心知识点**：

- `@TypedShellRoute` 注解定义 Shell 路由
- `ShellRouteData` 数据类
- 共享 UI 容器的实现
- 底部导航栏集成
- 多个子路由共享同一个容器

**简要说明**：

该示例展示了 Shell Route 的基本用法。Shell Route 允许多个路由共享同一个 UI 容器（如底部导航栏），非常适合实现 Tab 导航等场景。

**相关示例**：

- [shell_route_with_keys_example.dart](#shell_route_with_keys_exampledart) - 带 Navigator Key 的版本
- [stateful_shell_route_example.dart](#stateful_shell_route_exampledart) - Stateful Shell Route

### shell_route_with_keys_example.dart

**文件路径**：`lib/shell_route_with_keys_example.dart`

**核心知识点**：

- `$navigatorKey` 为 Shell 路由设置 Navigator Key
- `$parentNavigatorKey` 指定父级 Navigator
- 多层级导航器的管理
- 对话框覆盖导航栏的实现

**简要说明**：

该示例演示了如何在 Shell Route 中使用 Navigator Key 来控制导航层级。通过设置不同的 Navigator Key，可以实现对话框覆盖导航栏等复杂场景。

**相关示例**：

- [shell_route_example.dart](#shell_route_exampledart) - 基础 Shell Route
- [shell_route_with_observers_example.dart](#shell_route_with_observers_exampledart) - 带观察者的版本

### shell_route_with_observers_example.dart

**文件路径**：`lib/shell_route_with_observers_example.dart`

**核心知识点**：

- `$observers` 静态属性配置观察者
- `NavigatorObserver` 的使用
- 路由生命周期监听
- 导航事件的追踪

**简要说明**：

该示例展示了如何在 Shell Route 中添加 Navigator Observer 来监听路由的生命周期事件。这对于实现路由分析、日志记录等功能非常有用。

**相关示例**：

- [shell_route_example.dart](#shell_route_exampledart) - 基础 Shell Route
- [shell_route_with_keys_example.dart](#shell_route_with_keys_exampledart) - 带 Navigator Key 的版本

### stateful_shell_route_example.dart

**文件路径**：`lib/stateful_shell_route_example.dart`

**核心知识点**：

- `@TypedStatefulShellRoute` 注解定义 Stateful Shell Route
- `StatefulShellRouteData` 数据类
- `StatefulNavigationShell` 的使用
- `goBranch()` 方法切换分支
- 状态保持机制
- 分支导航器的独立管理
- 自定义分支容器构建器

**简要说明**：

该示例展示了 Stateful Shell Route 的完整用法。Stateful Shell Route 允许每个分支维护独立的导航状态，非常适合需要保持多个 Tab 页面状态的场景。

**相关示例**：

- [shell_route_example.dart](#shell_route_exampledart) - 基础 Shell Route
- [stateful_shell_route_initial_location_example.dart](#stateful_shell_route_initial_location_exampledart) - 带初始位置的版本

### stateful_shell_route_initial_location_example.dart

**文件路径**：`lib/stateful_shell_route_initial_location_example.dart`

**核心知识点**：

- `$initialLocation` 静态属性设置分支初始位置
- 分支初始位置配置
- Tab 导航集成
- 多分支的初始路由设置

**简要说明**：

该示例演示了如何为 Stateful Shell Route 的每个分支设置初始位置。当用户首次切换到某个分支时，会导航到指定的初始路由，而不是分支的第一个路由。

**相关示例**：

- [stateful_shell_route_example.dart](#stateful_shell_route_exampledart) - 基础 Stateful Shell Route

## 第七组：其他

### readme_excerpts.dart

**文件路径**：`lib/readme_excerpts.dart`

**核心知识点**：

- 各种用法的代码片段参考
- 文档示例集合
- 常见场景的实现方式

**简要说明**：

该文件包含了 README 文档中使用的各种代码片段，涵盖了 `go_router_builder` 的主要功能特性。可以作为快速参考手册使用。

**相关示例**：

- 所有其他示例都可以在这里找到对应的代码片段

### separate_file_route.dart

**文件路径**：`lib/separate_file_route.dart`

**核心知识点**：

- 路由定义的文件组织
- 跨文件路由引用
- 模块化路由管理

**简要说明**：

该示例展示了如何将路由定义分离到不同的文件中，以便更好地组织代码。这对于大型项目的路由管理非常重要。

**相关示例**：

- [stateful_shell_route_initial_location_example.dart](#stateful_shell_route_initial_location_exampledart) - 使用了分离的路由定义

## 学习路径建议

### 初学者路径

如果您是第一次使用 `go_router_builder`，建议按照以下顺序学习：

1. **基础入门**
   - 从 [simple_example.dart](#simple_exampledart) 开始，了解基本的路由定义和导航
   - 然后学习 [main.dart](#maindart)，了解完整应用中的路由使用

2. **路由参数**
   - 学习 [all_types.dart](#all_typesdart)，了解支持的数据类型
   - 如果需要传递复杂对象，学习 [json_example.dart](#json_exampledart)

3. **导航特性**
   - 学习 [go_relative.dart](#go_relativedart) 了解相对路由
   - 学习 [extra_example.dart](#extra_exampledart) 了解非 URL 参数传递

### 进阶路径

如果您已经掌握了基础知识，可以深入学习：

1. **高级配置**
   - [case_sensitive_example.dart](#case_sensitive_exampledart) - 路由匹配规则
   - [custom_encoder_example.dart](#custom_encoder_exampledart) - 自定义编码器

2. **Shell Route**
   - 从 [shell_route_example.dart](#shell_route_exampledart) 开始
   - 学习 [shell_route_with_keys_example.dart](#shell_route_with_keys_exampledart) 了解导航器管理
   - 学习 [stateful_shell_route_example.dart](#stateful_shell_route_exampledart) 了解状态保持

3. **特殊场景**
   - [on_exit_example.dart](#on_exit_exampledart) - 路由拦截
   - [json_nested_example.dart](#json_nested_exampledart) - 复杂数据结构

### 参考资源

- 查看 [readme_excerpts.dart](#readme_excerptsdart) 获取各种场景的代码片段
- 参考 [separate_file_route.dart](#separate_file_routedart) 了解大型项目的路由组织方式
