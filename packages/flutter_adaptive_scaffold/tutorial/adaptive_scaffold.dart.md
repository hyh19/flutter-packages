# AdaptiveScaffold 类详解

## 引言

`AdaptiveScaffold` 是 `flutter_adaptive_scaffold` 包中提供的高级响应式布局组件，它基于 `AdaptiveLayout` 构建，提供了更易用的 API 和预设的布局结构。`AdaptiveScaffold` 实现了 [Material Design 3](https://m3.material.io/foundations/adaptive-design/overview) 自适应设计规范，能够根据屏幕尺寸自动切换导航元素（BottomNavigationBar、NavigationRail、Drawer），并处理不同断点下的布局变化。

`AdaptiveScaffold` 通过封装 `AdaptiveLayout` 的复杂配置，简化了响应式应用的开发。它自动处理导航元素的转换、断点切换和布局动画，让开发者只需关注内容本身，而无需手动管理复杂的布局逻辑。

## 类概述

### 设计目的

`AdaptiveScaffold` 类的主要目的是：

1. **提供预设的响应式布局结构**：基于 Material Design 3 规范，提供开箱即用的布局方案
2. **自动处理导航元素的转换**：根据屏幕尺寸自动在 BottomNavigationBar、NavigationRail、Drawer 之间切换
3. **简化响应式应用的开发**：减少配置代码，提供更高级的抽象

### 与 AdaptiveLayout 的关系

`AdaptiveScaffold` 与 `AdaptiveLayout` 是组合关系：

- **内部实现**：`AdaptiveScaffold` 内部使用 `AdaptiveLayout` 来实现布局（见第 607-827 行）
- **抽象层次**：`AdaptiveScaffold` 提供更高级的抽象，减少配置代码
- **自动处理**：自动处理导航元素的断点切换，无需手动配置 `SlotLayout`
- **灵活性权衡**：更易用但可定制性较低，需要更精细控制时可以使用 `AdaptiveLayout`

**代码示例**：

```dart
// AdaptiveScaffold 内部使用 AdaptiveLayout
body: AdaptiveLayout(
  transitionDuration: widget.transitionDuration,
  bodyOrientation: widget.bodyOrientation,
  bodyRatio: widget.bodyRatio,
  internalAnimations: widget.internalAnimations,
  primaryNavigation: SlotLayout(...),  // 自动配置
  bottomNavigation: SlotLayout(...),    // 自动配置
  body: SlotLayout(...),                // 自动配置
  secondaryBody: SlotLayout(...),       // 自动配置
)
```

### Material Design 3 规范

`AdaptiveScaffold` 严格遵循 Material Design 3 规范：

- **间距规范**：使用 `kMaterialCompactSpacing`（0）和 `kMaterialMediumAndUpSpacing`（24）
- **边距规范**：使用 `kMaterialCompactMargin`（16）和 `kMaterialMediumAndUpMargin`（24）
- **内边距规范**：使用 `kMaterialPadding`（4）和 `kNavigationRailDefaultPadding`（8）
- **标准断点**：使用 `Breakpoints` 中定义的标准断点
- **导航组件**：提供符合规范的 NavigationRail、BottomNavigationBar 和 Drawer

## 核心组件详解

### 常量定义

`AdaptiveScaffold` 定义了多个 Material Design 3 相关的常量（第 11-32 行）：

#### kMaterialCompactSpacing

```dart
const double kMaterialCompactSpacing = 0;
```

**作用**：紧凑断点（compact breakpoint）的间距值，根据 Material Design 3 规范定义。

**用途**：用于小屏幕设备的布局间距。

#### kMaterialMediumAndUpSpacing

```dart
const double kMaterialMediumAndUpSpacing = 24;
```

**作用**：中等及以上断点的间距值，根据 Material Design 3 规范定义。

**用途**：用于平板和桌面设备的布局间距。

#### kMaterialCompactMargin

```dart
const double kMaterialCompactMargin = 16;
```

**作用**：紧凑断点的边距值，根据 Material Design 3 规范定义。

**用途**：用于小屏幕设备的内容边距。

#### kMaterialMediumAndUpMargin

```dart
const double kMaterialMediumAndUpMargin = 24;
```

**作用**：中等断点的边距值，根据 Material Design 3 规范定义。

**用途**：用于平板和桌面设备的内容边距。

#### kMaterialPadding

```dart
const double kMaterialPadding = 4;
```

**作用**：紧凑断点的内边距值，根据 Material Design 3 规范定义。

**用途**：用于组件内部的内边距。

#### kNavigationRailDefaultPadding

```dart
const double kNavigationRailDefaultPadding = 8;
```

**作用**：NavigationRail 的默认内边距值。

**用途**：用于 `standardNavigationRail` 方法的默认内边距（第 354 行）。

### 类型定义

#### NavigationRailDestinationBuilder

```dart
typedef NavigationRailDestinationBuilder = NavigationRailDestination Function(
  int index,
  NavigationDestination destination,
);
```

**作用**：用于自定义 `NavigationDestination` 到 `NavigationRailDestination` 的转换。

**参数说明**：

- `index`：导航目标在列表中的索引
- `destination`：原始的 `NavigationDestination` 对象

**返回值**：转换后的 `NavigationRailDestination` 对象

**使用场景**：当需要自定义 NavigationRail 的显示方式时，可以使用此构建器。

**示例**：

```dart
AdaptiveScaffold(
  destinations: destinations,
  navigationRailDestinationBuilder: (index, destination) {
    return NavigationRailDestination(
      label: Text('自定义 ${destination.label}'),
      icon: destination.icon,
      selectedIcon: destination.selectedIcon,
    );
  },
)
```

### AdaptiveScaffold 类

#### AdaptiveScaffold 构造函数

```dart
const AdaptiveScaffold({
  super.key,
  required this.destinations,
  this.selectedIndex = 0,
  this.leadingUnextendedNavRail,
  this.leadingExtendedNavRail,
  this.trailingNavRail,
  this.navigationRailPadding = const EdgeInsets.all(kNavigationRailDefaultPadding),
  this.smallBody,
  this.body,
  this.mediumLargeBody,
  this.largeBody,
  this.extraLargeBody,
  this.smallSecondaryBody,
  this.secondaryBody,
  this.mediumLargeSecondaryBody,
  this.largeSecondaryBody,
  this.extraLargeSecondaryBody,
  this.bodyRatio,
  this.smallBreakpoint = Breakpoints.small,
  this.mediumBreakpoint = Breakpoints.medium,
  this.mediumLargeBreakpoint = Breakpoints.mediumLarge,
  this.largeBreakpoint = Breakpoints.large,
  this.extraLargeBreakpoint = Breakpoints.extraLarge,
  this.drawerBreakpoint = Breakpoints.smallDesktop,
  this.internalAnimations = true,
  this.transitionDuration = const Duration(seconds: 1),
  this.bodyOrientation = Axis.horizontal,
  this.onSelectedIndexChange,
  this.useDrawer = true,
  this.appBar,
  this.navigationRailWidth = 72,
  this.extendedNavigationRailWidth = 192,
  this.appBarBreakpoint,
  this.navigationRailDestinationBuilder,
  this.groupAlignment,
}) : assert(
      destinations.length >= 2,
      'At least two destinations are required',
    );
```

**特点**：

- 使用 `const` 构造函数，支持编译时常量
- 所有参数都有合理的默认值
- 通过断言确保至少有两个导航目标

#### 构造函数参数详解

##### destinations - 导航目标列表

```dart
final List<NavigationDestination> destinations;
```

**作用**：定义导航目标列表，这些目标会被转换为 `NavigationRailDestination` 和 `BottomNavigationBarItem`。

**要求**：必须提供至少两个目标（通过断言检查）。

**转换机制**：

- 在 `_AdaptiveScaffoldState.build()` 中（第 575-580 行），通过 `navigationRailDestinationBuilder` 或 `toRailDestination` 转换为 `NavigationRailDestination`
- 在 `standardBottomNavigationBar` 中（第 433 行），直接使用 `NavigationDestination` 列表

**示例**：

```dart
destinations: const [
  NavigationDestination(icon: Icon(Icons.inbox), label: '收件箱'),
  NavigationDestination(icon: Icon(Icons.article), label: '文章'),
  NavigationDestination(icon: Icon(Icons.chat), label: '聊天'),
]
```

##### selectedIndex - 当前选中索引

```dart
final int? selectedIndex;
```

**作用**：指定当前选中的导航目标索引。

**默认值**：`0`（第一个目标）。

**使用场景**：通常与状态管理结合，当用户选择不同目标时更新此值。

##### Body 参数系列

`AdaptiveScaffold` 提供了多个 body 参数，用于在不同断点下显示不同的内容：

- `smallBody`：小屏幕断点的 body（第 162-167 行）
- `body`：默认 body，用于中等断点（第 169-172 行）
- `mediumLargeBody`：中等大屏幕断点的 body（第 174-179 行）
- `largeBody`：大屏幕断点的 body（第 181-186 行）
- `extraLargeBody`：超大屏幕断点的 body（第 188-193 行）

**回退机制**：如果没有指定特定断点的 body，则使用默认的 `body`。如果指定为 `null`，则该槽位保持为空。

**实现逻辑**（第 713-771 行）：

```dart
body: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.standard: SlotLayout.from(
      key: const Key('body'),
      builder: widget.body,  // 默认 body
    ),
    if (widget.smallBody != null)
      widget.smallBreakpoint: SlotLayout.from(
        key: const Key('smallBody'),
        builder: widget.smallBody,
      ),
    // ... 其他断点
  },
)
```

##### SecondaryBody 参数系列

与 body 参数类似，`AdaptiveScaffold` 也提供了多个 secondaryBody 参数：

- `smallSecondaryBody`：小屏幕断点的 secondaryBody（第 195-201 行）
- `secondaryBody`：默认 secondaryBody（第 203-206 行）
- `mediumLargeSecondaryBody`：中等大屏幕断点的 secondaryBody（第 208-214 行）
- `largeSecondaryBody`：大屏幕断点的 secondaryBody（第 216-222 行）
- `extraLargeSecondaryBody`：超大屏幕断点的 secondaryBody（第 224-230 行）

**用途**：用于实现主从视图（Master-Detail）模式，在较大屏幕上同时显示列表和详情。

##### 断点配置参数

`AdaptiveScaffold` 允许自定义所有断点：

- `smallBreakpoint`：小屏幕断点，默认 `Breakpoints.small`（第 241-245 行）
- `mediumBreakpoint`：中等屏幕断点，默认 `Breakpoints.medium`（第 247-251 行）
- `mediumLargeBreakpoint`：中等大屏幕断点，默认 `Breakpoints.mediumLarge`（第 253-257 行）
- `largeBreakpoint`：大屏幕断点，默认 `Breakpoints.large`（第 259-263 行）
- `extraLargeBreakpoint`：超大屏幕断点，默认 `Breakpoints.extraLarge`（第 265-269 行）
- `drawerBreakpoint`：Drawer 断点，默认 `Breakpoints.smallDesktop`（第 294-298 行）

**使用场景**：当默认断点不符合应用需求时，可以自定义断点。

##### 导航相关参数

- `leadingUnextendedNavRail`：中等断点下 NavigationRail 顶部的 Widget（第 144-146 行）
- `leadingExtendedNavRail`：大屏幕断点下扩展 NavigationRail 顶部的 Widget（第 148-150 行）
- `trailingNavRail`：扩展 NavigationRail 底部的 Widget（第 152-154 行）
- `navigationRailPadding`：NavigationRail 的内边距，默认 `EdgeInsets.all(8)`（第 156-157 行）
- `groupAlignment`：导航目标在 NavigationRail 中的对齐方式（第 159-160 行）

**使用示例**：

```dart
AdaptiveScaffold(
  destinations: destinations,
  leadingExtendedNavRail: IconButton(
    icon: Icon(Icons.menu),
    onPressed: () {},
  ),
  trailingNavRail: IconButton(
    icon: Icon(Icons.settings),
    onPressed: () {},
  ),
)
```

##### 动画相关参数

- `transitionDuration`：布局过渡动画时长，默认 `Duration(seconds: 1)`（第 277-280 行）
- `internalAnimations`：是否启用内部动画，默认 `true`（第 271-275 行）

**说明**：这些参数直接传递给内部的 `AdaptiveLayout`（第 608-611 行）。

##### 其他重要参数

- `bodyRatio`：body 和 secondaryBody 的比例（第 232-239 行）
- `bodyOrientation`：body 的排列方向，默认 `Axis.horizontal`（第 282-286 行）
- `useDrawer`：是否使用 Drawer，默认 `true`（第 288-292 行）
- `appBar`：自定义 AppBar（第 307-309 行）
- `appBarBreakpoint`：AppBar 显示的断点（第 300-305 行）
- `navigationRailWidth`：NavigationRail 的宽度，默认 72（第 314-315 行）
- `extendedNavigationRailWidth`：扩展 NavigationRail 的宽度，默认 192（第 317-319 行）
- `onSelectedIndexChange`：导航目标选择回调（第 311-312 行）

#### 静态辅助方法

##### toRailDestination

```dart
static NavigationRailDestination toRailDestination(
  NavigationDestination destination,
) {
  return NavigationRailDestination(
    label: Text(destination.label),
    icon: destination.icon,
    selectedIcon: destination.selectedIcon,
  );
}
```

**作用**：将 `NavigationDestination` 转换为 `NavigationRailDestination`。

**转换逻辑**：

- `label`：使用 `Text` Widget 包装原始标签
- `icon`：直接使用原始图标
- `selectedIcon`：直接使用原始选中图标

**使用场景**：在 `_AdaptiveScaffoldState.build()` 中（第 575-580 行），如果没有提供自定义的 `navigationRailDestinationBuilder`，则使用此方法进行转换。

##### standardNavigationRail

```dart
static Builder standardNavigationRail({
  required List<NavigationRailDestination> destinations,
  double width = 72,
  int? selectedIndex,
  bool extended = false,
  Color? backgroundColor,
  EdgeInsetsGeometry padding = const EdgeInsets.all(kNavigationRailDefaultPadding),
  Widget? leading,
  Widget? trailing,
  void Function(int)? onDestinationSelected,
  double? groupAlignment,
  IconThemeData? selectedIconTheme,
  IconThemeData? unselectedIconTheme,
  TextStyle? selectedLabelTextStyle,
  TextStyle? unSelectedLabelTextStyle,
  NavigationRailLabelType? labelType = NavigationRailLabelType.none,
})
```

**作用**：创建符合 Material 3 设计规范的 `NavigationRail`。

**实现细节**（第 348-404 行）：

1. **宽度处理**：如果 `extended` 为 `true` 且宽度为默认值 72，则自动设置为 192（第 366-368 行）
2. **布局结构**：
   - 使用 `Padding` 包裹，应用内边距
   - 使用 `SizedBox` 固定宽度和高度
   - 使用 `LayoutBuilder` 获取可用高度
   - 使用 `SingleChildScrollView` 支持滚动
   - 使用 `ConstrainedBox` 和 `IntrinsicHeight` 确保正确布局

**使用场景**：在 `_AdaptiveScaffoldState.build()` 中（第 616-631 行、第 635-651 行等），用于创建不同断点下的 NavigationRail。

##### standardBottomNavigationBar

```dart
static Builder standardBottomNavigationBar({
  required List<NavigationDestination> destinations,
  int? currentIndex,
  double iconSize = 24,
  ValueChanged<int>? onDestinationSelected,
})
```

**作用**：创建符合 Material 3 设计规范的 `BottomNavigationBar`（实际使用 `NavigationBar`）。

**实现细节**（第 408-440 行）：

1. **主题处理**：从 `NavigationBarTheme` 获取当前主题（第 416-417 行）
2. **图标大小**：通过 `NavigationBarTheme` 的 `iconTheme` 设置图标大小（第 419-427 行）
3. **MediaQuery 处理**：移除顶部内边距，避免与系统状态栏冲突（第 429-430 行）
4. **NavigationBar 使用**：直接使用 `NavigationBar` Widget，传入 `NavigationDestination` 列表（第 431-435 行）

**使用场景**：在 `_AdaptiveScaffoldState.build()` 中（第 704-708 行），用于创建小屏幕下的底部导航。

##### toMaterialGrid

```dart
static Builder toMaterialGrid({
  List<Widget> widgets = const <Widget>[],
  List<Breakpoint> breakpoints = Breakpoints.all,
  double? margin,
  int? itemColumns,
})
```

**作用**：创建符合 Material 3 规范的网格布局（瀑布流布局）。

**实现细节**（第 442-478 行）：

1. **断点检测**：使用 `Breakpoint.activeBreakpointIn()` 获取当前断点（第 451-452 行）
2. **边距计算**：使用传入的 `margin` 或当前断点的 `margin` 或 `kMaterialCompactMargin`（第 453-454 行）
3. **列数计算**：使用传入的 `itemColumns` 或当前断点的 `recommendedPanes` 或 1（第 455-456 行）
4. **布局实现**：使用 `_BrickLayout` 实现瀑布流布局（第 467-472 行）

**使用场景**：用于创建响应式的网格布局，如卡片列表、图片网格等。

**示例**：

```dart
AdaptiveScaffold.toMaterialGrid(
  widgets: cardWidgets,
  margin: 16,
  itemColumns: 2,
)
```

##### 动画辅助方法

`AdaptiveScaffold` 提供了多个静态动画辅助方法，用于 `SlotLayoutConfig` 的 `inAnimation` 和 `outAnimation`：

- `bottomToTop`：从底部滑入（第 480-489 行）
- `topToBottom`：向底部滑出（第 491-500 行）
- `leftOutIn`：从左侧滑入（第 502-511 行）
- `leftInOut`：向左侧滑出（第 513-522 行）
- `rightOutIn`：从右侧滑入（第 524-533 行）
- `fadeIn`：淡入（第 535-541 行）
- `fadeOut`：淡出（第 543-552 行）
- `stayOnScreen`：保持在屏幕上（第 554-560 行）

**实现方式**：这些方法都返回 `AnimatedWidget`，使用 `SlideTransition` 或 `FadeTransition` 实现动画效果。

**使用场景**：在自定义 `SlotLayoutConfig` 时，可以使用这些方法作为动画函数。

**示例**：

```dart
SlotLayout.from(
  key: const Key('body'),
  inAnimation: AdaptiveScaffold.fadeIn,
  outAnimation: AdaptiveScaffold.fadeOut,
  builder: (_) => ContentWidget(),
)
```

### _AdaptiveScaffoldState 状态类

`_AdaptiveScaffoldState` 是 `AdaptiveScaffold` 的状态类，负责管理 Scaffold 状态和 Drawer 的打开/关闭。

#### Scaffold 状态管理

```dart
final GlobalKey<ScaffoldState> _scaffoldKey = GlobalKey<ScaffoldState>();
```

**作用**：用于管理 Scaffold 的状态，特别是 Drawer 的打开/关闭。

**使用场景**：在 `build()` 方法中（第 583 行），将 `_scaffoldKey` 传递给 `Scaffold`，以便后续控制 Drawer。

#### Drawer 的打开/关闭处理

`_AdaptiveScaffoldState` 实现了 `_onDrawerDestinationSelected` 方法（第 831-843 行），用于处理 Drawer 中导航目标的选择：

```dart
void _onDrawerDestinationSelected(int index) {
  if (widget.useDrawer) {
    final ScaffoldState? scaffoldCurrentContext = _scaffoldKey.currentState;
    if (scaffoldCurrentContext != null) {
      if (scaffoldCurrentContext.isDrawerOpen) {
        scaffoldCurrentContext.closeDrawer();
      }
    }
  }
  widget.onSelectedIndexChange?.call(index);
}
```

**逻辑说明**：

1. 检查 `useDrawer` 是否为 `true`
2. 获取 Scaffold 的当前状态
3. 如果 Drawer 是打开的，则关闭它（符合 Material Design 规范）
4. 调用 `onSelectedIndexChange` 回调，通知外部导航目标已改变

**设计原因**：根据 Material Design 规范，当用户在 Drawer 中选择导航目标时，Drawer 应该自动关闭。

#### 导航目标选择回调

`_AdaptiveScaffoldState` 通过 `widget.onSelectedIndexChange` 将导航目标选择事件传递给外部（第 623 行、第 643 行、第 707 行等）。

**使用场景**：外部可以通过此回调更新应用状态，如切换页面内容。

### \_BrickLayout 和 \_BrickLayoutDelegate

`_BrickLayout` 和 `_BrickLayoutDelegate` 用于实现 Material 3 规范的网格布局（瀑布流布局）。

#### _BrickLayout

```dart
class _BrickLayout extends StatelessWidget {
  const _BrickLayout({
    this.columns = 1,
    this.itemPadding = EdgeInsets.zero,
    this.columnSpacing = 0,
    required this.children,
  });

  final int columns;
  final double columnSpacing;
  final EdgeInsetsGeometry itemPadding;
  final List<Widget> children;
}
```

**作用**：实现瀑布流布局的 Widget。

**实现方式**（第 860-881 行）：

1. 使用 `Column` 作为容器
2. 使用 `CustomMultiChildLayout` 和 `_BrickLayoutDelegate` 进行布局
3. 为每个子 Widget 分配唯一的 `LayoutId`

#### _BrickLayoutDelegate

```dart
class _BrickLayoutDelegate extends MultiChildLayoutDelegate {
  _BrickLayoutDelegate({
    this.columns = 1,
    this.columnSpacing = 0,
    this.itemPadding = EdgeInsets.zero,
  });

  final int columns;
  final EdgeInsetsGeometry itemPadding;
  final double columnSpacing;
}
```

**作用**：实现瀑布流布局算法的布局委托。

**布局算法**（第 896-936 行）：

1. **测量阶段**：
   - 计算每个子 Widget 的尺寸（第 911-913 行）
   - 计算列宽：`(size.width - totalColumnSpacing) / columns`（第 918 行）

2. **定位阶段**：
   - 维护每列的使用高度数组 `columnUsage`（第 920-921 行）
   - 将每个子 Widget 放置在当前最短的列中（第 922-934 行）
   - 更新该列的使用高度

**算法特点**：

- **瀑布流效果**：通过将 Widget 放置在最短列中，实现瀑布流效果
- **均匀分布**：通过轮询列索引（`columnIndex = (columnIndex + 1) % columns`），确保 Widget 均匀分布
- **间距处理**：考虑列间距和项目内边距

**使用场景**：在 `toMaterialGrid` 方法中使用（第 467 行），用于创建响应式的网格布局。

## 布局机制详解

### 导航元素自动切换

`AdaptiveScaffold` 根据屏幕尺寸自动切换导航元素，实现逻辑在 `_AdaptiveScaffoldState.build()` 中（第 607-827 行）。

#### 小屏幕：BottomNavigationBar

**触发条件**：`smallBreakpoint` 激活且不在 `drawerBreakpoint` 或 `useDrawer` 为 `false`。

**实现代码**（第 697-712 行）：

```dart
bottomNavigation:
    !widget.drawerBreakpoint.isActive(context) || !widget.useDrawer
        ? SlotLayout(
            config: <Breakpoint, SlotLayoutConfig>{
              widget.smallBreakpoint: SlotLayout.from(
                key: const Key('bottomNavigation'),
                builder: (_) =>
                    AdaptiveScaffold.standardBottomNavigationBar(
                  currentIndex: widget.selectedIndex,
                  destinations: widget.destinations,
                  onDestinationSelected: widget.onSelectedIndexChange,
                ),
              ),
            },
          )
        : null,
```

**特点**：

- 使用 `NavigationBar` Widget（在 `standardBottomNavigationBar` 中）
- 直接使用 `NavigationDestination` 列表
- 位于屏幕底部

#### 中等屏幕：NavigationRail（未扩展）

**触发条件**：`mediumBreakpoint` 激活。

**实现代码**（第 614-632 行）：

```dart
widget.mediumBreakpoint: SlotLayout.from(
  key: const Key('primaryNavigation'),
  builder: (_) => AdaptiveScaffold.standardNavigationRail(
    width: widget.navigationRailWidth,  // 默认 72
    leading: widget.leadingUnextendedNavRail,
    trailing: widget.trailingNavRail,
    padding: widget.navigationRailPadding,
    selectedIndex: widget.selectedIndex,
    destinations: destinations,
    onDestinationSelected: widget.onSelectedIndexChange,
    // ... 主题相关参数
  ),
),
```

**特点**：

- 宽度为 72（默认值）
- 不显示标签（`labelType: NavigationRailLabelType.none`）
- 位于屏幕左侧（LTR）或右侧（RTL）

#### 大屏幕：NavigationRail（扩展）

**触发条件**：`mediumLargeBreakpoint`、`largeBreakpoint` 或 `extraLargeBreakpoint` 激活。

**实现代码**（第 633-694 行）：

```dart
widget.mediumLargeBreakpoint: SlotLayout.from(
  key: const Key('primaryNavigation1'),
  builder: (_) => AdaptiveScaffold.standardNavigationRail(
    width: widget.extendedNavigationRailWidth,  // 默认 192
    extended: true,  // 扩展模式
    leading: widget.leadingExtendedNavRail,
    trailing: widget.trailingNavRail,
    // ... 其他参数
  ),
),
```

**特点**：

- 宽度为 192（默认值）
- `extended: true`，显示标签
- 可以使用 `leadingExtendedNavRail` 和 `trailingNavRail`

#### 小桌面：Drawer

**触发条件**：`drawerBreakpoint` 激活且 `useDrawer` 为 `true`。

**实现代码**（第 588-606 行）：

```dart
drawer: widget.drawerBreakpoint.isActive(context) && widget.useDrawer
    ? Drawer(
        child: NavigationRail(
          extended: true,
          leading: widget.leadingExtendedNavRail,
          trailing: widget.trailingNavRail,
          selectedIndex: widget.selectedIndex,
          destinations: destinations,
          onDestinationSelected: _onDrawerDestinationSelected,
          // ... 主题相关参数
        ),
      )
    : null,
```

**特点**：

- 使用 `Drawer` Widget 包裹 `NavigationRail`
- `extended: true`，显示标签
- 选择目标后自动关闭（在 `_onDrawerDestinationSelected` 中处理）

### Body 和 SecondaryBody 的断点处理

`AdaptiveScaffold` 根据断点选择不同的 body 内容，实现逻辑在 `_AdaptiveScaffoldState.build()` 中（第 713-826 行）。

#### Body 的断点处理

**实现代码**（第 713-771 行）：

```dart
body: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.standard: SlotLayout.from(
      key: const Key('body'),
      inAnimation: AdaptiveScaffold.fadeIn,
      outAnimation: AdaptiveScaffold.fadeOut,
      builder: widget.body,  // 默认 body
    ),
    if (widget.smallBody != null)
      widget.smallBreakpoint: SlotLayout.from(
        key: const Key('smallBody'),
        inAnimation: AdaptiveScaffold.fadeIn,
        outAnimation: AdaptiveScaffold.fadeOut,
        builder: widget.smallBody,
      ),
    // ... 其他断点
  },
)
```

**回退机制**：

1. 如果指定了特定断点的 body（如 `smallBody`），则使用该 body
2. 如果没有指定，则使用默认的 `body`
3. 如果 `body` 为 `null`，则使用 `emptyBuilder`（返回 `SizedBox()`）

**空值处理**（第 723-729 行）：

```dart
if (widget.smallBody != null)
  widget.smallBreakpoint:
      (widget.smallBody != AdaptiveScaffold.emptyBuilder)
          ? SlotLayout.from(...)
          : null,
```

如果 `smallBody` 等于 `emptyBuilder`，则在该断点下不显示 body。

#### SecondaryBody 的断点处理

**实现代码**（第 773-826 行）：

与 body 类似，但使用 `stayOnScreen` 作为 `outAnimation`，确保在切换时保持在屏幕上。

**特点**：

- 使用 `stayOnScreen` 动画，避免在切换时消失
- 支持主从视图模式

### Drawer 处理逻辑

#### 何时显示 Drawer

Drawer 在以下条件下显示（第 588 行）：

```dart
widget.drawerBreakpoint.isActive(context) && widget.useDrawer
```

- `drawerBreakpoint` 激活（默认 `Breakpoints.smallDesktop`）
- `useDrawer` 为 `true`（默认值）

#### Drawer 中的 NavigationRail 配置

Drawer 中的 NavigationRail 配置（第 590-604 行）：

```dart
Drawer(
  child: NavigationRail(
    extended: true,  // 扩展模式，显示标签
    leading: widget.leadingExtendedNavRail,
    trailing: widget.trailingNavRail,
    selectedIndex: widget.selectedIndex,
    destinations: destinations,
    onDestinationSelected: _onDrawerDestinationSelected,  // 自定义回调
    // ... 主题相关参数
  ),
)
```

**特点**：

- `extended: true`，显示标签
- 使用 `_onDrawerDestinationSelected` 作为回调，实现自动关闭功能

#### Drawer 关闭逻辑

Drawer 关闭逻辑在 `_onDrawerDestinationSelected` 方法中（第 831-843 行）：

```dart
void _onDrawerDestinationSelected(int index) {
  if (widget.useDrawer) {
    final ScaffoldState? scaffoldCurrentContext = _scaffoldKey.currentState;
    if (scaffoldCurrentContext != null) {
      if (scaffoldCurrentContext.isDrawerOpen) {
        scaffoldCurrentContext.closeDrawer();  // 关闭 Drawer
      }
    }
  }
  widget.onSelectedIndexChange?.call(index);  // 通知外部
}
```

**设计原因**：符合 Material Design 规范，选择导航目标后自动关闭 Drawer。

## 导航元素转换机制

### NavigationDestination 到 NavigationRailDestination

#### toRailDestination 方法的实现

`toRailDestination` 方法（第 329-337 行）实现了基本的转换逻辑：

```dart
static NavigationRailDestination toRailDestination(
  NavigationDestination destination,
) {
  return NavigationRailDestination(
    label: Text(destination.label),
    icon: destination.icon,
    selectedIcon: destination.selectedIcon,
  );
}
```

**转换规则**：

- `label`：使用 `Text` Widget 包装
- `icon`：直接使用
- `selectedIcon`：直接使用

#### 自定义转换器 navigationRailDestinationBuilder 的使用

在 `_AdaptiveScaffoldState.build()` 中（第 575-580 行），使用自定义转换器或默认转换器：

```dart
final List<NavigationRailDestination> destinations = widget.destinations
    .map((NavigationDestination destination) =>
        widget.navigationRailDestinationBuilder
            ?.call(widget.destinations.indexOf(destination), destination) ??
        AdaptiveScaffold.toRailDestination(destination))
    .toList();
```

**优先级**：

1. 如果提供了 `navigationRailDestinationBuilder`，则使用自定义转换器
2. 否则，使用 `toRailDestination` 进行默认转换

**使用场景**：当需要自定义 NavigationRail 的显示方式时，如添加徽章、自定义标签样式等。

### NavigationDestination 到 BottomNavigationBarItem

在 `standardBottomNavigationBar` 方法中（第 431-435 行），直接使用 `NavigationBar` Widget，传入 `NavigationDestination` 列表：

```dart
NavigationBar(
  selectedIndex: currentIndex ?? 0,
  destinations: destinations,  // 直接使用 NavigationDestination 列表
  onDestinationSelected: onDestinationSelected,
)
```

**说明**：`NavigationBar` 原生支持 `NavigationDestination` 列表，无需转换。

## 代码示例

### 基本使用示例

最简单的使用方式：

```dart
AdaptiveScaffold(
  destinations: const [
    NavigationDestination(icon: Icon(Icons.inbox), label: '收件箱'),
    NavigationDestination(icon: Icon(Icons.article), label: '文章'),
    NavigationDestination(icon: Icon(Icons.chat), label: '聊天'),
  ],
  body: (_) => Center(
    child: Text('主要内容'),
  ),
)
```

**说明**：

- 只需提供 `destinations` 和 `body`
- 导航元素会根据屏幕尺寸自动切换
- 小屏幕显示 BottomNavigationBar，中等屏幕显示 NavigationRail

### 多断点配置示例

为不同断点配置不同的 body：

```dart
AdaptiveScaffold(
  destinations: destinations,
  smallBody: (_) => ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) => ListTile(
      title: Text(items[index].title),
    ),
  ),
  body: (_) => GridView.count(
    crossAxisCount: 2,
    children: items.map((item) => Card(
      child: Text(item.title),
    )).toList(),
  ),
  mediumLargeBody: (_) => GridView.count(
    crossAxisCount: 3,
    children: items.map((item) => Card(
      child: Text(item.title),
    )).toList(),
  ),
)
```

**说明**：

- `smallBody`：小屏幕使用列表视图
- `body`：中等屏幕使用 2 列网格
- `mediumLargeBody`：大屏幕使用 3 列网格

### 主从视图示例

使用 secondaryBody 实现主从视图：

```dart
class MasterDetailExample extends StatefulWidget {
  @override
  _MasterDetailExampleState createState() => _MasterDetailExampleState();
}

class _MasterDetailExampleState extends State<MasterDetailExample> {
  Item? selectedItem;

  @override
  Widget build(BuildContext context) {
    return AdaptiveScaffold(
      destinations: destinations,
      body: (_) => ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) => ListTile(
          title: Text(items[index].title),
          selected: items[index] == selectedItem,
          onTap: () {
            setState(() => selectedItem = items[index]);
          },
        ),
      ),
      secondaryBody: selectedItem != null
          ? (_) => DetailView(item: selectedItem!)
          : null,
      bodyRatio: 0.4,  // 主视图占 40%，详情视图占 60%
    );
  }
}
```

**说明**：

- `body`：显示主列表
- `secondaryBody`：显示详情视图，仅在选中项时显示
- `bodyRatio`：控制主从视图的比例

### 自定义导航示例

使用自定义的导航元素：

```dart
AdaptiveScaffold(
  destinations: destinations,
  leadingExtendedNavRail: IconButton(
    icon: Icon(Icons.menu),
    onPressed: () {
      // 打开菜单
    },
  ),
  trailingNavRail: Column(
    mainAxisAlignment: MainAxisAlignment.end,
    children: [
      IconButton(
        icon: Icon(Icons.settings),
        onPressed: () {
          // 打开设置
        },
      ),
      IconButton(
        icon: Icon(Icons.help),
        onPressed: () {
          // 显示帮助
        },
      ),
    ],
  ),
  navigationRailDestinationBuilder: (index, destination) {
    return NavigationRailDestination(
      label: Text('自定义 ${destination.label}'),
      icon: Badge(
        child: destination.icon,
        label: Text('${index + 1}'),
      ),
      selectedIcon: destination.selectedIcon,
    );
  },
)
```

**说明**：

- `leadingExtendedNavRail`：在扩展 NavigationRail 顶部添加菜单按钮
- `trailingNavRail`：在扩展 NavigationRail 底部添加设置和帮助按钮
- `navigationRailDestinationBuilder`：自定义导航目标的显示方式，添加徽章

## 最佳实践

### 断点配置建议

#### 何时使用默认断点

**建议**：在大多数情况下，使用默认断点即可满足需求。

**原因**：

- 默认断点符合 Material Design 3 规范
- 经过充分测试，兼容性好
- 减少配置代码

#### 何时自定义断点

**场景**：

1. **特殊设备**：需要适配特殊尺寸的设备
2. **特定布局需求**：应用的布局需求与标准断点不匹配
3. **A/B 测试**：需要测试不同的断点配置

**示例**：

```dart
AdaptiveScaffold(
  destinations: destinations,
  mediumBreakpoint: Breakpoint(
    start: 600,
    end: 840,
    margin: 24,
  ),
  // ... 其他配置
)
```

### 性能优化建议

#### 合理使用 const

**建议**：尽可能使用 `const` 构造函数。

**原因**：

- 减少 Widget 重建
- 提高性能
- 减少内存占用

**示例**：

```dart
AdaptiveScaffold(
  destinations: const [  // const 列表
    NavigationDestination(icon: Icon(Icons.inbox), label: '收件箱'),
    NavigationDestination(icon: Icon(Icons.article), label: '文章'),
  ],
  body: (_) => const ContentWidget(),  // const Widget
)
```

#### 避免不必要的重建

**建议**：使用 `StatefulWidget` 管理状态，避免在 `build()` 方法中创建新对象。

**错误示例**：

```dart
AdaptiveScaffold(
  destinations: [
    NavigationDestination(icon: Icon(Icons.inbox), label: '收件箱'),  // 每次重建都创建新对象
  ],
  body: (_) => ListView.builder(
    itemBuilder: (context, index) => ItemWidget(items[index]),  // 每次重建都创建新列表
  ),
)
```

**正确示例**：

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final List<NavigationDestination> destinations = const [
    NavigationDestination(icon: Icon(Icons.inbox), label: '收件箱'),
  ];
  
  final List<Item> items = [];  // 在 State 中管理
  
  @override
  Widget build(BuildContext context) {
    return AdaptiveScaffold(
      destinations: destinations,
      body: (_) => ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) => ItemWidget(items[index]),
      ),
    );
  }
}
```

### 常见问题解决

#### 导航元素不显示

**可能原因**：

- `destinations` 列表为空或只有一个元素
- 断点配置不正确
- `useDrawer` 设置错误

**解决方法**：

步骤 1：检查 destinations

确保至少有两个导航目标：

```dart
assert(
  destinations.length >= 2,
  'At least two destinations are required',
);
```

步骤 2：检查断点

确保断点配置正确：

```dart
AdaptiveScaffold(
  destinations: destinations,
  smallBreakpoint: Breakpoints.small,  // 确保断点正确
  mediumBreakpoint: Breakpoints.medium,
)
```

步骤 3：检查 useDrawer

如果在小桌面设备上，确保 `useDrawer` 设置正确：

```dart
AdaptiveScaffold(
  destinations: destinations,
  useDrawer: true,  // 确保启用 Drawer
)
```

#### 动画不流畅

**可能原因**：

- `transitionDuration` 设置过长
- `internalAnimations` 被禁用
- 设备性能不足

**解决方法**：

步骤 1：调整动画时长

```dart
AdaptiveScaffold(
  destinations: destinations,
  transitionDuration: const Duration(milliseconds: 300),  // 缩短时长
)
```

步骤 2：启用内部动画

```dart
AdaptiveScaffold(
  destinations: destinations,
  internalAnimations: true,  // 确保启用
)
```

步骤 3：性能优化

在低端设备上禁用动画：

```dart
AdaptiveScaffold(
  destinations: destinations,
  internalAnimations: false,  // 禁用动画以提高性能
  transitionDuration: Duration.zero,
)
```

#### Drawer 不工作

**可能原因**：

- `drawerBreakpoint` 未激活
- `useDrawer` 为 `false`
- `_scaffoldKey` 未正确设置

**解决方法**：

步骤 1：检查断点

确保 `drawerBreakpoint` 正确激活：

```dart
AdaptiveScaffold(
  destinations: destinations,
  drawerBreakpoint: Breakpoints.smallDesktop,  // 确保断点正确
)
```

步骤 2：启用 Drawer

```dart
AdaptiveScaffold(
  destinations: destinations,
  useDrawer: true,  // 确保启用
)
```

步骤 3：检查 Scaffold Key

`_AdaptiveScaffoldState` 内部已正确设置，无需手动处理。

## 总结

`AdaptiveScaffold` 是一个强大的响应式布局组件，它基于 `AdaptiveLayout` 构建，提供了更易用的 API 和预设的布局结构。它支持：

- **自动导航切换**：根据屏幕尺寸自动在 BottomNavigationBar、NavigationRail、Drawer 之间切换
- **多断点支持**：支持为不同断点配置不同的 body 和 secondaryBody
- **Material Design 3 规范**：严格遵循 M3 的间距、边距、内边距规范
- **主从视图**：通过 secondaryBody 实现主从视图模式
- **流畅动画**：提供平滑的布局过渡效果
- **易于使用**：减少配置代码，提供更高级的抽象

通过合理使用 `AdaptiveScaffold`，可以快速构建出适配各种屏幕尺寸和设备的现代化应用界面。当需要更精细的控制时，可以使用 `AdaptiveLayout` 进行自定义配置。
