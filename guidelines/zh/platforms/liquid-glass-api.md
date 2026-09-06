---
id: platforms/liquid-glass-api
lang: zh
version: 3
source-lang: en
status: active
digest: c43a4acd
---

# Liquid Glass API 参考

## 范围与核验

面向使用 Xcode 27 构建的 iOS 27 和原生 macOS 27 应用，最低系统版本均设为 27.0。核验日期为 2026-09-06，依据 Apple 文档和 Xcode 27 Beta 构建 `27A5252f`（`iPhoneOS27.0.sdk`、`MacOSX27.sdk`）。SDK 声明决定平台可用性，运行中的系统决定渲染和交互行为。Beta 声明仍可能变化。

下表只说明这两个平台的支持情况，不记录历史引入版本。iOS 使用 UIKit，原生 macOS 使用 AppKit。平台专有 API 通过 `#if os(iOS)`／`#if os(macOS)` 或平台专用文件隔离。

## SwiftUI 玻璃 API

本节效果和样式在 iOS 27 与 macOS 27 均可用。导入 `SwiftUI` 即可；部分声明位于其依赖 `SwiftUICore` 中。

| 符号                                                          | 可用性与用途                                      |
| ------------------------------------------------------------- | ------------------------------------------------- |
| `glassEffect(_:in:)`                                          | 在内容后方添加玻璃；默认 `.regular` 和胶囊形      |
| `Glass.regular` / `.clear` / `.identity`                      | 自适应／清透／无玻璃效果                          |
| `Glass.tint(_:)` / `.interactive(_:)`                         | 配置着色与交互反馈，包括 macOS 鼠标交互           |
| `GlassEffectContainer(spacing:content:)`                      | 共同渲染相关效果；spacing 控制融合距离            |
| `glassEffectID(_:in:)`                                        | 命名空间内用于变形的稳定元素身份                  |
| `glassEffectUnion(id:namespace:)`                             | 合并同一命名空间和合并 ID 下的相同形状与变体      |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`、`.materialize` 或 `.identity` |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | 系统玻璃按钮样式；保留操作语义与角色              |

## SwiftUI 导航与布局

| 符号                                                                   | 可用性与用途                                |
| ---------------------------------------------------------------------- | ------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                                           | iOS：滚动时收起 iPhone 标签栏               |
| `ToolbarSpacer(_:placement:)` / `sharedBackgroundVisibility(_:)`       | 两者：工具栏分组与背景控制                  |
| `scrollEdgeEffectStyle(_:for:)` / `scrollEdgeEffectHidden(_:for:)`     | 两者：按滚动边缘配置样式或可见性            |
| `backgroundExtensionEffect()`                                          | 两者：将背景内容延展到相邻界面下方          |
| `tabViewBottomAccessory(content:)` / `tabViewBottomAccessoryPlacement` | iOS：附件适应 `.inline`／`.expanded`／`nil` |
| `tabViewBottomAccessory(isEnabled:content:)`                           | iOS：动态显示或隐藏附件                     |

## UIKit 与 AppKit

| 符号                                                                                                | 可用性与用途                                                                           |
| --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `UIGlassEffect(style:)` / `UIGlassContainerEffect`                                                  | iOS：在 `UIVisualEffectView` 中创建单个或成组效果，配置 `tintColor` 和 `isInteractive` |
| `UIButton.Configuration.glass()` / `.prominentGlass()` / `.clearGlass()` / `.prominentClearGlass()` | iOS：系统按钮配置                                                                      |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`                                                  | iOS：配置各边缘效果的 `style` 和 `isHidden`                                            |
| `UIScrollEdgeElementContainerInteraction` / `UIBackgroundExtensionView`                             | iOS：注册自定义覆盖层／延展背景                                                        |
| `UIBarButtonItem.hidesSharedBackground` / `.isHidden`                                               | iOS：隐藏共享背景／项目本身                                                            |
| `UITabBarController.tabBarMinimizeBehavior`                                                         | iOS：UIKit 标签栏最小化                                                                |
| `NSGlassEffectView` / `NSGlassEffectContainerView`                                                  | macOS：通过 `contentView` 和 `spacing` 配置自定义玻璃与分组                            |
| `NSButton.BezelStyle.glass` / `NSBackgroundExtensionView`                                           | macOS：玻璃按钮／背景延展                                                              |
| `NSGlassEffectView.effectIsInteractive`                                                             | macOS：鼠标交互反馈                                                                    |

## 自适应导航与窗口行为

| 符号                                                                                       | 可用性与用途                                                                                          |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `toolbarMinimizationBehavior(_:for:)`                                                      | iOS 导航栏：最小化策略；使用当前名称，不使用 Beta 旧名 `toolbarMinimizeBehavior`                      |
| `toolbarMinimizationSafeAreaAdjustment(_:for:)` / `toolbarMinimizationRestoration(_:for:)` | iOS 导航栏：安全区调整和恢复策略                                                                      |
| `UINavigationItem.navigationBarMinimization`                                               | iOS：`UIBarMinimization` 值包含 `minimizationBehavior`、`safeAreaAdjustment` 和 `restorationBehavior` |
| `TabRole.prominent` / `UITabBarController.prominentTabIdentifier`                          | iOS：指定突出显示的标签                                                                               |
| `UITabBarController.Sidebar.preferredPlacement`                                            | iOS：空间允许时选择侧边栏布局，检查 `isAvailable`                                                     |
| `ToolbarContent.visibilityPriority(_:)`                                                    | 两者：工具栏溢出优先级                                                                                |
| `ToolbarOverflowMenu` / `.topBarPinnedTrailing`                                            | 仅 iOS；原生 macOS 不可用                                                                             |
| `toolbarColorScheme(_:for:)` / `toolbarVisibility(_:for:)` with `.statusBar`               | iOS：状态栏样式和可见性                                                                               |
| `appearsActive`                                                                            | 两者：自定义内容适应窗口活动状态                                                                      |
| `UIMenuElement.preferredImageVisibility`                                                   | iOS：按需覆盖菜单图标的默认可见性                                                                     |

界面验收应覆盖外观滑块、非活动窗口样式和滚动边缘。UIKit 应用必须使用场景生命周期并提供启动屏幕。布局须适应可调整大小的 iPhone 窗口，保留重要操作入口。

## 实现检查

- 分别针对 iOS 27 和 macOS 27 构建并做类型检查。共享 SwiftUI 代码仍须隔离导航、工具栏和框架专有视图的平台差异。
- 外观修饰符位于 `glassEffect` 之前，身份、合并或过渡修饰符位于之后。相关效果使用容器，并测量实际渲染成本。
- 操作使用有语义的按钮。`ToolbarItem` 没有 `isHidden` 属性；SwiftUI 通过条件分支包含或移除项目。
- 在无障碍设置下验证自定义标签、动画、安全区和背景。系统材质适应不等于应用内容已经通过验证。
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

设计见 [Liquid Glass 设计](liquid-glass-design.md)，实现见 [Liquid Glass 实现模式](liquid-glass-patterns.md)。
