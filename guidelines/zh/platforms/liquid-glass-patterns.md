---
id: platforms/liquid-glass-patterns
lang: zh
version: 3
source-lang: en
status: active
digest: 1faba719
---

# Liquid Glass 实现模式

## 范围

这些模式用于通过 Xcode 27 构建的 iOS 27 和原生 macOS 27 应用，对应最低系统版本设为 27.0。优先采用 SwiftUI；自定义集成在 iOS 使用 UIKit，在 macOS 使用 AppKit。共享代码须隔离平台专有 API，详见 [API 参考](liquid-glass-api.md)。设计决策遵循 [Liquid Glass 设计](liquid-glass-design.md)。

## 从系统组件开始

- 采用 27 系列 SDK 提供的系统设计。标准导航、标签栏、工具栏、sheet 和菜单无需额外添加玻璃背景。
- 避免用自定义栏背景、边框和 sheet 样式遮盖系统材质。仅保留设计确有需要的定制，并验证效果。

## 按钮

优先使用有语义的 `Button` 和系统样式。破坏性操作应明确指定角色；突出样式用于主要操作，不能替代破坏性角色。

```swift
Button("Save", action: save)
    .buttonStyle(.glassProminent)
Button("Cancel", action: cancel)
    .buttonStyle(.glass)
Button("Delete", role: .destructive, action: delete)
    .buttonStyle(.glass)
```

操作闭包由应用提供。仅在满足设计文档中 clear 的使用条件时采用 `.glass(.clear)`。UIKit 的 `UIButton.Configuration` 提供 `.glass()`、`.prominentGlass()`、`.clearGlass()` 和 `.prominentClearGlass()`；AppKit 提供 `.glass` 按钮边框样式。

## 自定义玻璃

在尺寸和外观修饰符之后应用 `glassEffect`，默认材质为 `.regular`，形状为胶囊形。仅在控件几何形状确有需要时显式指定形状。

```swift
Text("3 selected")
    .font(.headline)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))
```

`.interactive()` 配置材质反馈，不会提供操作、键盘激活或无障碍特征。可操作内容仍应使用 `Button`。macOS 中，SwiftUI 使用 `Glass.interactive(_:)`，AppKit 使用 `effectIsInteractive`，提供针对鼠标优化的反馈。

使用 clear 玻璃时，应把背景一并纳入设计。Apple 示例在效果下方使用 30% 不透明度的黑色；应结合实际媒体内容调整，并检查前景对比度。

```swift
Label("Flag", systemImage: "flag.fill")
    .padding()
    .glassEffect(.clear)
    .background(.black.opacity(0.3))
```

## 容器、变形与合并

相关自定义玻璃效果放入同一个 `GlassEffectContainer`，由系统共同渲染并融合形状。`spacing` 控制邻近形状开始交互的距离；大于布局间距时，形状可能在静止状态就发生融合。容器能改善渲染效率，但不保证固定的渲染次数。

在同一命名空间内使用稳定且不同的 `glassEffectID`，处理元素加入和移除。以下完整控件保留按钮语义，并在启用“减弱动态效果”时关闭自定义动画：

```swift
import SwiftUI

struct FloatingTools: View {
    @Environment(\.accessibilityReduceMotion) private var reduceMotion
    @Namespace private var namespace
    @State private var isExpanded = false
    let mark: () -> Void

    var body: some View {
        GlassEffectContainer(spacing: 24) {
            HStack(spacing: 16) {
                Button(isExpanded ? "Hide tools" : "Show tools",
                       systemImage: "slider.horizontal.3") {
                    withAnimation(reduceMotion ? nil : .smooth) {
                        isExpanded.toggle()
                    }
                }
                .glassEffectID("toggle", in: namespace)

                if isExpanded {
                    Button("Mark", systemImage: "pencil", action: mark)
                        .glassEffectID("mark", in: namespace)
                }
            }
            .buttonStyle(.glass)
            .labelStyle(.iconOnly)
            .controlSize(.large)
        }
    }
}
```

邻近形状适合 `.matchedGeometry`，没有合适邻近形状的过渡使用 `.materialize`。`glassEffectTransition` 放在 `glassEffect` 之后。静态合并使用 `glassEffectUnion(id:namespace:)`，同样放在各自效果之后；合并标识符、命名空间、形状和玻璃变体一致的成员会合为一个表面。合并标识符用于表面分组，与元素的变形身份不同。

## 标签栏与附件视图

- iPhone 布局适合收起标签栏时，对 `TabView` 应用 `.tabBarMinimizeBehavior(.onScrollDown)`。收起后仍须让用户容易找到导航入口。
- 播放控制等持久操作使用 `.tabViewBottomAccessory { ... }`。附件内部读取 `tabViewBottomAccessoryPlacement`，分别适应 `.inline`、`.expanded` 和未定义的 `nil` 状态。
- 使用 `.searchable` 和搜索标签角色，让系统安排搜索入口。不要仅为模仿系统外观而叠加自定义搜索框。

## 工具栏与导航

- 用 `ToolbarSpacer(.fixed, placement:)` 按功能分组；需要弹性间距时使用 `.flexible`。自带视觉样式的工具栏内容可设置 `sharedBackgroundVisibility(.hidden)`。
- SwiftUI 中通过条件分支决定是否包含 `ToolbarItem`。只隐藏标签可能留下项目背景或占位。UIKit 使用 `UIBarButtonItem.isHidden`。
- iOS 27 中，用 `visibilityPriority` 指定溢出顺序，`ToolbarOverflowMenu` 放置始终位于溢出菜单的操作，`.topBarPinnedTrailing` 保留末端操作。后两者在原生 macOS 不可用，详见平台表。
- iOS 27 的 `TabRole.prominent` 支持独立的末端标签。应在 `TabView` 中定义带标签的完整项目，角色本身不是完整的标签声明。

iOS 直接使用导航栏最小化 API。以下条件编译用于区分 iOS 导航栏行为和原生 macOS 布局：

```swift
#if os(iOS)
struct AdaptiveNavigation<Content: View>: View {
    let content: Content

    var body: some View {
        NavigationStack {
            content.toolbarMinimizationBehavior(
                .onScrollDown, for: .navigationBar)
        }
    }
}
#endif
```

## 滚动边缘与背景延展

- 优先保留自动滚动边缘样式。确需更明显的边界时才使用 `.scrollEdgeEffectStyle(.hard, for: .top)`；`nil` 恢复默认。自动样式可能随系统演进，适配新系统时应重新检查显式覆盖。
- UIKit 在 `UIScrollView` 上提供各边缘效果。自定义覆盖层通过 `UIScrollEdgeElementContainerInteraction` 注册，不再叠加第二层模糊。
- 对背景图片应用 `backgroundExtensionEffect()`，再通过覆盖层添加文字和控件。不要把效果应用到包含完整控件的侧边栏。UIKit 和 AppKit 分别提供 `UIBackgroundExtensionView` 和 `NSBackgroundExtensionView`。

## UIKit 与 AppKit 集成

UIKit 自定义玻璃将内容添加到 `UIVisualEffectView.contentView`，并为效果视图和内容提供约束。以下工厂函数创建有尺寸约束的玻璃标签，调用方负责放置返回的视图：

```swift
import UIKit

@MainActor
func makeGlassLabel() -> UIVisualEffectView {
    let glass = UIVisualEffectView(effect: UIGlassEffect(style: .regular))
    let label = UILabel()
    label.text = "3 selected"
    label.font = .preferredFont(forTextStyle: .headline)
    label.adjustsFontForContentSizeCategory = true
    label.translatesAutoresizingMaskIntoConstraints = false
    glass.contentView.addSubview(label)
    NSLayoutConstraint.activate([
        label.leadingAnchor.constraint(equalTo: glass.contentView.leadingAnchor, constant: 16),
        label.trailingAnchor.constraint(equalTo: glass.contentView.trailingAnchor, constant: -16),
        label.topAnchor.constraint(equalTo: glass.contentView.topAnchor, constant: 12),
        label.bottomAnchor.constraint(equalTo: glass.contentView.bottomAnchor, constant: -12)
    ])
    return glass
}
```

- 多个相邻 UIKit 效果放入配置了 `UIGlassContainerEffect` 的 `UIVisualEffectView.contentView`，其 `spacing` 控制交互距离。
- macOS 使用 `NSGlassEffectView` 的 `contentView`、`style`、`cornerRadius` 和可选的 `tintColor`。相关视图用 `NSGlassEffectContainerView` 分组。按钮优先使用 `.glass` 边框样式的 `NSButton`。
- 对需要响应鼠标交互的 AppKit 玻璃设置 `effectIsInteractive = true`。按[设计验收要求](liquid-glass-design.md)检查性能和无障碍。

## 官方资料

- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views)
- [Adopting Liquid Glass](https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass)
- [TabViewBottomAccessoryPlacement](https://developer.apple.com/documentation/swiftui/tabviewbottomaccessoryplacement)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Modernize your UIKit app — WWDC26](https://developer.apple.com/videos/play/wwdc2026/278/)
