---
id: platforms/liquid-glass-design
lang: zh
version: 1
source-lang: en
status: active
digest: d2f0845b
---

# Liquid Glass 设计（苹果平台）

## 结论

Liquid Glass 是 iOS/iPadOS 26+、macOS 26+、tvOS 26+、watchOS 26+ 的系统设计语言，并在 iOS 27 / macOS 27 这一代做了细化。新的苹果平台项目必须采用；本文档是设计任何新 App UI 前的必读内容。它不给存量 pre-iOS-26 App 施加迁移义务——旧 App 改造不在本文档范围内。

先采用系统默认行为，只在系统 chrome 无法表达设计时才做定制。该材质仍在演进（iOS 26.1 时代和 iOS 27 各重调过一次渲染）；把苹果 HIG 当作活的唯一信息源，每次大版本发布时重新核对。

## 双层模型

- Liquid Glass 是一种动态材质：它实时弯曲并汇聚光线（lensing），而不是散射光线。色调、阴影和动态范围随背后内容持续自适应。
- UI 拆成两层：由控件和导航组成的功能层（标签栏、侧边栏、工具栏、导航栏、菜单）浮在内容层之上。内容在玻璃下方滚动，玻璃保证控件可读。内容层是 Liquid Glass 的禁区——它可以是纯不透明，也可以用标准材质，但绝不用玻璃。
- 存在两族材质且职责不互换：Liquid Glass 用于控件/导航层；标准材质（blur、vibrancy、厚度层级）用于内容层内部的结构。
- 小的玻璃元素随背景在明暗间翻转；大表面（菜单、侧边栏）自适应但从不翻转。元素通过调节 lensing 显形（materialize），而不是淡入淡出。

## 玻璃的归属

- 归功能层：导航栏、工具栏、标签栏、侧边栏（iPad/Mac 上为内嵌和浮动形态）、菜单、sheet 和 action sheet、控件的瞬时激活态（slider、toggle 激活时呈现玻璃），以及自定义浮动控件。
- 绝不进内容层——App 背景、卡片、表格/集合内容属于内容层，绝不使用 Liquid Glass（HIG："Don't use Liquid Glass in the content layer"）。唯一例外是内容层控件被激活的瞬时状态。
- 绝不玻璃叠玻璃；元素落在玻璃上时，用填充、透明度或 vibrancy，让它读起来是材质的一部分。
- 绝不把玻璃放在下方没有可折射内容的位置——玻璃属于紧贴内容层的控件，不属于静态内容区。
- 玻璃不是装饰。滚动边缘效果的存在意义是让内容融入浮动 chrome 下方，不是装饰品。

## 变体：regular 与 clear

- regular 是默认项：模糊并调节背景亮度，对任何内容自适应，任何尺寸下都可读。文本密集的 chrome（alert、侧边栏、popover）以及背景可能损害可读性的场合一律用它。
- clear 永久更透明，没有任何自适应可读性行为。只在三个条件全部满足时使用：浮在媒体富内容之上；内容层不会被一层压暗层伤害；玻璃上的内容本身粗体且明亮。
- clear 压着明亮内容时，在玻璃下方加约 35% 黑色不透明度的压暗层；内容本身已暗则跳过。
- 相关元素之间绝不混用 regular 和 clear。

35% 压暗值、三条件测试和着色限制都是苹果当前公布的默认值与场景测试——照做，但在每个大版本重新核对 HIG，不要把它们硬编码成永久参数。

## 着色与颜色

- 玻璃本身没有固有颜色。tint 会把一个颜色映射成随背后亮度变化的一组色调（彩玻璃行为），并换来对比度。
- tint 只留给强调：一个主操作或状态。系统会自动用强调色给 prominent 按钮的玻璃着色。
- 不要给一批控件着色；颜色放进内容层。背景多彩时，栏保持单色，或选一个区分度好的强调色。
- 小型栏上的符号和文字默认单色并随背景翻转——不要强行固定明暗文字颜色去对抗翻转。

## 形状与同心性

- 三种形状类型：fixed（固定圆角半径）、capsule（半径 = 高度一半，天然同心，是栏/slider/switch/分组圆角的默认）、concentric（内半径由父级半径减去内边距推导，由系统计算）。
- 嵌套容器（卡片中的封面、栏中的按钮）必须用同心形状让内半径自动计算。绝不捏出或张开圆角。
- 靠近 iPhone 屏幕边缘时用 capsule 并留出额外边距；iPad/Mac 上与窗口边缘同心对齐。既会嵌套又会独立出现的组件，用带兜底半径的同心形状。
- macOS 27 上所有窗口统一使用更紧的圆角半径——不要假设旧的按窗口区分的半径。

## 布局：边到边

- 背景和全幅视觉内容延伸到显示边缘；可滚动内容延续到浮动 chrome 下方。尊重系统安全区（灵动岛、摄像头区域、栏），并为 iPad 的整个窗口尺寸区间做设计。
- 滚动边缘效果取代浮动玻璃与滚动内容之间的硬分隔线。soft 是 iOS/iPadOS 默认（细微过渡）；hard 主要用于 macOS（文本类控件、无边框控件、固定 header 的均匀不透明边界）。每个视图一个效果，split view 各窗格高度一致，绝不堆叠，没有浮动 UI 的地方不使用。
- 内容不满铺窗口时（侧边栏、inspector 布局），用 background extension 让内容看起来延续到 chrome 下方；文字和控件叠在扩展层之上以免变形。
- iOS 27：内容滚到浮动栏下方时，顶部会浮现统一的工具栏（标准工具栏自动获得）；iPhone App 在 iPad 和 iPhone Mirroring 中变为可调整尺寸——为动态的尺寸和宽高比区间设计，用工具栏重排（溢出菜单、优先级、钉住项）而不是固定布局。

## 无障碍

- 系统组件自动适配，无需主动开启：Reduce Transparency 让玻璃更磨砂、遮蔽更多；Increase Contrast 把元素变成接近黑/白并加对比描边；Reduce Motion 收敛弹性/液态行为。自定义玻璃必须在三个设置加 Dynamic Type 下全部测试。
- 满足对比度下限：17 pt 以下文字 4.5:1，18 pt 及以上或粗体 3:1。明暗两种外观都要验证；开启 Increase Contrast 时提供更高对比的配色方案。
- iOS 27 新增系统级的玻璃外观滑杆（超透明 ↔ 完全着色），macOS 27 增加 "show borders" 无障碍值。绝不假设所有用户、所有系统版本只有一种玻璃渲染。
- Reduce Motion 下的 morph 与液态动效：收紧弹簧、直接跟随手势、优先淡入淡出、避免模糊状态的进出动画。

## iOS 27 的变化

- 材质再次重调：玻璃对复杂背景内容的漫射更好，边缘变暗、镜面高光更亮。已采用 Liquid Glass 的 App 无需重编译即获得新渲染。
- 用户可以全局调整玻璃外观（超透明 ↔ 完全着色）——设计必须容忍整个区间。
- 侧边栏在 iPad 和 Mac 上扩展到屏幕边缘，图标恢复强调色；macOS/iPadOS 上菜单图标默认隐藏（通过 API 呈现关键操作）。
- 窗口呈现明确的非激活外观（自定义视图依据 `appearsActive` 适配）；macOS 上自定义玻璃可交互（为鼠标优化）。
- App 图标渲染更锐利、半透明感降低；Icon Composer 支持多层 Liquid Glass 图标、折射标注，并可预览旧系统效果。

## 该做与不该做

| 该做                                                       | 不该做                                    |
| ---------------------------------------------------------- | ----------------------------------------- |
| 玻璃只给浮动控件和导航                                     | 把玻璃用在内容层视图（表格、背景、卡片）  |
| 默认用 regular，文本密集的 chrome 尤其如此                 | 在可读性或内容层可能受损的场合用 clear    |
| clear 只压媒体富内容，背景亮时加约 35% 压暗层              | 相关元素混用 regular 和 clear             |
| 只给一个主操作的玻璃着色                                   | 给一批控件着色，或给符号着色              |
| 符号保持单色并交给系统自适应                               | 硬编码明/暗文字颜色对抗材质翻转           |
| 用 capsule/同心形状，半径交给系统计算                      | 捏扁、张开或手工计算嵌套圆角              |
| 内容边到边，用滚动边缘效果融合                             | 玻璃叠玻璃；重新加栏背景、描边、分隔线    |
| 每个视图一个滚动边缘效果                                   | 把滚动边缘效果当装饰                      |
| 即使 App 只有一种外观也提供明暗两套颜色                    | 忽略 Increase Contrast 或 iOS 27 外观滑杆 |
| 测试 Reduce Transparency、Increase Contrast、Reduce Motion | 自定义玻璃只在默认设置下测过就上线        |

## 官方参考

- HIG Materials — <https://developer.apple.com/design/human-interface-guidelines/materials>
- HIG Color（Liquid Glass color）— <https://developer.apple.com/design/human-interface-guidelines/color>
- HIG Layout — <https://developer.apple.com/design/human-interface-guidelines/layout>
- HIG Accessibility — <https://developer.apple.com/design/human-interface-guidelines/accessibility>
- HIG 设计原则 — <https://developer.apple.com/design/human-interface-guidelines/design-principles>
- WWDC25-219 Meet Liquid Glass — <https://developer.apple.com/videos/play/wwdc2025/219/>
- WWDC25-356 Get to know the new design system — <https://developer.apple.com/videos/play/wwdc2025/356/>
- WWDC26-102 Platforms State of the Union（iOS 27 细化）— <https://developer.apple.com/videos/play/wwdc2026/102/>
- WWDC26-250 Principles of great design — <https://developer.apple.com/videos/play/wwdc2026/250/>
- Adopting Liquid Glass 技术综述 — <https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass>
- WWDC26 设计指南汇总 — <https://developer.apple.com/wwdc26/guides/design/>
- Apple 设计资源（Figma/Sketch 套件、安全区参考）— <https://developer.apple.com/design/resources/>
- Icon Composer — <https://developer.apple.com/icon-composer/>

实现代码与配方见[Liquid Glass 实现模式](liquid-glass-patterns.md)；符号级细节见[Liquid Glass API 参考](liquid-glass-api.md)。
