# all_types.dart - 集合类型路由

本文档详细讲解 `all_types.dart` 文件中涉及集合类型的路由实现，包括 `Iterable`、`List`、`Set` 等集合类型的处理方式，以及多种元素类型的支持。

## 概述

集合类型路由展示了 `go_router_builder` 如何处理 Dart 中的集合类型（`Iterable`、`List`、`Set`）。集合类型在 URL 中通过查询参数传递，支持多种元素类型（`int`、`double`、`String`、`bool`、枚举等）。

## 集合类型说明

Dart 中有三种主要的集合类型：

1. **Iterable**：可迭代集合的抽象接口，`List` 和 `Set` 都实现了 `Iterable`
2. **List**：有序集合，允许重复元素，元素有索引
3. **Set**：无序集合，不允许重复元素

## IterableRoute

`IterableRoute` 展示了如何处理各种集合类型和元素类型的组合。

```dart 289:360:example/lib/all_types.dart
class IterableRoute extends GoRouteData with $IterableRoute {
  IterableRoute({
    this.intIterableField,
    this.doubleIterableField,
    this.stringIterableField,
    this.boolIterableField,
    this.enumIterableField,
    this.enumOnlyInIterableField,
    this.intListField,
    this.doubleListField,
    this.stringListField,
    this.boolListField,
    this.enumListField,
    this.enumOnlyInListField,
    this.intSetField,
    this.doubleSetField,
    this.stringSetField,
    this.boolSetField,
    this.enumSetField,
    this.enumOnlyInSetField,
  });

  final Iterable<int>? intIterableField;
  final List<int>? intListField;
  final Set<int>? intSetField;

  final Iterable<double>? doubleIterableField;
  final List<double>? doubleListField;
  final Set<double>? doubleSetField;

  final Iterable<String>? stringIterableField;
  final List<String>? stringListField;
  final Set<String>? stringSetField;

  final Iterable<bool>? boolIterableField;
  final List<bool>? boolListField;
  final Set<bool>? boolSetField;

  final Iterable<SportDetails>? enumIterableField;
  final List<SportDetails>? enumListField;
  final Set<SportDetails>? enumSetField;

  final Iterable<CookingRecipe>? enumOnlyInIterableField;
  final List<CookingRecipe>? enumOnlyInListField;
  final Set<CookingRecipe>? enumOnlyInSetField;

  @override
  Widget build(BuildContext context, GoRouterState state) => IterablePage(
    dataTitle: 'IterableRoute',
    intIterableField: intIterableField,
    doubleIterableField: doubleIterableField,
    stringIterableField: stringIterableField,
    boolIterableField: boolIterableField,
    enumIterableField: enumIterableField,
    intListField: intListField,
    doubleListField: doubleListField,
    stringListField: stringListField,
    boolListField: boolListField,
    enumListField: enumListField,
    intSetField: intSetField,
    doubleSetField: doubleSetField,
    stringSetField: stringSetField,
    boolSetField: boolSetField,
    enumSetField: enumSetField,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('IterableRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **所有参数都是可选的**：所有字段都是可空类型（`T?`），作为查询参数传递
- **多种集合类型**：支持 `Iterable`、`List`、`Set` 三种类型
- **多种元素类型**：支持 `int`、`double`、`String`、`bool`、枚举类型
- **URL 路径**：`/iterable-route`（没有路径参数，所有参数都是查询参数）

### 路由定义

```dart 30:30:example/lib/all_types.dart
    TypedGoRoute<IterableRoute>(path: 'iterable-route'),
```

注意路径中没有任何参数，因为集合类型只能作为查询参数传递。

### 字段分类

`IterableRoute` 中的字段可以按以下方式分类：

#### 按集合类型分类

- **Iterable 类型**：`intIterableField`、`doubleIterableField`、`stringIterableField`、`boolIterableField`、`enumIterableField`、`enumOnlyInIterableField`
- **List 类型**：`intListField`、`doubleListField`、`stringListField`、`boolListField`、`enumListField`、`enumOnlyInListField`
- **Set 类型**：`intSetField`、`doubleSetField`、`stringSetField`、`boolSetField`、`enumSetField`、`enumOnlyInSetField`

#### 按元素类型分类

- **int 类型**：`intIterableField`、`intListField`、`intSetField`
- **double 类型**：`doubleIterableField`、`doubleListField`、`doubleSetField`
- **String 类型**：`stringIterableField`、`stringListField`、`stringSetField`
- **bool 类型**：`boolIterableField`、`boolListField`、`boolSetField`
- **枚举类型**：
  - `SportDetails`：`enumIterableField`、`enumListField`、`enumSetField`
  - `CookingRecipe`：`enumOnlyInIterableField`、`enumOnlyInListField`、`enumOnlyInSetField`

### 使用示例

在 `BasePage` 的 drawer 中可以看到 `IterableRoute` 的使用示例：

```dart 489:514:example/lib/all_types.dart
          IterableRoute(
            intIterableField: <int>[1, 2, 3],
            doubleIterableField: <double>[.3, .4, .5],
            stringIterableField: <String>['quo usque tandem'],
            boolIterableField: <bool>[true, false, false],
            enumIterableField: <SportDetails>[
              SportDetails.football,
              SportDetails.hockey,
            ],
            intListField: <int>[1, 2, 3],
            doubleListField: <double>[.3, .4, .5],
            stringListField: <String>['quo usque tandem'],
            boolListField: <bool>[true, false, false],
            enumListField: <SportDetails>[
              SportDetails.football,
              SportDetails.hockey,
            ],
            intSetField: <int>{1, 2, 3},
            doubleSetField: <double>{.3, .4, .5},
            stringSetField: <String>{'quo usque tandem'},
            boolSetField: <bool>{true, false},
            enumSetField: <SportDetails>{
              SportDetails.football,
              SportDetails.hockey,
            },
          ).drawerTile(context),
```

## IterableRouteWithDefaultValues

`IterableRouteWithDefaultValues` 展示了带默认值的集合类型参数。

```dart 362:430:example/lib/all_types.dart
class IterableRouteWithDefaultValues extends GoRouteData
    with $IterableRouteWithDefaultValues {
  const IterableRouteWithDefaultValues({
    this.intIterableField = const <int>[0],
    this.doubleIterableField = const <double>[0, 1, 2],
    this.stringIterableField = const <String>['defaultValue'],
    this.boolIterableField = const <bool>[false],
    this.enumIterableField = const <SportDetails>[
      SportDetails.tennis,
      SportDetails.hockey,
    ],
    this.intListField = const <int>[0],
    this.doubleListField = const <double>[1, 2, 3],
    this.stringListField = const <String>['defaultValue0', 'defaultValue1'],
    this.boolListField = const <bool>[true],
    this.enumListField = const <SportDetails>[SportDetails.football],
    this.intSetField = const <int>{0, 1},
    this.doubleSetField = const <double>{},
    this.stringSetField = const <String>{'defaultValue'},
    this.boolSetField = const <bool>{true, false},
    this.enumSetField = const <SportDetails>{SportDetails.hockey},
  });

  final Iterable<int> intIterableField;
  final List<int> intListField;
  final Set<int> intSetField;

  final Iterable<double> doubleIterableField;
  final List<double> doubleListField;
  final Set<double> doubleSetField;

  final Iterable<String> stringIterableField;
  final List<String> stringListField;
  final Set<String> stringSetField;

  final Iterable<bool> boolIterableField;
  final List<bool> boolListField;
  final Set<bool> boolSetField;

  final Iterable<SportDetails> enumIterableField;
  final List<SportDetails> enumListField;
  final Set<SportDetails> enumSetField;

  @override
  Widget build(BuildContext context, GoRouterState state) => IterablePage(
    dataTitle: 'IterableRouteWithDefaultValues',
    intIterableField: intIterableField,
    doubleIterableField: doubleIterableField,
    stringIterableField: stringIterableField,
    boolIterableField: boolIterableField,
    enumIterableField: enumIterableField,
    intListField: intListField,
    doubleListField: doubleListField,
    stringListField: stringListField,
    boolListField: boolListField,
    enumListField: enumListField,
    intSetField: intSetField,
    doubleSetField: doubleSetField,
    stringSetField: stringSetField,
    boolSetField: boolSetField,
    enumSetField: enumSetField,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('IterableRouteWithDefaultValues'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **所有参数都有默认值**：所有字段都是非空类型，带有 `const` 常量作为默认值
- **const 构造函数**：使用 `const` 构造函数，因为所有默认值都是编译时常量
- **默认值示例**：
  - 空集合：`const <double>{}`（空的 Set）
  - 单个元素：`const <int>[0]`、`const <bool>[true]`
  - 多个元素：`const <double>[0, 1, 2]`、`const <String>['defaultValue0', 'defaultValue1']`
  - 枚举集合：`const <SportDetails>[SportDetails.football]`
- **URL 路径**：`/iterable-route-with-default-values`

### 路由定义

```dart 31:33:example/lib/all_types.dart
    TypedGoRoute<IterableRouteWithDefaultValues>(
      path: 'iterable-route-with-default-values',
    ),
```

### 与 IterableRoute 的对比

| 特性 | IterableRoute | IterableRouteWithDefaultValues |
| --- | --- | --- |
| **参数类型** | 可空类型（`T?`） | 非空类型（`T`） |
| **默认值** | 无（`null`） | 有（`const` 常量） |
| **构造函数** | 普通构造函数 | `const` 构造函数 |
| **必需性** | 所有参数可选 | 所有参数有默认值 |

### 使用示例

```dart 515:515:example/lib/all_types.dart
          const IterableRouteWithDefaultValues().drawerTile(context),
```

由于所有参数都有默认值，可以不带任何参数直接创建实例。

## 集合在 URL 中的表示

### 查询参数格式

集合类型作为查询参数传递，URL 格式如下：

```text
/iterable-route?intListField=1&intListField=2&intListField=3
```

### 多个值的传递

对于集合类型，同一个参数名可以出现多次，每个值对应集合中的一个元素：

- **List**：`?intListField=1&intListField=2&intListField=3` → `[1, 2, 3]`
- **Set**：`?intSetField=1&intSetField=2&intSetField=3` → `{1, 2, 3}`
- **Iterable**：处理方式与 List 相同

### 不同类型元素的序列化

不同元素类型的序列化方式：

1. **int/double**：直接使用数字字符串，如 `1`、`3.14`
2. **String**：进行 URL 编码，如 `hello%20world`
3. **bool**：使用字符串 `true` 或 `false`
4. **枚举**：使用枚举值的名称（可能经过格式转换），如 `football`、`favorite-food`

### 完整 URL 示例

```text
/iterable-route?
  intListField=1&intListField=2&intListField=3&
  stringListField=hello&stringListField=world&
  boolListField=true&boolListField=false&
  enumListField=football&enumListField=hockey
```

## 集合类型的选择

### Iterable vs List vs Set

选择合适的集合类型：

1. **Iterable**：
   - 最通用，适用于只需要迭代的场景
   - 不关心具体实现（List 或 Set）
   - 性能稍差（因为需要动态类型检查）

2. **List**：
   - 需要保持元素顺序
   - 允许重复元素
   - 需要通过索引访问元素
   - URL 中元素的顺序会被保留

3. **Set**：
   - 不需要保持元素顺序
   - 不允许重复元素
   - URL 中重复的值会被自动去重

### 在路由中的使用建议

- **使用 List**：当元素的顺序重要时（如步骤列表、优先级列表）
- **使用 Set**：当需要确保元素唯一时（如标签集合、权限集合）
- **使用 Iterable**：当只需要迭代，不关心具体实现时（较少使用）

## 代码生成处理

代码生成器会为集合类型生成特殊的处理逻辑：

### 序列化

将集合转换为 URL 查询参数：

```dart
// 伪代码示例
if (intListField != null) {
  for (final value in intListField) {
    queryParams['intListField'] = value.toString();
  }
}
```

### 反序列化

从 URL 查询参数解析集合：

```dart
// 伪代码示例
final values = uri.queryParametersAll['intListField'] ?? [];
final intListField = values.map((v) => int.parse(v)).toList();
```

对于 Set，还需要去重：

```dart
final intSetField = values.map((v) => int.parse(v)).toSet();
```

## 限制和注意事项

### 路径参数限制

集合类型**不能**作为路径参数，只能作为查询参数：

```dart
// ❌ 错误：集合类型不能作为路径参数
TypedGoRoute<MyRoute>(path: 'route/:listField')

// ✅ 正确：集合类型作为查询参数
TypedGoRoute<MyRoute>(path: 'route')
// listField 作为查询参数传递
```

### URL 长度限制

URL 有长度限制（通常为 2048 字符），大型集合可能导致 URL 过长：

- 如果集合很大，考虑使用其他方式传递数据（如通过 `extra` 参数）
- 或者将数据存储在服务器，只传递 ID

### 空集合的处理

- **可空集合**（`List<int>?`）：`null` 表示参数未提供
- **非空集合**（`List<int>`）：空集合 `[]` 表示参数提供了但为空
- **带默认值的集合**：如果不提供参数，使用默认值

## 总结

集合类型路由展示了 `go_router_builder` 对集合类型的完整支持：

1. **多种集合类型**：支持 `Iterable`、`List`、`Set` 三种类型
2. **多种元素类型**：支持 `int`、`double`、`String`、`bool`、枚举等元素类型
3. **查询参数传递**：集合类型只能作为查询参数，不能作为路径参数
4. **默认值支持**：可以为集合类型参数设置默认值
5. **类型安全**：所有集合和元素类型都在编译时确定
6. **URL 序列化**：自动处理集合在 URL 中的表示和解析

集合类型路由为复杂的参数传递提供了灵活且类型安全的解决方案，适用于需要传递多个值的场景。
