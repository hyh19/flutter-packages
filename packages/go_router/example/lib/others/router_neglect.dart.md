# router_neglect.dart 代码解析

## 概述

这个示例演示了如何使用 `routerNeglect` 功能来控制浏览器历史记录的跟踪行为。在某些场景下，你可能不希望某些导航操作被记录到浏览器的历史记录中，`routerNeglect` 提供了这种控制能力。

## 核心概念

### routerNeglect 的作用

`routerNeglect` 用于控制 GoRouter 是否将导航操作记录到浏览器的历史记录中。当设置为 `true` 时，导航操作不会在浏览器历史记录中创建新条目，用户点击浏览器的后退按钮时不会返回到这些页面。

### 使用场景

1. **临时页面**：不需要在历史记录中保存的临时页面（如登录页、确认对话框）
2. **模态导航**：类似模态对话框的导航行为
3. **内部导航**：应用内部的临时状态切换
4. **用户体验优化**：避免用户通过后退按钮返回到不应该返回的页面

## 路由配置

### GoRouter 设置

```dart 22:38:example/lib/others/router_neglect.dart
  final GoRouter _router = GoRouter(
    // To turn off history tracking in the browser for the entire application,
    // set routerNeglect to true:
    // routerNeglect: true,
    routes: <GoRoute>[
      GoRoute(
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const Page1Screen(),
      ),
      GoRoute(
        path: '/page2',
        builder: (BuildContext context, GoRouterState state) =>
            const Page2Screen(),
      ),
    ],
  );
```

在这个示例中，`routerNeglect` 被注释掉了，表示默认情况下会跟踪历史记录。如果需要全局关闭历史记录跟踪，可以取消注释并设置为 `true`。

### 全局 vs 局部控制

- **全局控制**：在 `GoRouter` 构造函数中设置 `routerNeglect: true`，会影响所有导航操作
- **局部控制**：使用 `Router.neglect()` 方法包装特定的导航操作，只影响该操作

## 页面组件

### Page1Screen - 第一个页面

```dart 41:70:example/lib/others/router_neglect.dart
/// The screen of the first page.
class Page1Screen extends StatelessWidget {
  /// Creates a [Page1Screen].
  const Page1Screen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () => context.go('/page2'),
            child: const Text('Go to page 2'),
          ),
          const SizedBox(height: 8),
          ElevatedButton(
            // turn off history tracking in the browser for this navigation;
            // note that this isn't necessary when you've set routerNeglect
            // but it does illustrate the technique
            onPressed: () =>
                Router.neglect(context, () => context.push('/page2')),
            child: const Text('Push page 2'),
          ),
        ],
      ),
    ),
  );
}
```

这个页面包含两个按钮，演示了两种不同的导航方式：

1. **普通导航**：`context.go('/page2')`
   - 会创建历史记录条目
   - 用户可以通过后退按钮返回

2. **忽略历史记录的导航**：`Router.neglect(context, () => context.push('/page2'))`
   - 不会创建历史记录条目
   - 用户无法通过后退按钮返回到这个导航操作

### Router.neglect() 方法

```dart
Router.neglect(context, () => context.push('/page2'))
```

这个方法接受两个参数：

- `context`：当前的 `BuildContext`
- 回调函数：包含需要忽略历史记录的导航操作

在回调函数内部执行的所有导航操作都不会被记录到浏览器历史记录中。

### Page2Screen - 第二个页面

```dart 72:92:example/lib/others/router_neglect.dart
/// The screen of the second page.
class Page2Screen extends StatelessWidget {
  /// Creates a [Page2Screen].
  const Page2Screen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          ElevatedButton(
            onPressed: () => context.go('/'),
            child: const Text('Go to home page'),
          ),
        ],
      ),
    ),
  );
}
```

第二个页面包含一个返回首页的按钮，使用普通的 `context.go('/')` 导航。

## 行为对比

### 使用 context.go()（普通导航）

1. 用户点击 "Go to page 2" 按钮
2. 导航到 `/page2`
3. **浏览器历史记录**：`[/] -> [/page2]`
4. 用户点击浏览器后退按钮 → 返回到 `/`
5. 用户再次点击浏览器后退按钮 → 可能退出应用或返回到之前的页面

### 使用 Router.neglect()（忽略历史记录）

1. 用户点击 "Push page 2" 按钮
2. 导航到 `/page2`（使用 `push` 方法）
3. **浏览器历史记录**：`[/]`（没有新增条目）
4. 用户点击浏览器后退按钮 → 直接返回到 `/` 之前的历史记录，跳过 `/page2`

## 技术细节

### Router.neglect() 的实现原理

`Router.neglect()` 方法通过设置一个特殊的上下文标志来工作：

1. 在调用回调函数之前，设置一个标志表示忽略历史记录
2. 在回调函数执行期间，所有导航操作都会检查这个标志
3. 如果标志存在，导航操作不会更新浏览器历史记录
4. 回调函数执行完毕后，标志被清除

### 与 context.push() 的区别

- `context.push()`：通常会在导航栈中添加新页面，但配合 `Router.neglect()` 时不会更新浏览器历史记录
- `context.go()`：替换当前路由，通常会更新浏览器历史记录

### 与全局 routerNeglect 的交互

如果全局设置了 `routerNeglect: true`：

- 所有导航操作默认都不会记录历史
- `Router.neglect()` 仍然可以正常工作，但效果与全局设置相同
- 如果需要某些导航记录历史，需要使用其他机制

## 实际应用场景

### 场景 1：登录页面

```dart
// 用户从首页导航到登录页，但不希望登录页出现在历史记录中
Router.neglect(context, () {
  context.push('/login');
});
```

这样，用户登录后，点击浏览器后退按钮不会返回到登录页。

### 场景 2：确认对话框

```dart
// 显示确认对话框，但不记录到历史
Router.neglect(context, () {
  context.push('/confirm');
});
```

### 场景 3：临时状态页面

```dart
// 显示加载或处理页面，完成后自动返回
Router.neglect(context, () {
  context.push('/processing');
  // 处理完成后自动返回，用户不会在历史记录中看到这个页面
});
```

## 注意事项

### 1. 用户体验考虑

- **谨慎使用**：过度使用 `routerNeglect` 可能会让用户感到困惑，因为他们无法通过后退按钮返回到某些页面
- **明确意图**：确保使用 `routerNeglect` 的场景符合用户的预期

### 2. 与浏览器行为的交互

- **Web 平台**：在 Web 平台上，`routerNeglect` 主要影响浏览器的历史记录 API
- **移动平台**：在移动平台上，可能影响系统返回按钮的行为（取决于实现）

### 3. 深链接考虑

- 如果使用 `routerNeglect`，某些路由可能无法通过 URL 直接访问
- 需要确保应用的核心功能路由不使用 `routerNeglect`

### 4. 测试建议

- 测试浏览器后退按钮的行为
- 测试在不同平台上的表现
- 确保关键导航路径不受影响

## 最佳实践

### 何时使用 routerNeglect

✅ **适合使用**：

- 临时页面（登录、确认、加载）
- 模态式导航
- 内部状态切换

❌ **不适合使用**：

- 主要功能页面
- 需要支持深链接的页面
- 用户可能想要返回的页面

### 代码示例

```dart
// 好的实践：临时登录页
void navigateToLogin(BuildContext context) {
  Router.neglect(context, () {
    context.push('/login');
  });
}

// 好的实践：主要功能页面
void navigateToProfile(BuildContext context) {
  context.go('/profile'); // 正常记录历史
}

// 避免：主要页面使用 routerNeglect
void navigateToHome(BuildContext context) {
  Router.neglect(context, () {
    context.go('/'); // 不推荐，用户可能想返回
  });
}
```

## 总结

`routerNeglect` 是一个强大的功能，允许开发者精确控制哪些导航操作应该被记录到浏览器历史记录中。正确使用可以改善用户体验，但需要谨慎考虑使用场景，确保符合用户的预期和应用的导航逻辑。
