# 第 14 章：最佳实践和常见问题

## 本章目标

- 总结订阅开发的最佳实践
- 掌握错误处理和调试技巧
- 了解性能优化建议
- 解答常见问题

## 安全性最佳实践

### 1. 永远在服务器端验证购买

❌ **错误做法**

```dart
// 不安全：仅客户端验证
Future<void> handlePurchase(PurchaseDetails purchase) async {
  // 直接解锁，没有验证
  await unlockContent();
}
```

✅ **正确做法**

```dart
// 安全：服务器端验证
Future<void> handlePurchase(PurchaseDetails purchase) async {
  // 1. 发送到服务器验证
  final result = await apiService.verifyPurchase(purchase);
  
  // 2. 只在验证通过后解锁
  if (result.isValid) {
    await unlockContent();
  }
  
  // 3. 完成购买
  await completePurchase(purchase);
}
```

### 2. 保护敏感信息

❌ **错误做法**

```dart
// 不要在代码中硬编码 API 密钥
const String apiKey = 'sk_live_abc123...';
```

✅ **正确做法**

```dart
// 使用环境变量
const String apiKey = String.fromEnvironment('API_KEY');

// 或从安全存储获取
final apiKey = await secureStorage.read(key: 'api_key');
```

### 3. 防止中间人攻击

```dart
class ApiService {
  final http.Client _client;

  ApiService() : _client = http.Client() {
    // 使用证书 pinning（可选，高安全需求）
  }

  Future<Response> verifyPurchase(PurchaseDetails purchase) async {
    // 使用 HTTPS
    final response = await _client.post(
      Uri.parse('https://your-api.com/verify'),  // 注意 https
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
      body: jsonEncode(purchase.toJson()),
    );
    
    return response;
  }
}
```

## 用户体验最佳实践

### 1. 尽早初始化

✅ **在应用启动时初始化**

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 尽早初始化 IAP
  final iapService = SubscriptionService();
  await iapService.initialize();
  
  runApp(MyApp());
}
```

### 2. 提供清晰的价格信息

✅ **显示完整的价格信息**

```dart
Widget buildPriceInfo(SubscriptionProduct product) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      // 主价格
      Text(
        product.price,
        style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
      ),
      
      // 周期说明
      Text('每${product.periodDescription}自动续订'),
      
      // 试用期
      if (product.hasFreeTrial)
        Text(
          product.freeTrialDescription!,
          style: TextStyle(color: Colors.green),
        ),
      
      // 取消说明
      Text(
        '随时可在系统设置中取消',
        style: TextStyle(fontSize: 12, color: Colors.grey),
      ),
    ],
  );
}
```

### 3. 提供加载反馈

✅ **显示加载状态**

```dart
Widget buildSubscriptionButton({
  required bool isLoading,
  required VoidCallback onPressed,
}) {
  return ElevatedButton(
    onPressed: isLoading ? null : onPressed,
    child: isLoading
        ? Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              SizedBox(
                width: 16,
                height: 16,
                child: CircularProgressIndicator(strokeWidth: 2),
              ),
              SizedBox(width: 8),
              Text('处理中...'),
            ],
          )
        : Text('立即订阅'),
  );
}
```

### 4. 处理错误并提供重试

✅ **友好的错误处理**

```dart
Future<void> handlePurchase() async {
  try {
    await service.purchaseSubscription(product);
  } catch (e) {
    // 1. 记录错误
    logger.error('Purchase failed', error: e);
    
    // 2. 显示友好的错误消息
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('购买失败'),
        content: Text(_getUserFriendlyMessage(e)),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('取消'),
          ),
          ElevatedButton(
            onPressed: () {
              Navigator.pop(context);
              handlePurchase(); // 重试
            },
            child: Text('重试'),
          ),
        ],
      ),
    );
  }
}
```

## 代码组织最佳实践

### 1. 使用服务类封装逻辑

✅ **清晰的服务层**

```
lib/
├── services/
│   ├── subscription_service.dart     # 主服务类
│   ├── subscription_manager.dart     # 状态管理
│   └── api_service.dart              # API 调用
├── models/
│   ├── subscription_product.dart     # 产品模型
│   └── subscription_state.dart       # 状态模型
├── utils/
│   ├── subscription_constants.dart   # 常量定义
│   └── iap_error_handler.dart        # 错误处理
└── screens/
    └── subscription_screen.dart      # UI 界面
```

### 2. 使用依赖注入

✅ **便于测试和维护**

```dart
class SubscriptionService {
  final InAppPurchase _inAppPurchase;
  final ApiService _apiService;
  final SubscriptionManager _manager;

  SubscriptionService({
    InAppPurchase? inAppPurchase,
    ApiService? apiService,
    SubscriptionManager? manager,
  })  : _inAppPurchase = inAppPurchase ?? InAppPurchase.instance,
        _apiService = apiService ?? ApiService(),
        _manager = manager ?? SubscriptionManager();
}
```

### 3. 单一职责原则

✅ **每个类只做一件事**

```dart
// ✅ 好：职责单一
class PurchaseVerifier {
  Future<bool> verify(PurchaseDetails purchase) async {
    // 只负责验证
  }
}

class SubscriptionUnlocker {
  Future<void> unlock(String productId) async {
    // 只负责解锁
  }
}

// ❌ 差：职责混乱
class SubscriptionService {
  Future<void> doEverything() async {
    // 验证、解锁、UI 更新、网络请求...
  }
}
```

## 性能优化建议

### 1. 缓存产品信息

```dart
class SubscriptionService {
  List<ProductDetails>? _cachedProducts;
  DateTime? _cacheTime;
  static const _cacheDuration = Duration(hours: 1);

  Future<List<ProductDetails>> loadProducts({bool forceRefresh = false}) async {
    // 检查缓存
    if (!forceRefresh &&
        _cachedProducts != null &&
        _cacheTime != null &&
        DateTime.now().difference(_cacheTime!) < _cacheDuration) {
      print('使用缓存的产品列表');
      return _cachedProducts!;
    }

    // 重新加载
    final response = await _inAppPurchase.queryProductDetails(productIds);
    _cachedProducts = response.productDetails;
    _cacheTime = DateTime.now();

    return _cachedProducts!;
  }
}
```

### 2. 避免重复监听

```dart
class SubscriptionService {
  StreamSubscription? _subscription;

  void _listenToPurchaseUpdates() {
    // 先取消旧的订阅
    _subscription?.cancel();

    // 创建新的订阅
    _subscription = InAppPurchase.instance.purchaseStream.listen(
      _handlePurchaseUpdates,
    );
  }

  @override
  void dispose() {
    _subscription?.cancel();
  }
}
```

### 3. 延迟非关键操作

```dart
Future<void> handlePurchase(PurchaseDetails purchase) async {
  // 1. 关键操作：验证和解锁
  final valid = await verifyPurchase(purchase);
  if (valid) {
    await unlockContent(purchase);
  }
  await completePurchase(purchase);

  // 2. 非关键操作：延迟执行
  Future.delayed(Duration(seconds: 2), () {
    updateAnalytics(purchase);
    sendMarketingEmail(purchase);
  });
}
```

## 调试技巧

### 1. 详细的日志记录

```dart
class IAPLogger {
  static bool _enabled = true;

  static void enable() => _enabled = true;
  static void disable() => _enabled = false;

  static void log(String message, {String? tag}) {
    if (!_enabled) return;
    
    final timestamp = DateTime.now().toIso8601String();
    final tagStr = tag != null ? '[$tag]' : '';
    print('$timestamp [IAP]$tagStr $message');
  }

  static void logPurchase(PurchaseDetails purchase) {
    log('Purchase Details:', tag: 'DEBUG');
    log('  Product ID: ${purchase.productID}', tag: 'DEBUG');
    log('  Status: ${purchase.status}', tag: 'DEBUG');
    log('  Transaction Date: ${purchase.transactionDate}', tag: 'DEBUG');
    log('  Purchase ID: ${purchase.purchaseID}', tag: 'DEBUG');
  }
}
```

### 2. 模拟购买场景

```dart
class MockInAppPurchase implements InAppPurchase {
  @override
  Future<bool> isAvailable() async {
    await Future.delayed(Duration(seconds: 1));
    return true;
  }

  @override
  Future<ProductDetailsResponse> queryProductDetails(Set<String> ids) async {
    await Future.delayed(Duration(seconds: 1));
    
    // 返回模拟产品
    return ProductDetailsResponse(
      productDetails: _createMockProducts(ids),
      notFoundIDs: [],
    );
  }

  List<ProductDetails> _createMockProducts(Set<String> ids) {
    // 创建模拟产品
    return [];
  }
}
```

### 3. 使用调试面板

```dart
class IAPDebugPanel extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Drawer(
      child: ListView(
        children: [
          DrawerHeader(child: Text('IAP 调试')),
          ListTile(
            title: Text('清除缓存'),
            onTap: () => _clearCache(),
          ),
          ListTile(
            title: Text('重新初始化'),
            onTap: () => _reinitialize(),
          ),
          ListTile(
            title: Text('查看日志'),
            onTap: () => _showLogs(),
          ),
          ListTile(
            title: Text('模拟购买'),
            onTap: () => _simulatePurchase(),
          ),
        ],
      ),
    );
  }
}
```

## 常见问题解答

### Q1：为什么点击购买没反应？

**可能原因**：

1. 商店未初始化
2. 设备未登录商店账号
3. 产品未加载
4. 网络连接问题

**解决方法**：

```dart
Future<void> purchase(ProductDetails product) async {
  // 1. 检查初始化
  if (!service.isInitialized) {
    await service.initialize();
  }

  // 2. 检查商店可用性
  if (!service.isAvailable) {
    showError('应用内购买服务不可用');
    return;
  }

  // 3. 检查产品
  if (product == null) {
    showError('产品不存在');
    return;
  }

  // 4. 执行购买
  await service.purchaseSubscription(product);
}
```

### Q2：如何测试沙盒环境？

**iOS**：

1. 创建沙盒测试账号
2. 在设备的"设置" → "App Store" → "沙盒账号"中登录
3. 不要在 iTunes 中登录沙盒账号
4. 在应用中购买时会自动使用沙盒账号

**Android**：

1. 在 Play Console 添加许可测试员
2. 使用测试账号登录设备
3. 安装内部测试版本
4. 购买不会扣费

### Q3：购买成功但未解锁内容？

**检查清单**：

- [ ] purchaseStream 是否正在监听
- [ ] 购买状态是否为 `purchased`
- [ ] 是否调用了验证逻辑
- [ ] 是否调用了解锁逻辑
- [ ] 是否调用了 `completePurchase`

### Q4：如何处理订阅过期？

```dart
class SubscriptionService {
  /// 检查订阅是否过期
  Future<bool> isSubscriptionExpired() async {
    final expiryDateStr = await prefs.getString('subscription_expiry');
    
    if (expiryDateStr == null) return true;

    final expiryDate = DateTime.parse(expiryDateStr);
    return DateTime.now().isAfter(expiryDate);
  }

  /// 定期检查订阅状态
  void startExpiryCheck() {
    Timer.periodic(Duration(hours: 1), (timer) async {
      if (await isSubscriptionExpired()) {
        // 订阅已过期
        await handleExpiredSubscription();
      }
    });
  }

  Future<void> handleExpiredSubscription() async {
    // 1. 清除本地状态
    await clearSubscription();

    // 2. 尝试恢复购买
    await restorePurchases();

    // 3. 如果仍然过期，显示续订提示
    if (await isSubscriptionExpired()) {
      showRenewalPrompt();
    }
  }
}
```

### Q5：如何实现家庭共享？

**iOS 家庭共享**：

- iOS 14+ 支持订阅家庭共享
- 需要在 App Store Connect 中启用
- 代码无需特殊处理
- 购买验证时检查 `originalTransactionIdentifierIOS`

**Android**：

- Google Play 不支持自动家庭共享
- 需要自己实现账号系统
- 通过服务器管理家庭成员

### Q6：如何在开发环境禁用 IAP？

```dart
const bool kIAPEnabled = bool.fromEnvironment(
  'IAP_ENABLED',
  defaultValue: true,
);

class SubscriptionService {
  Future<bool> initialize() async {
    if (!kIAPEnabled) {
      print('IAP 在开发环境中禁用');
      _isInitialized = true;
      _isAvailable = false;
      return false;
    }

    // 正常初始化
    return await _actualInitialize();
  }
}

// 运行时禁用
// flutter run --dart-define=IAP_ENABLED=false
```

## 测试清单

### 功能测试

- [ ] 初始化成功
- [ ] 产品加载成功
- [ ] 购买流程完整
- [ ] 验证功能正常
- [ ] 恢复购买功能
- [ ] 升级/降级功能
- [ ] 错误处理正确

### 平台测试

- [ ] iOS 真机测试
- [ ] Android 真机测试
- [ ] 沙盒环境测试
- [ ] 生产环境测试

### 边缘情况

- [ ] 网络断开时的处理
- [ ] 应用在购买中被关闭
- [ ] 重复购买的防护
- [ ] 商店不可用的处理
- [ ] 产品加载失败的处理

## 发布前检查清单

### 代码检查

- [ ] 移除所有 debug 日志
- [ ] 移除测试代码
- [ ] 验证所有产品 ID 正确
- [ ] 检查服务器验证逻辑
- [ ] 确保调用了 `completePurchase`

### 商店配置

- [ ] iOS：产品已在 App Store Connect 激活
- [ ] Android：产品已在 Play Console 激活
- [ ] iOS：签署了付费应用协议
- [ ] Android：配置了应用签名
- [ ] 价格设置正确

### 隐私和合规

- [ ] 添加隐私政策链接
- [ ] 添加服务条款链接
- [ ] 说明订阅续订和取消方式
- [ ] 遵守 GDPR / CCPA 等法规

## 进阶主题

### 1. 离线支持

```dart
class OfflineSubscriptionManager {
  /// 离线缓存订阅状态
  Future<void> cacheSubscriptionStatus() async {
    final status = await fetchFromServer();
    await saveToLocal(status);
  }

  /// 离线时使用缓存
  Future<bool> hasActiveSubscription() async {
    if (await isOnline()) {
      return await checkWithServer();
    } else {
      return await checkLocalCache();
    }
  }
}
```

### 2. A/B 测试

```dart
class SubscriptionExperiment {
  /// 根据用户分组显示不同价格
  ProductDetails getProductForUser(String userId) {
    final group = _getUserGroup(userId);
    
    return switch (group) {
      'A' => products['monthly_premium_a']!,
      'B' => products['monthly_premium_b']!,
      _ => products['monthly_premium_default']!,
    };
  }
}
```

### 3. 推荐引擎

```dart
class SubscriptionRecommender {
  /// 根据用户行为推荐订阅
  SubscriptionProduct? recommendSubscription(UserProfile user) {
    if (user.isPowerUser) {
      return vipSubscription;
    } else if (user.usageFrequency > 10) {
      return premiumSubscription;
    } else {
      return basicSubscription;
    }
  }
}
```

## 有用的资源

### 官方文档

- [in_app_purchase 插件](https://pub.dev/packages/in_app_purchase)
- [Apple In-App Purchase](https://developer.apple.com/in-app-purchase/)
- [Google Play Billing](https://developer.android.com/google/play/billing)

### 社区资源

- [Flutter IAP Codelab](https://codelabs.developers.google.com/codelabs/flutter-in-app-purchases)
- [Stack Overflow - in-app-purchase](https://stackoverflow.com/questions/tagged/in-app-purchase)
- [Flutter GitHub Discussions](https://github.com/flutter/flutter/discussions)

### 工具

- [RevenueCat](https://www.revenuecat.com/) - 订阅管理平台
- [Adapty](https://adapty.io/) - 订阅分析
- [App Store Connect API](https://developer.apple.com/app-store-connect/api/)
- [Google Play Developer API](https://developers.google.com/android-publisher)

## 小结

在本章中，我们学习了：

- 订阅开发的安全性最佳实践
- 用户体验优化建议
- 代码组织和性能优化
- 调试技巧和常见问题解答
- 测试和发布检查清单

恭喜你完成了整个教程！现在你已经掌握了使用 `in_app_purchase` 插件开发订阅功能的完整知识。

## 结语

订阅功能的开发需要耐心和细心，涉及商店配置、代码实现、安全验证等多个方面。希望本教程能帮助你顺利实现订阅功能，为用户提供优质的服务。

如果在开发过程中遇到问题，记住：

1. 📖 查阅官方文档
2. 🔍 搜索类似问题
3. 🧪 在沙盒环境充分测试
4. 🛡️ 重视安全性
5. 💡 关注用户体验

祝你开发顺利！🎉
