# redirection.dart 代码解析

## 概述

这个示例展示了如何使用 GoRouter 的 `redirect` 功能来处理应用的登录流程。通过重定向机制，可以实现以下功能：

- 未登录用户访问受保护页面时自动跳转到登录页
- 已登录用户访问登录页时自动跳转到主页
- 登录状态变化时自动更新路由

## 核心组件

### LoginInfo 类：登录状态管理

```dart 16:35:example/lib/redirection.dart
/// The login information.
class LoginInfo extends ChangeNotifier {
  /// The username of login.
  String get userName => _userName;
  String _userName = '';

  /// Whether a user has logged in.
  bool get loggedIn => _userName.isNotEmpty;

  /// Logs in a user.
  void login(String userName) {
    _userName = userName;
    notifyListeners();
  }

  /// Logs out the current user.
  void logout() {
    _userName = '';
    notifyListeners();
  }
}
```

`LoginInfo` 类继承自 `ChangeNotifier`，用于管理应用的登录状态：

- **`userName`**：存储当前登录用户的用户名
- **`loggedIn`**：通过检查用户名是否为空来判断用户是否已登录
- **`login()`**：登录方法，设置用户名并通知所有监听者
- **`logout()`**：登出方法，清空用户名并通知所有监听者

这个类作为可监听对象（Listenable），当状态发生变化时会通知 GoRouter 重新评估路由。

### App 类：主应用和路由配置

```dart 40:96:example/lib/redirection.dart
/// The main app.
class App extends StatelessWidget {
  /// Creates an [App].
  App({super.key});

  final LoginInfo _loginInfo = LoginInfo();

  /// The title of the app.
  static const String title = 'GoRouter Example: Redirection';

  // add the login info into the tree as app state that can change over time
  @override
  Widget build(BuildContext context) => ChangeNotifierProvider<LoginInfo>.value(
    value: _loginInfo,
    child: MaterialApp.router(
      routerConfig: _router,
      title: title,
      debugShowCheckedModeBanner: false,
    ),
  );

  late final GoRouter _router = GoRouter(
    routes: <GoRoute>[
      GoRoute(
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const HomeScreen(),
      ),
      GoRoute(
        path: '/login',
        builder: (BuildContext context, GoRouterState state) =>
            const LoginScreen(),
      ),
    ],

    // redirect to the login page if the user is not logged in
    redirect: (BuildContext context, GoRouterState state) {
      // if the user is not logged in, they need to login
      final bool loggedIn = _loginInfo.loggedIn;
      final loggingIn = state.matchedLocation == '/login';
      if (!loggedIn) {
        return '/login';
      }

      // if the user is logged in but still on the login page, send them to
      // the home page
      if (loggingIn) {
        return '/';
      }

      // no need to redirect at all
      return null;
    },

    // changes on the listenable will cause the router to refresh it's route
    refreshListenable: _loginInfo,
  );
}
```

`App` 类是应用的主入口，负责：

1. **创建登录状态实例**：`_loginInfo` 作为应用级别的状态管理
2. **提供状态到 Widget 树**：通过 `ChangeNotifierProvider` 将 `LoginInfo` 注入到 Widget 树中，使得子组件可以通过 `context.read<LoginInfo>()` 访问
3. **配置路由**：定义了两个路由：
   - `/`：主页路由，对应 `HomeScreen`
   - `/login`：登录页路由，对应 `LoginScreen`

#### 重定向逻辑详解

```dart 75:91:example/lib/redirection.dart
    // redirect to the login page if the user is not logged in
    redirect: (BuildContext context, GoRouterState state) {
      // if the user is not logged in, they need to login
      final bool loggedIn = _loginInfo.loggedIn;
      final loggingIn = state.matchedLocation == '/login';
      if (!loggedIn) {
        return '/login';
      }

      // if the user is logged in but still on the login page, send them to
      // the home page
      if (loggingIn) {
        return '/';
      }

      // no need to redirect at all
      return null;
    },
```

`redirect` 回调在导航到新页面之前被调用，逻辑如下：

1. **未登录用户**：如果用户未登录（`!loggedIn`），无论访问哪个页面，都重定向到 `/login`
2. **已登录用户在登录页**：如果用户已登录但仍在登录页（`loggingIn`），重定向到主页 `/`
3. **正常情况**：返回 `null` 表示不需要重定向，允许正常导航

#### refreshListenable 的作用

```dart 93:94:example/lib/redirection.dart
    // changes on the listenable will cause the router to refresh it's route
    refreshListenable: _loginInfo,
```

`refreshListenable` 参数指定了 `_loginInfo` 作为监听对象。当 `LoginInfo` 调用 `notifyListeners()` 时（比如登录或登出），GoRouter 会自动重新执行 `redirect` 回调，从而根据新的登录状态更新路由。

### LoginScreen：登录页面

```dart 98:119:example/lib/redirection.dart
/// The login screen.
class LoginScreen extends StatelessWidget {
  /// Creates a [LoginScreen].
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: Center(
      child: ElevatedButton(
        onPressed: () {
          // log a user in, letting all the listeners know
          context.read<LoginInfo>().login('test-user');

          // router will automatically redirect from /login to / using
          // refreshListenable
        },
        child: const Text('Login'),
      ),
    ),
  );
}
```

登录页面包含一个登录按钮：

- 点击按钮时调用 `context.read<LoginInfo>().login('test-user')` 进行登录
- 登录后，`LoginInfo` 会调用 `notifyListeners()` 通知所有监听者
- GoRouter 监听到变化后，会重新执行 `redirect` 回调
- 由于用户已登录且当前在登录页，`redirect` 会返回 `/`，自动跳转到主页

### HomeScreen：主页

```dart 121:144:example/lib/redirection.dart
/// The home screen.
class HomeScreen extends StatelessWidget {
  /// Creates a [HomeScreen].
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final LoginInfo info = context.read<LoginInfo>();

    return Scaffold(
      appBar: AppBar(
        title: const Text(App.title),
        actions: <Widget>[
          IconButton(
            onPressed: info.logout,
            tooltip: 'Logout: ${info.userName}',
            icon: const Icon(Icons.logout),
          ),
        ],
      ),
      body: const Center(child: Text('HomeScreen')),
    );
  }
}
```

主页显示：

- 应用标题
- 右上角的登出按钮，点击后会调用 `info.logout()`
- 登出后，`LoginInfo` 会通知监听者，GoRouter 重新评估路由，将用户重定向到登录页

## 工作流程

### 场景 1：未登录用户访问主页

1. 用户尝试访问 `/`
2. GoRouter 调用 `redirect` 回调
3. 检测到用户未登录（`!loggedIn`）
4. 返回 `/login`，重定向到登录页

### 场景 2：用户登录

1. 用户在登录页点击 "Login" 按钮
2. 调用 `login('test-user')`，设置用户名并触发 `notifyListeners()`
3. GoRouter 监听到 `LoginInfo` 的变化，重新执行 `redirect` 回调
4. 检测到用户已登录且当前在登录页（`loggingIn`）
5. 返回 `/`，自动跳转到主页

### 场景 3：已登录用户访问登录页

1. 已登录用户直接访问 `/login`
2. GoRouter 调用 `redirect` 回调
3. 检测到用户已登录且当前在登录页
4. 返回 `/`，重定向到主页

### 场景 4：用户登出

1. 用户在主页点击登出按钮
2. 调用 `logout()`，清空用户名并触发 `notifyListeners()`
3. GoRouter 监听到变化，重新执行 `redirect` 回调
4. 检测到用户未登录
5. 返回 `/login`，自动跳转到登录页

## 关键设计模式

### 1. 状态管理

使用 `ChangeNotifier` 和 `Provider` 模式管理登录状态，实现了状态与 UI 的解耦。

### 2. 响应式路由

通过 `refreshListenable` 实现路由的响应式更新，当登录状态变化时自动重新评估路由。

### 3. 集中式重定向逻辑

所有路由重定向逻辑集中在 `redirect` 回调中，便于维护和理解。

## 使用建议

1. **扩展登录验证**：可以在 `redirect` 中添加更复杂的验证逻辑，如检查 token 是否过期
2. **多级路由保护**：可以为不同的路由配置不同的重定向逻辑
3. **记住目标路由**：可以在重定向到登录页时保存原始目标路由，登录后跳转回原页面

## 总结

这个示例展示了 GoRouter 重定向功能的典型应用场景。通过结合 `ChangeNotifier`、`Provider` 和 `refreshListenable`，实现了一个完整的、响应式的登录流程管理方案。核心思想是：将路由决策逻辑集中在 `redirect` 回调中，通过监听状态变化自动更新路由。
