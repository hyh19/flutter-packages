# simple_example.dart 代码解析

## 概述

这是一个使用 `go_router_builder` 包的简单示例，展示了如何创建类型安全的路由系统。该示例演示了基本的嵌套路由结构，包含一个首页路由和一个带参数的家庭详情路由。

## 文件结构

文件主要包含以下几个部分：

1. **应用入口和路由配置**：`App` 类和 `main` 函数
2. **路由定义**：`HomeRoute` 和 `FamilyRoute`
3. **UI 组件**：`HomeScreen` 和 `FamilyScreen`

## 代码生成机制

文件使用了 Dart 的代码生成功能：

```dart 12:12:example/lib/simple_example.dart
part 'simple_example.g.dart';
```

这行代码声明了该文件是 `simple_example.g.dart` 的一部分。`go_router_builder` 会在构建时生成这个 `.g.dart` 文件，其中包含：

- `$appRoutes`：应用的所有路由列表
- `$HomeRoute` 和 `$FamilyRoute`：为每个路由类生成的 mixin，提供类型安全的导航方法

## 应用入口

### main 函数

```dart 14:14:example/lib/simple_example.dart
void main() => runApp(App());
```

标准的 Flutter 应用入口点，启动 `App` 组件。

### App 类

```dart 16:24:example/lib/simple_example.dart
class App extends StatelessWidget {
  App({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp.router(routerConfig: _router, title: _appTitle);

  final GoRouter _router = GoRouter(routes: $appRoutes);
}
```

`App` 类负责初始化应用的路由系统：

- **`MaterialApp.router`**：使用 `go_router` 的 `MaterialApp.router` 构造函数，这是使用 `go_router` 的标准方式
- **`routerConfig`**：传入 `GoRouter` 实例，配置应用的路由
- **`$appRoutes`**：这是代码生成器自动生成的变量，包含所有通过 `@TypedGoRoute` 注解定义的路由

## 路由定义

### HomeRoute - 根路由

```dart 26:38:example/lib/simple_example.dart
@TypedGoRoute<HomeRoute>(
  path: '/',
  name: 'Home',
  routes: <TypedGoRoute<GoRouteData>>[
    TypedGoRoute<FamilyRoute>(path: 'family/:familyId'),
  ],
)
class HomeRoute extends GoRouteData with $HomeRoute {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) => const HomeScreen();
}
```

这是应用的根路由，展示了几个关键概念：

#### @TypedGoRoute 注解

- **`path: '/'`**：定义路由路径为根路径
- **`name: 'Home'`**：为路由指定一个名称，可用于命名路由导航
- **`routes`**：定义嵌套路由列表，这里包含了 `FamilyRoute` 作为子路由

#### 嵌套路由

`FamilyRoute` 被定义为 `HomeRoute` 的子路由，路径为 `'family/:familyId'`。由于是嵌套路由，完整的路径会是 `/family/:familyId`（父路径 `/` + 子路径 `family/:familyId`）。

#### 路由类结构

- **`extends GoRouteData`**：所有路由类必须继承 `GoRouteData`
- **`with $HomeRoute`**：混入代码生成器生成的 mixin，提供类型安全的导航方法（如 `go()`、`push()` 等）
- **`build` 方法**：返回该路由对应的 UI 组件

### FamilyRoute - 带参数的路由

```dart 40:49:example/lib/simple_example.dart
class FamilyRoute extends GoRouteData with $FamilyRoute {
  const FamilyRoute(this.familyId);

  final String familyId;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return FamilyScreen(family: familyById(familyId));
  }
}
```

这个路由展示了如何处理路径参数：

#### 路径参数

- **`path: 'family/:familyId'`**：在父路由的 `routes` 中定义，`:familyId` 表示这是一个路径参数
- **`final String familyId`**：路由类的字段会自动从 URL 路径中提取对应的参数值
- **类型安全**：参数类型在编译时确定，避免了运行时类型转换错误

#### 参数使用

在 `build` 方法中，`familyId` 被用于查找对应的家庭数据：

```dart 47:47:example/lib/simple_example.dart
    return FamilyScreen(family: familyById(familyId));
```

`familyById` 函数（定义在 `shared/data.dart` 中）根据 ID 查找对应的 `Family` 对象。

## UI 组件

### HomeScreen - 首页

```dart 51:67:example/lib/simple_example.dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(_appTitle)),
    body: ListView(
      children: <Widget>[
        for (final Family family in familyData)
          ListTile(
            title: Text(family.name),
            onTap: () => FamilyRoute(family.id).go(context),
          ),
      ],
    ),
  );
}
```

首页展示了所有家庭列表，关键点：

#### 数据展示

- **`familyData`**：从 `shared/data.dart` 导入的示例数据，包含多个家庭信息
- **`ListView`**：使用列表视图展示所有家庭

#### 类型安全导航

```dart 62:62:example/lib/simple_example.dart
            onTap: () => FamilyRoute(family.id).go(context),
```

这是 `go_router_builder` 的核心优势：

- **`FamilyRoute(family.id)`**：创建路由实例，传入必需的 `familyId` 参数
- **`.go(context)`**：这是代码生成器在 `$FamilyRoute` mixin 中生成的方法，执行导航
- **类型检查**：如果忘记传入 `familyId` 或传入错误类型，编译器会报错

### FamilyScreen - 家庭详情页

```dart 69:82:example/lib/simple_example.dart
class FamilyScreen extends StatelessWidget {
  const FamilyScreen({required this.family, super.key});
  final Family family;

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: Text(family.name)),
    body: ListView(
      children: <Widget>[
        for (final Person p in family.people) ListTile(title: Text(p.name)),
      ],
    ),
  );
}
```

家庭详情页显示该家庭的所有成员：

- **接收数据**：通过构造函数接收 `Family` 对象
- **显示成员**：遍历 `family.people` 列表，显示每个成员的姓名

## 路由导航流程

完整的导航流程如下：

1. **用户点击列表项**：在 `HomeScreen` 中点击某个家庭
2. **创建路由实例**：`FamilyRoute(family.id)` 创建路由对象
3. **执行导航**：调用 `.go(context)` 方法
4. **URL 生成**：代码生成器将路由参数转换为 URL（如 `/family/f1`）
5. **路由匹配**：`GoRouter` 匹配对应的路由
6. **参数提取**：从 URL 中提取 `familyId` 参数
7. **构建页面**：调用 `FamilyRoute.build()` 方法，创建 `FamilyScreen`

## 类型安全的优势

相比传统的字符串路由，这个示例展示了类型安全的优势：

### 传统方式的问题

```dart
// 传统方式 - 容易出错
context.go('/family/${family.id}'); // 路径拼写错误只能在运行时发现
```

### 类型安全方式

```dart
// 类型安全方式 - 编译时检查
FamilyRoute(family.id).go(context); // 如果参数缺失或类型错误，编译时就会报错
```

## 关键特性总结

1. **代码生成**：通过 `@TypedGoRoute` 注解和 `part` 声明，自动生成路由配置和导航方法
2. **类型安全**：路由参数在编译时进行类型检查
3. **嵌套路由**：支持路由的嵌套结构，便于组织复杂的路由层次
4. **路径参数**：通过 `:参数名` 语法定义路径参数，自动提取和类型转换
5. **IDE 支持**：完整的自动补全和类型提示

## 扩展建议

基于这个简单示例，可以进一步扩展：

- 添加查询参数（如分页、排序）
- 添加重定向逻辑
- 添加路由守卫（如登录检查）
- 使用 ShellRoute 实现更复杂的布局结构
- 添加路由过渡动画

这些高级特性在 `go_router_builder` 中都有支持，可以参考项目中的其他示例文件。
