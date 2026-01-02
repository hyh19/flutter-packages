# scaffold.dart 代码解析

本文档详细解释 `example/lib/books/src/screens/scaffold.dart` 文件的代码结构和实现逻辑。该文件定义了书店应用的主脚手架组件，提供了统一的导航界面，支持在不同页面（图书、作者、设置）之间切换。

## 文件概览

该文件包含两个核心组件：

1. **`ScaffoldTab` 枚举**：定义脚手架的三个标签页
2. **`BookstoreScaffold` 组件**：书店应用的主脚手架，使用自适应导航布局

## ScaffoldTab 枚举

```dart 10:20:example/lib/books/src/screens/scaffold.dart
/// The enum for scaffold tab.
enum ScaffoldTab {
  /// The books tab.
  books,

  /// The authors tab.
  authors,

  /// The settings tab.
  settings,
}
```

`ScaffoldTab` 枚举定义了应用中的三个主要标签页：

- **`books`**：图书列表页面
- **`authors`**：作者列表页面
- **`settings`**：设置页面

这个枚举用于标识当前选中的标签页，确保导航栏正确高亮显示对应的选项。

## BookstoreScaffold 组件

```dart 22:35:example/lib/books/src/screens/scaffold.dart
/// The scaffold for the book store.
class BookstoreScaffold extends StatelessWidget {
  /// Creates a [BookstoreScaffold].
  const BookstoreScaffold({
    required this.selectedTab,
    required this.child,
    super.key,
  });

  /// Which tab of the scaffold to display.
  final ScaffoldTab selectedTab;

  /// The scaffold body.
  final Widget child;
```

`BookstoreScaffold` 是一个无状态组件，作为整个应用的主框架：

- **`selectedTab`**：当前选中的标签页，类型为 `ScaffoldTab`，用于控制导航栏的高亮状态
- **`child`**：脚手架的主体内容，根据当前路由显示不同的页面组件（如 `BooksScreen`、`AuthorsScreen` 等）

### 组件构建逻辑

```dart 37:58:example/lib/books/src/screens/scaffold.dart
  @override
  Widget build(BuildContext context) => Scaffold(
    body: AdaptiveNavigationScaffold(
      selectedIndex: selectedTab.index,
      body: child,
      onDestinationSelected: (int idx) {
        switch (ScaffoldTab.values[idx]) {
          case ScaffoldTab.books:
            context.go('/books');
          case ScaffoldTab.authors:
            context.go('/authors');
          case ScaffoldTab.settings:
            context.go('/settings');
        }
      },
      destinations: const <AdaptiveScaffoldDestination>[
        AdaptiveScaffoldDestination(title: 'Books', icon: Icons.book),
        AdaptiveScaffoldDestination(title: 'Authors', icon: Icons.person),
        AdaptiveScaffoldDestination(title: 'Settings', icon: Icons.settings),
      ],
    ),
  );
}
```

构建方法的核心逻辑如下：

#### 1. 使用 AdaptiveNavigationScaffold

组件使用了 `adaptive_navigation` 包中的 `AdaptiveNavigationScaffold`，这是一个自适应导航组件，能够根据屏幕尺寸自动调整导航布局：

- 在小屏设备（如手机）上显示底部导航栏
- 在大屏设备（如平板、桌面）上显示侧边导航栏

#### 2. 选中索引映射

```dart
selectedIndex: selectedTab.index,
```

将 `ScaffoldTab` 枚举值转换为索引（`books` = 0，`authors` = 1，`settings` = 2），用于告诉 `AdaptiveNavigationScaffold` 当前应该高亮哪个导航项。

#### 3. 导航处理

```dart
onDestinationSelected: (int idx) {
  switch (ScaffoldTab.values[idx]) {
    case ScaffoldTab.books:
      context.go('/books');
    case ScaffoldTab.authors:
      context.go('/authors');
    case ScaffoldTab.settings:
      context.go('/settings');
  }
},
```

当用户点击导航项时，通过 `context.go()` 方法（来自 `go_router` 包）进行声明式导航：

- 使用 `context.go()` 而不是 `context.push()`，确保导航时替换当前路由栈，而不是压入新路由
- 每个标签页对应一个固定的路由路径：`/books`、`/authors`、`/settings`

#### 4. 导航目标配置

```dart
destinations: const <AdaptiveScaffoldDestination>[
  AdaptiveScaffoldDestination(title: 'Books', icon: Icons.book),
  AdaptiveScaffoldDestination(title: 'Authors', icon: Icons.person),
  AdaptiveScaffoldDestination(title: 'Settings', icon: Icons.settings),
],
```

定义了三个导航目标，每个目标包含：

- **`title`**：导航项显示的文本标题
- **`icon`**：导航项显示的图标

这些配置会显示在导航栏中，用户可以点击进行页面切换。

## 使用场景

在 `main.dart` 中，`BookstoreScaffold` 被用于包装不同的页面内容：

**图书页面**：

```dart
BookstoreScaffold(
  selectedTab: ScaffoldTab.books,
  child: BooksScreen(state.pathParameters['kind']!),
)
```

**作者页面**：

```dart
BookstoreScaffold(
  selectedTab: ScaffoldTab.authors,
  child: AuthorsScreen(),
)
```

**设置页面**：

```dart
BookstoreScaffold(
  selectedTab: ScaffoldTab.settings,
  child: SettingsScreen(),
)
```

## 设计模式

该组件体现了以下设计模式：

1. **组合模式**：通过 `child` 参数接收不同的页面内容，实现了脚手架与内容的分离
2. **策略模式**：通过 `selectedTab` 参数控制导航状态，使得同一组件可以适配不同的页面场景
3. **声明式导航**：使用 `go_router` 的声明式 API（`context.go()`），而非命令式的 `Navigator.push()`

## 依赖包

- **`adaptive_navigation`**：提供自适应导航布局，支持不同屏幕尺寸的设备
- **`flutter/material.dart`**：Flutter 基础组件
- **`go_router`**：声明式路由管理包，用于页面导航

## 总结

`scaffold.dart` 文件实现了一个统一的、自适应的应用脚手架，为书店应用提供了：

- 统一的导航体验（自适应布局）
- 声明式的路由导航（基于 URL 路径）
- 清晰的组件结构（脚手架与内容分离）

这种设计使得应用能够在不同设备上提供良好的用户体验，同时保持代码的可维护性和可扩展性。
