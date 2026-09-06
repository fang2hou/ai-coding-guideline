---
id: platforms/liquid-glass-api
lang: en
version: 2
source-lang: en
status: active
digest: f20ba3e6
---

# Liquid Glass API reference

## Scope and verification

Reference for new apps using the 26 release family, with 27 beta additions separated below. Verified on 2026-09-06 against Apple documentation and Xcode 27 beta build `27A5252f` (`iPhoneOS27.0.sdk`, `MacOSX27.sdk`). SDK declarations establish compile-time availability; the running OS determines rendering and interaction behavior. Beta declarations may change.

Versions below apply to the named symbol, not every API in its framework. iOS includes iPadOS; check Mac Catalyst separately. Core Liquid Glass effects are unavailable on visionOS, although several layout APIs exist there. An `if #available` check does not make an unavailable platform API usable; isolate such code with conditional compilation or platform-specific files.

## SwiftUI glass APIs

The following effects and styles are available from iOS/macOS/tvOS/watchOS 26 and unavailable on visionOS. Import `SwiftUI`; several declarations live in its `SwiftUICore` dependency.

| Symbol                                                        | Availability and use                                                      |
| ------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `glassEffect(_:in:)`                                          | Glass behind content; defaults to `.regular` and a capsule                |
| `Glass.regular` / `.clear` / `.identity`                      | Adaptive / clear / no glass effect                                        |
| `Glass.tint(_:)` / `.interactive(_:)`                         | Configure tint and response; macOS mouse response improves in 27          |
| `GlassEffectContainer(spacing:content:)`                      | Render related effects together; spacing controls blending                |
| `glassEffectID(_:in:)`                                        | Stable element identity for morphing within a namespace                   |
| `glassEffectUnion(id:namespace:)`                             | Combine matching shapes and variants with the same union ID and namespace |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`, `.materialize`, or `.identity`                        |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | Semantic button styles; on tvOS the glass styles apply even without focus |

Overload distinction: `GlassButtonStyle()` and `.glass(_:)` are declared from 26.0 in this SDK; the direct initializer `GlassButtonStyle(_:)` requires 26.1. Check the declaration of the overload actually called.

## SwiftUI navigation and layout

| Symbol                                                                 | Availability and use                                                                     |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                                           | 26 across Apple platforms; `.onScrollDown` behavior is iPhone-specific                   |
| `ToolbarSpacer(_:placement:)` / `sharedBackgroundVisibility(_:)`       | iOS/macOS 26; unavailable on tvOS/watchOS/visionOS                                       |
| `scrollEdgeEffectStyle(_:for:)` / `scrollEdgeEffectHidden(_:for:)`     | iOS/macOS/tvOS/watchOS 26; unavailable on visionOS despite the style type existing there |
| `backgroundExtensionEffect()`                                          | 26 across Apple platforms; extend background content beneath adjacent UI                 |
| `tabViewBottomAccessory(content:)` / `tabViewBottomAccessoryPlacement` | iOS 26; adapt accessory to `.inline` / `.expanded` / `nil`                               |
| `tabViewBottomAccessory(isEnabled:content:)`                           | iOS 26.1; dynamically show or hide the accessory                                         |

## UIKit and AppKit

| Symbol                                                                                              | Availability and use                                                                                                       |
| --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `UIGlassEffect(style:)` / `UIGlassContainerEffect`                                                  | iOS 26; custom and grouped effects in `UIVisualEffectView`; configure `tintColor` and `isInteractive`; no visionOS/watchOS |
| `UIButton.Configuration.glass()` / `.prominentGlass()` / `.clearGlass()` / `.prominentClearGlass()` | iOS/tvOS 26; system button configurations                                                                                  |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`                                                  | iOS/tvOS/visionOS 26; use `style` and `isHidden` on each edge effect                                                       |
| `UIScrollEdgeElementContainerInteraction` / `UIBackgroundExtensionView`                             | iOS/tvOS/visionOS 26; layout support, not proof of glass availability                                                      |
| `UIBarButtonItem.hidesSharedBackground` / `.isHidden`                                               | iOS 26 for the background property; iOS 16 for `isHidden`                                                                  |
| `UITabBarController.tabBarMinimizeBehavior`                                                         | iOS 26; UIKit tab bar minimization                                                                                         |
| `NSGlassEffectView` / `NSGlassEffectContainerView`                                                  | macOS 26; custom glass and grouping, with `contentView` and `spacing`                                                      |
| `NSButton.BezelStyle.glass` / `NSBackgroundExtensionView`                                           | macOS 26; glass buttons / background extension                                                                             |
| `NSGlassEffectView.effectIsInteractive`                                                             | macOS 27; interactive mouse response                                                                                       |

Mac Catalyst uses UIKit, not AppKit. The glass effect and all four button configurations type-check for a Catalyst 26 target with the verified SDK. Do not infer unavailability from an omitted website badge: check the Catalyst SDK declaration and compile the actual call for the target. UIKit does not provide a watchOS UI framework.

## Additions and changes in 27

| Symbol                                                                                     | Availability and use                                                                                             |
| ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| `toolbarMinimizationBehavior(_:for:)`                                                      | 27; current name replacing beta `toolbarMinimizeBehavior`; `.navigationBar` is the supported placement           |
| `toolbarMinimizationSafeAreaAdjustment(_:for:)` / `toolbarMinimizationRestoration(_:for:)` | 27; safe-area adjustment and restoration policy                                                                  |
| `UINavigationItem.navigationBarMinimization`                                               | iOS 27; a `UIBarMinimization` value with `minimizationBehavior`, `safeAreaAdjustment`, and `restorationBehavior` |
| `TabRole.prominent` / `UITabBarController.prominentTabIdentifier`                          | 27 / iOS 27; designate the prominent tab                                                                         |
| `UITabBarController.Sidebar.preferredPlacement`                                            | iOS 27; opt into sidebar placement when space permits; check `isAvailable`                                       |
| `ToolbarContent.visibilityPriority(_:)`                                                    | iOS/tvOS/watchOS/visionOS 27, macOS 26.1; overflow priority                                                      |
| `ToolbarOverflowMenu` / `.topBarPinnedTrailing`                                            | iOS/visionOS 27; unavailable on macOS/tvOS/watchOS                                                               |
| `toolbarColorScheme(_:for:)` / `toolbarVisibility(_:for:)` with `.statusBar`               | 27; status-bar styling and visibility support                                                                    |
| `appearsActive`                                                                            | Already available from iOS 18/macOS 15; use for the new inactive-window appearance                               |
| `UIMenuElement.preferredImageVisibility`                                                   | iOS 27; override default menu icon visibility where needed                                                       |

Runtime changes include refined glass rendering, the appearance slider, inactive-window styling, and scroll edge styling. Building with the 27 SDKs also changes app requirements: UIKit apps need the scene lifecycle and an appropriate launch screen. Layout must support resizable iPhone apps. These requirements are not solved by a glass availability check.

## Implementation checks

- Keep platform availability, deployment target, build SDK, and runtime behavior distinct. Gate 27 APIs when supporting 26; keep a working 26 path.
- Apply appearance modifiers before `glassEffect`, then identity, union, or transition modifiers. Use containers for related effects and measure actual rendering cost.
- Use semantic buttons for actions. `ToolbarItem` has no `isHidden` property; conditionally include the item in SwiftUI.
- Test custom labels, motion, safe areas, and backgrounds under accessibility settings. System material adaptation does not validate app content.
- `UIDesignRequiresCompatibility` is ignored when building with the 27 SDKs, including on 26 runtimes. It is not a fallback for new projects.
- Recheck beta names against the selected SDK; WWDC samples may retain earlier names.

## Official documentation

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

Design: [Liquid Glass design](liquid-glass-design.md). Recipes: [Liquid Glass patterns](liquid-glass-patterns.md).
