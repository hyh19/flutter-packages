# 第 11 章：恢复购买

## 本章目标

- 理解恢复购买的工作原理
- 实现恢复购买功能
- 处理恢复购买的结果
- 支持跨设备订阅同步

## 恢复购买概述

### 什么是恢复购买？

恢复购买允许用户在以下情况下重新获得已购买的订阅：

- 🔄 更换新设备
- 🔄 重新安装应用
- 🔄 清除应用数据
- 🔄 使用同一 Apple ID / Google 账号登录

### 为什么需要恢复购买？

1. **Apple 和 Google 要求**
   - App Review Guidelines 要求提供恢复购买功能
   - 用户换设备时必须能恢复订阅

2. **用户期望**
   - 用户认为订阅应该在所有设备上可用
   - 重装应用不应该丢失订阅

3. **法律合规**
   - 许多地区的消费者保护法要求

### 恢复购买 vs 新购买

| 特性 | 新购买 | 恢复购买 |
|------|--------|---------|
| 扣费 | 是 | 否 |
| 用户操作 | 主动购买 | 点击"恢复购买" |
| purchaseStream 状态 | purchased | restored |
| 需要验证 | 是 | 是 |
| 需要 completePurchase | 是 | 是 |

## 步骤 1：调用恢复购买 API

### 基本实现

```dart
import 'package:in_app_purchase/in_app_purchase.dart';

class SubscriptionService {
  final InAppPurchase _inAppPurchase = InAppPurchase.instance;

  // ...

  /// 恢复购买
  Future<bool> restorePurchases() async {
    if (!_isAvailable) {
      print('商店不可用，无法恢复购买');
      return false;
    }

    print('开始恢复购买...');

    try {
      // 调用恢复购买 API
      await _inAppPurchase.restorePurchases();
      print('恢复购买请求已发送');
      return true;
    } catch (e, stackTrace) {
      print('恢复购买时出错: $e');
      print('堆栈跟踪: $stackTrace');
      return false;
    }
  }
}
```

### 工作原理

调用 `restorePurchases()` 后：

1. 插件会向商店查询用户的购买历史
2. 找到的购买会通过 `purchaseStream` 发送
3. 状态为 `PurchaseStatus.restored`
4. 需要像处理新购买一样验证和完成

## 步骤 2：处理恢复的购买

### 在购买流监听中处理

我们在第 6 章已经实现了基础的处理：

```dart
class SubscriptionService {
  // ...

  Future<void> _handlePurchaseUpdate(PurchaseDetails purchaseDetails) async {
    switch (purchaseDetails.status) {
      // ... 其他状态 ...

      case PurchaseStatus.restored:
        await _handleRestoredPurchase(purchaseDetails);
        break;
    }

    // 完成购买
    if (purchaseDetails.pendingCompletePurchase) {
      await _completePurchase(purchaseDetails);
    }
  }

  /// 处理恢复的购买
  Future<void> _handleRestoredPurchase(PurchaseDetails purchaseDetails) async {
    print('恢复购买: ${purchaseDetails.productID}');

    // 1. 验证购买
    final bool valid = await _verifyPurchase(purchaseDetails);

    if (!valid) {
      print('恢复的购买验证失败: ${purchaseDetails.productID}');
      _showErrorToUser('恢复失败，请联系客服');
      return;
    }

    // 2. 解锁订阅内容
    await _unlockSubscription(purchaseDetails);

    // 3. 通知用户
    _showSuccessToUser('订阅已恢复！');

    print('订阅已恢复: ${purchaseDetails.productID}');
  }
}
```

## 步骤 3：创建恢复购买按钮

### 简单按钮

```dart
import 'package:flutter/material.dart';
import '../../../../services/subscription_service.dart';

class RestorePurchasesButton extends StatefulWidget {
  const RestorePurchasesButton({super.key});

  @override
  State<RestorePurchasesButton> createState() =>
      _RestorePurchasesButtonState();
}

class _RestorePurchasesButtonState extends State<RestorePurchasesButton> {
  final SubscriptionService _service = SubscriptionService();
  bool _isRestoring = false;

  Future<void> _handleRestore() async {
    setState(() => _isRestoring = true);

    try {
      final success = await _service.restorePurchases();
      
      if (!success) {
        _showMessage('恢复失败，请重试', isError: true);
      }
      // 成功消息在 _handleRestoredPurchase 中显示
    } finally {
      if (mounted) {
        setState(() => _isRestoring = false);
      }
    }
  }

  void _showMessage(String message, {bool isError = false}) {
    if (!mounted) return;
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(message),
        backgroundColor: isError ? Colors.red : Colors.green,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return TextButton.icon(
      onPressed: _isRestoring ? null : _handleRestore,
      icon: _isRestoring
          ? const SizedBox(
              width: 16,
              height: 16,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : const Icon(Icons.restore),
      label: const Text('恢复购买'),
    );
  }
}
```

### 带确认对话框的按钮

```dart
class RestorePurchasesButtonWithDialog extends StatefulWidget {
  const RestorePurchasesButtonWithDialog({super.key});

  @override
  State<RestorePurchasesButtonWithDialog> createState() =>
      _RestorePurchasesButtonWithDialogState();
}

class _RestorePurchasesButtonWithDialogState
    extends State<RestorePurchasesButtonWithDialog> {
  final SubscriptionService _service = SubscriptionService();
  bool _isRestoring = false;
  int _restoredCount = 0;

  Future<void> _handleRestore() async {
    // 显示确认对话框
    final confirmed = await _showConfirmDialog();
    if (confirmed != true) return;

    // 显示加载对话框
    _showLoadingDialog();

    _restoredCount = 0;

    try {
      // 监听恢复的购买
      final subscription = _service.purchaseStream.listen((purchases) {
        for (final purchase in purchases) {
          if (purchase.status == PurchaseStatus.restored) {
            _restoredCount++;
          }
        }
      });

      // 发起恢复
      final success = await _service.restorePurchases();

      // 等待一小段时间让购买流更新
      await Future.delayed(const Duration(seconds: 2));

      subscription.cancel();

      // 关闭加载对话框
      if (mounted) Navigator.of(context).pop();

      // 显示结果
      if (success) {
        _showResultDialog(_restoredCount);
      } else {
        _showErrorDialog();
      }
    } catch (e) {
      if (mounted) Navigator.of(context).pop();
      _showErrorDialog();
    }
  }

  Future<bool?> _showConfirmDialog() {
    return showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('恢复购买'),
        content: const Text(
          '将从您的 Apple ID / Google 账号恢复之前的购买。\n\n'
          '如果您之前没有购买过订阅，将不会恢复任何内容。',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(false),
            child: const Text('取消'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.of(context).pop(true),
            child: const Text('恢复'),
          ),
        ],
      ),
    );
  }

  void _showLoadingDialog() {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (context) => const AlertDialog(
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            CircularProgressIndicator(),
            SizedBox(height: 16),
            Text('正在恢复购买...'),
          ],
        ),
      ),
    );
  }

  void _showResultDialog(int count) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('恢复完成'),
        content: Text(
          count > 0
              ? '成功恢复 $count 个订阅'
              : '没有找到可恢复的购买\n\n'
                '请确保使用购买时的同一账号登录。',
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: const Text('确定'),
          ),
        ],
      ),
    );
  }

  void _showErrorDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('恢复失败'),
        content: const Text('恢复购买时出错，请检查网络连接后重试。'),
        actions: [
          TextButton(
            onPressed: () => Navigator.of(context).pop(),
            child: const Text('确定'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return TextButton.icon(
      onPressed: _handleRestore,
      icon: const Icon(Icons.restore),
      label: const Text('恢复购买'),
    );
  }
}
```

## 步骤 4：自动恢复购买

在某些情况下，可能需要在应用启动时自动尝试恢复购买。

### 静默恢复

```dart
class SubscriptionService {
  // ...

  /// 静默恢复购买（应用启动时调用）
  Future<void> silentRestorePurchases() async {
    if (!_isAvailable) return;

    print('静默恢复购买...');

    try {
      await _inAppPurchase.restorePurchases();
      // 不显示任何 UI，只在后台处理
    } catch (e) {
      print('静默恢复失败: $e');
      // 失败也不要打扰用户
    }
  }

  /// 初始化并恢复购买
  Future<bool> initializeWithRestore() async {
    // 先初始化
    final success = await initialize();
    
    if (success) {
      // 静默恢复购买
      await silentRestorePurchases();
    }
    
    return success;
  }
}
```

### 在应用启动时使用

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final subscriptionService = SubscriptionService();
  
  // 初始化并自动恢复
  await subscriptionService.initializeWithRestore();

  runApp(const MyApp());
}
```

## 步骤 5：处理特殊情况

### 没有可恢复的购买

```dart
class SubscriptionService {
  int _restoreAttemptCount = 0;

  Future<void> restorePurchases() async {
    _restoreAttemptCount = 0;

    // 监听购买流
    final completer = Completer<void>();
    late StreamSubscription subscription;

    subscription = _inAppPurchase.purchaseStream.listen(
      (purchases) {
        for (final purchase in purchases) {
          if (purchase.status == PurchaseStatus.restored) {
            _restoreAttemptCount++;
          }
        }
      },
      onDone: () {
        subscription.cancel();
        completer.complete();
      },
    );

    try {
      await _inAppPurchase.restorePurchases();
      
      // 等待一段时间让购买流更新
      await Future.delayed(const Duration(seconds: 3));
      
      if (_restoreAttemptCount == 0) {
        print('没有找到可恢复的购买');
        _showNoRestorableMessage();
      } else {
        print('恢复了 $_restoreAttemptCount 个购买');
      }
    } finally {
      subscription.cancel();
    }
  }

  void _showNoRestorableMessage() {
    // 显示给用户
    print('没有找到可恢复的购买');
  }
}
```

### 恢复消耗型产品

⚠️ **重要**：消耗型产品不能恢复！

```dart
Future<void> _handleRestoredPurchase(PurchaseDetails purchaseDetails) async {
  // 检查产品类型
  final product = getProductById(purchaseDetails.productID);
  
  if (product == null) {
    print('未找到产品: ${purchaseDetails.productID}');
    return;
  }

  // 订阅和非消耗型产品可以恢复
  // 消耗型产品需要从服务器恢复
  if (_isConsumable(product)) {
    print('警告：消耗型产品无法通过 restorePurchases 恢复');
    print('请从服务器恢复消耗型产品');
    return;
  }

  // 正常处理订阅恢复
  final valid = await _verifyPurchase(purchaseDetails);
  if (valid) {
    await _unlockSubscription(purchaseDetails);
  }

  if (purchaseDetails.pendingCompletePurchase) {
    await _completePurchase(purchaseDetails);
  }
}

bool _isConsumable(ProductDetails product) {
  // 根据产品 ID 或其他方式判断
  return product.id.contains('consumable');
}
```

## 步骤 6：跨设备同步

### 服务器端同步

对于需要跨设备同步的场景，应该使用服务器：

```dart
class SubscriptionService {
  final ApiService _apiService = ApiService();

  /// 从服务器同步订阅状态
  Future<void> syncSubscriptionFromServer(String userId) async {
    print('从服务器同步订阅状态...');

    try {
      // 调用服务器 API
      final response = await _apiService.getSubscriptionStatus(userId);

      if (response.hasActiveSubscription) {
        // 激活订阅
        await _subscriptionManager.activateSubscription(
          response.productId,
        );
        
        print('订阅已从服务器恢复: ${response.productId}');
      } else {
        print('服务器上没有活跃的订阅');
      }
    } catch (e) {
      print('从服务器同步失败: $e');
    }
  }

  /// 混合恢复策略：本地 + 服务器
  Future<void> restoreWithServerSync(String userId) async {
    // 1. 从本地商店恢复
    await restorePurchases();

    // 2. 从服务器同步
    await syncSubscriptionFromServer(userId);
  }
}
```

## 放置恢复购买按钮的位置

### 推荐位置

1. **订阅页面**：最常见的位置

```dart
class SubscriptionScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('选择订阅'),
        actions: [
          const RestorePurchasesButton(),
        ],
      ),
      body: _buildProductList(),
    );
  }
}
```

2. **设置页面**：

```dart
class SettingsScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ListView(
      children: [
        ListTile(
          leading: const Icon(Icons.restore),
          title: const Text('恢复购买'),
          subtitle: const Text('恢复之前购买的订阅'),
          onTap: () {
            // 恢复购买逻辑
          },
        ),
      ],
    );
  }
}
```

3. **登录后自动恢复**：

```dart
Future<void> onUserLogin(String userId) async {
  // 用户登录后自动恢复
  await _subscriptionService.restoreWithServerSync(userId);
}
```

## 测试恢复购买

### 测试场景

1. **正常恢复**
   - 购买订阅
   - 卸载应用
   - 重新安装
   - 点击"恢复购买"
   - 验证订阅已恢复

2. **没有购买历史**
   - 使用新账号
   - 点击"恢复购买"
   - 验证显示"没有可恢复的购买"

3. **多设备同步**
   - 设备 A 购买订阅
   - 设备 B 登录同一账号
   - 设备 B 恢复购买
   - 验证订阅已同步

### iOS 测试注意事项

- 使用沙盒测试账号
- 确保在"设置" → "App Store" → "沙盒账号"中登录
- 不要在 iTunes 中登录沙盒账号

### Android 测试注意事项

- 使用许可测试员账号
- 确保设备登录了测试 Google 账号
- 使用内部测试轨道的版本

## 常见问题

### Q1：恢复购买会扣费吗？

**不会**。恢复购买只是重新获取已有的购买记录，不会产生新的费用。

### Q2：消耗型产品可以恢复吗？

**不可以**。消耗型产品使用后就消失了，无法通过 `restorePurchases()` 恢复。如需跨设备同步，必须通过服务器实现。

### Q3：恢复购买需要多长时间？

通常 1-5 秒，取决于：

- 网络速度
- 购买历史数量
- 商店服务器响应速度

### Q4：用户换了 Apple ID / Google 账号怎么办？

**无法恢复**。购买与特定账号绑定，换账号后无法恢复。应该：

1. 提示用户使用原账号
2. 或通过服务器关联用户自己的账号系统

### Q5：需要在每次应用启动时恢复吗？

**不一定**。可以：

- 首次安装时自动恢复
- 用户主动点击"恢复购买"按钮
- 检测到没有订阅时提示恢复

频繁的自动恢复会影响性能和用户体验。

## 小结

在本章中，我们学习了：

- 恢复购买的工作原理和重要性
- 如何实现恢复购买功能
- 如何处理恢复的购买记录
- 如何实现跨设备订阅同步
- 如何放置和设计恢复购买按钮

完成恢复购买功能后，下一步是实现订阅的升级和降级。

## 下一步

[第 12 章：订阅升级和降级](12-subscription-management.md) - 学习如何实现订阅计划的升级和降级功能。
