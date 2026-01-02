# GoRouter Push 导航示例解析

## 概述

这个示例展示了如何在 Flutter 应用中使用 `go_router` 包的 `push` 方法进行页面导航。与 `go` 方法不同，`push` 方法会将新页面推入导航栈，而不是替换整个路由栈。这使得用户可以通过系统返回按钮或手势返回到上一个页面。

## 整体结构

这个示例包含以下主要组件：

1. **App 类**：应用的主入口，配置了 `GoRouter`
2. **Page1ScreenWithPush**：第一个页面，包含一个按钮用于 push 到第二个页面
3. **Page2ScreenWithPush**：第二个页面，可以继续 push 自身或使用 `go` 返回首页

## 路由配置

应用的路由配置在 `App` 类中定义：

```dart 22:37:example/lib/others/push.dart
late final GoRouter _router = GoRouter(
  routes: <GoRoute>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) =>
          const Page1ScreenWithPush(),
    ),
    GoRoute(
      path: '/page2',
      builder: (BuildContext context, GoRouterState state) =>
          Page2ScreenWithPush(
            int.parse(state.uri.queryParameters['push-count']!),
          ),
    ),
  ],
);
```

### 路由说明

- **`/`**：根路径，显示 `Page1ScreenWithPush` 页面
- **`/page2`**：第二个页面的路径，接收查询参数 `push-count` 来跟踪 push 的次数

### 查询参数的使用

在 `/page2` 路由中，通过 `state.uri.queryParameters['push-count']!` 获取查询参数，并转换为整数传递给页面组件。这个参数用于演示多次 push 同一页面时如何区分不同的实例。

## 页面组件详解

### Page1ScreenWithPush

第一个页面是一个简单的界面，包含一个按钮用于导航到第二个页面：

```dart 40:60:example/lib/others/push.dart
/// The screen of the first page.
class Page1ScreenWithPush extends StatelessWidget {
  /// Creates a [Page1ScreenWithPush].
  const Page1ScreenWithPush({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('${App.title}: page 1')),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () => context.push('/page2?push-count=1'),
            child: const Text('Push page 2'),
          ),
        ],
      ),
    ),
  );
}
```

**关键点**：

- 使用 `context.push('/page2?push-count=1')` 进行导航
- URL 中包含查询参数 `push-count=1`，表示这是第一次 push

### Page2ScreenWithPush

第二个页面展示了两种不同的导航方式：

```dart 62:98:example/lib/others/push.dart
/// The screen of the second page.
class Page2ScreenWithPush extends StatelessWidget {
  /// Creates a [Page2ScreenWithPush].
  const Page2ScreenWithPush(this.pushCount, {super.key});

  /// The push count.
  final int pushCount;

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(
      title: Text('${App.title}: page 2 w/ push count $pushCount'),
    ),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          Padding(
            padding: const EdgeInsets.all(8),
            child: ElevatedButton(
              onPressed: () => context.go('/'),
              child: const Text('Go to home page'),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8),
            child: ElevatedButton(
              onPressed: () =>
                  context.push('/page2?push-count=${pushCount + 1}'),
              child: const Text('Push page 2 (again)'),
            ),
          ),
        ],
      ),
    ),
  );
}
```

**关键特性**：

1. **pushCount 参数**：用于跟踪这是第几次 push 操作，显示在 AppBar 标题中
2. **两种导航按钮**：
   - **"Go to home page"**：使用 `context.go('/')` 直接跳转到首页，会清空整个导航栈
   - **"Push page 2 (again)"**：使用 `context.push('/page2?push-count=${pushCount + 1}')` 再次 push 同一个页面，但 push-count 会递增

## 导航方法对比

### `context.push()` vs `context.go()`

这个示例展示了两种不同的导航方式：

#### `context.push()`

```dart 53:53:example/lib/others/push.dart
onPressed: () => context.push('/page2?push-count=1'),
```

- **行为**：将新页面推入导航栈
- **栈管理**：保留之前的页面在栈中
- **返回行为**：用户可以通过系统返回按钮返回到上一个页面
- **使用场景**：适合需要保留导航历史的场景，如详情页、表单页等

#### `context.go()`

```dart 82:82:example/lib/others/push.dart
onPressed: () => context.go('/'),
```

- **行为**：替换整个路由栈，直接跳转到目标路由
- **栈管理**：清空之前的导航栈
- **返回行为**：无法返回到之前的页面（除非目标路由配置了返回路径）
- **使用场景**：适合需要重置导航栈的场景，如登出后跳转到登录页、切换主要功能模块等

## 导航栈演示

当用户执行以下操作时，导航栈的变化如下：

1. **初始状态**：`[Page1]`
2. **点击 "Push page 2"**：`[Page1, Page2(pushCount=1)]`
3. **点击 "Push page 2 (again)"**：`[Page1, Page2(pushCount=1), Page2(pushCount=2)]`
4. **再次点击 "Push page 2 (again)"**：`[Page1, Page2(pushCount=1), Page2(pushCount=2), Page2(pushCount=3)]`
5. **点击 "Go to home page"**：`[Page1]`（栈被清空并重置）

## 关键概念

### 查询参数传递

通过 URL 查询参数传递数据是 GoRouter 中常见的数据传递方式：

```dart 33:33:example/lib/others/push.dart
int.parse(state.uri.queryParameters['push-count']!),
```

- 从 `GoRouterState` 的 `uri.queryParameters` 中获取参数
- 使用 `!` 断言非空（在实际应用中应该添加空值检查）
- 将字符串转换为所需的数据类型

### 页面状态区分

虽然多次 push 同一个路由（`/page2`），但由于查询参数不同（`push-count` 递增），每个页面实例都是独立的。这展示了如何通过查询参数来区分同一路由的不同实例。

## 实际应用建议

在实际开发中，建议：

1. **参数验证**：不要使用 `!` 强制解包，应该添加空值检查和默认值处理
2. **类型安全**：考虑使用类型安全的路由或通过 `extra` 参数传递复杂对象
3. **导航策略**：
   - 使用 `push` 进行层级导航（如：列表 → 详情）
   - 使用 `go` 进行模块切换或重置导航栈
4. **用户体验**：根据业务需求合理选择导航方式，避免过深的导航栈影响用户体验

## 总结

这个示例清晰地演示了：

- `context.push()` 的基本用法
- 如何通过查询参数传递数据
- `push` 和 `go` 两种导航方式的区别
- 如何使用相同路由创建多个页面实例

通过运行这个示例，开发者可以直观地理解 GoRouter 中 push 导航的工作原理和适用场景。
