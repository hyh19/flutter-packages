# 第 5 章：初始化 IAP 插件

## 本章目标

- 了解 IAP 插件的初始化流程
- 检查商店可用性
- 实现初始化代码
- 处理初始化错误

## 为什么需要初始化

在使用 `in_app_purchase` 插件的任何功能前，需要先检查：

1. 当前设备是否支持应用内购买
2. 与应用商店的连接是否正常
3. 用户是否被限制使用应用内购买功能

## 初始化时机

推荐在应用启动时尽早初始化 IAP 插件：

- ✅ 在 `main()` 函数中
- ✅ 在应用的根 Widget 的 `initState` 中
- ✅ 在应用启动屏显示时
- ❌ 在用户点击购买按钮时（太晚了）

## 步骤 1：检查商店可用性

### 基本检查

最简单的检查方式：

```dart
import 'package:in_app_purchase/in_app_purchase.dart';

Future<bool> checkStoreAvailability() async {
  final bool available = await InAppPurchase.instance.isAvailable();
  return available;
}
```

### 完整的可用性检查

```dart
import 'dart:io';
import 'package:in_app_purchase/in_app_purchase.dart';

class IAPAvailability {
  // 商店是否可用
  static Future<bool> isStoreAvailable() async {
    // 检查平台是否支持
    if (!_isSupportedPlatform()) {
      print('当前平台不支持应用内购买');
      return false;
    }

    try {
      // 检查商店连接
      final bool available = await InAppPurchase.instance.isAvailable();
      
      if (!available) {
        print('应用内购买服务不可用');
        return false;
      }

      print('应用内购买服务可用');
      return true;
    } catch (e) {
      print('检查商店可用性时出错: $e');
      return false;
    }
  }

  // 检查平台是否支持
  static bool _isSupportedPlatform() {
    return Platform.isIOS || Platform.isAndroid;
  }

  // 获取不可用的原因
  static Future<String?> getUnavailableReason() async {
    if (!_isSupportedPlatform()) {
      return '当前平台（${Platform.operatingSystem}）不支持应用内购买';
    }

    final bool available = await InAppPurchase.instance.isAvailable();
    if (!available) {
      if (Platform.isIOS) {
        return '无法连接到 App Store。请检查网络连接，或者是否在"屏幕使用时间"中禁用了应用内购买。';
      } else {
        return '无法连接到 Google Play。请检查设备是否安装了 Google Play 服务。';
      }
    }

    return null;
  }
}
```

### 商店不可用的常见原因

#### iOS

1. **屏幕使用时间限制**
   - 用户在"屏幕使用时间"中禁用了应用内购买
   - 需要在设置中启用

2. **网络问题**
   - 无法连接到 App Store 服务器
   - 需要稳定的网络连接

3. **沙盒环境问题**
   - 测试账号未正确登录
   - 在设置中而非应用内登录了沙盒账号

#### Android

1. **缺少 Google Play 服务**
   - 设备未安装或禁用了 Google Play 服务
   - 部分国产 ROM 不包含 Google Play

2. **Play Protect 禁用**
   - Google Play Protect 被禁用
   - 需要在 Play 商店中启用

3. **应用未签名或签名不匹配**
   - 使用未签名的 APK
   - 签名与 Play Console 中注册的不一致

## 步骤 2：实现初始化服务

扩展我们在第 4 章创建的 `SubscriptionService` 类。

### 完整的服务类实现

编辑 `lib/services/subscription_service.dart`：

```dart
import 'dart:async';
import 'dart:io';
import 'package:in_app_purchase/in_app_purchase.dart';

class SubscriptionService {
  // ==================== 单例模式 ====================
  static final SubscriptionService _instance = SubscriptionService._internal();
  factory SubscriptionService() => _instance;
  SubscriptionService._internal();

  // ==================== 私有属性 ====================
  final InAppPurchase _inAppPurchase = InAppPurchase.instance;
  StreamSubscription<List<PurchaseDetails>>? _subscription;
  bool _isInitialized = false;
  bool _isAvailable = false;

  // ==================== 公共属性 ====================
  bool get isInitialized => _isInitialized;
  bool get isAvailable => _isAvailable;
  bool get isSupportedPlatform => Platform.isIOS || Platform.isAndroid;

  // ==================== 初始化方法 ====================
  
  /// 初始化 IAP 服务
  /// 
  /// 返回值：
  /// - true: 初始化成功，商店可用
  /// - false: 初始化失败或商店不可用
  Future<bool> initialize() async {
    if (_isInitialized) {
      print('IAP 服务已经初始化');
      return _isAvailable;
    }

    print('开始初始化 IAP 服务...');

    // 检查平台支持
    if (!isSupportedPlatform) {
      print('当前平台不支持应用内购买');
      _isInitialized = true;
      _isAvailable = false;
      return false;
    }

    try {
      // 检查商店可用性
      _isAvailable = await _inAppPurchase.isAvailable();

      if (!_isAvailable) {
        print('应用内购买服务不可用');
        _logUnavailableReason();
      } else {
        print('应用内购买服务可用');
      }

      _isInitialized = true;
      return _isAvailable;
    } catch (e, stackTrace) {
      print('初始化 IAP 服务时出错: $e');
      print('堆栈跟踪: $stackTrace');
      _isInitialized = true;
      _isAvailable = false;
      return false;
    }
  }

  // ==================== 辅助方法 ====================
  
  /// 记录商店不可用的可能原因
  void _logUnavailableReason() {
    if (Platform.isIOS) {
      print('''
      iOS 应用内购买不可用的可能原因：
      1. 网络连接问题
      2. 屏幕使用时间中禁用了应用内购买
      3. 沙盒测试账号未正确登录
      4. 设备所在地区不支持
      ''');
    } else if (Platform.isAndroid) {
      print('''
      Android 应用内购买不可用的可能原因：
      1. 设备未安装 Google Play 服务
      2. Google Play 服务被禁用
      3. 应用未正确签名
      4. 应用未上传到 Play Console
      ''');
    }
  }

  /// 获取用户友好的错误消息
  String getUnavailableMessage() {
    if (!isSupportedPlatform) {
      return '当前设备不支持应用内购买功能';
    }

    if (!_isAvailable) {
      if (Platform.isIOS) {
        return '无法连接到 App Store，请检查网络连接和设备设置';
      } else {
        return '无法连接到 Google Play，请确保设备已安装 Google Play 服务';
      }
    }

    return '应用内购买服务暂时不可用，请稍后再试';
  }

  // ==================== 资源清理 ====================
  
  /// 释放资源
  void dispose() {
    _subscription?.cancel();
    _subscription = null;
  }
}
```

## 步骤 3：在应用启动时初始化

### 方式 1：在 main() 函数中初始化

编辑 `lib/main.dart`：

```dart
import 'package:flutter/material.dart';
import '../../../../tutorial/services/subscription_service.dart';

Future<void> main() async {
  // 确保 Flutter 绑定已初始化
  WidgetsFlutterBinding.ensureInitialized();

  // 初始化订阅服务
  final subscriptionService = SubscriptionService();
  await subscriptionService.initialize();

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'IAP Demo',
      home: const HomePage(),
    );
  }
}
```

### 方式 2：在启动屏中初始化

如果应用有启动屏，可以在启动屏显示时初始化：

```dart
import 'package:flutter/material.dart';
import '../../../../tutorial/services/subscription_service.dart';

class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    _initialize();
  }

  Future<void> _initialize() async {
    // 初始化订阅服务
    final subscriptionService = SubscriptionService();
    final success = await subscriptionService.initialize();

    if (!success) {
      print('警告：应用内购买服务不可用');
      // 可以选择显示警告信息给用户
    }

    // 等待至少 2 秒（可选，用于显示启动画面）
    await Future.delayed(const Duration(seconds: 2));

    // 导航到主页
    if (mounted) {
      Navigator.of(context).pushReplacement(
        MaterialPageRoute(builder: (context) => const HomePage()),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 应用 Logo
            const FlutterLogo(size: 100),
            const SizedBox(height: 24),
            // 加载指示器
            const CircularProgressIndicator(),
            const SizedBox(height: 16),
            const Text('正在初始化...'),
          ],
        ),
      ),
    );
  }
}
```

### 方式 3：在根 Widget 中初始化

```dart
class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final SubscriptionService _subscriptionService = SubscriptionService();
  bool _isInitializing = true;

  @override
  void initState() {
    super.initState();
    _initializeIAP();
  }

  Future<void> _initializeIAP() async {
    await _subscriptionService.initialize();
    
    if (mounted) {
      setState(() {
        _isInitializing = false;
      });
    }
  }

  @override
  void dispose() {
    _subscriptionService.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (_isInitializing) {
      // 显示加载界面
      return const MaterialApp(
        home: Scaffold(
          body: Center(
            child: CircularProgressIndicator(),
          ),
        ),
      );
    }

    return MaterialApp(
      title: 'IAP Demo',
      home: const HomePage(),
    );
  }
}
```

## 步骤 4：处理初始化失败

### 显示错误信息给用户

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final subscriptionService = SubscriptionService();

    return Scaffold(
      appBar: AppBar(
        title: const Text('订阅服务'),
      ),
      body: Center(
        child: subscriptionService.isAvailable
            ? const Text('订阅服务正常')
            : _buildUnavailableWidget(subscriptionService),
      ),
    );
  }

  Widget _buildUnavailableWidget(SubscriptionService service) {
    return Padding(
      padding: const EdgeInsets.all(24.0),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Icon(
            Icons.error_outline,
            size: 64,
            color: Colors.red,
          ),
          const SizedBox(height: 16),
          const Text(
            '订阅服务不可用',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 8),
          Text(
            service.getUnavailableMessage(),
            textAlign: TextAlign.center,
            style: TextStyle(
              color: Colors.grey[600],
            ),
          ),
        ],
      ),
    );
  }
}
```

### 提供重试机制

```dart
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final SubscriptionService _subscriptionService = SubscriptionService();
  bool _isRetrying = false;

  Future<void> _retryInitialization() async {
    setState(() {
      _isRetrying = true;
    });

    await _subscriptionService.initialize();

    setState(() {
      _isRetrying = false;
    });

    if (_subscriptionService.isAvailable) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('订阅服务已连接')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('订阅服务'),
      ),
      body: Center(
        child: _subscriptionService.isAvailable
            ? const Text('订阅服务正常')
            : Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.error_outline, size: 64, color: Colors.red),
                  const SizedBox(height: 16),
                  const Text('订阅服务不可用'),
                  const SizedBox(height: 16),
                  ElevatedButton.icon(
                    onPressed: _isRetrying ? null : _retryInitialization,
                    icon: _isRetrying
                        ? const SizedBox(
                            width: 16,
                            height: 16,
                            child: CircularProgressIndicator(strokeWidth: 2),
                          )
                        : const Icon(Icons.refresh),
                    label: const Text('重试'),
                  ),
                ],
              ),
      ),
    );
  }
}
```

## 步骤 5：添加日志和监控

### 添加详细日志

```dart
class IAPLogger {
  static const String _tag = '[IAP]';

  static void info(String message) {
    print('$_tag INFO: $message');
  }

  static void warning(String message) {
    print('$_tag WARNING: $message');
  }

  static void error(String message, [dynamic error, StackTrace? stackTrace]) {
    print('$_tag ERROR: $message');
    if (error != null) {
      print('$_tag Error details: $error');
    }
    if (stackTrace != null) {
      print('$_tag Stack trace: $stackTrace');
    }
  }

  static void debug(String message) {
    print('$_tag DEBUG: $message');
  }
}
```

在服务类中使用：

```dart
Future<bool> initialize() async {
  IAPLogger.info('开始初始化 IAP 服务');
  
  if (!isSupportedPlatform) {
    IAPLogger.warning('当前平台不支持应用内购买');
    return false;
  }

  try {
    _isAvailable = await _inAppPurchase.isAvailable();
    
    if (_isAvailable) {
      IAPLogger.info('IAP 服务初始化成功');
    } else {
      IAPLogger.warning('IAP 服务不可用');
    }
    
    return _isAvailable;
  } catch (e, stackTrace) {
    IAPLogger.error('初始化失败', e, stackTrace);
    return false;
  }
}
```

## 测试初始化

### 测试场景

1. **正常初始化**
   - 在支持的设备上运行应用
   - 验证初始化成功

2. **商店不可用**
   - 关闭网络连接（iOS）
   - 禁用 Google Play 服务（Android）
   - 验证错误处理

3. **不支持的平台**
   - 在 Web 或 Desktop 上运行（如果配置了）
   - 验证平台检查

### 测试代码示例

```dart
void testInitialization() async {
  final service = SubscriptionService();
  
  print('测试 1: 检查平台支持');
  print('平台支持: ${service.isSupportedPlatform}');
  
  print('\n测试 2: 初始化服务');
  final success = await service.initialize();
  print('初始化结果: ${success ? "成功" : "失败"}');
  
  print('\n测试 3: 检查可用性');
  print('服务可用: ${service.isAvailable}');
  
  if (!service.isAvailable) {
    print('不可用原因: ${service.getUnavailableMessage()}');
  }
}
```

## 常见问题

### Q1：何时调用 initialize()？

**建议在应用启动时尽早调用**，最好在显示主界面前完成。这样可以：

- 尽早发现问题
- 确保购买流准备就绪
- 避免用户等待

### Q2：initialize() 可以多次调用吗？

**可以，但不推荐**。服务类内部有状态检查，多次调用会直接返回上次的结果。如需重新初始化，应先 dispose()。

### Q3：初始化失败后该怎么办？

**根据业务需求决定**：

- 关键功能：阻止用户继续，显示错误信息和重试按钮
- 非关键功能：允许用户继续使用应用，隐藏订阅相关功能

### Q4：如何在开发环境禁用 IAP？

可以添加环境标志：

```dart
const bool kIAPEnabled = bool.fromEnvironment('IAP_ENABLED', defaultValue: true);

Future<bool> initialize() async {
  if (!kIAPEnabled) {
    print('IAP 在开发环境中禁用');
    return false;
  }
  // 正常初始化流程
}
```

运行时：

```bash
flutter run --dart-define=IAP_ENABLED=false
```

## 小结

在本章中，我们学习了：

- IAP 插件初始化的重要性和时机
- 如何检查商店可用性
- 如何实现完整的初始化服务
- 如何处理初始化错误
- 如何添加日志和监控

完成初始化后，下一步是监听购买更新流。

## 下一步

[第 6 章：监听购买更新流](06-purchase-stream.md) - 学习如何监听和处理购买状态更新。
