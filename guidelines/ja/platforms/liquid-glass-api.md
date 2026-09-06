---
id: platforms/liquid-glass-api
lang: ja
version: 1
source-lang: en
status: active
digest: f5b58a7b
---

# Liquid Glass API リファレンス

## スコープと検証

Liquid Glass 関連 API のシンボル単位のリファレンス。Xcode 27 SDK のインターフェイス（`iPhoneOS27.0.sdk`、`MacOSX27.sdk`）と Apple のドキュメントページに対して 2026-09-06 に検証した。基線：SwiftUI／UIKit／AppKit 各節のシンボルは iOS 26 / macOS 26 / tvOS 26 / watchOS 26 から利用可能。「iOS 27 の新規」節は 27.0 以上と可用性ゲートが必要。

バックデプロイは存在しない：iOS 25 以前をサポートするアプリは `if #available(iOS 26.0, *)`（または `#available(macOS 26.0, *)`）でガラスのコードをゲートし、フォールバックを用意する。visionOS は Liquid Glass を採用しない——コアのガラスシンボルは `@available(visionOS, unavailable)` であり、このプラットフォームは独自のデザイン言語を保つ。設計規則は[Liquid Glass デザイン](liquid-glass-design.md)、レシピは[Liquid Glass 実装パターン](liquid-glass-patterns.md)を参照。

## SwiftUI：コアのガラス API

| シンボル                                                      | 用途                                                               |
| ------------------------------------------------------------- | ------------------------------------------------------------------ |
| `glassEffect(_:in:)`                                          | ビューの背後にガラス形状を描画。デフォルトは capsule + `.regular`  |
| `Glass`                                                       | マテリアル設定値：`.regular`、`.clear`、`.identity`                |
| `Glass.tint(_:)` / `Glass.interactive(_:)`                    | ティント付きバリアント／システムのタッチ・ポインタ反応             |
| `GlassEffectContainer(spacing:content:)`                      | グループを一回の描画に統合。融合とモーフを有効化                   |
| `glassEffectID(_:in:)`                                        | コンテナ内モーフ用の安定識別子。`Namespace` と併用                 |
| `glassEffectUnion(id:namespace:)`                             | 複数要素を静止状態で一つの形状にマージ（同形状・同バリアントのみ） |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`（近距離）または `.materialize`（遠距離）の遷移  |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | システムのガラスボタンスタイル——生の効果より優先                   |

SwiftUI フレームワーク。iOS/iPadOS/macCatalyst/macOS/tvOS/watchOS 26.0+——visionOS には存在しない（このマテリアルが無い）。tvOS はフォーカス状態にかかわらずガラスを適用する。

## SwiftUI：chrome 挙動 API

| シンボル                                           | 用途                                                                                                      |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                       | 浮遊タブバーがスクロールで最小化。`.onScrollDown` は iPhone 専用                                          |
| `ToolbarSpacer(.fixed/.flexible, placement:)`      | ツールバー項目を独立したガラスグループに分割（iOS/macOS のみ）                                            |
| `sharedBackgroundVisibility(_:)`（ToolbarContent） | 項目の共有ガラス背景を外し、独立グループにする（iOS/macOS のみ）                                          |
| `scrollEdgeEffectStyle(_:for:)`                    | 辺ごとに `.automatic` / `.soft` / `.hard`。`nil` でシステムデフォルト。モディファイアは visionOS 利用不可 |
| `scrollEdgeEffectHidden(_:for:)`                   | 指定辺のエッジエフェクトを除去（visionOS は同じ欠落）                                                     |
| `backgroundExtensionEffect()`                      | 内容がサイドバー／inspector の下に視覚的に延長                                                            |
| `tabViewBottomAccessory(content:)`                 | タブバー上部の永続アクセサリ。バーとともに折りたたむ（iOS 系プラットフォームのみ）                        |

SwiftUI フレームワーク。iOS 26.0+ が基線で、プラットフォーム差異は行ごとに注記する（`tabBarMinimizeBehavior`、`backgroundExtensionEffect`、および `ScrollEdgeEffectStyle` 型自体は visionOS でも利用可能）。

## UIKit API

| シンボル                                                     | 用途                                                             |
| ------------------------------------------------------------ | ---------------------------------------------------------------- |
| `UIGlassEffect(style:)`                                      | `UIVisualEffectView` 用ガラスマテリアル。`.regular` / `.clear`   |
| `UIGlassEffect.tintColor` / `.isInteractive`                 | ティント。インタラクティブなタッチ反応                           |
| `UIGlassContainerEffect`（`spacing`）                        | コンテナ効果：ネストしたガラスビューを一つの結合表面として描画   |
| `UIButton.Configuration.glass()`（prominent/clear 系を含む） | ガラスボタン設定（Mac Catalyst には無い）                        |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`           | 辺ごとの `UIScrollEdgeEffect`（`style`、`isHidden`）             |
| `UIScrollEdgeElementContainerInteraction`                    | カスタムオーバーレイをスクロールエッジエフェクトの形状計算に登録 |
| `UIBarButtonItem.hidesSharedBackground`                      | 項目を共有ガラス背景から外す                                     |
| `UITabBarController.tabBarMinimizeBehavior`                  | UIKit 側のタブバー最小化                                         |
| `UIBackgroundExtensionView`                                  | 背景延長コンテナー                                               |

UIKit。iOS/iPadOS/macCatalyst/tvOS 26.0+（ガラス効果とボタン設定は visionOS／watchOS 利用不可。ボタン設定は Catalyst にも無い）。

## AppKit API

| シンボル                                  | 用途                                                                  |
| ----------------------------------------- | --------------------------------------------------------------------- |
| `NSGlassEffectView`                       | ガラスコンテナー：`contentView`、`cornerRadius`、`tintColor`、`style` |
| `NSGlassEffectContainerView`（`spacing`） | 隣接ガラスビューをマージし描画パスを削減                              |
| `NSButton.BezelStyle.glass`               | ガラスベゼル——ボタン背面の自作ガラスより優先                          |
| `NSBackgroundExtensionView`               | 背景延長コンテナー（titlebar／サイドバー／inspector）                 |
| `NSGlassEffectView.effectIsInteractive`   | macOS のインタラクティブガラス——27.0+ 専用                            |

AppKit。macOS 26.0+（`effectIsInteractive` のみ 27.0+）。

## iOS 27 の新規

Xcode 27 SDK と iOS 27 リリースノートで検証済み。`#available(iOS 27, *)` / `macOS 27` でゲートする：

- `toolbarMinimizationBehavior(_:for:)`（と `toolbarMinimizationSafeAreaAdjustment`）——`toolbarMinimizeBehavior` を改名・拡張。対応 placement は `.navigationBar`。
- `UINavigationItem.navigationBarMinimization`（`UIBarMinimizationBehavior`：`.automatic/.never/.onScrollDown/.onScrollUp`）——ベータ期の `barMinimizeBehavior` 命名を置き換える。WWDC26 のサンプルコードには旧名称が残る可能性がある。
- `Tab(role: .prominent)` / `TabRole.prominent` と `UITabBarController.prominentTabIdentifier`——末尾にピン留めされる prominent タブ。
- `UITabBarController.Sidebar.preferredPlacement`（`.sidebar/.tabBar/.automatic`）——空間が許せば iPhone でもタブバーのサイドバー表現を使える。
- `ToolbarOverflowMenu` と `ToolbarContent.visibilityPriority(_:)`——空間不足時のツールバーリフロー。
- `ToolbarItemPlacement.topBarPinnedTrailing`——リフローに関係なく末尾にピン留めされる項目。
- `toolbarColorScheme(_:for: .statusBar)` / `toolbarVisibility(_:for: .statusBar)`——ステータスバー制御。
- 環境値 `appearsActive`——iOS 18 から存在する値。iOS 27 で非アクティブウィンドウが明確な見た目を持つようになり、カスタム chrome はこれに従って適応すべきである（新規シンボルではない）。
- `NSGlassEffectView.effectIsInteractive`（macOS 27）と `UIMenuElement.preferredImageVisibility`（メニューアイコンはデフォルト非表示）。
- 新 API を伴わない実行時の変化：マテリアルの再調整（拡散・暗い縁・鏡面ハイライト）、ユーザー向け外観スライダー（ウルトラクリア ↔ 完全ティント）、浮遊バー下へのスクロール時に現れる上部の均一ツールバー、iPad/Mac で端まで広がりアクセント色のアイコンを取り戻すサイドバー、macOS の統一された小さいウィンドウ角丸、リサイズ可能な iPhone アプリ。

## 開発上の注意

1. 機能領域ごとに一つの `GlassEffectContainer` とし、画面上の効果数を抑える——ガラスの層にはそれぞれ描画コストがかかり、コンテナ外のガラスはサンプリングも不一致になる（ガラスはガラスをサンプリングできない）。
2. `glassEffect` はビューの内容をキャプチャする——外観系モディファイアの後に置く。
3. モーフの規則：`.matchedGeometry` はコンテナの `spacing` 距離内でのみ機能し、超える場合は `.materialize` を使う。デフォルトアニメーションでは matchedGeometry がスケール・オフセットの演出を追加する——除外には明示的なアニメーションを渡す。`glassEffectUnion` はメンバー間で形状と `Glass` バリアントの完全一致が必要。
4. `.clear` は明るい内容の上では調光レイヤーが要る（Apple の例：`.background(.black.opacity(0.3))`）。関連要素で `.regular` と混ぜない。
5. アクセシビリティ適応はシステムコンポーネント専用——カスタムガラスは Reduce Transparency、Increase Contrast、Reduce Motion、iOS 27 の外観スライダーでテストする。
6. ゲートが必要なプラットフォームの欠落：コアのガラスシンボルは visionOS で利用不可（同プラットフォームは独自のデザイン言語を保持）だが watchOS 26 には存在する。`scrollEdgeEffectStyle`／`scrollEdgeEffectHidden` のモディファイアは visionOS 利用不可（`ScrollEdgeEffectStyle` 型は存在する）。Apple の可用性メタデータによれば `UIButton.Configuration.glass()` 系は Mac Catalyst に無い（Catalyst のコードでは利用可能な `UIGlassEffect` を使う）。ネイティブ macOS は `NSGlassEffectView`。
7. ツールバー項目の非表示は項目本体（`ToolbarItem`／`UIBarButtonItem` の `isHidden`）で行い、内容ビューでは行わない。
8. iOS 27 の `.automatic` スクロールエッジエフェクトは独自の見た目を持つ（soft/hard を切り替えない）——27 SDK 採用時に明示的な `.soft` オーバーライドを再評価する。
9. iOS 27 SDK でビルドすると iPhone アプリがリサイズ可能になり、UIScene ライフサイクルと launch screen キーが必須になる——新規プロジェクトは現在のテンプレートで既定で満たす。
10. `UIDesignRequiresCompatibility`（Info.plist）はガラス以前の見た目を復元する。最終手段とし、新規プロジェクトのデフォルトには絶対にしない。

## 公式ドキュメント索引

- Adopting Liquid Glass — <https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass>
- Applying Liquid Glass to custom views — <https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views>
- Liquid Glass 概要 — <https://developer.apple.com/documentation/TechnologyOverviews/liquid-glass>
- サンプル：Landmarks — building an app with Liquid Glass — <https://developer.apple.com/documentation/swiftui/landmarks-building-an-app-with-liquid-glass>
- WWDC25-323 Build a SwiftUI app with the new design — <https://developer.apple.com/videos/play/wwdc2025/323/>
- WWDC26-269 What's new in SwiftUI — <https://developer.apple.com/videos/play/wwdc2026/269/>
- WWDC26-278 Modernize your UIKit app — <https://developer.apple.com/videos/play/wwdc2026/278/>
- iOS & iPadOS 27 リリースノート — <https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes>
