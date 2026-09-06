# Glossary

Canonical terminology across the three language trees. Translations of
`guidelines/**` must use these equivalents. Product names (mise, pnpm, oxlint,
oxfmt, uv, ruff, prek, Cocogitto, Vite, Next.js, Tailwind CSS, shadcn/ui,
FastAPI, Pydantic, Uvicorn, Databricks, GitHub Actions, Go, Rust, ...) and
industry-standard terms (Conventional Commits, monorepo, ...) stay in
English in all languages.

When a translation introduces a recurring term not listed here, add a row
to this table in the same change.

| en                                 | zh                | ja                          |
| ---------------------------------- | ----------------- | --------------------------- |
| guideline                          | 指南              | ガイドライン                |
| toolchain                          | 工具链            | ツールチェーン              |
| linter                             | Lint 工具         | リンター                    |
| formatter                          | 格式化工具        | フォーマッター              |
| package manager                    | 包管理器          | パッケージマネージャー      |
| pre-commit hook                    | 预提交钩子        | プリコミットフック          |
| architecture decision record (ADR) | 架构决策记录(ADR) | アーキテクチャ決定記録(ADR) |
| single source of truth             | 单一事实来源      | 信頼できる唯一の情報源      |
| validation                         | 校验              | 検証                        |
| type checking                      | 类型检查          | 型チェック                  |
| dependency                         | 依赖              | 依存関係                    |
| breaking change                    | 破坏性变更        | 破壊的変更                  |
| secret                             | 敏感凭据          | シークレット                |
| task runner                        | 任务运行器        | タスクランナー              |
| root cause                         | 根本原因          | 根本原因                    |
| code review                        | 代码评审          | コードレビュー              |
| pull request                       | pull request      | プルリクエスト              |
| AI agent                           | AI Agent          | AI エージェント             |
| quality gates                      | 质量门禁          | 品質ゲート                  |
| invariant                          | 不变量            | 不変条件                    |
| coverage                           | 覆盖率            | カバレッジ                  |
| over-engineering                   | 过度设计          | 過剰な設計                  |
| literal value                      | 字面值            | リテラル値                  |
| test fixture                       | 测试夹具          | テストフィクスチャ          |
| unit test                          | 单元测试          | ユニットテスト              |
| E2E test                           | E2E 测试          | E2E テスト                  |
| workflow                           | 工作流            | ワークフロー                |
| identifier                         | 标识符            | 識別子                      |
| lockfile                           | lockfile          | ロックファイル              |
| framework-agnostic                 | 框架无关          | フレームワーク非依存        |
| long-term support (LTS)            | LTS               | LTS                         |
| hallucination                      | 幻觉              | 幻覚                        |
| secret scanning                    | 凭据扫描          | シークレットスキャニング    |
| design token                       | 设计令牌          | デザイントークン            |
| tradeoff                           | 权衡              | トレードオフ                |
| workload                           | 工作负载          | ワークロード                |
| discriminated union                | 可辨识联合        | 判別可能なユニオン          |
| barrel file                        | 桶文件            | バレルファイル              |
| Atomic Design Methodology          | 原子设计方法论    | アトミックデザイン手法      |
| least privilege                    | 最小权限          | 最小権限                    |
| branch protection                  | 分支保护          | ブランチ保護                |
| status check                       | 状态检查          | ステータスチェック          |
| script injection                   | 脚本注入          | スクリプトインジェクション  |
| pipeline                           | 流水线            | パイプライン                |
| stage                              | 阶段              | ステージ                    |
| whitelist                          | 白名单            | ホワイトリスト              |
| quick start                        | 快速开始          | クイックスタート            |
| screenshot                         | 截图              | スクリーンショット          |
| badge                              | 徽章              | バッジ                      |
| tagline                            | 标语              | タグライン                  |
| table of contents                  | 目录              | 目次                        |
| license                            | 许可证            | ライセンス                  |
| feature                            | 特性              | 機能                        |
| concept                            | 概念              | 概念                        |
| how-to guide                       | 操作指南          | 利用方法                    |
| reference                          | 参考              | 参考資料                    |
| explanation                        | 解释              | 解説                        |
| performance                        | 性能              | 性能                        |
| experiments                        | 实验              | 実験                        |
| design language                    | 设计语言          | デザイン言語                |
| material                           | 材质              | マテリアル                  |
| control                            | 控件              | コントロール                |
| navigation bar                     | 导航栏            | ナビゲーションバー          |
| tab bar                            | 标签栏            | タブバー                    |
| toolbar                            | 工具栏            | ツールバー                  |
| sidebar                            | 侧边栏            | サイドバー                  |
| safe area                          | 安全区            | セーフエリア                |
| scroll edge effect                 | 滚动边缘效果      | スクロールエッジエフェクト  |
| concentricity                      | 同心性            | 同心性                      |
| accessibility                      | 无障碍            | アクセシビリティ            |

## Forbidden renderings

Machine-enforced by `tools/check-docs.ts`: these strings must not appear in
the body of any document in the corresponding language tree. When you fix a
recurring mistranslation, add a row here so it cannot come back.

| English term    | Lang | Forbidden | Use instead    |
| --------------- | ---- | --------- | -------------- |
| workflow        | zh   | 工作流程  | 工作流         |
| pull request    | zh   | 拉取请求  | pull request   |
| secret scanning | zh   | 秘密扫描  | 凭据扫描       |
| quality gate    | zh   | 质量门槛  | 质量门禁       |
| pull request    | ja   | プル要求  | プルリクエスト |
