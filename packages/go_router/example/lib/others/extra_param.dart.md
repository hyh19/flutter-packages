# extra_param.dart 代码解析

## 概述

这个示例演示了如何在 GoRouter 中使用 `extra` 参数在路由导航时传递额外的数据。与通过 URL 路径参数传递数据不同，`extra` 参数允许传递任意类型的对象，这些数据不会出现在 URL 中，适合传递复杂对象或敏感信息。

## 数据结构

### Family 类

```dart 8:18:example/lib/others/extra_param.dart
/// Family data class.
class Family {
  /// Create a family.
  const Family({required this.name, required this.people});

  /// The last name of the family.
  final String name;

  /// The people in the family.
  final Map<String, Person> people;
}
```

`Family` 类表示一个家庭，包含：

- `name`: 家庭的姓氏
- `people`: 家庭成员映射，键为人员 ID，值为 `Person` 对象

### Person 类

```dart 20:27:example/lib/others/extra_param.dart
/// Person data class.
class Person {
  /// Creates a person.
  const Person({required this.name});

  /// The first name of the person.
  final String name;
}
```

`Person` 类表示一个人，包含：

- `name`: 人员的名字

### 数据源

```dart 29:44:example/lib/others/extra_param.dart
const Map<String, Family> _families = <String, Family>{
  'f1': Family(
    name: 'Doe',
    people: <String, Person>{
      'p1': Person(name: 'Jane'),
      'p2': Person(name: 'John'),
    },
  ),
  'f2': Family(
    name: 'Wong',
    people: <String, Person>{
      'p1': Person(name: 'June'),
      'p2': Person(name: 'Xin'),
    },
  ),
};
```

这里定义了两个家庭作为示例数据：

- `f1`: Doe 家庭，包含 Jane 和 John
- `f2`: Wong 家庭，包含 June 和 Xin

## 路由配置

### GoRouter 设置

```dart 60:81:example/lib/others/extra_param.dart
  late final GoRouter _router = GoRouter(
    routes: <GoRoute>[
      GoRoute(
        name: 'home',
        path: '/',
        builder: (BuildContext context, GoRouterState state) =>
            const HomeScreen(),
        routes: <GoRoute>[
          GoRoute(
            name: 'family',
            path: 'family',
            builder: (BuildContext context, GoRouterState state) {
              final Map<String, Object> params =
                  state.extra! as Map<String, String>;
              final fid = params['fid']! as String;
              return FamilyScreen(fid: fid);
            },
          ),
        ],
      ),
    ],
  );
```

关键点：

1. **路由定义**：`family` 路由的路径是 `'family'`，完整路径为 `/family`
2. **extra 参数获取**：在 `builder` 中通过 `state.extra` 获取传递的额外数据
3. **类型转换**：将 `state.extra` 转换为 `Map<String, String>`，然后提取 `fid`（家庭 ID）

**注意**：这里使用了 `!` 非空断言，实际项目中应该添加空值检查。

## 页面组件

### HomeScreen - 首页

```dart 84:105:example/lib/others/extra_param.dart
/// The home screen that shows a list of families.
class HomeScreen extends StatelessWidget {
  /// Creates a [HomeScreen].
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text(App.title)),
    body: ListView(
      children: <Widget>[
        for (final MapEntry<String, Family> entry in _families.entries)
          ListTile(
            title: Text(entry.value.name),
            onTap: () => context.goNamed(
              'family',
              extra: <String, String>{'fid': entry.key},
            ),
          ),
      ],
    ),
  );
}
```

功能说明：

1. **显示家庭列表**：遍历 `_families` 映射，为每个家庭创建一个 `ListTile`
2. **导航时传递 extra**：点击列表项时，使用 `context.goNamed()` 导航到 `family` 路由
3. **extra 参数**：通过 `extra` 参数传递一个包含 `fid` 的 Map，这是家庭 ID

**关键代码**：

```dart
context.goNamed(
  'family',
  extra: <String, String>{'fid': entry.key},
)
```

这里 `entry.key` 是家庭 ID（如 `'f1'` 或 `'f2'`），通过 `extra` 参数传递给目标路由。

### FamilyScreen - 家庭成员页面

```dart 107:127:example/lib/others/extra_param.dart
/// The screen that shows a list of persons in a family.
class FamilyScreen extends StatelessWidget {
  /// Creates a [FamilyScreen].
  const FamilyScreen({required this.fid, super.key});

  /// The family to display.
  final String fid;

  @override
  Widget build(BuildContext context) {
    final Map<String, Person> people = _families[fid]!.people;
    return Scaffold(
      appBar: AppBar(title: Text(_families[fid]!.name)),
      body: ListView(
        children: <Widget>[
          for (final Person p in people.values) ListTile(title: Text(p.name)),
        ],
      ),
    );
  }
}
```

功能说明：

1. **接收参数**：通过构造函数接收 `fid`（家庭 ID）
2. **获取数据**：使用 `fid` 从 `_families` 中查找对应的家庭数据
3. **显示成员列表**：遍历家庭成员并显示在列表中
4. **AppBar 标题**：显示家庭的姓氏

## 使用场景

### 何时使用 extra 参数

1. **传递复杂对象**：当需要传递的数据不适合放在 URL 中时
2. **保护敏感信息**：避免在 URL 中暴露敏感数据
3. **传递非字符串数据**：传递对象、列表等复杂数据结构
4. **临时数据**：不需要在浏览器历史记录中保存的数据

### 与路径参数的区别

- **路径参数**：出现在 URL 中，如 `/family/f1`，可以通过浏览器地址栏直接访问
- **extra 参数**：不出现在 URL 中，只能通过程序导航传递，无法通过 URL 直接访问

## 注意事项

1. **空值检查**：实际项目中应该检查 `state.extra` 是否为 null
2. **类型安全**：确保 `extra` 的类型与预期一致，避免类型转换错误
3. **数据持久化**：`extra` 参数不会保存在浏览器历史记录中，刷新页面会丢失
4. **深链接限制**：使用 `extra` 参数的路由无法通过 URL 直接访问，因为 URL 中不包含这些数据

## 改进建议

### 更安全的实现

```dart
builder: (BuildContext context, GoRouterState state) {
  if (state.extra == null) {
    // 处理 extra 为 null 的情况
    return const ErrorScreen(message: 'Missing family ID');
  }
  
  final Map<String, Object>? params = state.extra as Map<String, Object>?;
  final fid = params?['fid'] as String?;
  
  if (fid == null || !_families.containsKey(fid)) {
    // 处理无效的 fid
    return const ErrorScreen(message: 'Invalid family ID');
  }
  
  return FamilyScreen(fid: fid);
}
```

这样可以更好地处理边界情况，提高代码的健壮性。
