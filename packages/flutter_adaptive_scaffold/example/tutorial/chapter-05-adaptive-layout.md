# 第 5 章 AdaptiveLayout 高级用法

## 引言

`AdaptiveLayout` 是 `flutter_adaptive_scaffold` 库的底层 API，提供了完全的控制权。虽然配置更复杂，但它能实现 `AdaptiveScaffold` 无法实现的复杂布局。本章将基于 `adaptive_layout_demo.dart` 示例，深入讲解 `AdaptiveLayout` 的完整 API 和高级用法。

## AdaptiveLayout 架构

### 与 AdaptiveScaffold 的关系

`AdaptiveScaffold` 内部使用 `AdaptiveLayout` 实现，但做了大量预设：

```dart
// AdaptiveScaffold 内部大致实现
AdaptiveLayout(
  primaryNavigation: _buildPrimaryNavigation(),
  body: _buildBody(),
  bottomNavigation: _buildBottomNavigation(),
  // ...
)
```

使用 `AdaptiveLayout` 可以直接控制这些槽位，实现更灵活的布局。

### 槽位系统

`AdaptiveLayout` 支持 6 个槽位：

1. `topNavigation`：顶部导航栏
2. `primaryNavigation`：主导航（通常左侧）
3. `secondaryNavigation`：次导航（通常右侧）
4. `body`：主内容区
5. `secondaryBody`：次内容区
6. `bottomNavigation`：底部导航栏

每个槽位都是可选的，可以独立配置。

## 完整 API 解析

### 基础结构

```dart
AdaptiveLayout(
  transitionDuration: Duration(milliseconds: 1000),
  primaryNavigation: SlotLayout(...),
  secondaryNavigation: SlotLayout(...),
  topNavigation: SlotLayout(...),
  body: SlotLayout(...),
  secondaryBody: SlotLayout(...),
  bottomNavigation: SlotLayout(...),
)
```

### transitionDuration

控制所有槽位切换时的动画时长：

```dart
AdaptiveLayout(
  transitionDuration: Duration(milliseconds: 500),
  // ...
)
```

**默认值**：`Duration(milliseconds: 300)`

## PrimaryNavigation 配置

### 基础配置

`primaryNavigation` 通常用于显示 `NavigationRail`：

```dart
primaryNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Primary Navigation Medium'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => AdaptiveScaffold.standardNavigationRail(
        selectedIndex: selectedNavigation,
        onDestinationSelected: (int newIndex) {
          setState(() {
            selectedNavigation = newIndex;
          });
        },
        leading: const Icon(Icons.menu),
        destinations: destinations
            .map((NavigationDestination destination) =>
                AdaptiveScaffold.toRailDestination(destination))
            .toList(),
      ),
    ),
  },
)
```

### 多断点配置

在不同断点显示不同样式的导航：

```dart
primaryNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    // 中等屏幕：紧凑 NavigationRail
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Primary Navigation Medium'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => AdaptiveScaffold.standardNavigationRail(
        selectedIndex: selectedNavigation,
        onDestinationSelected: (int newIndex) {
          setState(() {
            selectedNavigation = newIndex;
          });
        },
        leading: const Icon(Icons.menu),
        destinations: destinations
            .map((d) => AdaptiveScaffold.toRailDestination(d))
            .toList(),
      ),
    ),
    
    // 中等大屏幕及以上：扩展 NavigationRail
    Breakpoints.mediumLarge: SlotLayout.from(
      key: const Key('Primary Navigation MediumLarge'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => AdaptiveScaffold.standardNavigationRail(
        selectedIndex: selectedNavigation,
        onDestinationSelected: (int newIndex) {
          setState(() {
            selectedNavigation = newIndex;
          });
        },
        extended: true,  // 扩展模式
        leading: Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: [
            Text('REPLY', style: headerColor),
            const Icon(Icons.menu_open),
          ],
        ),
        trailing: trailingNavRail,  // 添加 trailing
        destinations: destinations
            .map((d) => AdaptiveScaffold.toRailDestination(d))
            .toList(),
      ),
    ),
  },
)
```

### 实际应用：adaptive_layout_demo.dart

示例中展示了完整的 `primaryNavigation` 配置：

```dart
primaryNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Primary Navigation Medium'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => AdaptiveScaffold.standardNavigationRail(
        selectedIndex: selectedNavigation,
        onDestinationSelected: (int newIndex) {
          setState(() {
            selectedNavigation = newIndex;
          });
        },
        leading: const Icon(Icons.menu),
        destinations: destinations
            .map((NavigationDestination destination) =>
                AdaptiveScaffold.toRailDestination(destination))
            .toList(),
        backgroundColor: navRailTheme.backgroundColor,
        selectedIconTheme: navRailTheme.selectedIconTheme,
        unselectedIconTheme: navRailTheme.unselectedIconTheme,
        selectedLabelTextStyle: navRailTheme.selectedLabelTextStyle,
        unSelectedLabelTextStyle: navRailTheme.unselectedLabelTextStyle,
      ),
    ),
    Breakpoints.mediumLarge: SlotLayout.from(
      key: const Key('Primary Navigation MediumLarge'),
      inAnimation: AdaptiveScaffold.leftOutIn,
      builder: (_) => AdaptiveScaffold.standardNavigationRail(
        selectedIndex: selectedNavigation,
        onDestinationSelected: (int newIndex) {
          setState(() {
            selectedNavigation = newIndex;
          });
        },
        extended: true,
        leading: Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: [
            Text('REPLY', style: headerColor),
            const Icon(Icons.menu_open),
          ],
        ),
        trailing: trailingNavRail,
        destinations: destinations
            .map((d) => AdaptiveScaffold.toRailDestination(d))
            .toList(),
        // ... 主题配置
      ),
    ),
    // large 和 extraLarge 配置类似
  },
)
```

## Body 配置

### 多断点 Body

`body` 是最重要的槽位，通常需要为不同断点配置不同的布局：

```dart
body: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayout.from(
      key: const Key('Body Small'),
      builder: (_) => ListView.builder(
        itemCount: children.length,
        itemBuilder: (BuildContext context, int index) => children[index],
      ),
    ),
    Breakpoints.medium: SlotLayout.from(
      key: const Key('Body Medium'),
      builder: (_) => GridView.count(
        crossAxisCount: 2,
        children: children,
      ),
    ),
    Breakpoints.mediumLarge: SlotLayout.from(
      key: const Key('Body MediumLarge'),
      builder: (_) => GridView.count(
        crossAxisCount: 3,
        children: children,
      ),
    ),
    Breakpoints.large: SlotLayout.from(
      key: const Key('Body Large'),
      builder: (_) => GridView.count(
        crossAxisCount: 4,
        children: children,
      ),
    ),
    Breakpoints.extraLarge: SlotLayout.from(
      key: const Key('Body ExtraLarge'),
      builder: (_) => GridView.count(
        crossAxisCount: 5,
        children: children,
      ),
    ),
  },
)
```

### 条件渲染

可以根据状态条件渲染不同的内容：

```dart
body: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Body'),
      builder: (_) => selectedNavigation == 0
          ? ListView(...)  // 第一个导航项
          : OtherView(...), // 其他导航项
    ),
  },
)
```

## BottomNavigation 配置

### 仅小屏幕显示

`bottomNavigation` 通常只在小屏幕上显示：

```dart
bottomNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.small: SlotLayout.from(
      key: const Key('Bottom Navigation Small'),
      inAnimation: AdaptiveScaffold.bottomToTop,
      outAnimation: AdaptiveScaffold.topToBottom,
      builder: (_) => AdaptiveScaffold.standardBottomNavigationBar(
        destinations: destinations,
        currentIndex: selectedNavigation,
        onDestinationSelected: (int newIndex) {
          setState(() {
            selectedNavigation = newIndex;
          });
        },
      ),
    ),
  },
)
```

### 动画配置

注意 `inAnimation` 和 `outAnimation` 的使用：

- `inAnimation: AdaptiveScaffold.bottomToTop`：从底部滑入
- `outAnimation: AdaptiveScaffold.topToBottom`：向顶部滑出

这提供了自然的过渡效果。

## SecondaryBody 配置

### 主从视图模式

`secondaryBody` 常用于实现主从视图（Master-Detail）模式：

```dart
secondaryBody: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.mediumAndUp: SlotLayout.from(
      key: const Key('Secondary Body'),
      builder: (_) => selectedItem != null
          ? DetailView(item: selectedItem)
          : PlaceholderView(),
    ),
  },
)
```

### 条件显示

可以根据主内容的状态决定是否显示 `secondaryBody`：

```dart
secondaryBody: selectedItem != null
    ? SlotLayout(
        config: <Breakpoint, SlotLayoutConfig>{
          Breakpoints.mediumAndUp: SlotLayout.from(
            key: const Key('Secondary Body'),
            builder: (_) => DetailView(item: selectedItem),
          ),
        },
      )
    : null,
```

## TopNavigation 和 SecondaryNavigation

### TopNavigation

`topNavigation` 用于显示应用栏或工具栏：

```dart
topNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Top Navigation'),
      builder: (_) => AppBar(
        title: Text('My App'),
        actions: [
          IconButton(icon: Icon(Icons.search), onPressed: () {}),
        ],
      ),
    ),
  },
)
```

**注意**：`topNavigation` 必须定义固定高度。

### SecondaryNavigation

`secondaryNavigation` 较少使用，可用于辅助导航：

```dart
secondaryNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig>{
    Breakpoints.extraLarge: SlotLayout.from(
      key: const Key('Secondary Navigation'),
      builder: (_) => Toolbar(...),
    ),
  },
)
```

## 完整示例分析

让我们完整分析 `adaptive_layout_demo.dart` 的实现：

```dart
class _MyHomePageState extends State<MyHomePage> {
  int selectedNavigation = 0;
  int _transitionDuration = 1000;

  @override
  void initState() {
    super.initState();
    setState(() {
      _transitionDuration = widget.transitionDuration;
    });
  }

  @override
  Widget build(BuildContext context) {
    final NavigationRailThemeData navRailTheme =
        Theme.of(context).navigationRailTheme;

    // 定义子组件
    final List<Widget> children = List<Widget>.generate(10, (int index) {
      return Padding(
        padding: const EdgeInsets.all(8.0),
        child: Container(
          color: const Color.fromARGB(255, 255, 201, 197),
          height: 400,
        ),
      );
    });

    // 定义 trailingNavRail
    final Widget trailingNavRail = Column(
      children: [
        const Divider(color: Colors.black),
        const SizedBox(height: 10),
        const Row(
          children: [
            SizedBox(width: 27),
            Text('Folders', style: TextStyle(fontSize: 16)),
          ],
        ),
        // ... 文件夹列表
      ],
    );

    // 定义导航目标
    const List<NavigationDestination> destinations = <NavigationDestination>[
      NavigationDestination(
        label: 'Inbox',
        icon: Icon(Icons.inbox_outlined),
        selectedIcon: Icon(Icons.inbox),
      ),
      // ... 更多目标
    ];

    return AdaptiveLayout(
      transitionDuration: Duration(milliseconds: _transitionDuration),
      
      // Primary Navigation 配置
      primaryNavigation: SlotLayout(
        config: <Breakpoint, SlotLayoutConfig>{
          Breakpoints.medium: SlotLayout.from(
            key: const Key('Primary Navigation Medium'),
            inAnimation: AdaptiveScaffold.leftOutIn,
            builder: (_) => AdaptiveScaffold.standardNavigationRail(
              selectedIndex: selectedNavigation,
              onDestinationSelected: (int newIndex) {
                setState(() {
                  selectedNavigation = newIndex;
                });
              },
              leading: const Icon(Icons.menu),
              destinations: destinations
                  .map((d) => AdaptiveScaffold.toRailDestination(d))
                  .toList(),
              backgroundColor: navRailTheme.backgroundColor,
              // ... 主题配置
            ),
          ),
          Breakpoints.mediumLarge: SlotLayout.from(
            key: const Key('Primary Navigation MediumLarge'),
            inAnimation: AdaptiveScaffold.leftOutIn,
            builder: (_) => AdaptiveScaffold.standardNavigationRail(
              selectedIndex: selectedNavigation,
              onDestinationSelected: (int newIndex) {
                setState(() {
                  selectedNavigation = newIndex;
                });
              },
              extended: true,
              leading: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  Text('REPLY', style: headerColor),
                  const Icon(Icons.menu_open),
                ],
              ),
              trailing: trailingNavRail,
              destinations: destinations
                  .map((d) => AdaptiveScaffold.toRailDestination(d))
                  .toList(),
              // ... 主题配置
            ),
          ),
          // large 和 extraLarge 配置类似
        },
      ),
      
      // Body 配置
      body: SlotLayout(
        config: <Breakpoint, SlotLayoutConfig>{
          Breakpoints.small: SlotLayout.from(
            key: const Key('Body Small'),
            builder: (_) => ListView.builder(
              itemCount: children.length,
              itemBuilder: (BuildContext context, int index) => children[index],
            ),
          ),
          Breakpoints.medium: SlotLayout.from(
            key: const Key('Body Medium'),
            builder: (_) => GridView.count(
              crossAxisCount: 2,
              children: children,
            ),
          ),
          // ... 更多断点配置
        },
      ),
      
      // Bottom Navigation 配置
      bottomNavigation: SlotLayout(
        config: <Breakpoint, SlotLayoutConfig>{
          Breakpoints.small: SlotLayout.from(
            key: const Key('Bottom Navigation Small'),
            inAnimation: AdaptiveScaffold.bottomToTop,
            outAnimation: AdaptiveScaffold.topToBottom,
            builder: (_) => AdaptiveScaffold.standardBottomNavigationBar(
              destinations: destinations,
              currentIndex: selectedNavigation,
              onDestinationSelected: (int newIndex) {
                setState(() {
                  selectedNavigation = newIndex;
                });
              },
            ),
          ),
        },
      ),
    );
  }
}
```

**关键设计点**：

1. **状态同步**：`selectedNavigation` 在 `primaryNavigation` 和 `bottomNavigation` 之间共享
2. **主题一致性**：使用 `navRailTheme` 保持视觉一致性
3. **渐进增强**：从紧凑到扩展的导航栏设计
4. **动画协调**：使用合适的动画方向

## 性能优化技巧

### 1. 使用 Builder 延迟构建

`SlotLayout.from` 的 `builder` 参数是延迟执行的，只在需要时构建 Widget：

```dart
SlotLayout.from(
  key: const Key('Body'),
  builder: (_) => ExpensiveWidget(),  // 只在匹配时构建
)
```

### 2. 合理使用 Key

为每个 `SlotLayoutConfig` 提供唯一的 `Key`，帮助 Flutter 正确识别 Widget：

```dart
SlotLayout.from(
  key: const Key('Body Small'),  // 唯一且稳定
  builder: (_) => Widget(),
)
```

### 3. 避免不必要的重建

将静态内容提取到常量：

```dart
// ✅ 推荐
const List<NavigationDestination> destinations = [...];

// ❌ 避免
final List<NavigationDestination> destinations = [...];  // 每次重建都会创建新列表
```

### 4. 条件配置

只在需要时配置槽位：

```dart
// ✅ 推荐：使用 null 表示不显示
secondaryBody: shouldShowDetail
    ? SlotLayout(...)
    : null,

// ❌ 避免：使用空 Widget
secondaryBody: SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Empty'),
      builder: (_) => SizedBox.shrink(),  // 浪费资源
    ),
  },
)
```

## 高级用法

### 1. 动态槽位配置

根据应用状态动态配置槽位：

```dart
AdaptiveLayout(
  primaryNavigation: isLoggedIn
      ? SlotLayout(...)  // 登录后显示导航
      : null,            // 未登录隐藏
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayout.from(
        key: const Key('Body'),
        builder: (_) => isLoggedIn
            ? AuthenticatedView()
            : LoginView(),
      ),
    },
  ),
)
```

### 2. 嵌套 AdaptiveLayout

可以在 `body` 中嵌套另一个 `AdaptiveLayout`：

```dart
body: SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Body'),
      builder: (_) => AdaptiveLayout(
        // 嵌套布局
        body: SlotLayout(...),
        secondaryBody: SlotLayout(...),
      ),
    ),
  },
)
```

### 3. 自定义动画

为每个槽位定义自定义动画（详见第 7 章）。

## 总结

本章我们深入学习了：

- **AdaptiveLayout 架构**：与 AdaptiveScaffold 的关系和槽位系统
- **完整 API**：所有槽位的配置方法
- **多断点配置**：如何为不同断点配置不同的布局
- **性能优化**：延迟构建、Key 管理、条件配置等技巧
- **高级用法**：动态配置、嵌套布局等

在下一章中，我们将深入探讨 `SlotLayout` 的工作原理。

## 练习

1. 修改 `adaptive_layout_demo.dart`，添加 `topNavigation` 槽位
2. 实现条件渲染：根据 `selectedNavigation` 显示不同的 body 内容
3. 尝试嵌套 `AdaptiveLayout`，实现更复杂的布局

## 检查清单

- [ ] 理解 `AdaptiveLayout` 的完整 API
- [ ] 掌握所有槽位的配置方法
- [ ] 能够实现多断点配置
- [ ] 了解性能优化技巧
- [ ] 能够实现动态和嵌套布局
