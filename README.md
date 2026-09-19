<div align="center">
<img src="Scale/Assets.xcassets/AppIcon.appiconset/Scale_1024.png" width="128" alt="Scale App Icon" />

# Scale · 蚂蚁阿福体脂秤

一个原生 iOS App，通过蓝牙连接蚂蚁阿福体脂秤，读取体重与阻抗，本地估算身体成分，并同步到 Apple「健康」。

</div>

## 简介

Scale 是用 SwiftUI 写的 iOS 体脂秤 App。手机作为蓝牙中心设备直接与体脂秤通信，无需依赖原厂 App 或云端：

- 自动扫描并连接沃莱系列体脂秤（服务 UUID `FFB0`），并记住上次连过的秤下次直连。
- 连接后向秤下发用户资料（身高 / 年龄 / 性别），触发它做阻抗测量。
- 解析秤的 "Scale27" 蓝牙协议（体重包 / 阻抗包），实时显示体重、稳定后锁定读数。
- 基于阻抗 + 用户资料，本地用 BIA 公式估算体脂率、BMI、水分率、肌肉率、基础代谢、去脂体重。
- 测量完成后自动把体重、体脂率、BMI、去脂体重写入 Apple「健康」App。

> ⚠️ 体脂等身体成分为使用体脂秤提供的原始阻值以本地公式估算，与原厂 App 的计算公式不完全一致，仅供参考（反正都差不多准）。

## 使用方法

本项目未上架 App Store，需要下载打包好的 IPA 后自行安装到 iPhone。整体流程：**下载 IPA → 签名 → 安装 → 首次运行授权**。

### 1. 下载 IPA

从 Releases 下载最新安装包：

> 📦 [Scale.v0.1.ipa](https://github.com/Aoko-Aozaki/ant-afu-scale/releases/download/ipa/Scale.v0.1.ipa)

IPA 是没有签名的，直接装不上，需要用下面任一方式签名。**不确定选哪个：想免费、能接受每 7 天重签一次，选方式 A；想省事、装完能用大半年，选方式 B。**

### 2A. 自签名（免费，证书 7 天有效）

用一个普通 Apple ID（免费开发者账号）给 IPA 签名。证书只有 **7 天有效期**，到期后 App 会打不开，需要重新签一次；期间不要删除签名工具。

需要：一台电脑（Windows / macOS）+ 数据线 + 一个 Apple ID。

以 **Sideloadly**（[sideloadly.io](https://sideloadly.io)，Win/Mac 都有）为例：

1. 电脑装好 Sideloadly，用数据线连上 iPhone，手机上点「信任此电脑」。
2. 打开 Sideloadly，把下载好的 `Scale.v0.1.ipa` 拖进去。
3. 在 Apple account 处填你的 Apple ID，点 **Start**，按提示输入密码。
4. 等进度条走完，App 就装到手机上了。

> AltStore、Feather、爱思助手等工具同理

### 2B. 闲鱼代签（几块钱，签名约 1 年有效）

原理是让卖家用他们的**99 美元/年的付费开发者账号**把你的设备加进去，再签名给你，有效期约 1 年。在闲鱼搜「**iOS 开发者签名 / 代签名 / p12 签名**」，几块钱一个。

流程通常是：

1. 提供你 iPhone 的 **UDID**（设备唯一标识）。
   - 手机 Safari 打开卖家给的 UDID 获取链接，安装一个描述文件即可读出。
2. 卖家给你签名的证书（p12）和描述文件（mobileprovision）
3. 用 [**zsign**](zhlynn/zsign) 之类支持改 Bundle ID 的工具重签名，**必须把 Bundle ID 改成描述文件里授权的那个**，例如：

   ```bash
   zsign -k cert.p12 -p 密码 -m profile.mobileprovision -b 描述文件里的BundleID -o Scale-signed.ipa Scale.v0.1.ipa
   ```

4. 签好的 IPA 用爱思助手 / Sideloadly 装上

> ⚠️ **别直接用爱思助手的「证书签名」功能**：它不改 Bundle ID，签完打开会报 `Missing com.apple.developer.healthkit entitlement`，数据无法写入苹果健康。
>
> 原因：用了新证书却没改 Bundle ID，iOS 会把**整份 entitlements 判为无效**，而不是只丢某一条，所以明明签进去的 HealthKit 也会报 missing。只有把 Bundle ID 改成描述文件授权的那个，权限才会生效。

> 也有企业签等更便宜的方案，但企业签容易被苹果吊销（掉签），装完随时可能打不开；追求稳定优先选个人开发者账号代签。

### 3. 首次运行

装好后第一次打开，按下面处理：

1. **信任开发者**：若提示「未受信任的开发者」，去 *设置 → 通用 → VPN与设备管理*，点开对应描述文件选择「信任」。
2. **打开开发者模式**（仅自签名需要，iOS 16+）：*设置 → 隐私与安全性 → 开发者模式*，打开后重启手机。
3. **授予权限**：首次进入会依次请求
   - **蓝牙**权限——用来连接体脂秤，必须允许；
   - **健康**写入权限——用来把测量结果同步到「健康」App，按需允许（不想同步可在 [我的资料](Scale/UserProfile.swift) 里关掉「测量后自动写入健康」开关）。如果不小心按错了可以在健康App里重新授予
4. **填写资料**：点右上角头像图标，填身高 / 年龄 / 性别（体脂估算需要），点保存。
5. **开始测量**：确保手机蓝牙已开、体脂秤有电，点「开始测量」，光脚站上秤保持不动，等读数稳定即可。

## 运行截图

| 测量结果 | 我的资料 |
| :---: | :---: |
| <img src="readme.assets/IMG_2596.PNG" width="300" alt="测量结果" /> | <img src="readme.assets/IMG_2595.PNG" width="300" alt="我的资料" /> |

## 项目结构

| 文件 | 作用 |
| --- | --- |
| [ContentView.swift](Scale/ContentView.swift) | 主界面：测量状态、结果卡片、资料设置页 |
| [BluetoothManager.swift](Scale/BluetoothManager.swift) | CoreBluetooth：扫描 / 连接 / 订阅通知 / 下发资料 / 稳定判定 |
| [Scale27.swift](Scale/Scale27.swift) | 沃莱 "Scale27" 蓝牙协议的编解码 |
| [BodyComposition.swift](Scale/BodyComposition.swift) | 阻抗 + 资料 → 身体成分的 BIA 估算公式 |
| [HealthKitManager.swift](Scale/HealthKitManager.swift) | 把测量结果写入 Apple「健康」 |
| [UserProfile.swift](Scale/UserProfile.swift) | 身高 / 年龄 / 性别，存本地 UserDefaults |

## 从源码构建并运行到 iPhone

除了安装 Releases 中的 IPA，也可以直接使用 Xcode 从源码构建并运行。

以下流程主要用于开发、调试或维护本项目。

### 环境要求

- macOS
- Xcode（需安装对应的 iOS Platform Support）
- Apple ID
- 一台真实 iPhone
- 支持的体脂秤

> 蓝牙称重的完整流程需要使用真实 iPhone + 真实体脂秤验证，不能仅依赖 Simulator。

### 1. Clone 仓库

Clone 自己需要运行或维护的仓库：

```bash
git clone <your-repository-url>
cd ant-afu-scale-ios
```

然后使用 Xcode 打开：

```text
Scale.xcodeproj
```

如果 Git 命令提示尚未同意 Xcode / Apple SDK License，可以先执行：

```bash
sudo xcodebuild -license
```

按照提示阅读并接受 License 后重新执行 Git / Xcode 操作。

### 2. 登录 Apple ID

打开：

`Xcode → Settings → Accounts`

登录用于开发签名的 Apple ID。

免费 Apple ID 的 Personal Team 即可用于个人真机测试。

### 3. 配置 Signing

在 Xcode 中进入：

`TARGETS → Scale → Signing & Capabilities`

确认：

- `Automatically manage signing` 已开启；
- `Team` 选择自己的 Apple ID / Personal Team；
- Bundle Identifier 使用自己可签名的唯一标识，例如：

  ```text
  com.yourname.antafuscale
  ```

- HealthKit capability 正常存在；
- 项目所需的 Bluetooth 权限配置正常存在。

不要直接复用其他开发者的个人 Bundle Identifier。

### 4. 连接 iPhone

首次配置建议使用数据线连接 iPhone。

1. 解锁 iPhone；
2. 如果手机询问是否信任这台 Mac，选择「信任」；
3. 根据 iOS 提示开启 Developer Mode；
4. 等待 Xcode 完成设备初始化；
5. 在 Xcode 顶部选择真实 iPhone 作为 Run Destination。

注意：

```text
Any iOS Device
```

属于 Build Only Device，不能直接运行 App。

需要选择实际连接的 iPhone。

### 5. Build & Run

点击 Xcode 顶部的：

```text
▶ Run
```

Xcode 会完成：

```text
Build
↓
Signing
↓
Install
↓
Launch
```

如果 App 已经安装但提示开发者证书不受信任，在 iPhone 中进入：

`设置 → 通用 → VPN与设备管理`

找到对应开发者并选择「信任」。

然后重新打开 App。

### 6. 首次授权

首次运行后，按照前文「首次运行」章节：

1. 授予蓝牙权限；
2. 授予 Apple「健康」写入权限；
3. 填写正确的身高 / 年龄 / 性别；
4. 保存个人资料；
5. 完成一次真实称重。

### 7. Smoke Test

源码环境恢复完成后，不应只检查「App 能不能打开」。

建议完成一次完整 smoke test：

```text
App 启动
↓
扫描 / 连接体脂秤
↓
实时读取体重
↓
获取阻抗
↓
生成身体成分结果
↓
写入 HealthKit
↓
Apple「健康」出现新记录
```

确认：

- [ ] App 可以正常启动
- [ ] 可以扫描并连接体脂秤
- [ ] 可以实时显示体重
- [ ] 可以获取阻抗并完成测量
- [ ] 可以正常显示身体成分估算结果
- [ ] 体重可以写入 Apple「健康」
- [ ] 体脂率可以写入 Apple「健康」
- [ ] BMI 可以写入 Apple「健康」
- [ ] 去脂体重可以写入 Apple「健康」

> 只有完整链路通过，才认为当前开发环境恢复成功。

---

## 开发与维护

### 维护原则

为了让个人 Fork 始终保留一个可恢复的 working baseline，建议遵循：

```text
main 保持可运行
↓
一个问题创建一个 branch
↓
先理解问题，再修改
↓
尽量使用最小修改
↓
Review diff
↓
Xcode Build
↓
真实 iPhone + 体脂秤测试
↓
Apple Health 验证
↓
Commit
↓
Merge
```

不要在没有 checkpoint 的情况下同时修改多个不相关问题。

### 免费开发签名过期

如果使用免费 Apple ID / Personal Team 部署，开发签名到期后 App 可能无法继续打开。

通常不需要重新配置整个项目。

1. Mac 打开 `Scale.xcodeproj`；
2. Xcode 登录自己的 Apple ID；
3. 连接并解锁 iPhone；
4. 进入 `Signing & Capabilities`；
5. 确认正确的 Personal Team；
6. 选择真实 iPhone 作为 Run Destination；
7. 再次点击 Run。

Xcode 会重新完成签名、安装和启动。

重新安装后建议完成一次真实称重，确认蓝牙和 HealthKit 仍然正常。

### 换 Mac

项目应能够仅依赖 Git 仓库重新建立开发环境。

在新 Mac 上：

1. 安装 Xcode；
2. 安装对应的 iOS Platform Support；
3. Xcode 登录 Apple ID；
4. Clone 仓库；
5. 打开 `Scale.xcodeproj`；
6. 重新配置 Personal Team；
7. 必要时设置自己唯一的 Bundle Identifier；
8. 连接真实 iPhone；
9. Build & Run；
10. 完成一次真实称重；
11. 检查 Apple「健康」是否收到数据。

不应把以下内容提交到 Git：

- Apple ID 密码；
- 私钥；
- `.p12` 开发证书；
- Provisioning Profile；
- 其他个人签名凭据。

### 换 iPhone

换手机后：

1. 将新 iPhone 连接到 Mac；
2. 信任 Mac；
3. 开启 Developer Mode；
4. 在 Xcode 中选择新 iPhone；
5. 检查 Signing；
6. Build & Run；
7. 重新授予蓝牙权限；
8. 重新授予 Apple「健康」写入权限；
9. 填写 / 检查个人资料；
10. 完成一次真实称重 smoke test。

### Xcode / iOS 升级

升级 Xcode 或 iOS 前建议先确认仓库状态：

```bash
git status
```

如果存在已经完成的修改，先 commit / push，确保有明确 rollback point。

升级完成后，不建议立即顺手修改代码。

先使用原代码执行：

```text
Build
↓
真机 Run
↓
蓝牙连接
↓
真实称重
↓
HealthKit 写入
```

如果出现新的 warning / error，再判断问题属于：

- Xcode / Swift 编译变化；
- iOS API 行为变化；
- Signing / Provisioning；
- HealthKit entitlement；
- CoreBluetooth 行为变化；
- 项目代码本身。

尽量避免把「工具链升级」和「功能重构」放在同一个 commit 中。

### HealthKit 写入异常

如果 App 可以正常称重，但 Apple「健康」没有出现新数据：

1. 检查 iPhone 中 Scale 的健康数据写入权限；
2. 确认对应指标仍然允许写入；
3. 检查 Xcode 中 HealthKit capability；
4. 检查项目 entitlement / signing 是否正常；
5. 完成一次新的真实测量；
6. 在 Apple「健康」中检查记录的数据来源和时间。

当前项目写入：

- 体重
- 体脂率
- BMI
- 去脂体重

注意：每次成功测试都可能创建新的真实 HealthKit sample，因此调试时避免无意义地重复写入。

### 修改代码前

开始新的修改前：

```bash
git status
git branch --show-current
git log -1 --oneline
```

理想状态：

```text
branch: main
working tree: clean
latest commit: 已知可运行版本
```

然后为单独问题创建 branch，例如：

```bash
git switch -c fix/reset-measurement-state
```

完成修改后先检查：

```bash
git status
git diff
```

真机测试通过以后再 commit。

---

## 开发 Roadmap

以下 Roadmap 主要用于本 Fork 的个人维护与学习，不代表原项目作者的开发计划。

### V1.1 · 稳定与可信

优先处理低风险、容易验证的问题：

- [ ] 新一轮测量开始时正确重置上一轮 measurement state；
- [ ] HealthKit 授权失败时提供明确反馈；
- [ ] 检查并修正 HealthKit 权限用途文案；
- [ ] 整理 deployment target 配置；
- [ ] 清理未使用状态变量；
- [ ] 将健康数据相关 console log 限制在 Debug 环境；
- [x] 增加源码构建与维护说明。

### V1.2 · 好用与可维护

在基本功能稳定后，再提高工程健壮性：

- [ ] 为蓝牙扫描增加 timeout；
- [ ] 为连接过程增加 timeout；
- [ ] 为服务 / Characteristic discovery 增加错误处理；
- [ ] 为等待阻抗结果增加 timeout；
- [ ] 处理 CoreBluetooth callback error；
- [ ] 改善用户可理解的蓝牙错误提示；
- [ ] 避免在未确认个人资料时静默使用默认资料估算体脂；
- [ ] 增加 BIA 测量条件 / 估算性质说明；
- [ ] 增加基础协议解析测试；
- [ ] 增加身体成分公式固定输入测试；
- [ ] 增加 measurement state transition 测试。

### V2.0 · 协议与产品化探索

这些修改可能影响设备兼容性或核心协议，需在充分理解和测试后进行：

- [ ] 验证并精确匹配 notify / write Characteristic UUID；
- [ ] 增加 Scale27 packet checksum 校验；
- [ ] 研究 grouped ADC / impedance packet；
- [ ] 支持设备选择 / 多体脂秤场景；
- [ ] 增加测量质量和异常阻抗提示；
- [ ] 如确有需求，再评估读取 HealthKit 历史数据与趋势展示。

## 参考项目：ant-afu-welland-scale

本项目的蓝牙协议与估算逻辑参考了 [`ant-afu-welland-scale`](https://github.com/Mzdyl/ant-afu-welland-scale) —— 一个用 Python 写的 macOS 命令行读秤工具。Scale 把它的核心算法移植成了 Swift，具体借鉴的部分：

- **"Scale27" 蓝牙协议解析**：[Scale27.swift](Scale/Scale27.swift) 移植自其 `protocol.py`，包括 20 字节数据包的包头 / 包类型判定、体重编码的取整与刻度换算（`gunit`）、阻抗归一化，以及下发时间 + 用户资料（`encode_time_and_user_info_27`）以触发体脂测量的逻辑。
- **身体成分估算公式**：[BodyComposition.swift](Scale/BodyComposition.swift) 的 BIA 公式源自其使用的 `ha-miscale2` 算法，用体重、阻抗、身高、年龄、性别估算体脂率、水分、骨量、肌肉率等。
  

区别在于：`ant-afu-welland-scale` 是 macOS 上跑的 Python CLI，而 Scale 是 iOS 原生 App（靠 CoreBluetooth + SwiftUI），并额外接入了 Apple「健康」。参考项目自身的用法请见其仓库：<https://github.com/Mzdyl/ant-afu-welland-scale>。



