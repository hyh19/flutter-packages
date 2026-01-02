# extra_codec.dart 代码解析

## 概述

这个示例应用演示了如何在 GoRouter 中为复杂的 `extra` 数据提供自定义编解码器（codec）。当你在导航时传递复杂对象作为 `extra` 参数时，特别是在 Web 平台上，这些数据需要被序列化和反序列化以支持浏览器的前进/后退功能。如果没有提供合适的编解码器，复杂对象可能会在序列化过程中丢失。

## 应用结构

### 主入口和路由配置

```dart 11:23:example/lib/extra_codec.dart
void main() => runApp(const MyApp());

/// The router configuration.
final GoRouter _router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) =>
          const HomeScreen(),
    ),
  ],
  extraCodec: const MyExtraCodec(),
);
```

应用入口很简单，直接运行 `MyApp`。路由配置中关键的部分是 `extraCodec: const MyExtraCodec()`，这告诉 GoRouter 使用自定义的编解码器来处理 `extra` 数据。

### 主应用组件

```dart 25:34:example/lib/extra_codec.dart
/// The main app.
class MyApp extends StatelessWidget {
  /// Constructs a [MyApp]
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(routerConfig: _router);
  }
}
```

标准的 Flutter 应用结构，使用 `MaterialApp.router` 并传入配置好的 `_router`。

## 首页界面

```dart 36:68:example/lib/extra_codec.dart
/// The home screen.
class HomeScreen extends StatelessWidget {
  /// Constructs a [HomeScreen].
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home Screen')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              "If running in web, use the browser's backward and forward button to test extra codec after setting extra several times.",
            ),
            Text(
              'The extra for this page is: ${GoRouterState.of(context).extra}',
            ),
            ElevatedButton(
              onPressed: () => context.go('/', extra: ComplexData1('data')),
              child: const Text('Set extra to ComplexData1'),
            ),
            ElevatedButton(
              onPressed: () => context.go('/', extra: ComplexData2('data')),
              child: const Text('Set extra to ComplexData2'),
            ),
          ],
        ),
      ),
    );
  }
}
```

首页展示了如何使用 `extra` 参数：

1. **显示当前 extra 数据**：通过 `GoRouterState.of(context).extra` 获取当前路由的 `extra` 数据并显示
2. **设置不同的 extra 数据**：两个按钮分别设置 `ComplexData1` 和 `ComplexData2` 类型的 `extra` 数据

在 Web 平台上，当你点击按钮设置 `extra` 后，使用浏览器的前进/后退按钮时，`extra` 数据应该能够正确恢复，这得益于自定义编解码器的工作。

## 复杂数据类

示例中定义了两个简单的复杂数据类：

```dart 70:80:example/lib/extra_codec.dart
/// A complex class.
class ComplexData1 {
  /// Create a complex object.
  ComplexData1(this.data);

  /// The data.
  final String data;

  @override
  String toString() => 'ComplexData1(data: $data)';
}
```

```dart 82:92:example/lib/extra_codec.dart
/// A complex class.
class ComplexData2 {
  /// Create a complex object.
  ComplexData2(this.data);

  /// The data.
  final String data;

  @override
  String toString() => 'ComplexData2(data: $data)';
}
```

这两个类结构相同，都包含一个 `String` 类型的 `data` 字段。它们代表需要在导航时传递的复杂对象。在实际应用中，这些类可能包含更多字段或更复杂的结构。

## 自定义编解码器

### Codec 接口实现

```dart 94:103:example/lib/extra_codec.dart
/// A codec that can serialize both [ComplexData1] and [ComplexData2].
class MyExtraCodec extends Codec<Object?, Object?> {
  /// Create a codec.
  const MyExtraCodec();
  @override
  Converter<Object?, Object?> get decoder => const _MyExtraDecoder();

  @override
  Converter<Object?, Object?> get encoder => const _MyExtraEncoder();
}
```

`MyExtraCodec` 实现了 Dart 的 `Codec<Object?, Object?>` 接口，它需要提供两个转换器：

- **encoder**：将复杂对象编码为可序列化的格式（通常是基本类型或列表/映射）
- **decoder**：将序列化后的数据解码回原始对象

### 编码器实现

```dart 123:139:example/lib/extra_codec.dart
class _MyExtraEncoder extends Converter<Object?, Object?> {
  const _MyExtraEncoder();
  @override
  Object? convert(Object? input) {
    if (input == null) {
      return null;
    }
    switch (input) {
      case ComplexData1 _:
        return <Object?>['ComplexData1', input.data];
      case ComplexData2 _:
        return <Object?>['ComplexData2', input.data];
      default:
        throw FormatException('Cannot encode type ${input.runtimeType}');
    }
  }
}
```

编码器的工作流程：

1. **处理 null 值**：如果输入为 `null`，直接返回 `null`
2. **类型识别**：使用模式匹配（pattern matching）识别对象类型
3. **序列化**：将对象转换为列表格式 `[类型标识符, 数据]`
   - `ComplexData1` 编码为 `['ComplexData1', data]`
   - `ComplexData2` 编码为 `['ComplexData2', data]`
4. **错误处理**：如果遇到不支持的类型，抛出 `FormatException`

这种编码方式简单有效，使用列表的第一个元素作为类型标识符，第二个元素存储实际数据。

### 解码器实现

```dart 105:121:example/lib/extra_codec.dart
class _MyExtraDecoder extends Converter<Object?, Object?> {
  const _MyExtraDecoder();
  @override
  Object? convert(Object? input) {
    if (input == null) {
      return null;
    }
    final inputAsList = input as List<Object?>;
    if (inputAsList[0] == 'ComplexData1') {
      return ComplexData1(inputAsList[1]! as String);
    }
    if (inputAsList[0] == 'ComplexData2') {
      return ComplexData2(inputAsList[1]! as String);
    }
    throw FormatException('Unable to parse input: $input');
  }
}
```

解码器的工作流程：

1. **处理 null 值**：如果输入为 `null`，直接返回 `null`
2. **类型转换**：将输入转换为列表类型
3. **类型识别**：根据列表的第一个元素（类型标识符）判断应该创建哪种对象
4. **对象重建**：使用列表的第二个元素（数据）创建对应的对象实例
5. **错误处理**：如果无法识别类型，抛出 `FormatException`

## 工作原理

### 序列化流程

当你在导航时传递 `extra` 数据时：

1. **编码阶段**：GoRouter 调用 `MyExtraCodec.encoder` 将 `ComplexData1` 或 `ComplexData2` 对象编码为列表格式
2. **存储阶段**：编码后的数据（可序列化的列表）被存储到浏览器的历史记录中
3. **解码阶段**：当用户使用浏览器前进/后退按钮时，GoRouter 从历史记录中读取数据，调用 `MyExtraCodec.decoder` 将列表解码回原始对象
4. **使用阶段**：解码后的对象可以通过 `GoRouterState.of(context).extra` 访问

### 为什么需要编解码器？

在 Web 平台上，浏览器的历史记录 API 只能存储可序列化的数据（JSON 可序列化的基本类型、列表、映射等）。如果你直接传递一个 Dart 对象作为 `extra`，它无法被正确序列化，导致：

- 数据丢失
- 类型信息丢失
- 浏览器前进/后退时无法恢复原始对象

通过提供自定义编解码器，你可以：

- 将复杂对象转换为可序列化的格式
- 在需要时恢复原始对象
- 保持类型安全

## 使用场景

这个示例特别适用于以下场景：

1. **Web 应用**：需要在浏览器历史记录中保存复杂状态
2. **深度链接**：通过 URL 传递复杂数据对象
3. **状态恢复**：在应用重启或页面刷新后恢复导航状态
4. **类型安全**：需要在导航时传递强类型的复杂对象

## 扩展建议

在实际应用中，你可能需要：

1. **支持更多类型**：在编码器和解码器中添加更多数据类型的支持
2. **版本控制**：如果数据结构可能变化，在编码格式中包含版本号
3. **错误恢复**：提供更优雅的错误处理，而不是直接抛出异常
4. **性能优化**：对于大型对象，考虑使用更高效的序列化格式（如 JSON）

## 总结

这个示例展示了 GoRouter 中 `extraCodec` 的完整使用方式，包括：

- 如何定义复杂数据类
- 如何实现自定义编解码器
- 如何在导航时使用 `extra` 参数
- 如何在页面中访问 `extra` 数据

通过这个模式，你可以在 Web 应用中安全地传递和恢复复杂对象，同时保持类型安全和良好的用户体验。
