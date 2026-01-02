# all_types.dart - 基础数据类型路由

本文档详细讲解 `all_types.dart` 文件中涉及基础数据类型的路由实现，包括 `BigInt`、`bool`、`DateTime`、`double`、`int`、`num`、`String`、`Uri` 等类型的路由处理。

## 概述

基础数据类型路由展示了如何在 `go_router_builder` 中处理 Dart 的基本数据类型。这些路由都遵循相似的模式，主要区别在于参数的类型和处理方式。

## 路由列表

本部分涵盖以下路由类：

1. **BigIntRoute**：处理 `BigInt` 类型（任意精度整数）
2. **BoolRoute**：处理 `bool` 类型（布尔值）
3. **DateTimeRoute**：处理 `DateTime` 类型（日期时间）
4. **DoubleRoute**：处理 `double` 类型（双精度浮点数）
5. **IntRoute**：处理 `int` 类型（整数）
6. **NumRoute**：处理 `num` 类型（数字，`int` 和 `double` 的父类）
7. **StringRoute**：处理 `String` 类型（字符串）
8. **UriRoute**：处理 `Uri` 类型（URI）

## BigIntRoute

`BigIntRoute` 展示了如何处理任意精度的整数类型。

```dart 45:63:example/lib/all_types.dart
class BigIntRoute extends GoRouteData with $BigIntRoute {
  BigIntRoute({required this.requiredBigIntField, this.bigIntField});

  final BigInt requiredBigIntField;
  final BigInt? bigIntField;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<BigInt>(
    dataTitle: 'BigIntRoute',
    param: requiredBigIntField,
    queryParam: bigIntField,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('BigIntRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredBigIntField` 是必需的 `BigInt` 类型参数，定义在路径中
- **查询参数**：`bigIntField` 是可选的 `BigInt?` 类型，通过查询参数传递
- **URL 路径**：`/big-int-route/:requiredBigIntField`

### 路由定义

```dart 17:17:example/lib/all_types.dart
    TypedGoRoute<BigIntRoute>(path: 'big-int-route/:requiredBigIntField'),
```

## BoolRoute

`BoolRoute` 展示了如何处理布尔类型，包括带默认值的参数。

```dart 65:89:example/lib/all_types.dart
class BoolRoute extends GoRouteData with $BoolRoute {
  BoolRoute({
    required this.requiredBoolField,
    this.boolField,
    this.boolFieldWithDefaultValue = true,
  });

  final bool requiredBoolField;
  final bool? boolField;
  final bool boolFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<bool>(
    dataTitle: 'BoolRoute',
    param: requiredBoolField,
    queryParam: boolField,
    queryParamWithDefaultValue: boolFieldWithDefaultValue,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('BoolRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **必需参数**：`requiredBoolField` 是必需的 `bool` 类型，从路径参数中提取
- **可选参数**：`boolField` 是可空的 `bool?` 类型，从查询参数中提取
- **默认值参数**：`boolFieldWithDefaultValue` 有默认值 `true`，如果查询参数不存在则使用默认值
- **URL 路径**：`/bool-route/:requiredBoolField`

### 三种参数模式

`BoolRoute` 展示了三种不同的参数处理模式：

1. **必需路径参数**：`required bool requiredBoolField`
2. **可选查询参数**：`bool? boolField`
3. **带默认值的查询参数**：`bool boolFieldWithDefaultValue = true`

## DateTimeRoute

`DateTimeRoute` 展示了如何处理日期时间类型。

```dart 91:109:example/lib/all_types.dart
class DateTimeRoute extends GoRouteData with $DateTimeRoute {
  DateTimeRoute({required this.requiredDateTimeField, this.dateTimeField});

  final DateTime requiredDateTimeField;
  final DateTime? dateTimeField;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<DateTime>(
    dataTitle: 'DateTimeRoute',
    param: requiredDateTimeField,
    queryParam: dateTimeField,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('DateTimeRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredDateTimeField` 是必需的 `DateTime` 类型
- **查询参数**：`dateTimeField` 是可选的 `DateTime?` 类型
- **URL 路径**：`/date-time-route/:requiredDateTimeField`
- **序列化**：`DateTime` 在 URL 中通常被序列化为 ISO 8601 格式字符串

## DoubleRoute

`DoubleRoute` 展示了如何处理双精度浮点数类型，包含默认值示例。

```dart 111:135:example/lib/all_types.dart
class DoubleRoute extends GoRouteData with $DoubleRoute {
  DoubleRoute({
    required this.requiredDoubleField,
    this.doubleField,
    this.doubleFieldWithDefaultValue = 1.0,
  });

  final double requiredDoubleField;
  final double? doubleField;
  final double doubleFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<double>(
    dataTitle: 'DoubleRoute',
    param: requiredDoubleField,
    queryParam: doubleField,
    queryParamWithDefaultValue: doubleFieldWithDefaultValue,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('DoubleRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredDoubleField` 是必需的 `double` 类型
- **可选参数**：`doubleField` 是可空的 `double?` 类型
- **默认值**：`doubleFieldWithDefaultValue` 默认值为 `1.0`
- **URL 路径**：`/double-route/:requiredDoubleField`

## IntRoute

`IntRoute` 展示了如何处理整数类型。

```dart 137:161:example/lib/all_types.dart
class IntRoute extends GoRouteData with $IntRoute {
  IntRoute({
    required this.requiredIntField,
    this.intField,
    this.intFieldWithDefaultValue = 1,
  });

  final int requiredIntField;
  final int? intField;
  final int intFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<int>(
    dataTitle: 'IntRoute',
    param: requiredIntField,
    queryParam: intField,
    queryParamWithDefaultValue: intFieldWithDefaultValue,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('IntRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredIntField` 是必需的 `int` 类型
- **可选参数**：`intField` 是可空的 `int?` 类型
- **默认值**：`intFieldWithDefaultValue` 默认值为 `1`
- **URL 路径**：`/int-route/:requiredIntField`

## NumRoute

`NumRoute` 展示了如何处理 `num` 类型（`int` 和 `double` 的父类）。

```dart 163:187:example/lib/all_types.dart
class NumRoute extends GoRouteData with $NumRoute {
  NumRoute({
    required this.requiredNumField,
    this.numField,
    this.numFieldWithDefaultValue = 1,
  });

  final num requiredNumField;
  final num? numField;
  final num numFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<num>(
    dataTitle: 'NumRoute',
    param: requiredNumField,
    queryParam: numField,
    queryParamWithDefaultValue: numFieldWithDefaultValue,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('NumRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **类型灵活性**：`num` 类型可以接受 `int` 或 `double` 值
- **路径参数**：`requiredNumField` 是必需的 `num` 类型
- **可选参数**：`numField` 是可空的 `num?` 类型
- **默认值**：`numFieldWithDefaultValue` 默认值为 `1`（整数）
- **URL 路径**：`/num-route/:requiredNumField`

## StringRoute

`StringRoute` 展示了如何处理字符串类型，包括特殊字符的处理。

```dart 243:267:example/lib/all_types.dart
class StringRoute extends GoRouteData with $StringRoute {
  StringRoute({
    required this.requiredStringField,
    this.stringField,
    this.stringFieldWithDefaultValue = 'defaultValue',
  });

  final String requiredStringField;
  final String? stringField;
  final String stringFieldWithDefaultValue;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<String>(
    dataTitle: 'StringRoute',
    param: requiredStringField,
    queryParam: stringField,
    queryParamWithDefaultValue: stringFieldWithDefaultValue,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('StringRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredStringField` 是必需的 `String` 类型
- **可选参数**：`stringField` 是可空的 `String?` 类型
- **默认值**：`stringFieldWithDefaultValue` 默认值为 `'defaultValue'`
- **URL 路径**：`/string-route/:requiredStringField`
- **URL 编码**：字符串中的特殊字符会自动进行 URL 编码

### 使用示例

在代码中可以看到字符串参数的使用，包括特殊字符：

```dart 473:476:example/lib/all_types.dart
          StringRoute(
            requiredStringField: r'$!/#bob%%20',
            stringField: r'$!/#bob%%20',
          ).drawerTile(context),
```

这里使用了原始字符串（`r''`），包含特殊字符，演示了 URL 编码的处理。

## UriRoute

`UriRoute` 展示了如何处理 URI 类型。

```dart 269:287:example/lib/all_types.dart
class UriRoute extends GoRouteData with $UriRoute {
  UriRoute({required this.requiredUriField, this.uriField});

  final Uri requiredUriField;
  final Uri? uriField;

  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<Uri>(
    dataTitle: 'UriRoute',
    param: requiredUriField,
    queryParam: uriField,
  );

  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('UriRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
}
```

### 特点

- **路径参数**：`requiredUriField` 是必需的 `Uri` 类型
- **查询参数**：`uriField` 是可选的 `Uri?` 类型
- **URL 路径**：`/uri-route/:requiredUriField`
- **序列化**：`Uri` 对象会被序列化为字符串形式存储在 URL 中

### 使用示例

```dart 485:488:example/lib/all_types.dart
          UriRoute(
            requiredUriField: Uri.parse('https://dart.dev'),
            uriField: Uri.parse('https://dart.dev'),
          ).drawerTile(context),
```

## 通用模式

### 参数类型总结

所有基础数据类型路由都遵循相同的参数模式：

1. **必需路径参数**：使用 `required` 关键字，类型为非空类型（如 `BigInt`、`bool`、`DateTime` 等）
2. **可选查询参数**：使用可空类型（如 `BigInt?`、`bool?` 等）
3. **带默认值的查询参数**：使用非空类型 + 默认值（如 `bool boolFieldWithDefaultValue = true`）

### drawerTile 方法

所有路由类都实现了 `drawerTile` 方法，用于在导航抽屉中显示列表项：

```dart 58:62:example/lib/all_types.dart
  Widget drawerTile(BuildContext context) => ListTile(
    title: const Text('BigIntRoute'),
    onTap: () => go(context),
    selected: GoRouterState.of(context).uri.path == location,
  );
```

**功能说明**：

- **标题**：显示路由名称
- **点击事件**：调用 `go(context)` 进行导航
- **选中状态**：根据当前路径判断是否选中（高亮显示）

### build 方法

所有路由类都实现了 `build` 方法，返回 `BasePage` 组件：

```dart 51:56:example/lib/all_types.dart
  @override
  Widget build(BuildContext context, GoRouterState state) => BasePage<BigInt>(
    dataTitle: 'BigIntRoute',
    param: requiredBigIntField,
    queryParam: bigIntField,
  );
```

**参数传递**：

- **dataTitle**：页面标题，用于显示路由名称
- **param**：路径参数的值
- **queryParam**：查询参数的值（可选）
- **queryParamWithDefaultValue**：带默认值的查询参数（如果路由支持）

## URL 格式示例

以下是各种路由类型的 URL 格式示例：

- **BigIntRoute**：`/big-int-route/12345678901234567890?bigIntField=987654321`
- **BoolRoute**：`/bool-route/true?boolField=false&boolFieldWithDefaultValue=false`
- **DateTimeRoute**：`/date-time-route/2023-05-20T10:30:00Z?dateTimeField=2023-05-21T12:00:00Z`
- **DoubleRoute**：`/double-route/3.14?doubleField=-3.14`
- **IntRoute**：`/int-route/42?intField=-42`
- **NumRoute**：`/num-route/2.71828?numField=-2.71828`
- **StringRoute**：`/string-route/hello?stringField=world`
- **UriRoute**：`/uri-route/https%3A%2F%2Fdart.dev?uriField=https%3A%2F%2Fflutter.dev`

## 总结

基础数据类型路由展示了 `go_router_builder` 对各种基本数据类型的支持：

1. **类型安全**：所有参数都有明确的类型，编译时检查
2. **灵活的参数处理**：支持必需参数、可选参数和带默认值的参数
3. **URL 编码**：特殊字符自动进行 URL 编码/解码
4. **统一的模式**：所有路由类遵循相同的代码结构
5. **类型转换**：框架自动处理字符串与目标类型之间的转换

这些路由为开发者提供了清晰的参考，展示了如何在类型安全的路由系统中处理各种基础数据类型。
