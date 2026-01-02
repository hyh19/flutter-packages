# Go Router 示例指南

本文档列出了 `example/lib` 目录下的所有示例，按照从基础到高级的学习顺序排列。每个示例都包含简要介绍和核心知识点说明。

## 文档结构

本文档按照学习难度和功能特性，将所有示例分为以下几个部分：

1. **基础入门**（4 个示例）- 最基础的路由和导航功能
2. **中级进阶**（5 个示例）- 路由控制、异常处理、动画等
3. **Shell 路由**（4 个示例）- 嵌套导航和 Shell 路由相关
4. **高级特性**（3 个示例）- 动态配置、编解码器、顶层拦截等
5. **状态恢复**（3 个示例）- 应用状态恢复相关
6. **综合示例**（1 个）- 完整的应用示例
7. **其他示例**（8 个）- 补充特性示例

## 第一部分：基础入门

本部分介绍 GoRouter 最基础的功能，适合初学者入门。建议按顺序学习，掌握基本的路由配置和导航方法。

### 1. main.dart

- **文件路径**: [`lib/main.dart`](lib/main.dart)
- **简介**: 最基础的 GoRouter 示例，展示如何配置简单的路由和进行页面导航。
- **核心知识点**:
  - `GoRouter` 的基本配置和使用
  - `GoRoute` 的定义和嵌套路由
  - `context.go()` 导航方法
  - `MaterialApp.router` 的集成
  - 深链接支持

### 2. path_and_query_parameters.dart

- **文件路径**: [`lib/path_and_query_parameters.dart`](lib/path_and_query_parameters.dart)
- **简介**: 演示如何在路由中使用路径参数和查询参数，实现动态路由和参数传递。
- **核心知识点**:
  - 路径参数（`:param`）的定义和使用
  - `GoRouterState.pathParameters` 访问路径参数
  - 查询参数（query parameters）的使用
  - `GoRouterState.uri.queryParameters` 访问查询参数
  - `context.goNamed()` 使用命名路由导航

### 3. named_routes.dart

- **文件路径**: [`lib/named_routes.dart`](lib/named_routes.dart)
- **简介**: 展示如何使用命名路由替代硬编码的 URL，提高代码的可维护性。
- **核心知识点**:
  - `GoRoute.name` 属性的使用
  - `context.namedLocation()` 生成命名路由的 URL
  - 命名路由的优势（避免硬编码 URL）
  - 多层嵌套路由的命名

### 4. go_relative.dart

- **文件路径**: [`lib/go_relative.dart`](lib/go_relative.dart)
- **简介**: 演示如何使用相对路径进行导航，相对于当前路由位置。
- **核心知识点**:
  - 相对路径导航（`context.go('./path')`）
  - 相对路径与绝对路径的区别
  - 在当前路由基础上进行导航

## 第二部分：中级进阶

本部分介绍路由控制、异常处理、动画等进阶功能。建议在掌握基础功能后学习。

### 5. redirection.dart

- **文件路径**: [`lib/redirection.dart`](lib/redirection.dart)
- **简介**: 展示如何使用重定向功能实现基于状态的路由控制，例如登录验证。
- **核心知识点**:
  - `GoRouter.redirect` 回调的使用
  - 基于状态的导航控制（如登录状态）
  - `refreshListenable` 监听状态变化
  - 路由拦截和重定向逻辑

### 6. async_redirection.dart

- **文件路径**: [`lib/async_redirection.dart`](lib/async_redirection.dart)
- **简介**: 演示如何处理异步认证状态的重定向，适用于需要异步验证的场景。
- **核心知识点**:
  - 异步重定向（async redirect）
  - `InheritedNotifier` 的使用
  - StreamAuth 模式的实现
  - 依赖注入和状态依赖

### 7. exception_handling.dart

- **文件路径**: [`lib/exception_handling.dart`](lib/exception_handling.dart)
- **简介**: 展示如何处理路由异常和未知路由，提供友好的错误页面。
- **核心知识点**:
  - `GoRouter.onException` 异常处理
  - 404 错误页面处理
  - 异常时的路由跳转
  - `state.extra` 传递额外数据

### 8. transition_animations.dart

- **文件路径**: [`lib/transition_animations.dart`](lib/transition_animations.dart)
- **简介**: 演示如何为路由切换添加自定义的过渡动画效果。
- **核心知识点**:
  - `pageBuilder` 的使用
  - `CustomTransitionPage` 自定义过渡动画
  - `FadeTransition`、`SlideTransition` 等动画效果
  - 过渡动画的配置和自定义

### 9. on_exit.dart

- **文件路径**: [`lib/on_exit.dart`](lib/on_exit.dart)
- **简介**: 展示如何在用户离开页面时进行拦截和确认，例如未保存的数据警告。
- **核心知识点**:
  - `GoRoute.onExit` 回调的使用
  - 页面退出拦截
  - 异步确认对话框
  - 阻止或允许导航离开

## 第三部分：Shell 路由

本部分介绍嵌套导航和 Shell 路由相关的高级功能。Shell 路由是构建复杂应用导航结构的重要工具。

### 10. shell_route.dart

- **文件路径**: [`lib/shell_route.dart`](lib/shell_route.dart)
- **简介**: 展示如何使用 `ShellRoute` 创建嵌套导航结构，实现底部导航栏等 UI 模式。
- **核心知识点**:
  - `ShellRoute` 的定义和使用
  - 嵌套 `Navigator` 的实现
  - `BottomNavigationBar` 与路由集成
  - `navigatorKey` 和 `parentNavigatorKey` 的区别
  - 深链接与 Shell 路由的结合

### 11. shell_route_top_route.dart

- **文件路径**: [`lib/shell_route_top_route.dart`](lib/shell_route_top_route.dart)
- **简介**: 演示如何在 `ShellRoute` 中根据当前路由动态更新顶部栏（如 AppBar）的内容。
- **核心知识点**:
  - `ShellRoute.topRoute` 的使用
  - 动态 AppBar 标题的实现
  - 基于路由的 UI 更新
  - Shell 路由中顶部导航的处理

### 12. push_with_shell_route.dart

- **文件路径**: [`lib/push_with_shell_route.dart`](lib/push_with_shell_route.dart)
- **简介**: 展示在 `ShellRoute` 环境中如何进行页面推送导航。
- **核心知识点**:
  - 在 `ShellRoute` 中使用 `context.push()`
  - 不同 `Navigator` 层级的推送
  - `parentNavigatorKey` 的高级用法

### 13. stateful_shell_route.dart

- **文件路径**: [`lib/stateful_shell_route.dart`](lib/stateful_shell_route.dart)
- **简介**: 演示如何使用 `StatefulShellRoute` 实现每个标签页维护独立导航栈的高级导航模式。
- **核心知识点**:
  - `StatefulShellRoute` 的定义和使用
  - `NavigationShell` 的使用
  - 每个标签页独立的导航状态
  - 底部导航栏与有状态路由的集成

## 第四部分：高级特性

本部分介绍动态配置、编解码器、顶层拦截等高级特性。适合需要深度定制路由行为的场景。

### 14. routing_config.dart

- **文件路径**: [`lib/routing_config.dart`](lib/routing_config.dart)
- **简介**: 展示如何动态修改路由配置，在运行时添加或修改路由。
- **核心知识点**:
  - `GoRouter.routingConfig` 构造函数
  - `RoutingConfig` 的动态配置
  - `ValueNotifier` 用于路由配置更新
  - 运行时添加路由

### 15. extra_codec.dart

- **文件路径**: [`lib/extra_codec.dart`](lib/extra_codec.dart)
- **简介**: 演示如何为路由的 `extra` 参数配置自定义编解码器。
- **核心知识点**:
  - `GoRouter.extraCodec` 的使用
  - 自定义对象的序列化和反序列化
  - `state.extra` 的高级用法

### 16. top_level_on_enter.dart

- **文件路径**: [`lib/top_level_on_enter.dart`](lib/top_level_on_enter.dart)
- **简介**: 展示如何在顶层统一处理路由进入逻辑，例如处理推荐链接、深链接等。
- **核心知识点**:
  - `GoRouter.onEnter` 顶层拦截
  - 深链接处理
  - 推荐码处理等业务逻辑
  - 路由进入前的统一处理

## 第五部分：状态恢复

本部分介绍应用状态恢复相关功能，帮助应用在重启后恢复用户状态。

### 17. go_route_state_restoration.dart

- **文件路径**: [`lib/state_restoration/go_route_state_restoration.dart`](lib/state_restoration/go_route_state_restoration.dart)
- **简介**: 演示如何为普通 `GoRoute` 配置状态恢复功能。
- **核心知识点**:
  - `GoRoute` 的状态恢复配置
  - `RestorableTextField` 等可恢复组件
  - 应用重启后状态恢复

### 18. shell_route_state_restoration.dart

- **文件路径**: [`lib/state_restoration/shell_route_state_restoration.dart`](lib/state_restoration/shell_route_state_restoration.dart)
- **简介**: 展示如何为 `ShellRoute` 配置状态恢复。
- **核心知识点**:
  - `ShellRoute` 的状态恢复
  - Shell 路由中状态恢复的特殊处理

### 19. stateful_shell_route_state_restoration.dart

- **文件路径**: [`lib/state_restoration/stateful_shell_route_state_restoration.dart`](lib/state_restoration/stateful_shell_route_state_restoration.dart)
- **简介**: 演示如何为 `StatefulShellRoute` 配置状态恢复，恢复多个独立的导航栈。
- **核心知识点**:
  - `StatefulShellRoute` 的状态恢复
  - 多个导航栈的状态恢复

## 第六部分：综合示例

本部分包含一个完整的应用示例，综合运用了多种 GoRouter 特性。

### 20. books/（完整应用示例）

- **文件路径**: [`lib/books/`](lib/books/)
- **简介**: 一个完整的图书管理应用示例，综合运用了 GoRouter 的多种特性，包括认证、Shell 路由、参数传递等，是最佳实践的参考实现。
- **核心知识点**:
  - 完整的应用架构
  - 认证流程集成
  - 复杂路由结构
  - 数据模型与路由结合
  - 多个特性的综合使用

## 第七部分：其他示例

本部分包含其他补充特性的示例，可以根据需要参考学习。

### 21. error_screen.dart

- **文件路径**: [`lib/others/error_screen.dart`](lib/others/error_screen.dart)
- **简介**: 展示如何自定义路由错误页面。
- **核心知识点**:
  - `errorBuilder` 自定义错误页面
  - 错误页面的设计

### 22. push.dart

- **文件路径**: [`lib/others/push.dart`](lib/others/push.dart)
- **简介**: 演示 `push` 导航方式的使用。
- **核心知识点**:
  - `context.push()` 与 `context.go()` 的区别
  - 页面栈管理

### 23. nav_observer.dart

- **文件路径**: [`lib/others/nav_observer.dart`](lib/others/nav_observer.dart)
- **简介**: 展示如何监听路由导航事件。
- **核心知识点**:
  - `NavigatorObserver` 的使用
  - 路由导航监听

### 24. transitions.dart

- **文件路径**: [`lib/others/transitions.dart`](lib/others/transitions.dart)
- **简介**: 提供更多过渡动画的实现示例。
- **核心知识点**:
  - 更多过渡动画示例

### 25. extra_param.dart

- **文件路径**: [`lib/others/extra_param.dart`](lib/others/extra_param.dart)
- **简介**: 演示如何通过 `extra` 参数传递自定义数据。
- **核心知识点**:
  - `state.extra` 参数的使用
  - 传递自定义对象

### 26. init_loc.dart

- **文件路径**: [`lib/others/init_loc.dart`](lib/others/init_loc.dart)
- **简介**: 展示如何设置应用的初始路由位置。
- **核心知识点**:
  - `initialLocation` 初始位置配置

### 27. router_neglect.dart

- **文件路径**: [`lib/others/router_neglect.dart`](lib/others/router_neglect.dart)
- **简介**: 演示如何控制哪些导航操作被记录到历史记录中。
- **核心知识点**:
  - `RouterNeglect` 的使用
  - 忽略某些导航操作

### 28. custom_stateful_shell_route.dart

- **文件路径**: [`lib/others/custom_stateful_shell_route.dart`](lib/others/custom_stateful_shell_route.dart)
- **简介**: 展示如何自定义 `StatefulShellRoute` 的实现。
- **核心知识点**:
  - 自定义 `StatefulShellRoute` 的构建
  - 更高级的 Shell 路由定制

## 学习建议

1. **初学者**: 建议按照文档顺序，从第一部分开始学习，逐步掌握基础功能。
2. **有经验的开发者**: 可以根据项目需求，直接跳转到相关的部分学习。
3. **实战项目**: 建议参考第六部分的 `books/` 示例，了解如何在实际项目中综合运用各种特性。

## 相关资源

- [GoRouter 官方文档](https://pub.dev/packages/go_router)
- [GoRouter GitHub 仓库](https://github.com/flutter/packages/tree/main/packages/go_router)
