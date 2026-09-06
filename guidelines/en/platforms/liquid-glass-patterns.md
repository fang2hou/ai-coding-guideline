---
id: platforms/liquid-glass-patterns
lang: en
version: 3
source-lang: en
status: active
digest: 4d335908
---

# Liquid Glass implementation patterns

## Scope

Use these recipes for iOS 27 and native macOS 27 apps built with Xcode 27. Set the matching deployment target to 27.0. SwiftUI is the default; use UIKit on iOS and AppKit on macOS for custom integration. Shared code needs platform separation where APIs differ; consult the [API reference](liquid-glass-api.md). Design decisions follow [Liquid Glass design](liquid-glass-design.md).

## Start with system components

- Use the system design supplied by the 27 SDKs. Use standard navigation, tab bars, toolbars, sheets, and menus without adding another glass background.
- Avoid custom bar backgrounds, borders, and sheet styling that obscure the system material. Keep customization only when the design needs it and validate the result.

## Buttons

Prefer semantic `Button` controls and system styles. Keep destructive roles explicit; prominent styling is for the primary action, not a substitute for a destructive role.

```swift
Button("Save", action: save)
    .buttonStyle(.glassProminent)
Button("Cancel", action: cancel)
    .buttonStyle(.glass)
Button("Delete", role: .destructive, action: delete)
    .buttonStyle(.glass)
```

The action closures are supplied by the app. Use `.glass(.clear)` only under the clear-variant conditions in the design document. UIKit provides `.glass()`, `.prominentGlass()`, `.clearGlass()`, and `.prominentClearGlass()` on `UIButton.Configuration`; AppKit provides the `.glass` button bezel.

## Custom glass

Apply `glassEffect` after sizing and appearance modifiers. Its defaults are `.regular` and a capsule. Add an explicit shape only when the control’s geometry calls for it.

```swift
Text("3 selected")
    .font(.headline)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))
```

`.interactive()` configures the material’s response; it does not supply an action, keyboard activation, or accessibility traits. Keep actionable content in a `Button`. On macOS, use `Glass.interactive(_:)` in SwiftUI or `effectIsInteractive` in AppKit for mouse-optimized responses.

For clear glass, treat the background as part of the design. Apple’s example uses 30% black beneath the effect; adjust it against the actual media and check foreground contrast.

```swift
Label("Flag", systemImage: "flag.fill")
    .padding()
    .glassEffect(.clear)
    .background(.black.opacity(0.3))
```

## Containers, morphing, and unions

Group related custom glass effects in a `GlassEffectContainer` so the system renders them together and can blend their shapes. `spacing` controls when nearby shapes interact; spacing larger than the layout gap can blend them at rest. Containers improve rendering efficiency but do not guarantee a fixed render-pass count.

Use stable, distinct `glassEffectID` values in one namespace for insertion and removal. This complete control keeps button semantics and disables custom motion when Reduce Motion is enabled:

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

Use `.matchedGeometry` for nearby shapes and `.materialize` for transitions without a suitable nearby shape. Apply `glassEffectTransition` after `glassEffect`. For a static union, apply `glassEffectUnion(id:namespace:)` after each effect: members with matching union identifiers, namespaces, shapes, and glass variants combine. A union identifier groups surfaces; it is not an element’s morph identity.

## Tab bars and accessories

- Apply `.tabBarMinimizeBehavior(.onScrollDown)` to the `TabView` when the iPhone layout benefits from minimization. Keep navigation discoverable after collapse.
- Use `.tabViewBottomAccessory { ... }` for persistent controls such as playback. Inside the accessory, read `tabViewBottomAccessoryPlacement` and adapt to `.inline`, `.expanded`, or an undefined (`nil`) placement.
- Use `.searchable` and the search tab role for system search placement. Do not build an overlapping custom search field solely to imitate the system design.

## Toolbars and navigation

- Group items by function with `ToolbarSpacer(.fixed, placement:)`; use `.flexible` when flexible separation is intended. Set `sharedBackgroundVisibility(.hidden)` on toolbar content that supplies its own visual treatment.
- In SwiftUI, conditionally include the `ToolbarItem` to remove it. Hiding only its label can leave the item’s background or space behind. UIKit uses `UIBarButtonItem.isHidden`.
- In iOS 27, use `visibilityPriority` to rank overflow candidates, `ToolbarOverflowMenu` for actions always in overflow, and `.topBarPinnedTrailing` for a persistent trailing action. The last two are unavailable on native macOS; see the platform table.
- `TabRole.prominent` supports a distinct trailing tab in iOS 27. Provide a labeled tab inside `TabView`; the role alone is not a complete tab declaration.

Apply navigation-bar minimization directly on iOS. The conditional compilation below separates the iOS navigation-bar behavior from the native macOS layout:

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

## Scroll edges and background extension

- Keep automatic scroll edge styling first. Use `.scrollEdgeEffectStyle(.hard, for: .top)` only for a deliberately stronger boundary; `nil` restores the default. Recheck explicit overrides when adopting a new OS release; automatic styling can evolve.
- UIKit exposes per-edge effects on `UIScrollView`. Register custom overlays with `UIScrollEdgeElementContainerInteraction` rather than layering a second blur.
- Apply `backgroundExtensionEffect()` to the background artwork before adding text and controls in an overlay. Do not apply it to an entire sidebar full of controls. UIKit and AppKit provide `UIBackgroundExtensionView` and `NSBackgroundExtensionView`.

## UIKit and AppKit integration

For UIKit custom glass, add content to `UIVisualEffectView.contentView` and supply constraints for both the effect view and its content. This factory creates a sized glass label; the caller positions the returned view:

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

- For several nearby UIKit effects, put their effect views inside the `contentView` of a `UIVisualEffectView` configured with `UIGlassContainerEffect`. Its `spacing` controls the interaction distance.
- On macOS, set `NSGlassEffectView.contentView`, `style`, `cornerRadius`, and optional `tintColor`. Group related views with `NSGlassEffectContainerView`. Prefer `NSButton` with the `.glass` bezel for buttons.
- Set AppKit’s `effectIsInteractive = true` for glass that responds to mouse interaction. Check performance and accessibility with the [design validation criteria](liquid-glass-design.md).

## Official references

- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views)
- [Adopting Liquid Glass](https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass)
- [TabViewBottomAccessoryPlacement](https://developer.apple.com/documentation/swiftui/tabviewbottomaccessoryplacement)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Modernize your UIKit app — WWDC26](https://developer.apple.com/videos/play/wwdc2026/278/)
