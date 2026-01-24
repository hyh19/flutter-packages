# 第 9 章 实战案例 - 邮件应用

## 引言

`main.dart` 中的邮件应用示例是一个完整、功能丰富的应用，展示了 `AdaptiveLayout` 在实际项目中的应用。本章将深入分析这个应用的架构设计、实现细节和设计模式，帮助你理解如何在实际项目中应用自适应布局。

## 应用架构概览

### 功能特性

邮件应用包含以下功能：

1. **邮件列表**：显示邮件线程列表
2. **邮件详情**：显示选中邮件的详细信息
3. **响应式导航**：根据屏幕尺寸切换导航方式
4. **搜索功能**：搜索邮件
5. **文件夹管理**：管理邮件文件夹
6. **方向切换**：支持 LTR/RTL 切换

### 架构设计

应用采用以下架构：

```text
MyApp
└── MyHomePage (StatefulWidget)
    └── AdaptiveLayout
        ├── primaryNavigation (NavigationRail)
        ├── body (邮件列表或示例页面)
        ├── secondaryBody (邮件详情)
        └── bottomNavigation (BottomNavigationBar)
```

## 核心组件分析

### MyHomePage 状态管理

`MyHomePage` 使用 `StatefulWidget` 管理应用状态：

```dart
class _MyHomePageState extends State<MyHomePage>
    with TickerProviderStateMixin, ChangeNotifier {
  // 导航索引
  int _navigationIndex = 0;
  
  // 选中的邮件项
  int? selected;
  
  // 方向覆盖
  TextDirection directionalityOverride = TextDirection.ltr;
  
  // 动画控制器（用于导航项的交错动画）
  late AnimationController _inboxIconSlideController;
  late AnimationController _articleIconSlideController;
  late AnimationController _chatIconSlideController;
  late AnimationController _videoIconSlideController;
  
  // 网格视图显示状态
  ValueNotifier<bool?> showGridView = ValueNotifier<bool?>(false);
}
```

**关键设计点**：

1. **TickerProviderStateMixin**：提供动画控制器
2. **ChangeNotifier**：用于通知状态变化
3. **多个动画控制器**：实现导航项的交错动画
4. **ValueNotifier**：管理网格视图显示状态

### 导航配置

定义导航目标：

```dart
const List<NavigationDestination> destinations = <NavigationDestination>[
  NavigationDestination(
    label: 'Inbox',
    icon: Icon(Icons.inbox),
  ),
  NavigationDestination(
    label: 'Articles',
    icon: Icon(Icons.article_outlined),
  ),
  NavigationDestination(
    label: 'Chat',
    icon: Icon(Icons.chat_bubble_outline),
  ),
  NavigationDestination(
    label: 'Video',
    icon: Icon(Icons.video_call_outlined),
  ),
];
```

### PrimaryNavigation 配置

主导航在不同断点下显示不同样式：

```dart
primaryNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    // 中等屏幕：紧凑 NavigationRail，带交错动画
    Breakpoints.medium: SlotLayout.from(
      key: const Key('primaryNavigation'),
      builder: (_) {
        return AdaptiveScaffold.standardNavigationRail(
          onDestinationSelected: (int index) {
            setState(() {
              _navigationIndex = index;
            });
          },
          selectedIndex: _navigationIndex,
          leading: ScaleTransition(
            scale: _articleIconSlideController,
            child: const _MediumComposeIcon(),
          ),
          backgroundColor: const Color.fromARGB(0, 255, 255, 255),
          destinations: <NavigationRailDestination>[
            slideInNavigationItem(
              begin: -1,
              controller: _inboxIconSlideController,
              icon: Icons.inbox,
              label: 'Inbox',
            ),
            slideInNavigationItem(
              begin: -2,
              controller: _articleIconSlideController,
              icon: Icons.article_outlined,
              label: 'Articles',
            ),
            // ... 更多导航项
          ],
        );
      },
    ),
    
    // 中等大屏幕及以上：扩展 NavigationRail，带 trailing
    Breakpoints.mediumLarge: SlotLayout.from(
      key: const Key('MediumLarge primaryNavigation'),
      builder: (_) => AdaptiveScaffold.standardNavigationRail(
        leading: const _LargeComposeIcon(),
        onDestinationSelected: (int index) {
          setState(() {
            _navigationIndex = index;
          });
        },
        selectedIndex: _navigationIndex,
        trailing: trailingNavRail,  // 文件夹列表
        extended: true,
        destinations: destinations
            .map((NavigationDestination destination) {
          return AdaptiveScaffold.toRailDestination(destination);
        }).toList(),
      ),
    ),
    // large 和 extraLarge 配置类似
  },
)
```

**关键特性**：

1. **交错动画**：导航项依次滑入
2. **扩展模式**：大屏幕显示扩展的 NavigationRail
3. **Trailing 组件**：显示文件夹列表

### Body 配置

Body 根据导航索引显示不同内容：

```dart
body: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.standard: SlotLayout.from(
      key: const Key('body'),
      builder: (_) => (_navigationIndex == 0)
          ? Padding(
              padding: const EdgeInsets.fromLTRB(0, 32, 0, 0),
              child: _ItemList(
                selected: selected,
                items: _allItems,
                selectCard: selectCard,
              ),
            )
          : const _ExamplePage(),
    ),
  },
)
```

**设计模式**：

- **条件渲染**：根据 `_navigationIndex` 显示不同内容
- **第一个导航项**：显示邮件列表
- **其他导航项**：显示示例页面

### SecondaryBody 配置

详情视图仅在大屏幕上显示：

```dart
secondaryBody: _navigationIndex == 0
    ? SlotLayout(
        config: <Breakpoint, SlotLayoutConfig?>{
          Breakpoints.mediumAndUp: SlotLayout.from(
            // 使用 stayOnScreen 保持显示
            outAnimation: AdaptiveScaffold.stayOnScreen,
            key: const Key('Secondary Body'),
            builder: (_) => SafeArea(
              child: _DetailTile(item: _allItems[selected ?? 0]),
            ),
          ),
        },
      )
    : null,
```

**关键设计**：

1. **条件显示**：仅在第一个导航项时显示
2. **stayOnScreen**：退出动画保持显示，避免闪烁
3. **空值处理**：使用 `selected ?? 0` 提供默认值

### BottomNavigation 配置

底部导航仅在小屏幕显示：

```dart
bottomNavigation: SlotLayout(
  config: <Breakpoint, SlotLayoutConfig?>{
    Breakpoints.small: SlotLayout.from(
      key: const Key('bottomNavigation'),
      outAnimation: AdaptiveScaffold.topToBottom,
      builder: (_) => AdaptiveScaffold.standardBottomNavigationBar(
        destinations: destinations,
      ),
    ),
  },
)
```

## 邮件列表实现

### _ItemList 组件

`_ItemList` 显示邮件列表：

```dart
class _ItemList extends StatelessWidget {
  const _ItemList({
    required this.items,
    required this.selectCard,
    required this.selected,
  });

  final List<_Item> items;
  final int? selected;
  final _CardSelectedCallback selectCard;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color.fromARGB(0, 0, 0, 0),
      // 小屏幕显示浮动按钮
      floatingActionButton: Breakpoints.mediumAndUp.isActive(context)
          ? null
          : const _SmallComposeIcon(),
      body: Column(
        children: <Widget>[
          // 搜索栏
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: TextField(
              decoration: InputDecoration(
                prefixIcon: const Padding(
                  padding: EdgeInsets.fromLTRB(20, 0, 20, 0),
                  child: Icon(Icons.search),
                ),
                // ... 搜索栏配置
              ),
            ),
          ),
          // 邮件列表
          Expanded(
            child: ListView.builder(
              itemCount: items.length,
              itemBuilder: (BuildContext context, int index) => _ItemListTile(
                item: items[index],
                email: items[index].emails![0],
                selectCard: selectCard,
                selected: selected,
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**关键特性**：

1. **响应式 FAB**：小屏幕显示，大屏幕隐藏
2. **搜索功能**：顶部搜索栏
3. **列表显示**：使用 `ListView.builder` 优化性能

### _ItemListTile 组件

`_ItemListTile` 显示单个邮件项：

```dart
class _ItemListTile extends StatelessWidget {
  const _ItemListTile({
    required this.item,
    required this.email,
    required this.selectCard,
    required this.selected,
  });

  final _Item item;
  final _Email email;
  final int? selected;
  final _CardSelectedCallback selectCard;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        selectCard(_allItems.indexOf(item));
        // 小屏幕：使用模态显示详情
        if (!Breakpoints.mediumAndUp.isActive(context)) {
          Navigator.of(context).pushNamed(
            _ExtractRouteArguments.routeName,
            arguments: _ScreenArguments(
              item: item,
              selectCard: selectCard,
            ),
          );
        } else {
          // 大屏幕：直接显示在 secondaryBody
          selectCard(_allItems.indexOf(item));
        }
      },
      child: Padding(
        padding: const EdgeInsets.all(8.0),
        child: Container(
          decoration: BoxDecoration(
            color: selected == _allItems.indexOf(item)
                ? const Color.fromARGB(255, 234, 222, 255)
                : const Color.fromARGB(255, 243, 237, 247),
            borderRadius: const BorderRadius.all(Radius.circular(10)),
          ),
          child: Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                // 发件人信息
                ListTile(
                  contentPadding: EdgeInsets.zero,
                  leading: CircleAvatar(
                    radius: 18,
                    child: Image.asset(email.image, ...),
                  ),
                  title: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: <Widget>[
                      Text(email.sender, ...),
                      Text('${email.time} ago', ...),
                    ],
                  ),
                  trailing: Container(...),
                ),
                // 邮件标题和内容
                Text(item.title, ...),
                Text(email.body.replaceRange(116, email.body.length, '...'), ...),
                // 邮件图片（如果有）
                if (email.bodyImage != '')
                  Image.asset(email.bodyImage),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

**关键设计**：

1. **响应式行为**：小屏幕使用模态，大屏幕使用侧边栏
2. **选中状态**：高亮选中的邮件项
3. **内容截断**：长文本自动截断

## 详情视图实现

### _DetailTile 组件

`_DetailTile` 显示邮件详情：

```dart
class _DetailTile extends StatelessWidget {
  const _DetailTile({required this.item});

  final _Item item;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.all(8.0),
      child: SizedBox(
        height: MediaQuery.sizeOf(context).height,
        child: Container(
          decoration: const BoxDecoration(
            color: Color.fromARGB(255, 245, 241, 248),
            borderRadius: BorderRadius.all(Radius.circular(10)),
          ),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              // 标题和操作按钮
              Padding(
                padding: const EdgeInsets.fromLTRB(16, 16, 16, 0),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: <Widget>[
                    Expanded(
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: <Widget>[
                          Text(item.title, ...),
                          Text('${item.emails!.length} Messages', ...),
                        ],
                      ),
                    ),
                    // 操作按钮
                    Row(
                      children: <Widget>[
                        Container(...),  // 删除按钮
                        Container(...),  // 更多按钮
                      ],
                    ),
                  ],
                ),
              ),
              // 邮件列表
              Expanded(
                child: ListView.builder(
                  itemCount: item.emails!.length,
                  itemBuilder: (BuildContext context, int index) {
                    final _Email thisEmail = item.emails![index];
                    return _EmailTile(
                      sender: thisEmail.sender,
                      time: thisEmail.time,
                      senderIcon: thisEmail.image,
                      recipients: thisEmail.recipients,
                      body: thisEmail.body,
                      bodyImage: thisEmail.bodyImage,
                    );
                  },
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### _EmailTile 组件

`_EmailTile` 显示单封邮件：

```dart
class _EmailTile extends StatelessWidget {
  const _EmailTile({
    required this.sender,
    required this.time,
    required this.senderIcon,
    required this.recipients,
    required this.body,
    required this.bodyImage,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.fromLTRB(0, 4, 0, 4),
      child: Container(
        decoration: const BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.all(Radius.circular(10)),
        ),
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              // 发件人信息
              Row(
                children: <Widget>[
                  CircleAvatar(...),
                  Column(...),
                  const Spacer(),
                  Container(...),  // 收藏按钮
                ],
              ),
              // 收件人（如果有）
              if (recipients != '')
                Text('To $recipients', ...),
              // 邮件正文
              Text(body, ...),
              // 邮件图片（如果有）
              if (bodyImage != '')
                Image.asset(bodyImage),
              // 操作按钮
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: <Widget>[
                  OutlinedButton(...),  // Reply
                  OutlinedButton(...),  // Reply all
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## 模态对话框处理

### 小屏幕模态

在小屏幕上，点击邮件项会打开模态对话框：

```dart
class _ExtractRouteArguments extends StatelessWidget {
  const _ExtractRouteArguments();

  static const String routeName = '/detailView';

  @override
  Widget build(BuildContext context) {
    final _ScreenArguments args =
        ModalRoute.of(context)!.settings.arguments! as _ScreenArguments;

    return _RouteDetailView(
      item: args.item,
      selectCard: args.selectCard,
    );
  }
}

class _RouteDetailView extends StatelessWidget {
  const _RouteDetailView({
    required this.item,
    required this.selectCard,
  });

  final _Item item;
  final _CardSelectedCallback selectCard;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: <Widget>[
          // 返回按钮
          Align(
            alignment: Alignment.topLeft,
            child: TextButton(
              onPressed: () {
                Navigator.popUntil(
                  context,
                  (Route<dynamic> route) => route.settings.name == '/',
                );
                selectCard(null);
              },
              child: const Icon(Icons.arrow_back),
            ),
          ),
          // 详情内容
          Expanded(child: _DetailTile(item: item)),
        ],
      ),
    );
  }
}
```

**关键设计**：

1. **路由参数**：使用 `ModalRoute` 传递参数
2. **返回处理**：关闭模态并清除选中状态
3. **导航栈管理**：使用 `popUntil` 返回到根路由

## 状态管理策略

### 选中状态管理

使用 `int?` 管理选中的邮件项：

```dart
int? selected;

void selectCard(int? index) {
  setState(() {
    selected = index;
  });
}
```

**设计考虑**：

- **可空类型**：`null` 表示未选中
- **状态同步**：列表和详情视图共享状态

### 导航状态管理

使用 `_navigationIndex` 管理当前导航项：

```dart
int _navigationIndex = 0;

onDestinationSelected: (int index) {
  setState(() {
    _navigationIndex = index;
  });
}
```

### 方向管理

支持 LTR/RTL 切换：

```dart
TextDirection directionalityOverride = TextDirection.ltr;

// 在 build 中使用
return Directionality(
  textDirection: directionalityOverride,
  child: Scaffold(...),
);
```

## 设计模式总结

### 1. 条件渲染模式

根据状态和断点条件渲染不同内容：

```dart
builder: (_) => (_navigationIndex == 0)
    ? MailListView()
    : ExamplePage(),
```

### 2. 响应式行为模式

根据屏幕尺寸采用不同的交互方式：

```dart
if (!Breakpoints.mediumAndUp.isActive(context)) {
  // 小屏幕：模态
  Navigator.push(...);
} else {
  // 大屏幕：侧边栏
  selectCard(index);
}
```

### 3. 状态共享模式

多个组件共享状态：

```dart
_ItemList(
  selected: selected,
  selectCard: selectCard,
)
```

### 4. 动画协调模式

多个动画控制器协调工作：

```dart
_inboxIconSlideController.forward();
_articleIconSlideController.forward();
// ... 依次触发
```

## 性能优化技巧

### 1. 使用 ListView.builder

使用 `ListView.builder` 而不是 `ListView`：

```dart
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => _ItemListTile(...),
)
```

### 2. 延迟构建

使用 `builder` 延迟构建 Widget：

```dart
SlotLayout.from(
  key: const Key('Body'),
  builder: (_) => ExpensiveWidget(),  // 只在需要时构建
)
```

### 3. 条件渲染

避免构建不需要的 Widget：

```dart
secondaryBody: _navigationIndex == 0
    ? SlotLayout(...)
    : null,  // 不构建
```

## 总结

本章我们深入分析了邮件应用的实现：

- **应用架构**：整体结构和组件关系
- **核心组件**：PrimaryNavigation、Body、SecondaryBody 的配置
- **邮件列表**：列表和详情视图的实现
- **模态处理**：小屏幕模态对话框
- **状态管理**：选中状态、导航状态、方向管理
- **设计模式**：条件渲染、响应式行为、状态共享
- **性能优化**：ListView.builder、延迟构建、条件渲染

这个示例展示了如何在实际项目中应用 `AdaptiveLayout`，实现复杂的响应式布局。

## 练习

1. 添加新的邮件文件夹功能
2. 实现邮件搜索功能
3. 添加邮件标记为已读/未读功能

## 检查清单

- [ ] 理解邮件应用的架构设计
- [ ] 掌握核心组件的实现
- [ ] 了解响应式行为模式
- [ ] 理解状态管理策略
- [ ] 了解设计模式和性能优化技巧
