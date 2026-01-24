# 第 4 章 AdaptiveScaffold 基础与进阶

## 引言

`AdaptiveScaffold` 是 `flutter_adaptive_scaffold` 库提供的高级 API，它简化了响应式布局的实现。本章将基于 `adaptive_scaffold_demo.dart` 示例，深入讲解 `AdaptiveScaffold` 的使用方法、配置选项和进阶技巧。

## 基础用法

### 最简单的示例

让我们从最简单的 `AdaptiveScaffold` 使用开始：

```dart
AdaptiveScaffold(
  destinations: const [
    NavigationDestination(
      icon: Icon(Icons.inbox_outlined),
      selectedIcon: Icon(Icons.inbox),
      label: 'Inbox',
    ),
    NavigationDestination(
      icon: Icon(Icons.article_outlined),
      selectedIcon: Icon(Icons.article),
      label: 'Articles',
    ),
  ],
  body: (_) => Center(child: Text('Content')),
)
```

这个简单的配置就能实现：

- 小屏幕：底部显示 `BottomNavigationBar`
- 大屏幕：左侧显示 `NavigationRail`
- 自动的过渡动画

### 核心属性解析

#### destinations（必需）

`destinations` 定义了导航目标列表：

```dart
final List<NavigationDestination> destinations = const [
  NavigationDestination(
    icon: Icon(Icons.inbox_outlined),      // 未选中图标
    selectedIcon: Icon(Icons.inbox),       // 选中图标
    label: 'Inbox',                        // 标签文本
  ),
  // ... 更多目标
];
```

**特点**：

- 支持图标和标签
- 可以定义选中和未选中状态的不同图标
- 标签会在扩展的 `NavigationRail` 中显示

#### selectedIndex 和 onSelectedIndexChange

控制当前选中的导航项：

```dart
int _selectedTab = 0;

AdaptiveScaffold(
  selectedIndex: _selectedTab,
  onSelectedIndexChange: (int index) {
    setState(() {
      _selectedTab = index;
    });
  },
  destinations: destinations,
  body: (_) => _buildContent(_selectedTab),
)
```

## 不同断点的 Body 配置

### 基础配置

`AdaptiveScaffold` 允许为不同断点配置不同的 body：

```dart
AdaptiveScaffold(
  destinations: destinations,
  
  // 小屏幕：使用 ListView
  smallBody: (_) => ListView.builder(
    itemCount: items.length,
    itemBuilder: (_, idx) => items[idx],
  ),
  
  // 默认/中等屏幕：使用 2 列 GridView
  body: (_) => GridView.count(
    crossAxisCount: 2,
    children: items,
  ),
  
  // 中等大屏幕：使用 3 列 GridView
  mediumLargeBody: (_) => GridView.count(
    crossAxisCount: 3,
    children: items,
  ),
  
  // 大屏幕：使用 4 列 GridView
  largeBody: (_) => GridView.count(
    crossAxisCount: 4,
    children: items,
  ),
  
  // 超大屏幕：使用 5 列 GridView
  extraLargeBody: (_) => GridView.count(
    crossAxisCount: 5,
    children: items,
  ),
)
```

### 断点 Body 的优先级

如果同时配置了多个断点的 body，优先级如下：

1. **特定断点**：`smallBody`、`mediumLargeBody`、`largeBody`、`extraLargeBody`
2. **默认 body**：`body`（作为后备）

**示例**：

```dart
AdaptiveScaffold(
  body: (_) => DefaultLayout(),           // 默认布局
  smallBody: (_) => MobileLayout(),       // 小屏幕覆盖
  largeBody: (_) => DesktopLayout(),     // 大屏幕覆盖
  // 中等屏幕会使用 body（DefaultLayout）
)
```

### 实际应用：adaptive_scaffold_demo.dart 分析

让我们分析示例中的 body 配置：

```dart
// 定义子组件列表
final List<Widget> children = <Widget>[
  for (int i = 0; i < 10; i++)
    Padding(
      padding: const EdgeInsets.all(8.0),
      child: Container(
        color: const Color.fromARGB(255, 255, 201, 197),
        height: 400,
      ),
    )
];

return AdaptiveScaffold(
  // 小屏幕：ListView，单列显示
  smallBody: (_) => ListView.builder(
    itemCount: children.length,
    itemBuilder: (_, int idx) => children[idx],
  ),
  
  // 默认：2 列 GridView
  body: (_) => GridView.count(
    crossAxisCount: 2,
    children: children,
  ),
  
  // 中等大屏幕：3 列
  mediumLargeBody: (_) => GridView.count(
    crossAxisCount: 3,
    children: children,
  ),
  
  // 大屏幕：4 列
  largeBody: (_) => GridView.count(
    crossAxisCount: 4,
    children: children,
  ),
  
  // 超大屏幕：5 列
  extraLargeBody: (_) => GridView.count(
    crossAxisCount: 5,
    children: children,
  ),
)
```

**设计思路**：

- 小屏幕空间有限，使用单列 `ListView` 更合适
- 大屏幕空间充足，使用多列 `GridView` 充分利用空间
- 列数随屏幕增大而增加，提供渐进增强的体验

## SecondaryBody 配置

### 基础用法

`secondaryBody` 用于显示详情视图或辅助内容：

```dart
AdaptiveScaffold(
  destinations: destinations,
  body: (_) => ListView(...),              // 主列表
  secondaryBody: (_) => DetailView(...),   // 详情视图
)
```

### 小屏幕处理

在小屏幕上，`secondaryBody` 通常应该隐藏，使用模态显示：

```dart
AdaptiveScaffold(
  destinations: destinations,
  body: (_) => ListView(...),
  
  // 小屏幕：隐藏 secondaryBody
  smallSecondaryBody: AdaptiveScaffold.emptyBuilder,
  
  // 大屏幕：显示详情视图
  secondaryBody: (_) => DetailView(...),
)
```

### 不同断点的 SecondaryBody

可以为不同断点配置不同的 `secondaryBody`：

```dart
AdaptiveScaffold(
  destinations: destinations,
  body: (_) => ListView(...),
  
  // 小屏幕：隐藏
  smallSecondaryBody: AdaptiveScaffold.emptyBuilder,
  
  // 默认：简单详情视图
  secondaryBody: (_) => SimpleDetailView(...),
  
  // 大屏幕：完整详情视图
  largeSecondaryBody: (_) => FullDetailView(...),
)
```

## 导航元素自定义

### Leading 组件

`leadingUnextendedNavRail` 和 `leadingExtendedNavRail` 用于自定义导航栏的顶部组件：

```dart
AdaptiveScaffold(
  destinations: destinations,
  
  // 紧凑 NavigationRail 的 leading
  leadingUnextendedNavRail: IconButton(
    icon: Icon(Icons.menu),
    onPressed: () {},
  ),
  
  // 扩展 NavigationRail 的 leading
  leadingExtendedNavRail: Row(
    children: [
      Text('REPLY', style: TextStyle(...)),
      Icon(Icons.menu_open),
    ],
  ),
)
```

### Trailing 组件

`trailingNavRail` 用于在扩展的 `NavigationRail` 底部添加额外内容：

```dart
AdaptiveScaffold(
  destinations: destinations,
  trailingNavRail: Column(
    children: [
      Divider(),
      Text('Folders'),
      ListTile(
        leading: Icon(Icons.folder),
        title: Text('Freelance'),
      ),
      // ... 更多文件夹
    ],
  ),
)
```

### 实际应用示例

在 `adaptive_scaffold_demo.dart` 中，虽然没有使用这些属性，但在 `main.dart`（邮件应用）中有完整示例：

```dart
// 紧凑 NavigationRail 的 leading：菜单图标
leadingUnextendedNavRail: Icon(Icons.menu),

// 扩展 NavigationRail 的 leading：标题和菜单
leadingExtendedNavRail: Row(
  children: [
    Text('REPLY', style: TextStyle(...)),
    Icon(Icons.menu_open),
  ],
),

// Trailing：文件夹列表
trailingNavRail: Column(
  children: [
    Divider(),
    Text('Folders'),
    FolderItem('Freelance'),
    FolderItem('Mortgage'),
    // ...
  ],
)
```

## 过渡动画配置

### transitionDuration

控制布局切换时的动画时长：

```dart
AdaptiveScaffold(
  transitionDuration: Duration(milliseconds: 1000),  // 1 秒过渡
  destinations: destinations,
  body: (_) => Content(),
)
```

**默认值**：`Duration(milliseconds: 300)`

**建议**：

- 快速切换：200-300 ms
- 标准切换：300-500 ms
- 平滑切换：500-1000 ms

### 实际应用

在 `adaptive_scaffold_demo.dart` 中：

```dart
class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, this.transitionDuration = 1000});
  final int transitionDuration;
  // ...
}

AdaptiveScaffold(
  transitionDuration: Duration(milliseconds: _transitionDuration),
  // ...
)
```

这允许通过构造函数参数控制动画时长，方便测试和调试。

## 断点自定义

### 覆盖默认断点

`AdaptiveScaffold` 允许覆盖默认的断点定义：

```dart
AdaptiveScaffold(
  // 自定义小屏幕断点：0-700 dp
  smallBreakpoint: const Breakpoint(endWidth: 700),
  
  // 自定义中等屏幕断点：700-1000 dp
  mediumBreakpoint: const Breakpoint(
    beginWidth: 700,
    endWidth: 1000,
  ),
  
  // 自定义中等大屏幕断点：1000-1200 dp
  mediumLargeBreakpoint: const Breakpoint(
    beginWidth: 1000,
    endWidth: 1200,
  ),
  
  // 自定义大屏幕断点：1200-1600 dp
  largeBreakpoint: const Breakpoint(
    beginWidth: 1200,
    endWidth: 1600,
  ),
  
  // 自定义超大屏幕断点：1600+ dp
  extraLargeBreakpoint: const Breakpoint(beginWidth: 1600),
  
  destinations: destinations,
  body: (_) => Content(),
)
```

### 实际应用

在 `adaptive_scaffold_demo.dart` 中：

```dart
AdaptiveScaffold(
  smallBreakpoint: const Breakpoint(endWidth: 700),
  mediumBreakpoint: const Breakpoint(beginWidth: 700, endWidth: 1000),
  mediumLargeBreakpoint: const Breakpoint(beginWidth: 1000, endWidth: 1200),
  largeBreakpoint: const Breakpoint(beginWidth: 1200, endWidth: 1600),
  extraLargeBreakpoint: const Breakpoint(beginWidth: 1600),
  // ...
)
```

**使用场景**：

- 根据设计需求调整断点
- 适配特定设备的屏幕尺寸
- 测试不同断点的布局效果

## Drawer 配置

### useDrawer

控制是否在小屏幕桌面设备上使用 `Drawer`：

```dart
AdaptiveScaffold(
  useDrawer: true,  // 小屏幕桌面使用 Drawer
  destinations: destinations,
  body: (_) => Content(),
)
```

**默认值**：`true`

**行为**：

- `useDrawer: true`：小屏幕桌面设备显示 `Drawer`，移动设备显示 `BottomNavigationBar`
- `useDrawer: false`：所有小屏幕设备都显示 `BottomNavigationBar`

### 实际应用

在 `adaptive_scaffold_demo.dart` 中：

```dart
AdaptiveScaffold(
  useDrawer: false,  // 禁用 Drawer，统一使用 BottomNavigationBar
  // ...
)
```

## 静态辅助方法

`AdaptiveScaffold` 提供了一些静态辅助方法，用于创建标准的导航组件。

### standardNavigationRail

创建标准的 `NavigationRail`：

```dart
AdaptiveScaffold.standardNavigationRail(
  selectedIndex: selectedIndex,
  onDestinationSelected: (int index) {
    setState(() {
      selectedIndex = index;
    });
  },
  destinations: destinations.map((d) => 
    AdaptiveScaffold.toRailDestination(d)
  ).toList(),
)
```

### standardBottomNavigationBar

创建标准的 `BottomNavigationBar`：

```dart
AdaptiveScaffold.standardBottomNavigationBar(
  destinations: destinations,
  currentIndex: currentIndex,
  onDestinationSelected: (int index) {
    setState(() {
      currentIndex = index;
    });
  },
)
```

### toRailDestination

将 `NavigationDestination` 转换为 `NavigationRailDestination`：

```dart
NavigationRailDestination railDest = 
  AdaptiveScaffold.toRailDestination(navDest);
```

## 完整示例分析

让我们完整分析 `adaptive_scaffold_demo.dart` 的实现：

```dart
class _MyHomePageState extends State<MyHomePage> {
  int _selectedTab = 0;
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
    // 定义子组件
    final List<Widget> children = <Widget>[
      for (int i = 0; i < 10; i++)
        Padding(
          padding: const EdgeInsets.all(8.0),
          child: Container(
            color: const Color.fromARGB(255, 255, 201, 197),
            height: 400,
          ),
        )
    ];

    return AdaptiveScaffold(
      // 动画配置
      transitionDuration: Duration(milliseconds: _transitionDuration),
      
      // 断点自定义
      smallBreakpoint: const Breakpoint(endWidth: 700),
      mediumBreakpoint: const Breakpoint(beginWidth: 700, endWidth: 1000),
      mediumLargeBreakpoint: const Breakpoint(beginWidth: 1000, endWidth: 1200),
      largeBreakpoint: const Breakpoint(beginWidth: 1200, endWidth: 1600),
      extraLargeBreakpoint: const Breakpoint(beginWidth: 1600),
      
      // Drawer 配置
      useDrawer: false,
      
      // 导航状态
      selectedIndex: _selectedTab,
      onSelectedIndexChange: (int index) {
        setState(() {
          _selectedTab = index;
        });
      },
      
      // 导航目标
      destinations: const <NavigationDestination>[
        NavigationDestination(
          icon: Icon(Icons.inbox_outlined),
          selectedIcon: Icon(Icons.inbox),
          label: 'Inbox',
        ),
        NavigationDestination(
          icon: Icon(Icons.article_outlined),
          selectedIcon: Icon(Icons.article),
          label: 'Articles',
        ),
        NavigationDestination(
          icon: Icon(Icons.chat_outlined),
          selectedIcon: Icon(Icons.chat),
          label: 'Chat',
        ),
        NavigationDestination(
          icon: Icon(Icons.video_call_outlined),
          selectedIcon: Icon(Icons.video_call),
          label: 'Video',
        ),
        NavigationDestination(
          icon: Icon(Icons.home_outlined),
          selectedIcon: Icon(Icons.home),
          label: 'Inbox',
        ),
      ],
      
      // 不同断点的 Body
      smallBody: (_) => ListView.builder(
        itemCount: children.length,
        itemBuilder: (_, int idx) => children[idx],
      ),
      body: (_) => GridView.count(crossAxisCount: 2, children: children),
      mediumLargeBody: (_) =>
          GridView.count(crossAxisCount: 3, children: children),
      largeBody: (_) => GridView.count(crossAxisCount: 4, children: children),
      extraLargeBody: (_) =>
          GridView.count(crossAxisCount: 5, children: children),
      
      // SecondaryBody 配置
      smallSecondaryBody: AdaptiveScaffold.emptyBuilder,
      secondaryBody: (_) => Container(
        color: const Color.fromARGB(255, 234, 158, 192),
      ),
      mediumLargeSecondaryBody: (_) => Container(
        color: const Color.fromARGB(255, 234, 158, 192),
      ),
      largeSecondaryBody: (_) => Container(
        color: const Color.fromARGB(255, 234, 158, 192),
      ),
      extraLargeSecondaryBody: (_) => Container(
        color: const Color.fromARGB(255, 234, 158, 192),
      ),
    );
  }
}
```

**关键设计点**：

1. **状态管理**：使用 `_selectedTab` 跟踪当前选中的标签
2. **动态内容**：根据 `_selectedTab` 显示不同内容（虽然示例中所有标签显示相同内容）
3. **响应式布局**：不同断点使用不同的布局方式
4. **配置灵活性**：支持自定义断点和动画时长

## 进阶技巧

### 1. 条件渲染 SecondaryBody

根据选中项决定是否显示 `secondaryBody`：

```dart
AdaptiveScaffold(
  destinations: destinations,
  body: (_) => ListView(...),
  secondaryBody: _selectedTab == 0 
    ? (_) => DetailView() 
    : AdaptiveScaffold.emptyBuilder,
)
```

### 2. 动态更新 Destinations

根据应用状态动态更新导航目标：

```dart
List<NavigationDestination> get destinations {
  if (isLoggedIn) {
    return [
      NavigationDestination(icon: Icon(Icons.home), label: 'Home'),
      NavigationDestination(icon: Icon(Icons.profile), label: 'Profile'),
    ];
  } else {
    return [
      NavigationDestination(icon: Icon(Icons.login), label: 'Login'),
    ];
  }
}
```

### 3. 主题定制

通过 `Theme` 定制导航栏样式：

```dart
MaterialApp(
  theme: ThemeData.light().copyWith(
    navigationRailTheme: NavigationRailThemeData(
      selectedIconTheme: IconThemeData(color: Colors.red, size: 28),
      selectedLabelTextStyle: TextStyle(fontSize: 16, color: Colors.red),
    ),
    bottomNavigationBarTheme: BottomNavigationBarThemeData(
      type: BottomNavigationBarType.fixed,
    ),
  ),
  home: MyHomePage(),
)
```

## 总结

本章我们深入学习了：

- **基础用法**：destinations、selectedIndex、body 的配置
- **断点 Body**：如何为不同断点配置不同的布局
- **SecondaryBody**：详情视图的配置方法
- **导航自定义**：leading、trailing 的使用
- **动画配置**：transitionDuration 的设置
- **断点自定义**：如何覆盖默认断点
- **静态方法**：标准导航组件的创建

在下一章中，我们将学习 `AdaptiveLayout` 的高级用法，它提供了更强大的定制能力。

## 练习

1. 修改 `adaptive_scaffold_demo.dart`，为不同导航项显示不同的内容
2. 尝试添加 `leadingExtendedNavRail` 和 `trailingNavRail`
3. 调整断点配置，观察布局变化

## 检查清单

- [ ] 理解 `AdaptiveScaffold` 的基础用法
- [ ] 掌握不同断点的 body 配置
- [ ] 了解 secondaryBody 的使用方法
- [ ] 能够自定义导航元素
- [ ] 理解过渡动画和断点自定义
