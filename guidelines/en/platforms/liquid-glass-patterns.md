---
id: platforms/liquid-glass-patterns
lang: en
version: 1
source-lang: en
status: active
digest: 75bf42cb
---

# Liquid Glass implementation patterns

## Verdict

Task-oriented recipes for building with Liquid Glass, SwiftUI first, UIKit and AppKit where they apply. Baseline is iOS 26 / macOS 26 (the whole glass surface ships there); iOS 27-only APIs are marked and need availability gating. Snippets follow Apple's documented patterns; signatures were verified against the Xcode 27 SDK. Design rules live in [Liquid Glass design](liquid-glass-design.md); symbol details in [Liquid Glass API reference](liquid-glass-api.md).

## Adopt the system defaults first

- Standard chrome — navigation bars, tab bars, toolbars, sidebars, sheets, menus — gets Liquid Glass automatically on iOS 26+. Write no glass code for it.
- Delete the old customizations that fight the system look: bar background views, shadows and borders, divider logic, and `presentationBackground` on sheets.
- Remove custom search-bar and accessory styling; use accessory views for persistent features only, and group bar items by function and frequency of use.
- `UIDesignRequiresCompatibility` (Info.plist) restores the legacy look — a last resort for genuinely incompatible designs, never a default for new projects.

## Buttons

- Prefer system glass button styles over wrapping buttons in a raw glass effect:

```swift
Button("Save") { save() }
    .buttonStyle(.glass)             // standard glass
Button("Delete") { delete() }
    .buttonStyle(.glassProminent)    // accent-tinted, the primary action
Button("Filter") { toggleFilters() }
    .buttonStyle(.glass(.clear))     // over media-rich content only
```

- UIKit: `UIButton.Configuration` — `.glass()`, `.prominentGlass()`, `.clearGlass()`, `.prominentClearGlass()`. AppKit: `NSButton` bezel style `.glass`.
- One tinted prominent action per surface; the rest stay untinted glass (see the design document's tinting rules).

## Custom glass views

- Apply `glassEffect` to floating custom controls, ordered after appearance modifiers — it captures the view's content for rendering:

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect()                    // capsule shape, .regular by default
```

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))          // custom shape
```

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect(.regular.tint(.orange).interactive()) // tint + touch feedback
```

- `.interactive()` opts a custom glass element into the system's touch/pointer reactions; on macOS it requires macOS 27 (`NSGlassEffectView.effectIsInteractive` in AppKit).
- Use Clear only over media-rich content, with a dimming layer when the background is bright:

```swift
Label("Flag", systemImage: "flag.fill")
    .padding()
    .glassEffect(.clear)
    .background(.black.opacity(0.3))
```

## Blending with GlassEffectContainer

- Nearby glass elements must share a `GlassEffectContainer` — it renders them in one pass and lets them blend and morph. Without it, neighboring effects sample inconsistently.

```swift
GlassEffectContainer(spacing: 40) {
    HStack(spacing: 40) {
        toolButton("pencil")
        toolButton("eraser")
    }
}
```

- Larger `spacing` starts blending sooner. If container spacing exceeds layout spacing, shapes blend at rest. Keep one container per functional group and limit the number of on-screen effects — every glass layer costs render passes.

## Morphing and unions

- Give each glass element a stable identity to morph across hierarchy changes; animate the state change:

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

- `.matchedGeometry` (the default inside spacing distance) morphs smoothly; farther apart, switch the element to `.materialize` via `glassEffectTransition`. Under the default animation matchedGeometry adds scale/offset flourishes — pass an explicit animation (`.spring`, or `nil`) to opt out.
- `glassEffectUnion(id:namespace:)` merges several elements into one shared shape at rest; all members must share the same shape and the same `Glass` variant.

## Tab bar and toolbar behavior

- Floating tab bar minimization (iPhone only): `.tabBarMinimizeBehavior(.onScrollDown)`.
- Group toolbar items into separate glass bubbles with `ToolbarSpacer(.fixed)` / `.flexible`; drop an item's shared glass with `sharedBackgroundVisibility(.hidden)` (e.g. a profile photo).
- iOS 27 adds toolbar resilience APIs — gate them on availability:

```swift
StickerPageView().toolbar {
    ToolbarItemGroup { UndoButton(); RedoButton() }
        .visibilityPriority(.high)                 // overflow last
    ToolbarOverflowMenu {                          // always in overflow
        ChoosePhotoButton(); ExportButton()
    }
    ToolbarItem(placement: .topBarPinnedTrailing) { ShareButton() }
}
ScrollView { content }
    .toolbarMinimizationBehavior(.onScrollDown, for: .navigationBar) // iOS 27 rename
Tab(role: .prominent) { CartTab() }                // pinned trailing tab
```

- UIKit equivalents: `UITabBarController.tabBarMinimizeBehavior = .onScrollDown`; `UIBarButtonItem.hidesSharedBackground = true`; iOS 27 `UINavigationItem.navigationBarMinimization`.

## Scroll edge effects

- Tune how content dissolves under floating chrome: `.scrollEdgeEffectStyle(.soft, for: .top)` (default on iOS), `.hard` (opaque boundary, mostly macOS), or `nil` for the system default. One effect per view.
- UIKit: `scrollView.topEdgeEffect.style = .hard`, `.isHidden = true`; register custom overlays over a scroll view with `UIScrollEdgeElementContainerInteraction` instead of building custom blurs.
- iOS 27 changed `.automatic` to its own visuals (it no longer alternates soft/hard) — re-evaluate any explicit `.soft` override when building with the 27 SDK.

## Background extension

- When content does not span the window (sidebar, inspector layouts), extend it visually under the chrome: `.backgroundExtensionEffect()`. Use sparingly — one background instance; keep text and controls above the extension.

```swift
SidebarContent()
    .backgroundExtensionEffect()
```

- UIKit: `UIBackgroundExtensionView`; AppKit: `NSBackgroundExtensionView`.

## UIKit patterns

- Custom glass through `UIVisualEffectView` + `UIGlassEffect`; subviews go to `contentView`, never the effect view:

```swift
let effect = UIGlassEffect(style: .clear)   // or .regular
effect.tintColor = .systemBlue
effect.isInteractive = true
let glassView = UIVisualEffectView(effect: effect)
glassView.contentView.addSubview(label)     // contentView, not glassView
```

- Multiple nearby glass views: host a `UIGlassContainerEffect` (its `spacing` sets the merge distance) in a `UIVisualEffectView` and nest the individual glass effect views in its `contentView`.
- Hide a toolbar item by hiding the item (`isHidden` on the `ToolbarItem`/`UIBarButtonItem`), not its content view.

## AppKit patterns

- Custom glass container on macOS 26+: `NSGlassEffectView` — `contentView` is the only guaranteed in-glass placement; set `cornerRadius`, `tintColor`, `style`.
- macOS 27: `effectIsInteractive = true` enables interactive glass reactions for glass containing or sitting behind controls.
- Group nearby glass with `NSGlassEffectContainerView` (`spacing`, default 0) to merge render passes. Buttons use the `.glass` bezel style, not hand-wrapped glass.
