# 第 6 章 SlotLayout 槽位布局系统

## 引言

`SlotLayout` 是 `flutter_adaptive_scaffold` 库的核心组件，它负责根据当前屏幕条件选择并显示合适的 Widget。本章将深入分析 `SlotLayout` 的工作原理、配置机制、动画系统，以及槽位之间的协调机制。

## SlotLayout 工作原理

### 核心职责

`SlotLayout` 的主要职责是：

1. **断点匹配**：检测当前屏幕匹配哪个断点
2. **配置选择**：从配置映射中选择对应的 `SlotLayoutConfig`
3. **Widget 构建**：使用 `builder` 构建 Widget
4. **动画过渡**：在切换时执行动画

### 内部实现

`SlotLayout` 内部使用 `AnimatedSwitcher` 实现过渡：

```dart
class _SlotLayoutState extends State<SlotLayout> {
  SlotLayoutConfig? chosenWidget;

  @override
  Widget build(BuildContext context) {
    // 1. 选择匹配的配置
    chosenWidget = SlotLayout.pickWidget(context, widget.config);
    
    // 2. 使用 AnimatedSwitcher 实现过渡
    return AnimatedSwitcher(
      duration: chosenWidget?.inDuration ?? const Duration(milliseconds: 1000),
      transitionBuilder: (Widget child, Animation<double> animation) {
        final SlotLayoutConfig configChild = child as SlotLayoutConfig;
        if (child.key == chosenWidget?.key) {
          // 进入动画
          return configChild.inAnimation != null
              ? configChild.inAnimation!(child, animation)
              : child;
        } else {
          // 退出动画
          return configChild.outAnimation != null
              ? configChild.outAnimation!(child, ReverseAnimation(animation))
              : child;
        }
      },
      child: chosenWidget ?? SlotLayoutConfig.empty(),
    );
  }
}
```

### pickWidget 方法

`pickWidget` 方法负责选择匹配的配置：

```dart
static SlotLayoutConfig? pickWidget(
  BuildContext context,
  Map<Breakpoint, SlotLayoutConfig?> config,
) {
  // 1. 找到所有匹配的断点
  final Breakpoint? breakpoint = Breakpoint.activeBreakpointIn(
    context,
    config.keys.toList(),
  );
  
  // 2. 返回对应的配置
  return breakpoint != null && config.containsKey(breakpoint)
      ? config[breakpoint]
      : null;
}
```

**关键点**：

- `activeBreakpointIn` 返回**最后一个匹配的断点**
- 如果多个断点匹配，后面的会覆盖前面的
- 如果没有匹配的断点，返回 `null`

## SlotLayoutConfig 详解

### 结构定义

`SlotLayoutConfig` 包含以下属性：

```dart
class SlotLayoutConfig extends StatelessWidget {
  final WidgetBuilder? builder;           // Widget 构建器
  final Widget Function(Widget, Animation<double>)? inAnimation;   // 进入动画
  final Widget Function(Widget, Animation<double>)? outAnimation; // 退出动画
  final Duration? inDuration;             // 进入动画时长
  final Duration? outDuration;            // 退出动画时长
  final Curve? inCurve;                   // 进入动画曲线
  final Curve? outCurve;                  // 退出动画曲线
  final Key key;                          // 唯一标识
}
```

### SlotLayout.from 工厂方法

`SlotLayout.from` 是创建 `SlotLayoutConfig` 的便捷方法：

```dart
SlotLayout.from({
  WidgetBuilder? builder,
  Widget Function(Widget, Animation<double>)? inAnimation,
  Widget Function(Widget, Animation<double>)? outAnimation,
  Duration? inDuration,
  Duration? outDuration,
  Curve? inCurve,
  Curve? outCurve,
  required Key key,
})
```

### 必需属性

**key** 是唯一必需的属性，用于：

1. **Widget 识别**：帮助 Flutter 识别 Widget 的变化
2. **动画触发**：当 key 改变时触发动画
3. **状态保持**：保持 Widget 的状态

**最佳实践**：

```dart
// ✅ 推荐：使用描述性的 Key
SlotLayout.from(
  key: const Key('Body Small'),
  builder: (_) => Widget(),
)

// ❌ 避免：使用相同的 Key
SlotLayout.from(
  key: const Key('Body'),  // 多个配置使用相同 Key 会导致问题
  builder: (_) => Widget(),
)
```

## 配置映射机制

### 基础配置

`SlotLayout` 的 `config` 参数是一个映射：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.small: SlotLayout.from(
      key: const Key('Small'),
      builder: (_) => SmallWidget(),
    ),
    Breakpoints.large: SlotLayout.from(
      key: const Key('Large'),
      builder: (_) => LargeWidget(),
    ),
  },
)
```

### 可空配置

配置值可以是 `null`，表示在该断点下不显示：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.small: null,  // 小屏幕不显示
    Breakpoints.large: SlotLayout.from(
      key: const Key('Large'),
      builder: (_) => LargeWidget(),
    ),
  },
)
```

### 配置优先级

当多个断点匹配时，**最后一个匹配的断点**的配置会被使用：

```dart
SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Default'),
      builder: (_) => DefaultWidget(),
    ),
    Breakpoints.small: SlotLayout.from(
      key: const Key('Small'),
      builder: (_) => SmallWidget(),
    ),
    Breakpoints.largeAndUp: SlotLayout.from(
      key: const Key('Large'),
      builder: (_) => LargeWidget(),
    ),
  },
)
```

**匹配结果**：

- 400 dp：匹配 `Breakpoints.small` → 使用 `SmallWidget`
- 700 dp：匹配 `Breakpoints.standard` → 使用 `DefaultWidget`
- 1500 dp：匹配 `Breakpoints.largeAndUp` → 使用 `LargeWidget`

## 动画系统

### 内置动画

`AdaptiveScaffold` 提供了一些内置动画：

```dart
AdaptiveScaffold.leftOutIn      // 从左侧滑入/滑出
AdaptiveScaffold.rightOutIn    // 从右侧滑入/滑出
AdaptiveScaffold.topToBottom   // 从顶部滑入/滑出
AdaptiveScaffold.bottomToTop   // 从底部滑入/滑出
AdaptiveScaffold.stayOnScreen  // 保持在屏幕上（无动画）
```

### 使用内置动画

```dart
SlotLayout.from(
  key: const Key('Body'),
  inAnimation: AdaptiveScaffold.leftOutIn,   // 进入动画
  outAnimation: AdaptiveScaffold.rightOutIn, // 退出动画
  builder: (_) => Widget(),
)
```

### 自定义动画

可以创建完全自定义的动画：

```dart
SlotLayout.from(
  key: const Key('Body'),
  inAnimation: (Widget child, Animation<double> animation) {
    return FadeTransition(
      opacity: animation,
      child: SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(0.0, 0.3),
          end: Offset.zero,
        ).animate(animation),
        child: child,
      ),
    );
  },
  outAnimation: (Widget child, Animation<double> animation) {
    return FadeTransition(
      opacity: animation,
      child: child,
    );
  },
  builder: (_) => Widget(),
)
```

### 动画时长和曲线

可以单独控制进入和退出动画的时长和曲线：

```dart
SlotLayout.from(
  key: const Key('Body'),
  inDuration: Duration(milliseconds: 500),
  outDuration: Duration(milliseconds: 300),
  inCurve: Curves.easeInOut,
  outCurve: Curves.easeOut,
  builder: (_) => Widget(),
)
```

## 槽位协调机制

### 独立配置

每个槽位都是独立配置的，但它们需要协调工作：

```dart
AdaptiveLayout(
  // 主导航：中等屏幕及以上显示
  primaryNavigation: SlotLayout(
    config: {
      Breakpoints.mediumAndUp: SlotLayout.from(...),
    },
  ),
  
  // 主内容：所有屏幕都显示
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayout.from(...),
    },
  ),
  
  // 底部导航：仅小屏幕显示
  bottomNavigation: SlotLayout(
    config: {
      Breakpoints.small: SlotLayout.from(...),
    },
  ),
)
```

### 布局空间分配

`AdaptiveLayout` 负责分配空间：

1. **固定尺寸槽位**：`topNavigation`、`bottomNavigation`、`primaryNavigation`、`secondaryNavigation` 有固定或内容决定的尺寸
2. **灵活尺寸槽位**：`body` 和 `secondaryBody` 填充剩余空间

**布局顺序**：

```text
┌─────────────────────────────────┐
│      topNavigation (固定)        │
├──────┬──────────────────┬───────┤
│      │                  │       │
│ pri  │      body        │ sec   │
│ Nav  │    (灵活)        │ Nav   │
│      │                  │       │
│      ├──────────────────┤       │
│      │  secondaryBody   │       │
│      │    (灵活)        │       │
├──────┴──────────────────┴───────┤
│   bottomNavigation (固定)       │
└─────────────────────────────────┘
```

### 协调示例

实现主从视图模式：

```dart
AdaptiveLayout(
  // 主导航：中等屏幕及以上显示
  primaryNavigation: SlotLayout(
    config: {
      Breakpoints.mediumAndUp: SlotLayout.from(
        key: const Key('Nav'),
        builder: (_) => NavigationRail(...),
      ),
    },
  ),
  
  // 主列表：所有屏幕显示
  body: SlotLayout(
    config: {
      Breakpoints.standard: SlotLayout.from(
        key: const Key('Body'),
        builder: (_) => ListView(...),
      ),
    },
  ),
  
  // 详情视图：中等屏幕及以上显示
  secondaryBody: SlotLayout(
    config: {
      Breakpoints.mediumAndUp: SlotLayout.from(
        key: const Key('Detail'),
        builder: (_) => DetailView(...),
      ),
    },
  ),
  
  // 底部导航：仅小屏幕显示
  bottomNavigation: SlotLayout(
    config: {
      Breakpoints.small: SlotLayout.from(
        key: const Key('Bottom Nav'),
        builder: (_) => BottomNavigationBar(...),
      ),
    },
  ),
)
```

**行为**：

- **小屏幕**：`bottomNavigation` + `body`（列表）
- **中等屏幕及以上**：`primaryNavigation` + `body`（列表）+ `secondaryBody`（详情）

## 高级用法

### 1. 条件配置

根据应用状态动态配置：

```dart
SlotLayout(
  config: {
    Breakpoints.standard: isLoggedIn
        ? SlotLayout.from(
            key: const Key('Authenticated'),
            builder: (_) => AuthenticatedView(),
          )
        : SlotLayout.from(
            key: const Key('Login'),
            builder: (_) => LoginView(),
          ),
  },
)
```

### 2. 嵌套 SlotLayout

可以在 `builder` 中嵌套使用 `SlotLayout`：

```dart
SlotLayout(
  config: {
    Breakpoints.standard: SlotLayout.from(
      key: const Key('Body'),
      builder: (_) => SlotLayout(
        config: {
          Breakpoints.small: SlotLayout.from(
            key: const Key('Nested Small'),
            builder: (_) => SmallWidget(),
          ),
          Breakpoints.large: SlotLayout.from(
            key: const Key('Nested Large'),
            builder: (_) => LargeWidget(),
          ),
        },
      ),
    ),
  },
)
```

### 3. 共享状态

多个槽位可以共享状态：

```dart
class _MyHomePageState extends State<MyHomePage> {
  int selectedIndex = 0;
  
  @override
  Widget build(BuildContext context) {
    return AdaptiveLayout(
      primaryNavigation: SlotLayout(
        config: {
          Breakpoints.mediumAndUp: SlotLayout.from(
            key: const Key('Nav'),
            builder: (_) => NavigationRail(
              selectedIndex: selectedIndex,
              onDestinationSelected: (index) {
                setState(() {
                  selectedIndex = index;
                });
              },
            ),
          ),
        },
      ),
      body: SlotLayout(
        config: {
          Breakpoints.standard: SlotLayout.from(
            key: const Key('Body'),
            builder: (_) => ContentView(index: selectedIndex),
          ),
        },
      ),
    );
  }
}
```

## 性能考虑

### 1. Builder 延迟执行

`builder` 只在匹配的断点激活时执行：

```dart
// ✅ 推荐：延迟构建
SlotLayout.from(
  key: const Key('Body'),
  builder: (_) => ExpensiveWidget(),  // 只在需要时构建
)

// ❌ 避免：立即构建
final Widget expensiveWidget = ExpensiveWidget();
SlotLayout.from(
  key: const Key('Body'),
  builder: (_) => expensiveWidget,  // 即使不需要也会构建
)
```

### 2. Key 的稳定性

保持 Key 的稳定性，避免不必要的重建：

```dart
// ✅ 推荐：稳定的 Key
SlotLayout.from(
  key: const Key('Body Small'),  // const，稳定
  builder: (_) => Widget(),
)

// ❌ 避免：不稳定的 Key
SlotLayout.from(
  key: Key('Body_${DateTime.now()}'),  // 每次重建都不同
  builder: (_) => Widget(),
)
```

### 3. 避免不必要的配置

只配置需要的断点：

```dart
// ✅ 推荐：只配置需要的断点
SlotLayout(
  config: {
    Breakpoints.small: SlotLayout.from(...),
    Breakpoints.large: SlotLayout.from(...),
  },
)

// ❌ 避免：配置所有断点但内容相同
SlotLayout(
  config: {
    Breakpoints.small: SlotLayout.from(key: Key('A'), builder: (_) => Widget()),
    Breakpoints.medium: SlotLayout.from(key: Key('B'), builder: (_) => Widget()),
    Breakpoints.large: SlotLayout.from(key: Key('C'), builder: (_) => Widget()),
    // 如果内容相同，应该使用 standard
  },
)
```

## 总结

本章我们深入了解了：

- **SlotLayout 工作原理**：断点匹配、配置选择、Widget 构建、动画过渡
- **SlotLayoutConfig**：结构定义和配置方法
- **配置映射机制**：优先级和可空配置
- **动画系统**：内置动画和自定义动画
- **槽位协调**：独立配置和空间分配
- **高级用法**：条件配置、嵌套布局、状态共享
- **性能考虑**：延迟构建、Key 稳定性、配置优化

在下一章中，我们将深入学习动画与过渡效果的实现。

## 练习

1. 创建一个自定义动画，实现淡入淡出和缩放效果
2. 实现嵌套 `SlotLayout`，在 body 中根据断点显示不同内容
3. 分析 `main.dart` 中的 `SlotLayout` 配置，理解其协调机制

## 检查清单

- [ ] 理解 `SlotLayout` 的工作原理
- [ ] 掌握 `SlotLayoutConfig` 的配置方法
- [ ] 了解配置映射的优先级机制
- [ ] 能够使用内置和自定义动画
- [ ] 理解槽位协调机制
- [ ] 了解性能优化技巧
