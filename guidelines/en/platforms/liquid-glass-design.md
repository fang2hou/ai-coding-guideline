---
id: platforms/liquid-glass-design
lang: en
version: 1
source-lang: en
status: active
digest: 9704566d
---

# Liquid Glass design (Apple platforms)

## Verdict

Liquid Glass is the system design language on iOS/iPadOS 26+, macOS 26+, tvOS 26+, and watchOS 26+, refined in the iOS 27 / macOS 27 line. New Apple-platform projects adopt it; this document is mandatory reading before designing any new app UI. It imposes no migration duty on existing pre-iOS-26 apps — retrofitting is out of scope.

Adopt the system defaults first and customize only where system chrome cannot express the design. The material is still evolving (it was retuned in iOS 26.1-era updates and again in iOS 27); treat Apple's Human Interface Guidelines as the living source of truth and re-verify at each OS major release.

## The two-layer model

- Liquid Glass is a dynamic material that bends and concentrates light in real time (lensing) instead of scattering it. Tint, shadow, and dynamic range adapt continuously to the content behind it.
- The UI splits into two layers: a functional layer of controls and navigation (tab bars, sidebars, toolbars, navigation bars, menus) floating above the content layer. Content scrolls beneath the glass; glass keeps controls legible. The content layer is where Liquid Glass must not appear — it may be opaque or use standard materials, never glass.
- Two material families exist and they do not mix jobs: Liquid Glass for the control/navigation layer; standard materials (blur, vibrancy, thickness) for structure inside the content layer.
- Small glass elements flip light/dark with their background; large surfaces (menus, sidebars) adapt but never flip. Elements appear by modulating lensing (materializing), not by fading.

## Where glass belongs

- In the functional layer: navigation bars, toolbars, tab bars, sidebars (inset and floating on iPad/Mac), menus, sheets and action sheets, transient control states (sliders and toggles take on glass while active), and custom floating controls.
- Never in the content layer — app backgrounds, cards, and table/collection content belong to the content layer and never take Liquid Glass (HIG: "Don't use Liquid Glass in the content layer"). The only exception is the transient activation state of content-layer controls.
- Never stack glass on glass; if an element sits on glass, use fills, transparency, or vibrancy so it reads as part of the material.
- Never place glass where there is nothing beneath to refract — glass belongs on controls that hug the content layer, not on static content areas.
- Glass is not decoration. Scroll edge effects in particular exist to blend content under floating chrome, not to ornament it.

## Variants: regular and clear

- Regular is the default: it blurs and adjusts background luminosity, adapts to any content, and stays legible at any size. Use it for text-heavy chrome (alerts, sidebars, popovers) and wherever the background could hurt legibility.
- Clear is permanently more translucent with no adaptive legibility behavior. Apply it only when all three conditions hold: it floats over media-rich content; the content layer will not be harmed by a dimming layer; the content on the glass is bold and bright.
- With Clear over bright content, add a dark dimming layer at roughly 35% black opacity beneath the glass. Skip it when content is already dark.
- Never mix Regular and Clear on related elements.

The 35% dimming figure, the three-condition test, and the tint limits are Apple's current published defaults and context tests — follow them, but re-check HIG at each major release instead of hard-coding them as permanent tokens.

## Tinting and color

- Glass has no inherent color. A tint maps one color to a range of tones keyed to the brightness behind it (stained-glass behavior) and buys contrast.
- Reserve tint for emphasis: one primary action or status. The system tints prominent buttons' glass with the accent color automatically.
- Do not tint many controls; put color in the content layer instead. Over colorful backgrounds, keep bars monochromatic or pick a well-differentiated accent.
- Symbols and labels on small bars default to monochrome and flip with the background — do not force fixed light/dark text colors that fight the flip.

## Shapes and concentricity

- Three shape types: fixed (constant radius), capsule (radius = half the height; naturally concentric; the bar/slider/switch/grouped-corner default), and concentric (inner radius derived from the parent's radius minus padding, computed by the system).
- Nested containers (artwork in cards, buttons in bars) must use concentric shapes so inner radii compute automatically. Never pinch or flare corners.
- Near iPhone screen edges, use a capsule with extra margin; on iPad/Mac, align concentrically to the window edge. For components that live both nested and standalone, use a concentric shape with a fallback radius.
- macOS 27 gives every window the same tighter corner radius — do not assume the old per-window radii.

## Layout: edge to edge

- Extend backgrounds and full-bleed artwork to the display edges; scrollable content continues under floating chrome. Respect system safe areas (Dynamic Island, camera housing, bars) and design for the full window-size range on iPad.
- Scroll edge effects replace hard dividers between floating glass and scrolling content. Soft is the default on iOS/iPadOS (subtle transition); Hard suits mostly macOS (uniform opaque boundary for text-like controls, borderless controls, pinned headers). One effect per view, consistent heights across split-view panes, never stacked, and none where no floating UI exists.
- When content does not span the full window (sidebars, inspectors), use a background-extension view so content appears to continue behind the chrome; keep text and controls layered above it to avoid distortion.
- iOS 27: a uniform toolbar materializes across the top as content scrolls under floating bars (automatic for standard toolbars), and iPhone apps become resizable on iPad and in iPhone Mirroring — design for a dynamic range of sizes and aspect ratios, using toolbar reflow (overflow, priorities, pinned items) instead of fixed layouts.

## Accessibility

- System components adapt automatically — no opt-in: Reduce Transparency makes glass frostier and obscures more; Increase Contrast turns elements predominantly black/white with a contrasting border; Reduce Motion dampens elastic/liquid behaviors. Custom glass must be tested under all three settings plus Dynamic Type.
- Meet contrast minimums: 4.5:1 for text up to 17 pt, 3:1 for text 18 pt or larger or bold. Verify in light and dark appearance; provide a higher-contrast scheme when Increase Contrast is on.
- iOS 27 adds a user-facing appearance slider from ultra clear to fully tinted, and macOS 27 gains the "show borders" accessibility value. Never assume one fixed glass rendering across users or OS versions.
- For morphs and liquid motion under Reduce Motion: tighten springs, track gestures directly, prefer fades, and avoid animating into or out of blurs.

## What changed in iOS 27

- The material was retuned again: better diffusion of complex content behind glass, a darkened edge, and brighter specular highlights. Apps already using Liquid Glass get the new rendering without recompiling.
- Users can adjust glass appearance system-wide (ultra clear ↔ fully tinted) — designs must tolerate the full range.
- Sidebars expand to the screen edges on iPad and Mac and their icons regain accent color; menu icons are hidden by default on macOS/iPadOS (surf key actions via API).
- Windows show a distinct inactive appearance (key custom views off `appearsActive`); custom glass can be interactive on macOS (mouse-optimized).
- App icons render sharper with reduced translucency; Icon Composer now designs multi-layer Liquid Glass icons with refraction and previews on older OSes.

## Do and don't

| Do                                                                      | Don't                                                           |
| ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| Reserve glass for floating controls and navigation                      | Put glass on content-layer views (tables, backgrounds, cards)   |
| Use Regular by default, especially for text-heavy chrome                | Use Clear where legibility or the content layer could suffer    |
| Use Clear only over media-rich content, with a ~35% dim layer if bright | Mix Regular and Clear on related elements                       |
| Tint one primary action's glass                                         | Tint many controls or tint symbols                              |
| Let glyphs stay monochrome and system-adaptive                          | Hardcode light/dark text colors against the flipping material   |
| Use capsule/concentric shapes; let the system compute radii             | Pinch, flare, or hand-compute nested corner radii               |
| Keep content edge-to-edge; blend with scroll edge effects               | Stack glass on glass; re-add bar backgrounds, borders, dividers |
| One scroll edge effect per view                                         | Use scroll edge effects as decoration                           |
| Provide light and dark colors even for single-mode apps                 | Ignore Increase Contrast or the iOS 27 appearance slider        |
| Test Reduce Transparency, Increase Contrast, Reduce Motion              | Ship custom glass tested only at default settings               |

## Official references

- HIG Materials — <https://developer.apple.com/design/human-interface-guidelines/materials>
- HIG Color (Liquid Glass color) — <https://developer.apple.com/design/human-interface-guidelines/color>
- HIG Layout — <https://developer.apple.com/design/human-interface-guidelines/layout>
- HIG Accessibility — <https://developer.apple.com/design/human-interface-guidelines/accessibility>
- HIG Design principles — <https://developer.apple.com/design/human-interface-guidelines/design-principles>
- WWDC25-219 Meet Liquid Glass — <https://developer.apple.com/videos/play/wwdc2025/219/>
- WWDC25-356 Get to know the new design system — <https://developer.apple.com/videos/play/wwdc2025/356/>
- WWDC26-102 Platforms State of the Union (iOS 27 refinements) — <https://developer.apple.com/videos/play/wwdc2026/102/>
- WWDC26-250 Principles of great design — <https://developer.apple.com/videos/play/wwdc2026/250/>
- Adopting Liquid Glass (technology overview) — <https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass>
- WWDC26 design guide hub — <https://developer.apple.com/wwdc26/guides/design/>
- Apple Design Resources (Figma/Sketch kits, safe-area guides) — <https://developer.apple.com/design/resources/>
- Icon Composer — <https://developer.apple.com/icon-composer/>

Implementation code and recipes: [Liquid Glass implementation patterns](liquid-glass-patterns.md). Symbol-level details: [Liquid Glass API reference](liquid-glass-api.md).
