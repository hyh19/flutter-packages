# readme_excerpts.dart 代码详解

## 文件概述

这个文件包含了 `shared_preferences` 包的使用示例代码片段，主要用于演示如何在 Flutter 应用中使用共享偏好设置来存储和检索数据。文件中的代码片段被组织成多个函数，每个函数展示不同的使用场景。

文件开头导入了必要的包，并包含了多个异步函数，每个函数演示不同的功能特性。

## 主要功能示例

### 写入数据

```dart 12:26:shared_preferences/example/lib/readme_excerpts.dart
// #docregion Write
// Obtain shared preferences.
final SharedPreferences prefs = await SharedPreferences.getInstance();

// Save an integer value to 'counter' key.
await prefs.setInt('counter', 10);
// Save an boolean value to 'repeat' key.
await prefs.setBool('repeat', true);
// Save an double value to 'decimal' key.
await prefs.setDouble('decimal', 1.5);
// Save an String value to 'action' key.
await prefs.setString('action', 'Start');
// Save an list of strings to 'items' key.
await prefs.setStringList('items', <String>['Earth', 'Moon', 'Sun']);
// #enddocregion Write
```

这段代码展示了如何获取 `SharedPreferences` 实例并存储不同类型的数据：

- 使用 `SharedPreferences.getInstance()` 获取单例实例
- 支持存储整数（`setInt`）、布尔值（`setBool`）、双精度浮点数（`setDouble`）、字符串（`setString`）和字符串列表（`setStringList`）
- 所有设置方法都是异步的，返回 `Future<void>`

### 读取数据

```dart 28:39:shared_preferences/example/lib/readme_excerpts.dart
// #docregion Read
// Try reading data from the 'counter' key. If it doesn't exist, returns null.
final int? counter = prefs.getInt('counter');
// Try reading data from the 'repeat' key. If it doesn't exist, returns null.
final bool? repeat = prefs.getBool('repeat');
// Try reading data from the 'decimal' key. If it doesn't exist, returns null.
final double? decimal = prefs.getDouble('decimal');
// Try reading data from the 'action' key. If it doesn't exist, returns null.
final String? action = prefs.getString('action');
// Try reading data from the 'items' key. If it doesn't exist, returns null.
final List<String>? items = prefs.getStringList('items');
// #enddocregion Read
```

读取数据的方法对应存储方法：

- 如果键不存在，`getInt`、`getBool`、`getDouble`、`getString` 和 `getStringList` 都返回 `null`
- 返回类型都是可空的（使用 `?`），需要在使用前检查是否为 `null`
- 读取操作是同步的，不需要 `await`

### 删除数据

```dart 41:44:shared_preferences/example/lib/readme_excerpts.dart
// #docregion Clear
// Remove data for the 'counter' key.
await prefs.remove('counter');
// #enddocregion Clear
```

删除操作也很简单：

- 使用 `remove(key)` 方法删除指定键的值
- 删除操作是异步的
- 如果要清空所有数据，可以使用 `clear()` 方法（虽然这个示例中没有展示）

## 异步API使用

```dart 47:63:shared_preferences/example/lib/readme_excerpts.dart
Future<void> readmeSnippetsAsync() async {
  // #docregion Async
  final asyncPrefs = SharedPreferencesAsync();

  await asyncPrefs.setBool('repeat', true);
  await asyncPrefs.setString('action', 'Start');

  final bool? repeat = await asyncPrefs.getBool('repeat');
  final String? action = await asyncPrefs.getString('action');

  await asyncPrefs.remove('repeat');

  // Any time a filter option is included as a method parameter, strongly consider
  // using it to avoid potentially unwanted side effects.
  await asyncPrefs.clear(allowList: <String>{'action', 'repeat'});
  // #enddocregion Async
}
```

这个函数演示了新的异步API `SharedPreferencesAsync` 的使用：

- 创建 `SharedPreferencesAsync()` 实例（不需要 `await`）
- 所有的操作（设置、获取、删除、清空）都是异步的，都需要 `await`
- 清空方法支持 `allowList` 参数，只清空指定键的值，避免意外删除其他数据
- 注释提醒开发者在使用过滤选项时要谨慎考虑，以避免潜在的副作用

## 带缓存的偏好设置

```dart 65:86:shared_preferences/example/lib/readme_excerpts.dart
Future<void> readmeSnippetsWithCache() async {
  // #docregion WithCache
  final SharedPreferencesWithCache
  prefsWithCache = await SharedPreferencesWithCache.create(
    cacheOptions: const SharedPreferencesWithCacheOptions(
      // When an allowlist is included, any keys that aren't included cannot be used.
      allowList: <String>{'repeat', 'action'},
    ),
  );

  await prefsWithCache.setBool('repeat', true);
  await prefsWithCache.setString('action', 'Start');

  final bool? repeat = prefsWithCache.getBool('repeat');
  final String? action = prefsWithCache.getString('action');

  await prefsWithCache.remove('repeat');

  // Since the filter options are set at creation, they aren't needed during clear.
  await prefsWithCache.clear();
  // #enddocregion WithCache
}
```

这个函数展示了带缓存的偏好设置 `SharedPreferencesWithCache`：

- 使用 `SharedPreferencesWithCache.create()` 创建实例，需要传入 `cacheOptions`
- `SharedPreferencesWithCacheOptions` 可以设置 `allowList`，限制只能使用指定的键
- 设置操作是异步的，但获取操作是同步的（从缓存中读取）
- 删除和清空操作都是异步的
- 一旦在创建时设置了过滤选项，清空时就不需要再次指定

## 测试设置

```dart 92:97:shared_preferences/example/lib/readme_excerpts.dart
Future<void> readmeTestSnippets() async {
  // #docregion Tests
  final values = <String, Object>{'counter': 1};
  SharedPreferences.setMockInitialValues(values);
  // #enddocregion Tests
}
```

这个函数演示了如何在测试中设置模拟的初始值：

- 使用 `SharedPreferences.setMockInitialValues()` 设置测试用的初始数据
- 接受一个 `Map<String, Object>` 类型的参数
- 这是一个测试专用的方法，生产代码中不应该使用

## Android特定选项

```dart 99:107:shared_preferences/example/lib/readme_excerpts.dart
// #docregion Android_Options2
const SharedPreferencesAsyncAndroidOptions options =
    SharedPreferencesAsyncAndroidOptions(
      backend: SharedPreferencesAndroidBackendLibrary.SharedPreferences,
      originalSharedPreferencesOptions: AndroidSharedPreferencesStoreOptions(
        fileName: 'the_name_of_a_file',
      ),
    );
// #enddocregion Android_Options2
```

这个代码片段展示了Android平台特定的配置选项：

- `SharedPreferencesAsyncAndroidOptions` 用于配置Android异步API的行为
- `backend` 指定使用的后端库，这里使用标准的 `SharedPreferences`
- `originalSharedPreferencesOptions` 包含原始的Android SharedPreferences选项
- `AndroidSharedPreferencesStoreOptions` 中的 `fileName` 指定存储文件的名称

## 导入说明

```dart 5:9:shared_preferences/example/lib/readme_excerpts.dart
// ignore_for_file: public_member_api_docs, unused_local_variable, invalid_use_of_visible_for_testing_member
import 'package:shared_preferences/shared_preferences.dart';
// #docregion Android_Options1
import 'package:shared_preferences_android/shared_preferences_android.dart';
// #enddocregion Android_Options1
```

文件顶部包含了必要的导入：

- 主要的 `shared_preferences` 包
- Android特定的包 `shared_preferences_android`（用于Android特定选项）
- 忽略文件级别的lint规则，因为这是示例代码

## 总结

这个文件提供了 `shared_preferences` 包的完整使用示例，涵盖了：

1. 基本的同步API（设置、获取、删除）
2. 新的异步API `SharedPreferencesAsync`
3. 带缓存的偏好设置 `SharedPreferencesWithCache`
4. 测试设置方法
5. Android平台特定的配置

这些示例展示了从简单的数据存储到复杂的使用场景的各种用法，帮助开发者理解如何在Flutter应用中有效地使用共享偏好设置。
