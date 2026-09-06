---
id: platforms/liquid-glass-patterns
lang: zh
version: 1
source-lang: en
status: active
digest: fc61f492
---

# Liquid Glass 实现模式

## 结论

面向任务的实现配方，SwiftUI 优先，UIKit 和 AppKit 按适用场景给出。基线是 iOS 26 / macOS 26（整套玻璃 API 从该版本起可用）；iOS 27 专属 API 已标注，需要可用性门控。代码片段遵循苹果文档给出的模式；签名已对照 Xcode 27 SDK 核验。设计规则见[Liquid Glass 设计](liquid-glass-design.md)；符号细节见[Liquid Glass API 参考](liquid-glass-api.md)。

## 先采用系统默认

- 标准 chrome——导航栏、标签栏、工具栏、侧边栏、sheet、菜单——在 iOS 26+ 上自动获得 Liquid Glass。不要为它写任何玻璃代码。
- 删掉与系统外观冲突的旧定制：栏背景视图、阴影与描边、分隔线逻辑，以及 sheet 上的 `presentationBackground`。
- 移除自定义的搜索栏和附件样式；持久功能才用 accessory view，栏内项目按功能和使用频率分组。
- `UIDesignRequiresCompatibility`（Info.plist）可恢复旧外观——那是真正不兼容设计的最后手段，绝不是新项目的默认选项。

## 按钮

- 优先用系统玻璃按钮样式，而不是把按钮包进裸玻璃效果：

```swift
Button("Save") { save() }
    .buttonStyle(.glass)             // 标准玻璃
Button("Delete") { delete() }
    .buttonStyle(.glassProminent)    // 强调色着色的主操作
Button("Filter") { toggleFilters() }
    .buttonStyle(.glass(.clear))     // 仅用于媒体富内容之上
```

- UIKit：`UIButton.Configuration`——`.glass()`、`.prominentGlass()`、`.clearGlass()`、`.prominentClearGlass()`。AppKit：`NSButton` 的 `.glass` bezel 样式。
- 每个表面最多一个着色的主操作；其余保持无色玻璃（着色规则见设计文档）。

## 自定义玻璃视图

- 对自定义浮动控件施加 `glassEffect`，顺序放在外观类修饰符之后——它会捕获视图内容用于渲染：

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect()                    // 默认 capsule 形状、.regular
```

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))          // 自定义形状
```

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect(.regular.tint(.orange).interactive()) // 着色 + 触摸反馈
```

- `.interactive()` 让自定义玻璃获得系统的触摸/指针反应；macOS 上需要 macOS 27（AppKit 侧为 `NSGlassEffectView.effectIsInteractive`）。
- clear 只用于媒体富内容之上，背景亮时加压暗层：

```swift
Label("Flag", systemImage: "flag.fill")
    .padding()
    .glassEffect(.clear)
    .background(.black.opacity(0.3))
```

## 用 GlassEffectContainer 融合

- 相邻的玻璃元素必须共享一个 `GlassEffectContainer`——它把整组放进一次渲染，并让它们融合与变形。没有它，相邻效果采样不一致。

```swift
GlassEffectContainer(spacing: 40) {
    HStack(spacing: 40) {
        toolButton("pencil")
        toolButton("eraser")
    }
}
```

- `spacing` 越大，越早开始融合。容器 spacing 超过布局间距时，形状在静止态就会融合。每个功能区一个容器，并控制屏幕上的效果数量——每层玻璃都有渲染开销。

## 变形与合并

- 给每个玻璃元素一个稳定身份，让它在层级变化间变形；动画驱动状态切换：

```swift
@Namespace private var ns
@State private var isExpanded = false

GlassEffectContainer(spacing: 40) {
    HStack(spacing: 40) {
        toolIcon("scribble.variable")
            .glassEffect()
            .glassEffectID("pencil", in: ns)
        if isExpanded {
            toolIcon("eraser.fill")
                .glassEffect()
                .glassEffectID("eraser", in: ns)
        }
    }
}
// withAnimation { isExpanded.toggle() }
```

- `.matchedGeometry`（容器 spacing 距离内的默认项）平滑变形；距离更远时用 `glassEffectTransition` 切到 `.materialize`。默认动画下 matchedGeometry 会附加缩放/位移效果——传入显式动画（`.spring` 或 `nil`）可退出。
- `glassEffectUnion(id:namespace:)` 把多个元素在静止态合并为一个共享形状；所有成员必须形状相同且 `Glass` 变体相同。

## 标签栏与工具栏行为

- 浮动标签栏的最小化（仅 iPhone）：`.tabBarMinimizeBehavior(.onScrollDown)`。
- 用 `ToolbarSpacer(.fixed)` / `.flexible` 把工具栏项目拆成独立的玻璃分组；用 `sharedBackgroundVisibility(.hidden)` 去掉某个项目的共享玻璃（如头像）。
- iOS 27 新增工具栏韧性 API——需要可用性门控：

```swift
StickerPageView().toolbar {
    ToolbarItemGroup { UndoButton(); RedoButton() }
        .visibilityPriority(.high)                 // 最晚进溢出菜单
    ToolbarOverflowMenu {                          // 永远在溢出菜单
        ChoosePhotoButton(); ExportButton()
    }
    ToolbarItem(placement: .topBarPinnedTrailing) { ShareButton() }
}
ScrollView { content }
    .toolbarMinimizationBehavior(.onScrollDown, for: .navigationBar) // iOS 27 改名
Tab(role: .prominent) { CartTab() }                // 钉在尾部的 tab
```

- UIKit 对应物：`UITabBarController.tabBarMinimizeBehavior = .onScrollDown`；`UIBarButtonItem.hidesSharedBackground = true`；iOS 27 的 `UINavigationItem.navigationBarMinimization`。

## 滚动边缘效果

- 调整内容在浮动 chrome 下方的消隐方式：`.scrollEdgeEffectStyle(.soft, for: .top)`（iOS 默认）、`.hard`（不透明边界，主要用于 macOS）、`nil` 恢复系统默认。每个视图一个效果。
- UIKit：`scrollView.topEdgeEffect.style = .hard`、`.isHidden = true`；滚动视图上的自定义浮层用 `UIScrollEdgeElementContainerInteraction` 注册，不要自建模糊。
- iOS 27 把 `.automatic` 改为独立视觉（不再在 soft/hard 之间切换）——采用 27 SDK 时重新评估显式的 `.soft` 覆盖。

## 背景延展

- 内容不满铺窗口时（侧边栏、inspector 布局），让它在视觉上延展到 chrome 下方：`.backgroundExtensionEffect()`。节制使用——背景实例只放一个；文字和控件叠在延展层之上。

```swift
SidebarContent()
    .backgroundExtensionEffect()
```

- UIKit：`UIBackgroundExtensionView`；AppKit：`NSBackgroundExtensionView`。

## UIKit 模式

- 通过 `UIVisualEffectView` + `UIGlassEffect` 实现自定义玻璃；子视图加到 `contentView`，绝不加到 effect view 本身：

```swift
let effect = UIGlassEffect(style: .clear)   // 或 .regular
effect.tintColor = .systemBlue
effect.isInteractive = true
let glassView = UIVisualEffectView(effect: effect)
glassView.contentView.addSubview(label)     // contentView，不是 glassView
```

- 多个相邻玻璃视图：把 `UIGlassContainerEffect`（其 `spacing` 决定融合距离）装进一个 `UIVisualEffectView`，再把各个玻璃 effect view 嵌进它的 `contentView`。
- 隐藏工具栏项目时隐藏项目本身（`ToolbarItem`/`UIBarButtonItem` 的 `isHidden`），而不是它的内容视图。

## AppKit 模式

- macOS 26+ 的自定义玻璃容器：`NSGlassEffectView`——`contentView` 是唯一保证落在玻璃内的放置位置；设置 `cornerRadius`、`tintColor`、`style`。
- macOS 27：`effectIsInteractive = true` 为包含或承托控件的玻璃启用交互反应。
- 用 `NSGlassEffectContainerView`（`spacing`，默认 0）分组相邻玻璃以合并渲染。按钮用 `.glass` bezel 样式，不手工包玻璃。
