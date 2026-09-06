---
id: platforms/liquid-glass-api
lang: ja
version: 2
source-lang: en
status: active
digest: 55062175
---

# Liquid Glass API リファレンス

## 対象範囲と検証

26 系 OS 向けの新規アプリを対象とし、27 ベータ版の追加項目は別に示す。2026-09-06 に Apple の文書と Xcode 27 ベータ版のビルド `27A5252f`（`iPhoneOS27.0.sdk`、`MacOSX27.sdk`）で確認した。SDK の宣言はコンパイル時の可用性を、実行中の OS は描画と操作の挙動を決める。ベータ版の宣言は今後変わる可能性がある。

以下のバージョンは各シンボルに対応し、フレームワーク全体には適用しない。iOS には iPadOS を含む。Mac Catalyst は個別に確認する。visionOS では Liquid Glass の主要な効果を利用できないが、一部のレイアウト API は利用できる。`if #available` では、そのプラットフォームで利用不可の API は使えない。条件付きコンパイルやプラットフォーム別のファイルで分離する。

## SwiftUI のガラス API

以下の効果とスタイルは iOS/macOS/tvOS/watchOS 26 から利用でき、visionOS では利用できない。`SwiftUI` をインポートする。一部の宣言は依存先の `SwiftUICore` にある。

| シンボル                                                      | 可用性と用途                                                        |
| ------------------------------------------------------------- | ------------------------------------------------------------------- |
| `glassEffect(_:in:)`                                          | 内容の背後にガラスを描画。標準は `.regular` のカプセル形状          |
| `Glass.regular` / `.clear` / `.identity`                      | 自動調整／クリア／効果なし                                          |
| `Glass.tint(_:)` / `.interactive(_:)`                         | ティントと反応を設定。macOS のマウス反応は 27 で強化                |
| `GlassEffectContainer(spacing:content:)`                      | 関連する効果をまとめて描画。spacing で融合する距離を指定            |
| `glassEffectID(_:in:)`                                        | 名前空間内でモーフィングする要素の安定した ID                       |
| `glassEffectUnion(id:namespace:)`                             | 同じ名前空間と結合 ID の形状・バリアントが一致する効果を結合        |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`、`.materialize`、`.identity`                     |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | ボタンの意味を持つスタイル。tvOS では非フォーカス時にもガラスを適用 |

オーバーロードに注意する。この SDK では `GlassButtonStyle()` と `.glass(_:)` は 26.0 から利用できるが、直接呼び出す初期化子 `GlassButtonStyle(_:)` は 26.1 が必要である。実際に呼び出す宣言を確認する。

## SwiftUI のナビゲーションとレイアウト

| シンボル                                                               | 可用性と用途                                                                                 |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                                           | Apple 各プラットフォームの 26。`.onScrollDown` の挙動は iPhone 向け                          |
| `ToolbarSpacer(_:placement:)` / `sharedBackgroundVisibility(_:)`       | iOS/macOS 26。tvOS/watchOS/visionOS では利用不可                                             |
| `scrollEdgeEffectStyle(_:for:)` / `scrollEdgeEffectHidden(_:for:)`     | iOS/macOS/tvOS/watchOS 26。visionOS にスタイル型はあるが、この二つのモディファイアは利用不可 |
| `backgroundExtensionEffect()`                                          | Apple 各プラットフォームの 26。隣接する UI の背後へ背景を延長                                |
| `tabViewBottomAccessory(content:)` / `tabViewBottomAccessoryPlacement` | iOS 26。アクセサリを `.inline`／`.expanded`／`nil` に合わせる                                |
| `tabViewBottomAccessory(isEnabled:content:)`                           | iOS 26.1。アクセサリを動的に表示・非表示にする                                               |

## UIKit と AppKit

| シンボル                                                                                            | 可用性と用途                                                                                                        |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `UIGlassEffect(style:)` / `UIGlassContainerEffect`                                                  | iOS 26。`UIVisualEffectView` 内の単独・複数の効果。`tintColor` と `isInteractive` を設定。visionOS/watchOS は対象外 |
| `UIButton.Configuration.glass()` / `.prominentGlass()` / `.clearGlass()` / `.prominentClearGlass()` | iOS/tvOS 26。システムのボタン設定                                                                                   |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`                                                  | iOS/tvOS/visionOS 26。各辺の効果を `style` と `isHidden` で設定                                                     |
| `UIScrollEdgeElementContainerInteraction` / `UIBackgroundExtensionView`                             | iOS/tvOS/visionOS 26。レイアウト用 API であり、ガラスが利用可能であることを意味しない                               |
| `UIBarButtonItem.hidesSharedBackground` / `.isHidden`                                               | 背景のプロパティは iOS 26、`isHidden` は iOS 16                                                                     |
| `UITabBarController.tabBarMinimizeBehavior`                                                         | iOS 26。UIKit のタブバーの最小化                                                                                    |
| `NSGlassEffectView` / `NSGlassEffectContainerView`                                                  | macOS 26。`contentView` と `spacing` でカスタムのガラスとグループを設定                                             |
| `NSButton.BezelStyle.glass` / `NSBackgroundExtensionView`                                           | macOS 26。ガラスのボタン／背景の延長                                                                                |
| `NSGlassEffectView.effectIsInteractive`                                                             | macOS 27。マウス操作への反応                                                                                        |

Mac Catalyst は AppKit ではなく UIKit を使う。確認した SDK では、ガラス効果と四つのボタン設定が Catalyst 26 向けの型チェックを通る。Web ページの可用性表示に記載がないだけで利用不可と判断せず、Catalyst SDK の宣言を確認して対象向けに実際の呼び出しをコンパイルする。UIKit は watchOS 用の UI フレームワークを提供しない。

## 27 の追加と変更

| シンボル                                                                                   | 可用性と用途                                                                                                       |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `toolbarMinimizationBehavior(_:for:)`                                                      | 27。ベータ版の `toolbarMinimizeBehavior` に代わる名称。対応する配置は `.navigationBar`                             |
| `toolbarMinimizationSafeAreaAdjustment(_:for:)` / `toolbarMinimizationRestoration(_:for:)` | 27。セーフエリア調整と復元方針                                                                                     |
| `UINavigationItem.navigationBarMinimization`                                               | iOS 27。`UIBarMinimization` 値を設定し、`minimizationBehavior`、`safeAreaAdjustment`、`restorationBehavior` を指定 |
| `TabRole.prominent` / `UITabBarController.prominentTabIdentifier`                          | 27／iOS 27。目立たせるタブを指定                                                                                   |
| `UITabBarController.Sidebar.preferredPlacement`                                            | iOS 27。空間がある場合にサイドバー表示を選択。`isAvailable` を確認                                                 |
| `ToolbarContent.visibilityPriority(_:)`                                                    | iOS/tvOS/watchOS/visionOS 27、macOS 26.1。オーバーフローの優先順位                                                 |
| `ToolbarOverflowMenu` / `.topBarPinnedTrailing`                                            | iOS/visionOS 27。macOS/tvOS/watchOS では利用不可                                                                   |
| `toolbarColorScheme(_:for:)` / `toolbarVisibility(_:for:)` with `.statusBar`               | 27。ステータスバーの外観と表示制御                                                                                 |
| `appearsActive`                                                                            | iOS 18/macOS 15 から利用可能。新しい非アクティブ表示に合わせる                                                     |
| `UIMenuElement.preferredImageVisibility`                                                   | iOS 27。必要に応じてメニューアイコンの標準の表示設定を変更                                                         |

実行時にはガラス描画、外観スライダー、非アクティブ表示、スクロール端の表現が変わる。27 系 SDK でのビルドではアプリの要件も変わり、UIKit アプリにはシーンのライフサイクルと適切な起動画面が必要となる。iPhone アプリのリサイズにも対応する。ガラス API の可用性チェックだけでは、これらの要件を満たせない。

## 実装時の確認

- プラットフォーム上の可用性、最低対応 OS、ビルド用 SDK、実行時の挙動を区別する。26 をサポートする場合は 27 API の可用性を確認し、26 でも動く処理を残す。
- 外観のモディファイアは `glassEffect` の前に、識別・結合・遷移のモディファイアは後に置く。関連する効果にはコンテナーを使い、実際の描画コストを測定する。
- 操作には適切な意味を持つボタンを使う。`ToolbarItem` に `isHidden` プロパティはない。SwiftUI では条件分岐で項目自体を追加・削除する。
- アクセシビリティ設定を変え、独自のラベル、動き、セーフエリア、背景を検証する。システムのマテリアルが適応しても、アプリの内容まで検証されたことにはならない。
- 27 系 SDK でビルドすると、26 系 OS 上でも `UIDesignRequiresCompatibility` は無視される。新規プロジェクトの代替策としては使えない。
- ベータ版の名称は使用する SDK で再確認する。WWDC のサンプルには古い名称が残る場合がある。

## 公式ドキュメント

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

設計は [Liquid Glass の設計](liquid-glass-design.md)、実装は [Liquid Glass 実装パターン](liquid-glass-patterns.md)を参照。
