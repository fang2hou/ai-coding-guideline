---
id: platforms/liquid-glass-patterns
lang: ja
version: 2
source-lang: en
status: active
digest: 84b76b70
---

# Liquid Glass 実装パターン

## 対象範囲

iOS 26+ と macOS 26+ 向けの新規アプリを対象とする。SwiftUI を基本とし、独自の組み込みには UIKit と AppKit の例を使う。プラットフォームやオーバーロードによって可用性が異なるため、[API リファレンス](liquid-glass-api.md)を確認する。27 の例には Xcode 27 が必要であり、最低対応 OS が 26 の場合は可用性をチェックする。設計は [Liquid Glass の設計](liquid-glass-design.md)に従う。

## システムコンポーネントから始める

- 新しいシステムデザインを採用するには、26 以降の SDK でビルドする。標準のナビゲーション、タブバー、ツールバー、シート、メニューにはガラス背景を追加しない。
- 独自のバー背景、枠線、シートのスタイルでシステムのマテリアルを覆わない。設計上必要なカスタマイズだけを残し、結果を検証する。
- 新規プロジェクトでは互換モードを使わない。27 系 SDK でビルドすると、最低対応 OS が 26 でも `UIDesignRequiresCompatibility` は無視される。

## ボタン

操作には `Button` とシステムのスタイルを優先する。破壊的な操作はロールで明示する。主要な操作を強調するスタイルは、破壊的な操作のロールを代替しない。

```swift
Button("Save", action: save)
    .buttonStyle(.glassProminent)
Button("Cancel", action: cancel)
    .buttonStyle(.glass)
Button("Delete", role: .destructive, action: delete)
    .buttonStyle(.glass)
```

操作のクロージャはアプリ側で用意する。`.glass(.clear)` は設計文書の clear の使用条件を満たす場合に限る。UIKit の `UIButton.Configuration` には `.glass()`、`.prominentGlass()`、`.clearGlass()`、`.prominentClearGlass()` があり、AppKit ではボタンのベゼルに `.glass` を指定できる。

## カスタムのガラス効果

サイズと外観を設定するモディファイアの後に `glassEffect` を適用する。標準は `.regular` のカプセル形状となる。コントロールの形状に必要な場合だけ、明示的に形状を指定する。

```swift
Text("3 selected")
    .font(.headline)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))
```

`.interactive()` が設定するのはマテリアルの反応である。操作、キーボードからの実行、アクセシビリティ特性は追加されない。操作可能な内容には `Button` を使う。`Glass.interactive(_:)` 自体は macOS 26 から呼び出せる。macOS 27 ではマウス向けの反応と AppKit の `effectIsInteractive` プロパティが加わる。

clear のガラス効果では背景も含めて設計する。Apple の例は効果の背後に不透明度 30% の黒を置く。実際の画像や動画に合わせて調整し、前景のコントラストを確認する。

```swift
Label("Flag", systemImage: "flag.fill")
    .padding()
    .glassEffect(.clear)
    .background(.black.opacity(0.3))
```

## コンテナー、モーフィング、結合

関連するカスタムのガラス効果を `GlassEffectContainer` にまとめ、システムが一緒に描画して形状を融合できるようにする。`spacing` は近接する形状が相互作用を始める距離を指定する。レイアウトの間隔より大きいと、静止時にも融合する場合がある。コンテナーは描画効率を改善するが、描画パス数を固定する保証はない。

要素の追加と削除には、同じ名前空間で安定した重複のない `glassEffectID` を使う。次のコントロールはボタンの意味を保ち、「視差効果を減らす」が有効な場合は独自のアニメーションを無効にする。

```swift
import SwiftUI

@available(iOS 26.0, macOS 26.0, *)
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

近い形状には `.matchedGeometry`、近くに適切な形状がない遷移には `.materialize` を使う。`glassEffectTransition` は `glassEffect` の後に置く。静的に結合する場合は、各効果の後に `glassEffectUnion(id:namespace:)` を適用する。結合用の識別子、名前空間、形状、ガラスのバリアントが一致する要素が一つの面になる。結合用の識別子と、モーフィングで要素を識別する ID は役割が異なる。

## タブバーとアクセサリビュー

- iPhone のレイアウトで最小化が有用な場合は、`TabView` に `.tabBarMinimizeBehavior(.onScrollDown)` を適用する。折りたたんだ後もナビゲーションを見つけやすくする。
- 再生操作などの常設コントロールには `.tabViewBottomAccessory { ... }` を使う。アクセサリ内で `tabViewBottomAccessoryPlacement` を読み、`.inline`、`.expanded`、未定義の `nil` に応じて表示を変える。
- 検索の配置には `.searchable` と検索タブのロールを使う。システムの外観を模倣するためだけに、独自の検索欄を重ねない。

## ツールバーと 27 専用の挙動

- `ToolbarSpacer(.fixed, placement:)` で操作を機能別に分ける。伸縮する間隔には `.flexible` を使う。独自の外観を持つツールバー項目には `sharedBackgroundVisibility(.hidden)` を設定できる。
- SwiftUI では条件分岐で `ToolbarItem` 自体を含めるかどうか決める。ラベルだけを隠すと、背景や空間が残る場合がある。UIKit では `UIBarButtonItem.isHidden` を使う。
- iOS 27 では `visibilityPriority` でオーバーフローの優先順位を指定し、常にメニューへ収める操作を `ToolbarOverflowMenu` に置く。末尾に残す操作には `.topBarPinnedTrailing` を使う。後二者はネイティブの macOS では利用できない。プラットフォーム表を参照する。
- iOS 27 の `TabRole.prominent` は末尾の独立したタブに使える。`TabView` 内でラベルを備えたタブを定義する。ロールだけではタブの宣言は完成しない。

新しいナビゲーションバーの挙動は可用性を確認して使い、iOS 26 でも同じコンテンツを表示する。実行時のチェックを記述するには、そのシンボルを認識するコンパイラーと SDK が必要である。

```swift
#if os(iOS)
struct AdaptiveNavigation<Content: View>: View {
    let content: Content

    var body: some View {
        NavigationStack {
            if #available(iOS 27.0, *) {
                content.toolbarMinimizationBehavior(
                    .onScrollDown, for: .navigationBar)
            } else {
                content
            }
        }
    }
}
#endif
```

## スクロール端と背景の延長

- スクロール端のスタイルは原則として自動のままにする。境界を明確にする必要がある場合だけ `.scrollEdgeEffectStyle(.hard, for: .top)` を使う。`nil` で標準に戻る。自動スタイルは OS によって変わり得るため、新しい OS に対応する際は明示的な指定を見直す。
- UIKit では `UIScrollView` から各辺の効果を設定する。独自のオーバーレイは `UIScrollEdgeElementContainerInteraction` に登録し、別のぼかしを重ねない。
- 背景画像に `backgroundExtensionEffect()` を適用してから、文字とコントロールをオーバーレイで追加する。操作部品を含むサイドバー全体には適用しない。UIKit と AppKit にはそれぞれ `UIBackgroundExtensionView` と `NSBackgroundExtensionView` がある。

## UIKit と AppKit への組み込み

UIKit のカスタムガラスでは、`UIVisualEffectView.contentView` に内容を追加し、効果のビューと内容の両方に制約を設ける。次の関数はサイズ制約を持つガラスラベルを作る。返されたビューの配置は呼び出し側で行う。

```swift
import UIKit

@available(iOS 26.0, *)
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

- 隣接する複数の UIKit 効果は、`UIGlassContainerEffect` を設定した `UIVisualEffectView` の `contentView` 内に配置する。`spacing` が相互作用の距離を決める。
- macOS では `NSGlassEffectView` の `contentView`、`style`、`cornerRadius` と必要に応じて `tintColor` を設定する。関連するビューは `NSGlassEffectContainerView` でまとめる。ボタンには `.glass` ベゼルの `NSButton` を優先する。
- AppKit の `effectIsInteractive` は macOS 27 の可用性を確認して使う。[設計上の検証項目](liquid-glass-design.md)に沿って性能とアクセシビリティを確認する。

## 公式資料

- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views)
- [Adopting Liquid Glass](https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass)
- [TabViewBottomAccessoryPlacement](https://developer.apple.com/documentation/swiftui/tabviewbottomaccessoryplacement)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Modernize your UIKit app — WWDC26](https://developer.apple.com/videos/play/wwdc2026/278/)
- [UIDesignRequiresCompatibility](https://developer.apple.com/documentation/bundleresources/information-property-list/uidesignrequirescompatibility)
