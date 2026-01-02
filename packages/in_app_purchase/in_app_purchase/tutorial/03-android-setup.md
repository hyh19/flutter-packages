# 第 3 章：Android Google Play Console 配置

## 本章目标

- 了解 Google Play Console 的基本操作
- 创建和配置订阅产品
- 配置应用签名
- 设置测试环境和测试账号

## 前提条件

- 拥有 Google Play Console 开发者账号（$25 一次性注册费）
- 准备好应用的包名（Package Name）
- 了解 Android 应用签名流程

## Google Play Console 概述

Google Play Console 是 Google 提供的应用管理平台，用于：

- 发布和管理 Android 应用
- 配置应用内购买和订阅
- 管理测试轨道和测试用户
- 查看应用统计和收入报告

访问地址：[https://play.google.com/console](https://play.google.com/console)

## 步骤 1：创建应用

### 创建新应用

1. 登录 Google Play Console
2. 点击 **创建应用**（Create app）
3. 填写应用信息：
   - **应用名称**：你的应用名称
   - **默认语言**：简体中文
   - **应用类型**：应用
   - **免费或付费**：免费（或付费）
4. 同意开发者计划政策
5. 同意美国出口法律
6. 点击 **创建应用**

### 记录包名

创建应用后，系统会自动生成或要求你输入包名（Package Name），例如：

```
com.yourcompany.yourapp
```

⚠️ **重要**：包名一旦设置就无法更改，请确保与 Flutter 项目中的包名一致。

## 步骤 2：配置应用签名

Android 应用必须经过签名才能测试和发布应用内购买功能。

### 了解应用签名

Android 提供两种签名方式：

1. **Google Play 应用签名**（推荐）
   - Google 管理签名密钥
   - 更安全，支持密钥恢复
   - 简化签名流程

2. **自己管理签名密钥**
   - 你负责保管密钥文件
   - 丢失密钥无法恢复

### 启用 Google Play 应用签名

1. 在 Google Play Console 中打开你的应用
2. 进入 **设置** → **应用完整性**（App integrity）
3. 在 **应用签名** 部分，选择启用
4. 上传或生成签名密钥

### 创建上传密钥

如果你选择 Google Play 应用签名，需要创建一个上传密钥（Upload Key）：

#### 使用 Android Studio 生成

1. 在 Android Studio 中，选择 **Build** → **Generate Signed Bundle / APK**
2. 选择 **Android App Bundle**
3. 点击 **Create new** 创建新密钥库
4. 填写密钥信息：
   - **Key store path**：保存位置
   - **Password**：密钥库密码
   - **Key alias**：密钥别名（如 `upload`）
   - **Key password**：密钥密码
   - **Validity**：有效期（建议 25 年以上）
   - **Certificate**：填写你的信息
5. 保存密钥库文件

#### 使用命令行生成

```bash
keytool -genkey -v -keystore ~/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload
```

⚠️ **重要**：妥善保管密钥库文件和密码，丢失后无法恢复！

### 配置 Flutter 项目签名

1. 将密钥库文件复制到项目中（如 `android/app/upload-keystore.jks`）

2. 创建 `android/key.properties` 文件：

```properties
storePassword=你的密钥库密码
keyPassword=你的密钥密码
keyAlias=upload
storeFile=upload-keystore.jks
```

3. 编辑 `android/app/build.gradle`：

```gradle
// 在 android 块之前添加
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    // ...
    
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
        // 可选：debug 构建也使用签名（便于测试 IAP）
        debug {
            signingConfig signingConfigs.release
        }
    }
}
```

4. 将 `key.properties` 添加到 `.gitignore`：

```gitignore
# Android 签名文件
android/key.properties
android/app/upload-keystore.jks
```

## 步骤 3：创建订阅产品

### 进入产品配置页面

1. 在 Google Play Console 中打开你的应用
2. 在左侧菜单选择 **产品** → **订阅**（Subscriptions）
3. 点击 **创建订阅**（Create subscription）

### 配置订阅基础信息

#### 产品 ID

- **产品 ID**：`monthly_premium`
  - ⚠️ 产品 ID 创建后无法修改
  - 只能包含小写字母、数字、下划线、点号
  - 建议格式：`{period}_{level}`
  - 例如：`monthly_premium`、`yearly_vip`

#### 名称和描述

- **名称**：月度高级会员
  - 用户在购买界面看到的产品名称
  - 支持多语言

- **描述**（可选）：解锁所有高级功能，享受无限制使用
  - 产品的详细说明

### 配置基础方案（Base Plan）

Google Play 的订阅采用"基础方案 + 优惠"模式。

#### 基础方案 ID

- **基础方案 ID**：`monthly-standard`
  - 一个订阅可以有多个基础方案
  - 用于区分不同的计费周期或价格策略

#### 计费周期

选择自动续订周期：

- 每周
- 每月
- 每 2 个月
- 每 3 个月（季度）
- 每 4 个月
- 每 6 个月
- 每年

示例：选择 **每月**

#### 价格设置

1. 点击 **设置价格**（Set price）
2. 选择定价方式：
   - **按国家/地区设置价格**：为每个市场单独设置价格
   - **使用价格模板**：基于一个市场自动计算其他市场价格

3. 设置价格：
   - **中国**：CN¥18.00
   - **美国**：$2.99
   - **其他地区**：自动建议或手动设置

4. 点击 **应用价格**

#### 续订类型

- **自动续订**：用户订阅后会自动续费（推荐）
- **预付款**：需要用户手动续订（少见）

### 配置优惠（Offers）

优惠用于提供试用期或折扣价格。

#### 创建免费试用

1. 在基础方案页面，点击 **添加优惠**（Add offer）
2. 选择 **免费试用**（Free trial）
3. 填写优惠信息：
   - **优惠 ID**：`free-trial`
   - **优惠名称**：7 天免费试用
   - **试用期**：7 天
   - **合格性**：
     - 新用户：首次订阅的用户
     - 所有用户：包括之前订阅过的用户（慎用）
4. 点击 **激活**（Activate）

#### 创建首期折扣

1. 点击 **添加优惠**（Add offer）
2. 选择 **单期优惠**（Single-period offer）
3. 填写优惠信息：
   - **优惠 ID**：`intro-discount`
   - **优惠名称**：首月特惠
   - **折扣期数**：1 期
   - **折扣价格**：CN¥6.00
4. 点击 **激活**

#### 创建多期优惠

1. 点击 **添加优惠**（Add offer）
2. 选择 **多期优惠**（Multiple-period offer）
3. 配置多个优惠阶段：
   - **第 1 阶段**：1 期，CN¥6.00
   - **第 2 阶段**：2 期，CN¥12.00
   - **第 3 阶段**：正常价格 CN¥18.00
4. 点击 **激活**

### 配置宽限期（可选）

宽限期允许用户在支付失败后继续使用服务一段时间。

1. 在订阅设置中，找到 **宽限期**（Grace period）
2. 启用宽限期
3. 选择宽限期时长：3 天 / 7 天
4. 在宽限期内，应用应继续提供服务

### 配置账号保留（可选）

账号保留期间，用户无法使用服务，但保留订阅状态。

1. 在订阅设置中，找到 **账号保留**（Account hold）
2. 启用账号保留
3. 选择保留时长：最长 30 天
4. 在保留期内，应用应禁用订阅功能

## 步骤 4：创建更多订阅产品

创建不同等级或周期的订阅产品。

### 示例产品配置

| 产品 ID | 名称 | 基础方案 ID | 周期 | 价格（中国） |
|---------|------|------------|------|------------|
| `monthly_basic` | 基础月度会员 | `monthly-standard` | 每月 | CN¥12.00 |
| `monthly_premium` | 高级月度会员 | `monthly-standard` | 每月 | CN¥18.00 |
| `yearly_premium` | 高级年度会员 | `yearly-standard` | 每年 | CN¥168.00 |

### 创建年度订阅

对于同一产品的不同周期，可以创建多个基础方案：

1. 打开已创建的订阅产品
2. 点击 **添加基础方案**（Add base plan）
3. 配置年度方案：
   - **基础方案 ID**：`yearly-standard`
   - **计费周期**：每年
   - **价格**：CN¥168.00
4. 可以为年度方案单独配置优惠

## 步骤 5：激活订阅

创建完订阅产品后，需要激活才能使用。

1. 检查订阅产品的配置是否完整
2. 点击 **激活**（Activate）按钮
3. 确认激活

⚠️ **注意**：激活后某些配置（如产品 ID）无法修改，只能停用后重新创建。

## 步骤 6：设置测试环境

Google Play 提供多种测试方式，推荐使用内部测试轨道。

### 创建内部测试版本

1. 在左侧菜单选择 **测试** → **内部测试**（Internal testing）
2. 点击 **创建新版本**（Create new release）
3. 上传 AAB 或 APK 文件（稍后讲解如何构建）
4. 填写版本说明
5. 点击 **保存** 和 **审核版本**
6. 确认并 **开始发布到内部测试**

### 添加测试员

#### 创建测试员名单

1. 在内部测试页面，找到 **测试员** 部分
2. 点击 **创建电子邮件列表**（Create email list）
3. 填写列表名称：Internal Testers
4. 添加测试员的 Google 账号邮箱（每行一个）
5. 保存列表

#### 将测试员加入测试

1. 在内部测试页面，点击 **管理测试员**
2. 选择刚创建的测试员名单
3. 保存

### 获取测试链接

1. 在内部测试页面，找到 **测试员如何加入测试**
2. 复制 **加入链接**
3. 将链接发送给测试员
4. 测试员点击链接即可加入测试并下载应用

### 配置许可测试

许可测试允许特定账号免费测试付费功能。

1. 在左侧菜单选择 **设置** → **许可测试**（License testing）
2. 点击 **添加许可测试员**
3. 输入测试员的 Google 账号邮箱
4. 选择测试响应：**许可**（Licensed）
5. 保存

⚠️ **重要**：许可测试员可以免费测试应用内购买，不会真实扣费。

## 步骤 7：构建和上传签名版本

### 构建 Android App Bundle

在 Flutter 项目根目录执行：

```bash
flutter build appbundle --release
```

生成的文件位于：

```
build/app/outputs/bundle/release/app-release.aab
```

### 上传到 Google Play Console

1. 在内部测试页面，点击 **创建新版本**
2. 点击 **上传** 按钮
3. 选择生成的 `app-release.aab` 文件
4. 等待上传和处理完成
5. 填写版本说明
6. 保存并发布到内部测试

## 步骤 8：测试订阅

### 在测试设备上安装应用

1. 使用测试员账号登录 Android 设备
2. 访问测试链接并接受邀请
3. 在 Google Play 商店下载并安装应用

### 测试购买流程

1. 打开应用
2. 点击订阅产品
3. 系统会显示购买对话框
4. 确认购买（许可测试员不会扣费）
5. 验证订阅是否生效

### 测试自动续订

Google Play 会加速测试环境的续订周期：

| 真实周期 | 测试周期 |
|---------|---------|
| 1 周 | 5 分钟 |
| 1 个月 | 5 分钟 |
| 3 个月 | 10 分钟 |
| 6 个月 | 15 分钟 |
| 1 年 | 30 分钟 |

测试订阅最多续订 6 次后自动失效。

## 验证配置

完成以上步骤后，请检查：

- [ ] 创建了应用并记录了包名
- [ ] 配置了应用签名（密钥库和 build.gradle）
- [ ] 创建了至少一个订阅产品并激活
- [ ] 配置了至少一个基础方案和价格
- [ ] 设置了内部测试轨道
- [ ] 添加了测试员
- [ ] 上传了签名的 AAB 文件
- [ ] 配置了许可测试账号

## 常见问题

### Q1：产品 ID 可以修改吗？

**不可以**。产品 ID 一旦创建并激活就无法修改，只能停用后重新创建。停用的产品 ID 无法重用。

### Q2：必须使用 AAB 格式吗？

Google Play 从 2021 年 8 月起要求新应用使用 AAB 格式。虽然 APK 仍可用于测试，但建议使用 AAB。

### Q3：测试时为什么会扣费？

如果测试账号未添加到许可测试员名单，或未使用内部测试版本，可能会真实扣费。请确保：

- 测试账号在许可测试员名单中
- 使用内部测试轨道的版本

### Q4：如何取消测试订阅？

1. 打开 Google Play 商店应用
2. 点击个人资料图标 → **付款和订阅** → **订阅**
3. 选择要取消的订阅
4. 点击 **取消订阅**

### Q5：基础方案和优惠有什么区别？

- **基础方案**：定义订阅的基本属性（周期、价格）
- **优惠**：在基础方案基础上提供折扣或试用

一个订阅可以有多个基础方案（如月付和年付），每个基础方案可以有多个优惠。

### Q6：订阅升级和降级如何实现？

Android 没有类似 iOS 的订阅组概念，需要在代码中手动处理：

- 升级：用户切换到更高等级的订阅
- 降级：用户切换到更低等级的订阅

具体实现将在第 12 章详细讲解。

## 小结

在本章中,我们学习了：

- 如何在 Google Play Console 中创建应用
- 如何配置应用签名
- 如何创建和配置订阅产品
- 如何设置测试环境和测试员
- 如何上传和测试应用

完成 Android 配置后，我们已经完成了双平台的商店配置。

## 下一步

[第 4 章：Flutter 项目配置](04-flutter-setup.md) - 学习如何在 Flutter 项目中集成 `in_app_purchase` 插件。
