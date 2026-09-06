---
id: platforms/liquid-glass-api
lang: zh
version: 2
source-lang: en
status: active
digest: 523ec0cf
---

# Liquid Glass API 参考

## 范围与核验

面向采用 26 系列系统的新应用，27 Beta 新增内容单独列出。核验日期为 2026-09-06，依据 Apple 文档和 Xcode 27 Beta 构建 `27A5252f`（`iPhoneOS27.0.sdk`、`MacOSX27.sdk`）。SDK 声明决定编译时可用性，运行中的系统决定渲染和交互行为。Beta 声明仍可能变化。

下文版本只适用于对应符号，不代表整个框架的所有 API。iOS 包含 iPadOS；Mac Catalyst 需单独核查。visionOS 不提供核心 Liquid Glass 效果，但支持部分布局 API。`if #available` 不能让平台上不存在的 API 变得可用，应使用条件编译或平台专用文件隔离。

## SwiftUI 玻璃 API

以下效果和样式从 iOS/macOS/tvOS/watchOS 26 起可用，visionOS 不可用。导入 `SwiftUI` 即可；部分声明位于其依赖 `SwiftUICore` 中。

| 符号                                                          | 可用性与用途                                            |
| ------------------------------------------------------------- | ------------------------------------------------------- |
| `glassEffect(_:in:)`                                          | 在内容后方添加玻璃；默认 `.regular` 和胶囊形            |
| `Glass.regular` / `.clear` / `.identity`                      | 自适应／清透／无玻璃效果                                |
| `Glass.tint(_:)` / `.interactive(_:)`                         | 配置着色与反馈；macOS 的鼠标反馈在 27 中增强            |
| `GlassEffectContainer(spacing:content:)`                      | 共同渲染相关效果；spacing 控制融合距离                  |
| `glassEffectID(_:in:)`                                        | 命名空间内用于变形的稳定元素身份                        |
| `glassEffectUnion(id:namespace:)`                             | 合并同一命名空间和合并 ID 下的相同形状与变体            |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`、`.materialize` 或 `.identity`       |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | 有语义的按钮样式；tvOS 中这些玻璃样式在未获焦点时也生效 |

重载区别：此 SDK 中 `GlassButtonStyle()` 和 `.glass(_:)` 声明为 26.0 起可用，直接初始化器 `GlassButtonStyle(_:)` 则要求 26.1。应检查实际调用的重载声明。

## SwiftUI 导航与布局

| 符号                                                                   | 可用性与用途                                                           |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                                           | Apple 各平台 26；`.onScrollDown` 行为面向 iPhone                       |
| `ToolbarSpacer(_:placement:)` / `sharedBackgroundVisibility(_:)`       | iOS/macOS 26；tvOS/watchOS/visionOS 不可用                             |
| `scrollEdgeEffectStyle(_:for:)` / `scrollEdgeEffectHidden(_:for:)`     | iOS/macOS/tvOS/watchOS 26；visionOS 虽有样式类型，但不支持这两个修饰符 |
| `backgroundExtensionEffect()`                                          | Apple 各平台 26；将背景内容延展到相邻界面下方                          |
| `tabViewBottomAccessory(content:)` / `tabViewBottomAccessoryPlacement` | iOS 26；附件适应 `.inline`／`.expanded`／`nil`                         |
| `tabViewBottomAccessory(isEnabled:content:)`                           | iOS 26.1；动态显示或隐藏附件                                           |

## UIKit 与 AppKit

| 符号                                                                                                | 可用性与用途                                                                                                       |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `UIGlassEffect(style:)` / `UIGlassContainerEffect`                                                  | iOS 26；在 `UIVisualEffectView` 中创建单个或成组效果，配置 `tintColor` 和 `isInteractive`；不支持 visionOS/watchOS |
| `UIButton.Configuration.glass()` / `.prominentGlass()` / `.clearGlass()` / `.prominentClearGlass()` | iOS/tvOS 26；系统按钮配置                                                                                          |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`                                                  | iOS/tvOS/visionOS 26；通过各边缘效果的 `style` 和 `isHidden` 配置                                                  |
| `UIScrollEdgeElementContainerInteraction` / `UIBackgroundExtensionView`                             | iOS/tvOS/visionOS 26；提供布局支持，不代表平台支持玻璃                                                             |
| `UIBarButtonItem.hidesSharedBackground` / `.isHidden`                                               | 背景属性为 iOS 26；`isHidden` 为 iOS 16                                                                            |
| `UITabBarController.tabBarMinimizeBehavior`                                                         | iOS 26；UIKit 标签栏最小化                                                                                         |
| `NSGlassEffectView` / `NSGlassEffectContainerView`                                                  | macOS 26；通过 `contentView` 和 `spacing` 配置自定义玻璃与分组                                                     |
| `NSButton.BezelStyle.glass` / `NSBackgroundExtensionView`                                           | macOS 26；玻璃按钮／背景延展                                                                                       |
| `NSGlassEffectView.effectIsInteractive`                                                             | macOS 27；鼠标交互反馈                                                                                             |

Mac Catalyst 使用 UIKit，不使用 AppKit。使用本次核验的 SDK，玻璃效果和四种按钮配置均通过 Catalyst 26 目标的类型检查。不能因网页缺少可用性徽标就判定不支持；应检查 Catalyst SDK 声明，并针对目标平台编译实际调用。UIKit 不提供 watchOS 界面框架。

## 27 的新增与变化

| 符号                                                                                       | 可用性与用途                                                                                                     |
| ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `toolbarMinimizationBehavior(_:for:)`                                                      | 27；替代 Beta 名称 `toolbarMinimizeBehavior`；支持 `.navigationBar`                                              |
| `toolbarMinimizationSafeAreaAdjustment(_:for:)` / `toolbarMinimizationRestoration(_:for:)` | 27；安全区调整和恢复策略                                                                                         |
| `UINavigationItem.navigationBarMinimization`                                               | iOS 27；值类型为 `UIBarMinimization`，包含 `minimizationBehavior`、`safeAreaAdjustment` 和 `restorationBehavior` |
| `TabRole.prominent` / `UITabBarController.prominentTabIdentifier`                          | 27／iOS 27；指定突出显示的标签                                                                                   |
| `UITabBarController.Sidebar.preferredPlacement`                                            | iOS 27；空间允许时选择侧边栏布局，检查 `isAvailable`                                                             |
| `ToolbarContent.visibilityPriority(_:)`                                                    | iOS/tvOS/watchOS/visionOS 27、macOS 26.1；溢出优先级                                                             |
| `ToolbarOverflowMenu` / `.topBarPinnedTrailing`                                            | iOS/visionOS 27；macOS/tvOS/watchOS 不可用                                                                       |
| `toolbarColorScheme(_:for:)` / `toolbarVisibility(_:for:)` with `.statusBar`               | 27；状态栏样式和可见性支持                                                                                       |
| `appearsActive`                                                                            | iOS 18/macOS 15 已提供；用于适应新的非活动窗口外观                                                               |
| `UIMenuElement.preferredImageVisibility`                                                   | iOS 27；按需覆盖菜单图标的默认可见性                                                                             |

运行时变化包括玻璃渲染优化、外观滑块、非活动窗口样式和滚动边缘样式。使用 27 系列 SDK 构建还会改变应用要求：UIKit 应用需要场景生命周期和合适的启动屏幕，布局必须适应可调整大小的 iPhone 应用。仅检查玻璃 API 可用性不能满足这些要求。

## 实现检查

- 区分平台可用性、最低系统版本、构建 SDK 和运行时行为。支持 26 时检查 27 API 可用性，并保留可工作的 26 路径。
- 外观修饰符位于 `glassEffect` 之前，身份、合并或过渡修饰符位于之后。相关效果使用容器，并测量实际渲染成本。
- 操作使用有语义的按钮。`ToolbarItem` 没有 `isHidden` 属性；SwiftUI 通过条件分支包含或移除项目。
- 在无障碍设置下验证自定义标签、动画、安全区和背景。系统材质适应不等于应用内容已经通过验证。
- 使用 27 系列 SDK 构建时，`UIDesignRequiresCompatibility` 会被忽略，在 26 系统上运行也如此。它不是新项目的回退方案。
- 按所选 SDK 复核 Beta 名称，WWDC 示例可能保留早期命名。

## 官方文档

- [Glass](https://developer.apple.com/documentation/swiftui/glass)
- [GlassEffectContainer](https://developer.apple.com/documentation/swiftui/glasseffectcontainer)
- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views)
- [UIGlassEffect](https://developer.apple.com/documentation/uikit/uiglasseffect)
- [NSGlassEffectView](https://developer.apple.com/documentation/appkit/nsglasseffectview)
- [iOS and iPadOS 27 release notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Modernize your UIKit app — WWDC26](https://developer.apple.com/videos/play/wwdc2026/278/)
- [UIDesignRequiresCompatibility](https://developer.apple.com/documentation/bundleresources/information-property-list/uidesignrequirescompatibility)
- [Apple engineer clarification of compatibility mode](https://developer.apple.com/forums/thread/838637)

设计见 [Liquid Glass 设计](liquid-glass-design.md)，实现见 [Liquid Glass 实现模式](liquid-glass-patterns.md)。
