---
id: platforms/liquid-glass-design
lang: en
version: 3
source-lang: en
status: active
digest: d8fcfa6f
---

# Liquid Glass design (Apple platforms)

## Adoption and scope

For new iOS 27 and native macOS 27 apps, adopt Liquid Glass through system navigation and controls first. Add custom glass only where standard components cannot meet the interaction requirements. Set the deployment target to iOS 27.0 or macOS 27.0 and build with Xcode 27. This guideline covers these two platforms only.

The 27 SDK behavior was verified on 2026-09-06 against the available beta. Recheck Apple’s HIG, release notes, and SDK declarations when adopting the final SDK or a later update.

## Separate controls from content

- Use Liquid Glass for navigation and controls above content: bars, sidebars, menus, system presentations, and custom floating controls. Let the system choose materials for standard components and their interaction states.
- Keep backgrounds, article text, lists, tables, and content cards in the content layer. Use opaque surfaces or standard materials there. A control embedded in content may acquire glass temporarily during interaction.
- Do not stack independent glass surfaces. Controls on an existing glass surface should use the system’s foreground treatment, fills, or vibrancy.
- Glass adapts to its backdrop through refraction, luminosity, tint, and shadow. Preserve content beneath floating controls; do not add empty glass panels as decoration.

## Choose regular or clear

- Use `.regular` by default, especially for text and dense controls. It adjusts background blur and luminosity to support legibility.
- Use `.clear` only over media-rich content when dimming the background is acceptable and foreground content is bold and bright. It does not provide regular’s adaptive legibility treatment.
- For bright backgrounds, HIG suggests a black dimming layer at about 35% opacity; the SwiftUI `Glass.clear` example uses 30%. These are starting points, not a universal contrast guarantee. Verify the actual foreground and moving background together.
- Keep related elements on the same variant. Do not mix regular and clear within one control group.

## Tint and foreground color

- Reserve glass tint for the primary action or a meaningful status; keep secondary actions neutral. Put most branding and expressive color in the content layer.
- Prefer semantic colors and system button styles. Small glass controls can switch foreground appearance as their backdrop changes; fixed black or white labels can defeat that adaptation.
- Convey selection, status, and destructive actions through labels, symbols, or button roles as well as color. Tint alone is insufficient.

## Shapes and concentricity

- Use system control shapes first. Capsules suit standalone controls; nested rounded surfaces should maintain concentricity with their container.
- Prefer system-derived concentric shapes where available. Give reusable shapes a fallback radius for use outside a rounded container.
- Respect screen and window margins rather than copying a device’s corner radius. Recheck nested corners when padding, Dynamic Type, or window size changes. macOS 27 changes window corner geometry.

## Layout and scroll edges

- Extend backgrounds and artwork to the window edges, while keeping readable content and controls within appropriate safe areas. Do not apply `ignoresSafeArea` to the entire interactive hierarchy merely to achieve an edge-to-edge background.
- Let scroll edge effects separate scrolling content from floating bars. Keep the automatic style unless the layout needs an explicit soft or hard boundary. Avoid duplicate effects on the same edge; align their heights across adjacent panes.
- Use background extension on the artwork or main content that should appear beneath a sidebar or inspector. Keep text and controls out of the extended image.
- Base layout on available view size and size classes. Preserve essential actions when the window narrows, the keyboard appears, or text grows.

## Accessibility and validation

- Test Reduce Transparency, Increase Contrast, Reduce Motion, and Dynamic Type. System materials adapt, but custom foreground colors, animations, and layouts remain the app’s responsibility.
- Check text contrast against the actual background in light and dark appearances. Apple’s HIG specifies 4.5:1 for text up to 17 pt, and 3:1 for larger or bold text. Do not use this threshold as a reason to shrink text or make every label bold.
- Use semantic controls with accessible names and adequate hit areas. Verify VoiceOver order, keyboard access, and the platform’s pointer or focus behavior. A visual glass reaction does not create button semantics.
- Reduce custom morphing and spring motion when Reduce Motion is enabled; a nonanimated state change is a valid fallback.
- Validate bright, dark, busy, and moving backdrops; small windows; large text; and inactive windows. Profile scrolling and transitions with representative content on target hardware.

## System behavior in iOS 27 and macOS 27

- Liquid Glass rendering is refined, with revised diffusion, edges, and highlights. The system appearance slider changes the material’s tint; custom interfaces must remain readable across its range.
- Sidebars extend to the edges on iPad and Mac; inactive windows gain a clearer visual distinction. Use `appearsActive` for custom elements that need to follow window activity.
- Menus show fewer icons by default. Restore an icon only where it helps identify an important action.
- iPhone apps can resize in iPhone Mirroring and on iPad. Standard bars adapt; custom layouts must preserve actions through resizing and toolbar overflow.
- These are runtime and SDK changes with different requirements. Use the [API reference](liquid-glass-api.md) to distinguish available symbols from new runtime behavior.

## Official references

- [HIG: Materials](https://developer.apple.com/design/human-interface-guidelines/materials)
- [HIG: Color](https://developer.apple.com/design/human-interface-guidelines/color)
- [HIG: Layout](https://developer.apple.com/design/human-interface-guidelines/layout)
- [HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Glass.clear](https://developer.apple.com/documentation/swiftui/glass/clear)
- [Meet Liquid Glass — WWDC25](https://developer.apple.com/videos/play/wwdc2025/219/)
- [Get to know the new design system — WWDC25](https://developer.apple.com/videos/play/wwdc2025/356/)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Platforms State of the Union — WWDC26](https://developer.apple.com/videos/play/wwdc2026/102/)

Implementation: [Liquid Glass patterns](liquid-glass-patterns.md). Availability: [Liquid Glass API reference](liquid-glass-api.md).
