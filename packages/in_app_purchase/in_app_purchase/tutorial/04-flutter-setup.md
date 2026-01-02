# 第 4 章：Flutter 项目配置

## 本章目标

- 在 Flutter 项目中添加 `in_app_purchase` 依赖
- 配置 iOS 和 Android 平台特定设置
- 验证配置是否正确

## 前提条件

- 已完成 iOS App Store Connect 配置（第 2 章）
- 已完成 Android Google Play Console 配置（第 3 章）
- 拥有 Flutter 3.29.0+ 项目

## 步骤 1：添加依赖

### 使用命令行添加

在 Flutter 项目根目录执行：

```bash
flutter pub add in_app_purchase
```

### 手动添加到 pubspec.yaml

或者在 `pubspec.yaml` 文件中手动添加：

```yaml
dependencies:
  flutter:
    sdk: flutter
  in_app_purchase: ^3.2.3
```

然后运行：

```bash
flutter pub get
```

### 验证依赖安装

检查 `pubspec.yaml` 确保依赖已添加，并查看终端输出确认安装成功。

## 步骤 2：iOS 平台配置

### 2.1 配置 Bundle ID

确保 Xcode 项目中的 Bundle ID 与 App Store Connect 中配置的一致。

1. 在项目根目录运行：

```bash
open ios/Runner.xcworkspace
```

2. 在 Xcode 中：
   - 选择左侧的 **Runner** 项目
   - 选择 **TARGETS** 下的 **Runner**
   - 在 **General** 标签页中，找到 **Bundle Identifier**
   - 确保与 App Store Connect 中的 Bundle ID 一致（如 `com.yourcompany.yourapp`）

### 2.2 启用 In-App Purchase 能力

1. 在 Xcode 中，选择 **Runner** 目标
2. 切换到 **Signing & Capabilities** 标签
3. 点击左上角的 **+ Capability** 按钮
4. 搜索并添加 **In-App Purchase**

⚠️ **注意**：此步骤通常是自动的，但手动添加可以避免潜在问题。

### 2.3 配置 Deployment Target

确保最低部署目标为 iOS 12.0 或更高：

1. 在 Xcode 中，选择 **Runner** 目标
2. 在 **General** 标签页找到 **Deployment Info**
3. 设置 **iOS Deployment Target** 为 **12.0** 或更高

或者直接编辑 `ios/Podfile`：

```ruby
platform :ios, '12.0'
```

### 2.4 更新 Pods

在 `ios` 目录下运行：

```bash
cd ios
pod install
cd ..
```

### 2.5 配置 StoreKit 配置文件（可选）

从 iOS 14 开始，可以使用 StoreKit 配置文件进行本地测试，无需连接 App Store。

1. 在 Xcode 中，选择 **File** → **New** → **File**
2. 搜索 **StoreKit Configuration**
3. 命名为 `Products.storekit`
4. 添加产品配置（用于本地测试）

## 步骤 3：Android 平台配置

### 3.1 配置包名

确保包名与 Google Play Console 中配置的一致。

编辑 `android/app/build.gradle`：

```gradle
android {
    defaultConfig {
        applicationId "com.yourcompany.yourapp"  // 确保与 Play Console 一致
        // ...
    }
}
```

### 3.2 配置最低 SDK 版本

`in_app_purchase` 要求 Android SDK 21+（Android 5.0）。

在 `android/app/build.gradle` 中：

```gradle
android {
    defaultConfig {
        minSdkVersion 21  // 确保至少为 21
        // ...
    }
}
```

### 3.3 配置应用签名（如果未配置）

如果在第 3 章还未配置签名，请参考第 3 章的"步骤 2：配置应用签名"部分。

确保 `android/app/build.gradle` 包含签名配置：

```gradle
android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
        // 调试版本也使用签名（便于测试 IAP）
        debug {
            signingConfig signingConfigs.release
        }
    }
}
```

### 3.4 添加计费权限（自动包含）

`in_app_purchase_android` 插件会自动添加必要的权限，无需手动配置。

如需验证，可以检查 `android/app/src/main/AndroidManifest.xml`，应该包含：

```xml
<uses-permission android:name="com.android.vending.BILLING" />
```

如果使用的是 `in_app_purchase` 3.0+，此权限会自动合并，无需手动添加。

### 3.5 配置 ProGuard（如果使用）

如果启用了代码混淆，需要添加 ProGuard 规则。

创建或编辑 `android/app/proguard-rules.pro`：

```proguard
# In-App Purchase
-keep class com.android.vending.billing.**
```

然后在 `android/app/build.gradle` 中引用：

```gradle
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

## 步骤 4：配置项目代码结构

为了更好地组织代码，建议创建专门的文件来管理订阅功能。

### 推荐的文件结构

```
lib/
├── main.dart
├── models/
│   └── subscription_product.dart    # 订阅产品模型
├── services/
│   └── subscription_service.dart    # 订阅服务类
├── screens/
│   └── subscription_screen.dart     # 订阅界面
└── utils/
    └── iap_helper.dart              # IAP 辅助函数
```

### 创建基础服务类

创建 `lib/services/subscription_service.dart`：

```dart
import 'dart:async';
import 'package:in_app_purchase/in_app_purchase.dart';

class SubscriptionService {
  // 单例模式
  static final SubscriptionService _instance = SubscriptionService._internal();
  factory SubscriptionService() => _instance;
  SubscriptionService._internal();

  // InAppPurchase 实例
  final InAppPurchase _inAppPurchase = InAppPurchase.instance;

  // 购买更新流订阅
  StreamSubscription<List<PurchaseDetails>>? _subscription;

  // 是否初始化
  bool _isInitialized = false;

  // 获取初始化状态
  bool get isInitialized => _isInitialized;

  // 初始化方法（将在第 5 章实现）
  Future<void> initialize() async {
    // 实现代码将在后续章节添加
  }

  // 释放资源
  void dispose() {
    _subscription?.cancel();
  }
}
```

## 步骤 5：配置平台检测

创建 `lib/utils/platform_helper.dart` 用于检测平台：

```dart
import 'dart:io';

class PlatformHelper {
  // 是否为 iOS 平台
  static bool get isIOS => Platform.isIOS;

  // 是否为 Android 平台
  static bool get isAndroid => Platform.isAndroid;

  // 是否为支持的平台
  static bool get isSupportedPlatform => isIOS || isAndroid;

  // 获取平台名称
  static String get platformName {
    if (isIOS) return 'iOS';
    if (isAndroid) return 'Android';
    return 'Unknown';
  }
}
```

## 步骤 6：配置产品 ID 常量

创建 `lib/utils/subscription_constants.dart` 定义产品 ID：

```dart
class SubscriptionConstants {
  // 防止实例化
  SubscriptionConstants._();

  // iOS 产品 ID
  static const String iosMonthlyBasic = 'monthly_basic';
  static const String iosMonthlyPremium = 'monthly_premium';
  static const String iosYearlyPremium = 'yearly_premium';

  // Android 产品 ID
  static const String androidMonthlyBasic = 'monthly_basic';
  static const String androidMonthlyPremium = 'monthly_premium';
  static const String androidYearlyPremium = 'yearly_premium';

  // 所有产品 ID（如果两个平台使用相同的 ID）
  static const Set<String> allProductIds = {
    'monthly_basic',
    'monthly_premium',
    'yearly_premium',
  };

  // 根据平台获取产品 ID
  static Set<String> getProductIds() {
    // 如果 iOS 和 Android 使用相同的产品 ID
    return allProductIds;

    // 如果不同，可以这样区分：
    // if (Platform.isIOS) {
    //   return {iosMonthlyBasic, iosMonthlyPremium, iosYearlyPremium};
    // } else {
    //   return {androidMonthlyBasic, androidMonthlyPremium, androidYearlyPremium};
    // }
  }
}
```

## 步骤 7：验证配置

### 验证清单

完成配置后，请检查：

#### iOS

- [ ] Bundle ID 与 App Store Connect 一致
- [ ] 启用了 In-App Purchase 能力
- [ ] Deployment Target 为 iOS 12.0+
- [ ] Pods 已更新

#### Android

- [ ] 包名与 Google Play Console 一致
- [ ] minSdkVersion 为 21+
- [ ] 配置了应用签名
- [ ] 可以成功构建签名版本

#### Flutter

- [ ] `in_app_purchase` 依赖已添加
- [ ] `flutter pub get` 运行成功
- [ ] 创建了基础服务类
- [ ] 定义了产品 ID 常量

### 运行验证

#### 在 iOS 上验证

```bash
flutter run -d iPhone
```

或使用真机：

```bash
flutter run
```

#### 在 Android 上验证

```bash
flutter run -d Android
```

#### 构建 Release 版本

iOS：

```bash
flutter build ios --release
```

Android：

```bash
flutter build appbundle --release
```

如果构建成功，说明配置正确。

## 常见配置问题

### iOS 问题

#### Q1：Xcode 提示"No such module 'in_app_purchase_storekit'"

**解决方法**：

```bash
cd ios
pod install
cd ..
flutter clean
flutter pub get
```

#### Q2：真机测试时无法连接 App Store

**原因**：网络问题或未配置测试账号

**解决方法**：

1. 确保设备连接稳定网络
2. 使用沙盒测试账号登录（在应用内登录，不是设置中）

#### Q3：Bundle ID 不匹配

**解决方法**：

1. 在 Xcode 中修改 Bundle Identifier
2. 确保与 App Store Connect 完全一致（包括大小写）

### Android 问题

#### Q1：无法测试应用内购买

**原因**：应用未签名或未上传到 Play Console

**解决方法**：

1. 确保使用签名的 APK/AAB
2. 上传到内部测试轨道
3. 使用测试账号安装

#### Q2：签名配置错误

**解决方法**：

1. 检查 `key.properties` 文件路径和内容
2. 确保密钥库文件存在
3. 验证密码正确

#### Q3：minSdkVersion 过低

**错误信息**：Requires minSdkVersion 21

**解决方法**：

在 `android/app/build.gradle` 中设置：

```gradle
minSdkVersion 21
```

## 最佳实践

### 1. 分离配置和代码

将产品 ID、价格等配置信息放在独立的常量文件中，便于维护。

### 2. 使用环境变量

对于敏感信息（如签名密码），使用环境变量而不是硬编码。

### 3. 版本控制

确保将以下文件添加到 `.gitignore`：

```gitignore
# iOS
ios/Flutter/flutter_export_environment.sh
ios/Pods/
ios/.symlinks/
ios/ServiceDefinitions.json

# Android 签名
android/key.properties
android/app/upload-keystore.jks
android/app/debug-keystore.jks

# 本地配置
*.local
```

### 4. 文档化

在项目的 README 中记录：

- 产品 ID 列表
- 测试账号信息（不要提交到代码仓库）
- 配置步骤

### 5. 持续集成

配置 CI/CD 时，确保：

- 签名密钥通过加密方式存储
- 构建脚本包含必要的配置步骤

## 小结

在本章中，我们学习了：

- 如何在 Flutter 项目中添加 `in_app_purchase` 依赖
- iOS 和 Android 平台的详细配置步骤
- 如何组织订阅功能的代码结构
- 如何验证配置是否正确
- 常见配置问题的解决方法

完成项目配置后，我们就可以开始编写代码实现订阅功能了。

## 下一步

[第 5 章：初始化 IAP 插件](05-initialize-iap.md) - 学习如何在应用启动时初始化 `in_app_purchase` 插件。
