---
id: platforms/liquid-glass-design
lang: zh
version: 3
source-lang: en
status: active
digest: aeaf9a0f
---

# Liquid Glass 设计（Apple 平台）

## 采用原则与范围

面向 iOS 27 和原生 macOS 27 新应用，优先通过系统导航和控件采用 Liquid Glass。只有标准组件无法满足交互需求时，才添加自定义玻璃效果。最低系统版本设为 iOS 27.0 或 macOS 27.0，使用 Xcode 27 构建。本指南仅覆盖这两个平台。

27 SDK 行为已于 2026-09-06 对照当前 Beta 核验。采用正式 SDK 或后续更新时，重新核查 Apple HIG、发行说明和 SDK 声明。

## 区分操作层与内容层

- Liquid Glass 用于内容上方的导航和操作区域，包括各类栏、侧边栏、菜单、系统呈现界面和自定义浮动控件。标准组件及其交互状态的材质由系统决定。
- 背景、正文、列表、表格和内容卡片属于内容层，使用不透明表面或标准材质。嵌在内容中的控件可以在交互期间临时呈现玻璃效果。
- 不叠加独立的玻璃表面。已有玻璃表面上的控件使用系统前景样式、填充或鲜明效果。
- 玻璃通过折射、亮度、着色和阴影适应背景。让内容延续到浮动控件下方，不添加仅作装饰的空玻璃面板。

## 选择 regular 或 clear

- 默认使用 `.regular`，尤其适合文字和密集控件。它会调整背景模糊度和亮度，帮助保持可读性。
- `.clear` 仅用于以图片、视频等媒体为主的内容上方，且背景允许压暗、前景内容足够粗且明亮。它不具备 regular 的自适应可读性处理。
- 背景较亮时，HIG 建议以约 35% 不透明度的黑色遮罩压暗；SwiftUI 的 `Glass.clear` 示例使用 30%。这些数值是调试起点，不能保证任何背景下都满足对比度要求。应结合实际前景和动态背景验证。
- 相关元素使用同一变体，不在一组控件内混用 regular 和 clear。

## 着色与前景色

- 玻璃着色用于主要操作或有明确含义的状态，次要操作保持中性。品牌色和表现性配色主要用于内容层。
- 优先使用语义颜色和系统按钮样式。小型玻璃控件的前景外观可能随背景变化而切换，固定黑色或白色标签会妨碍这种适应。
- 选中状态、业务状态和破坏性操作还应通过标签、图标或按钮角色表达，不能只靠颜色区分。

## 形状与同心性

- 优先采用系统控件形状。独立控件适合胶囊形，嵌套的圆角表面应与容器保持同心性。
- 有相应 API 时，让系统计算同心形状。可复用形状应提供备用圆角半径，以便在圆角容器之外使用。
- 遵循屏幕和窗口边距，不直接复制设备圆角半径。内边距、Dynamic Type 或窗口尺寸变化后，重新检查嵌套圆角；macOS 27 也调整了窗口圆角。

## 布局与滚动边缘

- 背景和图片可延伸至窗口边缘，正文和控件仍须遵守相应安全区。不要为了背景铺满而对整个交互视图层级应用 `ignoresSafeArea`。
- 用滚动边缘效果区分滚动内容和浮动栏。默认保留自动样式，仅在布局确有需要时指定柔和或硬边界。同一边缘避免重复效果，相邻窗格的效果高度保持一致。
- 对需要延展到侧边栏或检查器下方的图片、主内容使用背景延展，文字和控件不应出现在延展图像中。
- 根据视图可用尺寸和尺寸类别布局。窗口变窄、键盘出现或文字放大时，重要操作仍须可用。

## 无障碍与验收

- 测试“降低透明度”“增强对比度”“减弱动态效果”和 Dynamic Type。系统材质会适应设置，但自定义前景色、动画和布局仍由应用负责。
- 在浅色和深色外观下，结合实际背景检查文字对比度。Apple HIG 对 17 pt 及以下文字要求 4.5:1，对更大或粗体文字要求 3:1。不要因此缩小文字或把所有标签改为粗体。
- 使用有语义的控件，提供无障碍名称和足够的点击区域。验证 VoiceOver 顺序、键盘操作，以及平台对应的指针或焦点行为。玻璃的视觉反馈不会自动赋予按钮语义。
- 启用“减弱动态效果”时，减少自定义变形和弹簧动画；直接切换状态也是有效的替代方案。
- 验收应覆盖明亮、暗色、复杂和动态背景，小窗口、大字号及非活动窗口。使用代表性内容，在目标设备上测量滚动和过渡性能。

## iOS 27 与 macOS 27 系统行为

- Liquid Glass 调整了扩散、边缘和高光表现。系统外观滑块会改变材质着色，自定义界面必须在整个调节范围内保持可读。
- iPad 和 Mac 的侧边栏延伸到边缘，非活动窗口的视觉区分更明显。需要随窗口活动状态变化的自定义元素使用 `appearsActive`。
- 菜单默认显示更少的图标，仅在图标有助于辨识重要操作时恢复显示。
- iPhone 应用可在 iPhone Mirroring 和 iPad 上调整窗口大小。标准栏会自动适应，自定义布局须在缩放和工具栏溢出时保留操作入口。
- 这些变化分别涉及运行时和 SDK，要求并不相同。通过 [API 参考](liquid-glass-api.md)区分符号可用性和新增运行时行为。

## 官方资料

- [HIG: Materials](https://developer.apple.com/design/human-interface-guidelines/materials)
- [HIG: Color](https://developer.apple.com/design/human-interface-guidelines/color)
- [HIG: Layout](https://developer.apple.com/design/human-interface-guidelines/layout)
- [HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Glass.clear](https://developer.apple.com/documentation/swiftui/glass/clear)
- [Meet Liquid Glass — WWDC25](https://developer.apple.com/videos/play/wwdc2025/219/)
- [Get to know the new design system — WWDC25](https://developer.apple.com/videos/play/wwdc2025/356/)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Platforms State of the Union — WWDC26](https://developer.apple.com/videos/play/wwdc2026/102/)

实现见 [Liquid Glass 实现模式](liquid-glass-patterns.md)，可用性见 [Liquid Glass API 参考](liquid-glass-api.md)。
