# 3 天源码阅读计划（第一遍）

## 学习目标

- 快速建立项目的整体结构和运行流程，不追求记住所有实现细节。
- 能说明：程序如何启动、App 如何注册和切换、输入如何进入、硬件如何被调用、界面如何刷新。
- 第一遍结束后，选出值得在第二遍深入阅读的 2～3 个模块。

## 第一遍阅读原则

- 按“入口 → 生命周期 → 输入 → HAL → 输出”的调用链阅读。
- 每个模块先看头文件、入口函数和核心状态，不逐行研究。
- 遇到第三方库，只确认项目如何调用它，不深入库内部实现。
- 图片、字体、音频等生成资源只确认格式和使用位置。
- 每个模块最多记录 3 个暂时没弄懂的问题，先继续向下阅读。

---

## Day 1：启动流程、框架与 Launcher

目标：理解系统从上电到 Launcher 出现，以及从 Launcher 打开一个 App 的完整主链路。

### 项目入口与 HAL

- [x] 浏览仓库目录、构建文件和 `main` 目录，了解项目边界。
- [x] 阅读 `main/main.cpp`，梳理 `app_main()` 的初始化和主循环。
- [x] 阅读 `hal/hal.h`、`hal/hal.cpp`，了解 HAL 管理的硬件资源。
- [x] 理解 `feedTheDog()`、`GetHAL().update()`、Mooncake `update()` 的职责。
- [x] 了解屏幕、Canvas、亮度和启动动画的基本调用方式。

### 键盘输入链路

- [x] 理解 TCA8418 的 7×8 物理矩阵、FIFO 和 IRQ 清除逻辑。
- [x] 理解物理按键位置到逻辑键值的映射。
- [x] 区分原始按键事件、字符事件和组合键状态。
- [x] 了解 Launcher 为什么需要关心按键的物理位置。

### Mooncake 与 App 生命周期

- [x] 理解 `AppAbility`、`AbilityManager` 和 `AppInfo` 的作用。
- [x] 区分 `_new_ability_list` 与 `_ability_list`。
- [x] 区分 `_app_ability_manager` 与 `_extension_ability_manager`。
- [x] 梳理 `onCreate()`、`onOpen()`、`onRunning()`、`onClose()` 的调用时机。
- [x] 确认当前项目采用静态注册 App，没有动态安装或内存不足时自动卸载 App 的机制。

### Launcher

- [x] 阅读 `Launcher::onCreate()` 的入口，理解 App 信息设置与创建日志。
- [x] 阅读 `boot_anim()`，了解 RGB565 图片、字体、亮度动画和启动音效。
- [x] 阅读 `start_menu()`：菜单数据如何建立、图标如何排列以及如何读取按键。
- [x] 阅读 `start_system_bar()` / `update_system_bar()`：电量、Wi-Fi 和状态栏如何刷新。
- [x] 阅读 `start_keyboard_bar()`：Fn、Shift、Ctrl、Alt 等按键状态如何显示。
- [x] 理解 `onCreate()` 最后的 `open()` 为什么用于启动 Launcher 自身。
- [x] 阅读 `Launcher::onRunning()`：系统栏、菜单和前台 App 状态如何持续更新。
- [x] 跟踪“选择图标 → 打开 App → App 运行 → 返回 Launcher”的完整调用链。

### Day 1 输出

- [x] 能口头说明：`app_main → HAL 初始化 → App 注册 → Launcher → 主循环`。
- [x] 能说明键盘事件如何从 TCA8418 到达 Launcher 或 App。
- [x] 在笔记中画一张主流程图，作为后两天的导航图。

---

## Day 2：典型 App 与通用开发模式

目标：通过几个代表性 App，总结本项目中 App 的通用写法，而不是逐个读完全部代码。

### 简单 App：建立模板

- [ ] 阅读 `app_wifi_scan`：观察最简单的初始化、扫描、状态更新和界面刷新流程。
- [ ] 阅读 `app_clock`：观察定时更新和持续渲染的写法。
- [ ] 阅读 `app_imu`：观察传感器数据读取与显示的基本模式。
- [ ] 总结一个 App 的通用骨架：构造/创建、打开、循环、输入、绘制、关闭。

### 有状态与事件驱动的 App

- [ ] 阅读 `app_set_wifi`：重点看页面状态、键盘输入、Wi-Fi 连接和配置保存。
- [ ] 阅读 `app_chat` 与 `ChatView`：重点看 App、View、回调和 ESP-NOW 数据之间的关系。
- [ ] 阅读 `app_remote` 与 `ReplView`：观察视图复用、命令输入和结果展示。
- [ ] 比较“主循环轮询”“状态机”“事件/回调”三种组织方式。

### 资源生命周期

- [ ] 检查代表性 App 在 `onOpen()` 中申请了哪些资源。
- [ ] 检查对应资源是否在 `onClose()` 中停止、释放或恢复。
- [ ] 记录共享资源：屏幕、键盘、Wi-Fi、ESP-NOW、音频、扩展接口。

### Day 2 输出

- [ ] 写出一份不超过 20 行的“通用 App 阅读模板”。
- [ ] 为三个代表性 App 各写一句职责说明。
- [ ] 能从入口追踪一次“按键 → 状态变化 → 屏幕刷新”。

---

## Day 3：硬件专项、复杂 App 与全局复盘

目标：认识项目能力边界，知道复杂功能位于哪里；第一遍不陷入协议和算法细节。

### 硬件专项 App

- [ ] 阅读 `app_record`：关注麦克风、缓冲区、录音状态和音频资源释放。
- [ ] 阅读 `app_keyboard`：关注键盘事件如何转换为 BLE/USB HID 输出。
- [ ] 阅读 `app_lora_chat`：关注 LoRa 模块初始化、收发流程和扩展硬件占用。
- [ ] 阅读 `app_gps`：关注串口数据、GPS 解析和位置信息展示。
- [ ] 阅读 `app_stringir_toolkit`：只梳理数据输入、编码、红外发送的主状态机。
- [ ] 阅读 `app_repl`：只确认 PikaScript 的初始化、输入、执行和输出重定向边界。

### 构建与依赖

- [ ] 阅读顶层及 `main` 的 `CMakeLists.txt`，了解源码和资源如何进入固件。
- [ ] 阅读 `idf_component.yml`，确认 ESP-IDF 组件依赖。
- [ ] 浏览 `sdkconfig.defaults` 和分区配置，只记录与硬件能力、Flash/RAM 有关的关键项。
- [ ] 将 M5GFX、M5Unified、Mooncake、Smooth UI Toolkit、PikaScript 标记为第三方边界。

### 全局复盘

- [ ] 完善架构图：`硬件/HAL → 框架 → Launcher/App → View`。
- [ ] 制作模块表：模块名、入口、职责、依赖、主要资源、退出时的清理动作。
- [ ] 回看主循环，确认每个已读模块由谁驱动、何时更新。
- [ ] 整理仍不清楚的问题，并区分“必须理解”和“可以查文档”。
- [ ] 选择第二遍精读的 2～3 个模块，并说明选择原因。

### Day 3 输出

- [ ] 能在 5 分钟内介绍整个项目的架构和运行流程。
- [ ] 能快速定位：新增 App、处理按键、画界面、访问硬件分别应修改哪里。
- [ ] 完成第一遍源码阅读总结。

---

## 第一遍暂不深入

- [ ] M5GFX、M5Unified 的底层驱动实现。
- [ ] BLE/USB HID 协议栈内部细节。
- [ ] PikaScript 解释器内部实现。
- [ ] RadioLib、TinyGPSPlus 等第三方库源码。
- [ ] Reed-Solomon 等编码算法的数学推导。
- [ ] 图片、字体、音频数组的逐字节内容。
- [ ] 所有键位映射、颜色值和 UI 坐标的细枝末节。

## 阅读问题记录

- [ ] 问题 1：
- [ ] 问题 2：
- [ ] 问题 3：

## 第二遍候选模块

- [ ] 候选 1：
- [ ] 候选 2：
- [ ] 候选 3：
