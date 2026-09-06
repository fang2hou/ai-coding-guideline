---
id: platforms/liquid-glass-patterns
lang: ja
version: 1
source-lang: en
status: active
digest: 81c26dfd
---

# Liquid Glass 実装パターン

## 判定

タスク指向のレシピ集。SwiftUI を最優先とし、UIKit と AppKit は該当箇所に示す。基線は iOS 26 / macOS 26（ガラス API 一式はここから利用可能）。iOS 27 専用 API は注記付きで、可用性ゲートが必要。コード断片は Apple のドキュメントが示すパターンに従い、シグネチャは Xcode 27 SDK と照合済みである。設計規則は[Liquid Glass デザイン](liquid-glass-design.md)、シンボル詳細は[Liquid Glass API リファレンス](liquid-glass-api.md)を参照。

## まずシステムのデフォルトを採用する

- 標準 chrome——ナビゲーションバー、タブバー、ツールバー、サイドバー、sheet、メニュー——は iOS 26+ で自動的に Liquid Glass を得る。ここにガラスのコードを書かない。
- システムの見た目と衝突する旧来のカスタマイズを削除する：バーの背景ビュー、影と境界線、区切り線のロジック、sheet の `presentationBackground`。
- カスタムの検索バーとアクセサリのスタイルを外す。永続的な機能だけがアクセサリビューを使い、バー項目は機能と利用頻度でグループ化する。
- `UIDesignRequiresCompatibility`（Info.plist）は従来の見た目を復元する——本当に両立できない設計のための最終手段であり、新規プロジェクトのデフォルトではない。

## ボタン

- 生のガラス効果でボタンを包むより、システムのガラスボタンスタイルを優先する：

```swift
Button("Save") { save() }
    .buttonStyle(.glass)             // 標準ガラス
Button("Delete") { delete() }
    .buttonStyle(.glassProminent)    // アクセント色の主要アクション
Button("Filter") { toggleFilters() }
    .buttonStyle(.glass(.clear))     // メディアリッチなコンテンツ上でのみ
```

- UIKit：`UIButton.Configuration`——`.glass()`、`.prominentGlass()`、`.clearGlass()`、`.prominentClearGlass()`。AppKit：`NSButton` の `.glass` ベゼルスタイル。
- 一つの表面につきティントを付けた主要アクションは一つまで。それ以外は無色のガラスのままにする（着色規則はデザイン文書を参照）。

## カスタムガラスビュー

- カスタムの浮遊コントロールに `glassEffect` を適用する。外観系モディファイアの後に置く——このモディファイアはビューの内容をキャプチャして描画に使う：

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect()                    // デフォルトは capsule 形状・.regular
```

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect(in: .rect(cornerRadius: 16))          // カスタム形状
```

```swift
Text("42")
    .font(.title)
    .padding()
    .glassEffect(.regular.tint(.orange).interactive()) // ティント + タッチ反応
```

- `.interactive()` はカスタムガラスにシステムのタッチ／ポインタ反応を与える。macOS では macOS 27 が必要（AppKit では `NSGlassEffectView.effectIsInteractive`）。
- clear はメディアリッチなコンテンツの上だけ。背景が明るい場合は調光レイヤーを置く：

```swift
Label("Flag", systemImage: "flag.fill")
    .padding()
    .glassEffect(.clear)
    .background(.black.opacity(0.3))
```

## GlassEffectContainer による融合

- 隣接するガラス要素は一つの `GlassEffectContainer` を共有しなければならない——グループ全体を一回の描画にまとめ、融合とモーフを可能にする。無い場合、隣接する効果のサンプリングが不一致になる。

```swift
GlassEffectContainer(spacing: 40) {
    HStack(spacing: 40) {
        toolButton("pencil")
        toolButton("eraser")
    }
}
```

- `spacing` を大きくすると融合の開始が早まる。コンテナの spacing がレイアウト間隔を超えると、静止状態でも形状が融合する。機能グループごとに一つのコンテナとし、画面上の効果数を抑える——ガラスの層にはそれぞれ描画コストがかかる。

## モーフとユニオン

- 各ガラス要素に安定した識別子を与え、階層変更をまたいでモーフさせる。状態切替をアニメーションで駆動する：

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

- `.matchedGeometry`（コンテナの spacing 距離内のデフォルト）は滑らかにモーフする。距離が離れる場合は `glassEffectTransition` で `.materialize` に切る。デフォルトアニメーションでは matchedGeometry がスケール・オフセットの演出を追加する——明示的なアニメーション（`.spring` または `nil`）を渡して除外する。
- `glassEffectUnion(id:namespace:)` は複数の要素を静止状態で一つの共有形状にマージする。全メンバーが同じ形状と同じ `Glass` バリアントを持つ必要がある。

## タブバーとツールバーの挙動

- 浮遊タブバーの最小化（iPhone のみ）：`.tabBarMinimizeBehavior(.onScrollDown)`。
- `ToolbarSpacer(.fixed)` / `.flexible` でツールバー項目を独立したガラスグループに分ける。`sharedBackgroundVisibility(.hidden)` で特定項目（プロフィール写真など）の共有ガラスを外す。
- iOS 27 はツールバーの適応性 API を追加する——可用性ゲートが必要：

```swift
StickerPageView().toolbar {
    ToolbarItemGroup { UndoButton(); RedoButton() }
        .visibilityPriority(.high)                 // 最後までオーバーフローしない
    ToolbarOverflowMenu {                          // 常にオーバーフローメニュー
        ChoosePhotoButton(); ExportButton()
    }
    ToolbarItem(placement: .topBarPinnedTrailing) { ShareButton() }
}
ScrollView { content }
    .toolbarMinimizationBehavior(.onScrollDown, for: .navigationBar) // iOS 27 での改名
Tab(role: .prominent) { CartTab() }                // 末尾にピン留めされたタブ
```

- UIKit 側の対応物：`UITabBarController.tabBarMinimizeBehavior = .onScrollDown`、`UIBarButtonItem.hidesSharedBackground = true`、iOS 27 の `UINavigationItem.navigationBarMinimization`。

## スクロールエッジエフェクト

- 浮遊 chrome の下での内容の溶け込み方を調整する：`.scrollEdgeEffectStyle(.soft, for: .top)`（iOS のデフォルト）、`.hard`（不透明な境界、主に macOS）、`nil` でシステムデフォルトに戻す。ビューごとに一つ。
- UIKit：`scrollView.topEdgeEffect.style = .hard`、`.isHidden = true`。スクロールビュー上のカスタムオーバーレイは自前のぼかしを構築せず `UIScrollEdgeElementContainerInteraction` で登録する。
- iOS 27 では `.automatic` が独自の見た目を持つ（soft/hard の切り替えをしない）——27 SDK 採用時に明示的な `.soft` オーバーライドを再評価する。

## 背景の延長

- 内容がウィンドウを満たさないとき（サイドバー、inspector レイアウト）、chrome の下へ視覚的に延長する：`.backgroundExtensionEffect()`。控えめに使う——背景インスタンスは一つ、テキストとコントロールはその上に重ねる。

```swift
SidebarContent()
    .backgroundExtensionEffect()
```

- UIKit：`UIBackgroundExtensionView`。AppKit：`NSBackgroundExtensionView`。

## UIKit のパターン

- カスタムガラスは `UIVisualEffectView` + `UIGlassEffect` で実装する。サブビューは `contentView` に追加し、effect view そのものには決して追加しない：

```swift
let effect = UIGlassEffect(style: .clear)   // または .regular
effect.tintColor = .systemBlue
effect.isInteractive = true
let glassView = UIVisualEffectView(effect: effect)
glassView.contentView.addSubview(label)     // contentView であり glassView ではない
```

- 隣接する複数のガラスビュー：`UIGlassContainerEffect`（`spacing` が融合距離を決める）を一つの `UIVisualEffectView` に宿らせ、個々のガラス effect view をその `contentView` にネストする。
- ツールバー項目の非表示は、内容ビューではなく項目そのもの（`ToolbarItem`／`UIBarButtonItem` の `isHidden`）で行う。

## AppKit のパターン

- macOS 26+ のカスタムガラスコンテナー：`NSGlassEffectView`——ガラス内に置かれる保証があるのは `contentView` だけ。`cornerRadius`、`tintColor`、`style` を設定する。
- macOS 27：`effectIsInteractive = true` で、コントロールを含む／支えるガラスにインタラクティブな反応を有効化する。
- 隣接するガラスは `NSGlassEffectContainerView`（`spacing`、デフォルト 0）でグループ化し描画パスをまとめる。ボタンは手作りのガラスではなく `.glass` ベゼルスタイルを使う。
