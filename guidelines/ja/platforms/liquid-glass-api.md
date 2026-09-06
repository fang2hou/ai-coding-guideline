---
id: platforms/liquid-glass-api
lang: ja
version: 3
source-lang: en
status: active
digest: 7e49f8f0
---

# Liquid Glass API リファレンス

## 対象範囲と検証

Xcode 27 でビルドし、最低対応 OS をそれぞれ 27.0 に設定した iOS 27 とネイティブ macOS 27 のアプリを対象とする。2026-09-06 に Apple の文書と Xcode 27 ベータ版のビルド `27A5252f`（`iPhoneOS27.0.sdk`、`MacOSX27.sdk`）で確認した。SDK の宣言はプラットフォーム上の可用性を、実行中の OS は描画と操作の挙動を決める。ベータ版の宣言は今後変わる可能性がある。

以下の表はこの二つのプラットフォームでの対応状況を示し、過去の導入バージョンは扱わない。iOS には UIKit、ネイティブ macOS には AppKit を使う。プラットフォーム専用の API は `#if os(iOS)`／`#if os(macOS)` または専用ファイルで分離する。

## SwiftUI のガラス API

この節の効果とスタイルは iOS 27 と macOS 27 の両方で利用できる。`SwiftUI` をインポートする。一部の宣言は依存先の `SwiftUICore` にある。

| シンボル                                                      | 可用性と用途                                                 |
| ------------------------------------------------------------- | ------------------------------------------------------------ |
| `glassEffect(_:in:)`                                          | 内容の背後にガラスを描画。標準は `.regular` のカプセル形状   |
| `Glass.regular` / `.clear` / `.identity`                      | 自動調整／クリア／効果なし                                   |
| `Glass.tint(_:)` / `.interactive(_:)`                         | ティントと操作への反応を設定。macOS のマウス操作にも対応     |
| `GlassEffectContainer(spacing:content:)`                      | 関連する効果をまとめて描画。spacing で融合する距離を指定     |
| `glassEffectID(_:in:)`                                        | 名前空間内でモーフィングする要素の安定した ID                |
| `glassEffectUnion(id:namespace:)`                             | 同じ名前空間と結合 ID の形状・バリアントが一致する効果を結合 |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`、`.materialize`、`.identity`              |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | システムのガラスボタンスタイル。操作の意味とロールを維持     |

## SwiftUI のナビゲーションとレイアウト

| シンボル                                                               | 可用性と用途                                               |
| ---------------------------------------------------------------------- | ---------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                                           | iOS：スクロール中に iPhone のタブバーを最小化              |
| `ToolbarSpacer(_:placement:)` / `sharedBackgroundVisibility(_:)`       | 両方：ツールバーのグループ分けと背景の制御                 |
| `scrollEdgeEffectStyle(_:for:)` / `scrollEdgeEffectHidden(_:for:)`     | 両方：スクロール端ごとにスタイルと表示を設定               |
| `backgroundExtensionEffect()`                                          | 両方：隣接する UI の背後へ背景を延長                       |
| `tabViewBottomAccessory(content:)` / `tabViewBottomAccessoryPlacement` | iOS：アクセサリを `.inline`／`.expanded`／`nil` に合わせる |
| `tabViewBottomAccessory(isEnabled:content:)`                           | iOS：アクセサリを動的に表示・非表示にする                  |

## UIKit と AppKit

| シンボル                                                                                            | 可用性と用途                                                                          |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `UIGlassEffect(style:)` / `UIGlassContainerEffect`                                                  | iOS：`UIVisualEffectView` 内の単独・複数の効果。`tintColor` と `isInteractive` を設定 |
| `UIButton.Configuration.glass()` / `.prominentGlass()` / `.clearGlass()` / `.prominentClearGlass()` | iOS：システムのボタン設定                                                             |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`                                                  | iOS：各辺の効果を `style` と `isHidden` で設定                                        |
| `UIScrollEdgeElementContainerInteraction` / `UIBackgroundExtensionView`                             | iOS：独自のオーバーレイの登録／背景の延長                                             |
| `UIBarButtonItem.hidesSharedBackground` / `.isHidden`                                               | iOS：共有背景／項目自体を非表示にする                                                 |
| `UITabBarController.tabBarMinimizeBehavior`                                                         | iOS：UIKit のタブバーの最小化                                                         |
| `NSGlassEffectView` / `NSGlassEffectContainerView`                                                  | macOS：`contentView` と `spacing` でカスタムのガラスとグループを設定                  |
| `NSButton.BezelStyle.glass` / `NSBackgroundExtensionView`                                           | macOS：ガラスのボタン／背景の延長                                                     |
| `NSGlassEffectView.effectIsInteractive`                                                             | macOS：マウス操作への反応                                                             |

## 適応的なナビゲーションとウィンドウ動作

| シンボル                                                                                   | 可用性と用途                                                                                             |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| `toolbarMinimizationBehavior(_:for:)`                                                      | iOS のナビゲーションバー：最小化方針。ベータ版の旧名 `toolbarMinimizeBehavior` ではなく現行名を使う      |
| `toolbarMinimizationSafeAreaAdjustment(_:for:)` / `toolbarMinimizationRestoration(_:for:)` | iOS のナビゲーションバー：セーフエリア調整と復元方針                                                     |
| `UINavigationItem.navigationBarMinimization`                                               | iOS：`UIBarMinimization` 値で `minimizationBehavior`、`safeAreaAdjustment`、`restorationBehavior` を指定 |
| `TabRole.prominent` / `UITabBarController.prominentTabIdentifier`                          | iOS：目立たせるタブを指定                                                                                |
| `UITabBarController.Sidebar.preferredPlacement`                                            | iOS：空間がある場合にサイドバー表示を選択。`isAvailable` を確認                                          |
| `ToolbarContent.visibilityPriority(_:)`                                                    | 両方：ツールバーのオーバーフロー優先順位                                                                 |
| `ToolbarOverflowMenu` / `.topBarPinnedTrailing`                                            | iOS 専用。ネイティブ macOS では利用不可                                                                  |
| `toolbarColorScheme(_:for:)` / `toolbarVisibility(_:for:)` with `.statusBar`               | iOS：ステータスバーの外観と表示制御                                                                      |
| `appearsActive`                                                                            | 両方：ウィンドウの活動状態に独自の内容を合わせる                                                         |
| `UIMenuElement.preferredImageVisibility`                                                   | iOS：必要に応じてメニューアイコンの標準の表示設定を変更                                                  |

外観スライダー、非アクティブ表示、スクロール端を UI の検証に含める。UIKit アプリはシーンのライフサイクルを使い、起動画面を用意する。iPhone のウィンドウサイズが変わっても主要な操作を利用できるように配置する。

## 実装時の確認

- iOS 27 と macOS 27 向けにそれぞれビルドし、型チェックを通す。共有の SwiftUI コードでも、ナビゲーション、ツールバー、各フレームワーク固有のビューはプラットフォーム別に分離する。
- 外観のモディファイアは `glassEffect` の前に、識別・結合・遷移のモディファイアは後に置く。関連する効果にはコンテナーを使い、実際の描画コストを測定する。
- 操作には適切な意味を持つボタンを使う。`ToolbarItem` に `isHidden` プロパティはない。SwiftUI では条件分岐で項目自体を追加・削除する。
- アクセシビリティ設定を変え、独自のラベル、動き、セーフエリア、背景を検証する。システムのマテリアルが適応しても、アプリの内容まで検証されたことにはならない。
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

設計は [Liquid Glass の設計](liquid-glass-design.md)、実装は [Liquid Glass 実装パターン](liquid-glass-patterns.md)を参照。
