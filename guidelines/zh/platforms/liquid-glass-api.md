---
id: platforms/liquid-glass-api
lang: zh
version: 1
source-lang: en
status: active
digest: 86327ae3
---

# Liquid Glass API 参考

## 范围与核验

Liquid Glass 相关 API 的符号级参考，已对照 Xcode 27 SDK 接口（`iPhoneOS27.0.sdk`、`MacOSX27.sdk`）和苹果文档页面核验（2026-09-06）。基线：SwiftUI/UIKit/AppKit 各节所列符号全部随 iOS 26 / macOS 26 / tvOS 26 / watchOS 26 发布；「iOS 27 新增」一节要求 27.0 起步并做可用性门控。

没有向后部署：需要支持 iOS 25 及更早版本的 App 必须用 `if #available(iOS 26.0, *)`（或 `#available(macOS 26.0, *)`）门控玻璃代码并提供回退。visionOS 不采用 Liquid Glass——核心玻璃符号均为 `@available(visionOS, unavailable)`，该平台保留自己的设计语言。设计规则见[Liquid Glass 设计](liquid-glass-design.md)；配方见[Liquid Glass 实现模式](liquid-glass-patterns.md)。

## SwiftUI：核心玻璃 API

| 符号                                                          | 用途                                                        |
| ------------------------------------------------------------- | ----------------------------------------------------------- |
| `glassEffect(_:in:)`                                          | 在视图后方渲染玻璃形状；默认 capsule + `.regular`           |
| `Glass`                                                       | 材质配置值：`.regular`、`.clear`、`.identity`               |
| `Glass.tint(_:)` / `Glass.interactive(_:)`                    | 着色变体 / 系统触摸与指针反应                               |
| `GlassEffectContainer(spacing:content:)`                      | 一组玻璃合并为一次渲染；启用融合与变形                      |
| `glassEffectID(_:in:)`                                        | 容器内变形所需的稳定身份，配合 `Namespace`                  |
| `glassEffectUnion(id:namespace:)`                             | 多个元素静止态合并为一个形状（仅限同形状同变体）            |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`（近距）或 `.materialize`（远距）增删过渡 |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | 系统玻璃按钮样式——优先于裸玻璃效果                          |

SwiftUI 框架；iOS/iPadOS/macCatalyst/macOS/tvOS/watchOS 26.0+——visionOS 全部不可用（该平台没有这种材质）；tvOS 无论焦点状态都应用玻璃。

## SwiftUI：chrome 行为 API

| 符号                                               | 用途                                                                                    |
| -------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                       | 浮动标签栏随滚动最小化；`.onScrollDown` 仅 iPhone 可用                                  |
| `ToolbarSpacer(.fixed/.flexible, placement:)`      | 把工具栏项目拆成独立玻璃分组（仅 iOS/macOS）                                            |
| `sharedBackgroundVisibility(_:)`（ToolbarContent） | 去掉某项目的共享玻璃背景（自成一组；仅 iOS/macOS）                                      |
| `scrollEdgeEffectStyle(_:for:)`                    | 按边设置 `.automatic` / `.soft` / `.hard`；`nil` 恢复系统默认。修饰符在 visionOS 不可用 |
| `scrollEdgeEffectHidden(_:for:)`                   | 移除指定边的边缘效果（visionOS 同样缺失）                                               |
| `backgroundExtensionEffect()`                      | 内容在视觉上延展到侧边栏/inspector 下方                                                 |
| `tabViewBottomAccessory(content:)`                 | 标签栏上方的持久附件，随栏一起收起（仅 iOS 系平台）                                     |

SwiftUI 框架；iOS 26.0+ 基线，平台差异按行标注（`tabBarMinimizeBehavior`、`backgroundExtensionEffect` 以及 `ScrollEdgeEffectStyle` 类型本身在 visionOS 也可用）。

## UIKit API

| 符号                                                        | 用途                                                          |
| ----------------------------------------------------------- | ------------------------------------------------------------- |
| `UIGlassEffect(style:)`                                     | 供 `UIVisualEffectView` 使用的玻璃材质；`.regular` / `.clear` |
| `UIGlassEffect.tintColor` / `.isInteractive`                | 着色；交互式触摸反馈                                          |
| `UIGlassContainerEffect`（`spacing`）                       | 容器效果：嵌套的玻璃视图渲染为一个合并表面                    |
| `UIButton.Configuration.glass()`（含 prominent/clear 变体） | 玻璃按钮配置（Mac Catalyst 无）                               |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`          | 按边的 `UIScrollEdgeEffect`（`style`、`isHidden`）            |
| `UIScrollEdgeElementContainerInteraction`                   | 注册自定义浮层以参与滚动边缘效果的形状计算                    |
| `UIBarButtonItem.hidesSharedBackground`                     | 让某个栏项目退出共享玻璃背景                                  |
| `UITabBarController.tabBarMinimizeBehavior`                 | UIKit 侧的标签栏最小化                                        |
| `UIBackgroundExtensionView`                                 | 背景延展容器                                                  |

UIKit；iOS/iPadOS/macCatalyst/tvOS 26.0+（玻璃效果与按钮配置在 visionOS/watchOS 不可用；按钮配置在 Catalyst 上也缺失）。

## AppKit API

| 符号                                      | 用途                                                          |
| ----------------------------------------- | ------------------------------------------------------------- |
| `NSGlassEffectView`                       | 玻璃容器：`contentView`、`cornerRadius`、`tintColor`、`style` |
| `NSGlassEffectContainerView`（`spacing`） | 合并相邻玻璃视图，减少渲染次数                                |
| `NSButton.BezelStyle.glass`               | 玻璃 bezel——优先于按钮后面的自定义玻璃                        |
| `NSBackgroundExtensionView`               | 背景延展容器（titlebar/侧边栏/inspector）                     |
| `NSGlassEffectView.effectIsInteractive`   | macOS 上的交互式玻璃——仅 27.0+                                |

AppKit；macOS 26.0+，`effectIsInteractive` 除外（27.0+）。

## iOS 27 新增

已在 Xcode 27 SDK 和 iOS 27 release notes 中核验；用 `#available(iOS 27, *)` / `macOS 27` 门控：

- `toolbarMinimizationBehavior(_:for:)`（及 `toolbarMinimizationSafeAreaAdjustment`）——重命名并扩展 `toolbarMinimizeBehavior`；支持的 placement 是 `.navigationBar`。
- `UINavigationItem.navigationBarMinimization`（`UIBarMinimizationBehavior`：`.automatic/.never/.onScrollDown/.onScrollUp`）——取代 beta 期的 `barMinimizeBehavior` 命名。WWDC26 示例代码可能还是旧名字。
- `Tab(role: .prominent)` / `TabRole.prominent` 与 `UITabBarController.prominentTabIdentifier`——钉在尾部边缘的 prominent tab。
- `UITabBarController.Sidebar.preferredPlacement`（`.sidebar/.tabBar/.automatic`）——空间允许时 iPhone 上的标签栏侧边栏形态。
- `ToolbarOverflowMenu` 与 `ToolbarContent.visibilityPriority(_:)`——空间不足时工具栏重排。
- `ToolbarItemPlacement.topBarPinnedTrailing`——无论重排都钉在尾部边缘的项目。
- `toolbarColorScheme(_:for: .statusBar)` / `toolbarVisibility(_:for: .statusBar)`——状态栏控制。
- 环境值 `appearsActive`——iOS 18 起就已存在；iOS 27 让非激活窗口有了明确外观，自定义 chrome 现在应依据它适配（并非新符号）。
- `NSGlassEffectView.effectIsInteractive`（macOS 27）与 `UIMenuElement.preferredImageVisibility`（菜单图标默认隐藏）。
- 无新 API 的运行时变化：材质重调（漫射、暗边、镜面高光）、用户外观滑杆（超透明 ↔ 完全着色）、内容滚入浮动栏下方时顶部统一工具栏、iPad/Mac 侧边栏到边并恢复强调色图标、macOS 统一更紧的窗口圆角、iPhone App 可调整尺寸。

## 开发注意事项

1. 每个功能区一个 `GlassEffectContainer`，控制屏幕上的效果数量——每层玻璃都有渲染开销；容器外的玻璃采样也不一致（玻璃无法采样玻璃）。
2. `glassEffect` 会捕获视图内容——放在外观类修饰符之后。
3. 变形规则：`.matchedGeometry` 只在容器 `spacing` 距离内生效；超出用 `.materialize`。默认动画下 matchedGeometry 附加缩放/位移效果——传显式动画退出。`glassEffectUnion` 要求各成员形状与 `Glass` 变体完全一致。
4. `.clear` 压明亮内容时需要压暗层（苹果示例：`.background(.black.opacity(0.3))`），且与 `.regular` 绝不在相关元素间混用。
5. 无障碍自适应只覆盖系统组件——自定义玻璃要在 Reduce Transparency、Increase Contrast、Reduce Motion 和 iOS 27 外观滑杆下测试。
6. 需要门控的平台缺口：核心玻璃符号在 visionOS 不可用（该平台保留自己的设计语言），但 watchOS 26 可用；`scrollEdgeEffectStyle`/`scrollEdgeEffectHidden` 修饰符在 visionOS 不可用（尽管 `ScrollEdgeEffectStyle` 类型存在）；按苹果可用性元数据，`UIButton.Configuration.glass()` 系列在 Mac Catalyst 缺失（Catalyst 代码用 `UIGlassEffect`，它在该平台可用）；原生 macOS 用 `NSGlassEffectView`。
7. 隐藏工具栏项目走项目本身（`ToolbarItem`/`UIBarButtonItem` 的 `isHidden`），绝不走它的内容视图。
8. iOS 27 的 `.automatic` 滚动边缘效果有独立视觉（不再在 soft/hard 间切换）——采用 27 SDK 时重新评估显式 `.soft` 覆盖。
9. 用 iOS 27 SDK 构建会让 iPhone App 变为可调整尺寸，并强制要求 UIScene 生命周期和 launch screen 键——新项目用当前模板默认即满足。
10. `UIDesignRequiresCompatibility`（Info.plist）恢复玻璃前外观；视为最后手段，绝不作为新项目默认。

## 官方文档索引

- Adopting Liquid Glass — <https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass>
- Applying Liquid Glass to custom views — <https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views>
- Liquid Glass 概述 — <https://developer.apple.com/documentation/TechnologyOverviews/liquid-glass>
- 示例：Landmarks — building an app with Liquid Glass — <https://developer.apple.com/documentation/swiftui/landmarks-building-an-app-with-liquid-glass>
- WWDC25-323 Build a SwiftUI app with the new design — <https://developer.apple.com/videos/play/wwdc2025/323/>
- WWDC26-269 What's new in SwiftUI — <https://developer.apple.com/videos/play/wwdc2026/269/>
- WWDC26-278 Modernize your UIKit app — <https://developer.apple.com/videos/play/wwdc2026/278/>
- iOS & iPadOS 27 release notes — <https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes>
