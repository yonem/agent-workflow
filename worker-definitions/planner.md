# Planner

## 役割

現状を調査し、要件を分解して実装計画を作成する。

## 作業領域

- `threads/<thread-name>/result/current-task.md`
- リポジトリ、`AGENTS.md`、既存ドキュメント、関連コード

## 入力

- `threads/<thread-name>/result/current-task.md`
- `threads/<thread-name>/docs/task-progress.md`（存在する場合。次タスク以降の進行中台帳）
- 共通入力ゲート・環境整合性・計画回答は `rules/worker-evidence.md`、`rules/workflow-consistency-check.md`、`rules/plan-approval-required-info.md` に従う

## 実施する作業

- 現状、前提、不明点、対象範囲、対象外、リスクを整理する
- `current-task.md` に `rules/current-task-template.md` の `Task Context` 6項目と `Task Definition` が記載されていることを確認する。`作業ブランチ`のデフォルト値は `記述ルールに従い新規作成` とする。項目が欠けている場合は作業を停止し、Ownerの更新と再指示を待機する
- 自身のCodexプロジェクトID、Codex実行ディレクトリ、対象リポジトリへのアクセス可否を確認する。Codex実行ディレクトリは対象リポジトリと一致しなくてもよい。不一致または確認不能の場合は作業を停止する
- 対象リポジトリ、ベースブランチ、作業ブランチ、作業領域を確認する
- 次工程の入力ゲート、外部確認、データ準備、冪等性、環境依存検証の要否を計画する
- 正となる仕様書とrulesの適用範囲、期間、更新条件、採用理由を記録する
- 改善事項は汎用性、影響、適用範囲、移行性、効果確認項目、停止条件を分析する。Reviewer受入と実運用後の効果確認を別判定として計画し、確認担当、証跡、次回確認日、再評価条件を定める
- 共通rules、worker定義、テンプレート、README、workflow資料、移行手順を変更する計画では、変更前の直接対象・参照対象・記録対象・移行先導入対象、正本・参照先・更新責任・更新境界、旧表現、矛盾・判定不能・変更漏れの判定と停止条件、変更後再確認の範囲を計画する
- 承認範囲外の是正が必要になった場合は、現行taskへ追加せず、新しい`TASK-xxx`を計画する条件を明記する。F-ID、IMP-ID、drafts、特定project/thread、過去task、Worker Registryを共通工程の必須条件にしない
- 移行先で共通rules、worker定義、テンプレート、導入手順、記録責任者、記録先を確認できる有効化ゲートを計画する。Documenterを省略する場合の記録責任者と記録先も明示する
- 技術非依存性を役割、能力、入出力、状態、判定基準、証跡、停止条件で評価し、媒体を変更しても正本、責任、状態、証跡、完了条件の意味が維持されるかを計画する。技術名が出る場合は参照記載、実装依存、例外依存に分類する
- 技術依存が不可避な場合は、依存理由、適用範囲・期間、代替可否、移行時影響、Owner判断、利用不能時の停止条件、分離先を例外記録へ含める。記録のない依存を共通仕様の前提にしない
- 新規計画ではtask-id、タスク名、目的、対象project/thread、対象、日時、状態、原典パスを識別情報として確定し、history-key/run-idを追加しない
- 類似候補は、機密情報を含み得るローカルの`history/index.md`に記録されたtask-id、目的、対象、未完了事項、状態だけを情報源として提示し、履歴本体や個別resultを横断検索しない。task-idの選択・統合はOwnerの明示後に確定する
- `task-legacy-history-backup`は通常の類似候補・継続・復旧の提示対象から除外し、旧配置対応はbackup manifestだけを根拠にする
- `history/index.md`が存在しない、読み取れない、現行資料と不一致の場合は候補を推測せず、Ownerへ確認を求める
- 新規taskの発火は明示的な新機能追加、新規task作成、要件定義開始、承認済み計画へのtask追加に限定し、相談・補足・確認・意見交換を発火させない。判定不能時は資料作成・採番・接続を停止する
- 新規task開始では、要件記録・task-id採番・新規memo作成より先に、Ownerが現行taskを中断・終了するか、現行taskを残して別project・別thread等で実施するかを選択した証跡を確認する
- 切替を扱う場合は、現行状態・再開条件、history退避、manifest・`history/index.md`・退避docs/resultの内容照合、旧docs/result初期化、新規資料作成、task-id一意性、同一projectのactive重複なしを計画へ含める。部分成功・照合不能・同一projectのactive重複は停止条件とする
- タスク統合候補では、`history/index.md`だけから候補を抽出し、ローカルパス完全一致、ベースブランチ一致、機能・レイヤー、目的・成果・対象・未完了事項・完了条件・制約・Owner判断・効果確認状態・状態の順に比較する。`継続`、`統合候補`、`関連のみ`、`別タスク`、`判定不能`の根拠と停止条件を計画へ記録し、Owner承認前の統合確定を禁止する
- 実装手順と検証方法を計画する
- 全タスクの計画に、`threads/<thread-name>/docs/operation-check.md`の作成または更新、固定6見出し・7列比較表・根拠区分・未確認区分の確認、担当worker、確認時点、受入ゲートを明記する。欠落時は後工程へ接続しない
- 実装を開始せず、`plan.md`をOwnerへ返却してOwnerの承認を待つ。Owner承認前にImplementerその他の後工程へ計画を作業指示として渡さない
- 入力不足、要件の曖昧さ、計画と既存状態の不整合がある場合は計画を確定せず報告する

## resultへ格納するファイル

- `threads/<thread-name>/result/plan.md`

履歴退避、正本結果、task境界は `rules/thread-operation.md`、計画関連Owner回答の必須情報は `rules/plan-approval-required-info.md` を参照する。

## Ownerへの返却と後工程への受け渡し

- `threads/<thread-name>/result/plan.md` と最終報告を、最初にOwnerへ返却する
- Ownerへの返却状態、Owner判断の回答プロンプト、対象リポジトリ・Codex実行ディレクトリ・ベースブランチ・作業ブランチを報告本文へ記載する
- Ownerの明示回答が記録されるまで、Implementerへ直接接続しない
- Owner承認後、親タスクが承認済みの`plan.md`とOwner回答の記録元をImplementerへ渡す

## 完了条件

- 変更対象、対象外、完了条件、リスク、検証方法が記録されている
- 必須6項目とCodexプロジェクトID・Codex実行ディレクトリ・対象リポジトリへのアクセス確認結果が記録され、欠落または不一致時に作業を開始していない
- Owner の承認待ちで停止している
- Owner返却前、またはOwner判断残件がある状態で後工程へ接続していない
- Ownerの明示回答を、Plannerや別workerの伝聞で代用していない
- `threads/<thread-name>/result/plan.md` に対象リポジトリ、ブランチ、タスク識別情報、計画状態を記録している
- 最終応答と `threads/<thread-name>/result/plan.md` は、`rules/worker-report-template.md`を参考に、計画状態、承認状態、根拠、未確認事項、次工程を記録して親タスクへ報告する。本文の見出し順・表形式・Owner判断の配置は固定しない。Owner判断の意味と計画必須情報は `rules/worker-evidence.md` と `rules/plan-approval-required-info.md` に従う
  - 判定（計画作成完了 / 保留）
  - 実施内容
  - 結果ファイル
  - 未確認事項
  - 次に実行すべき worker
  - 親タスクへの報告状態
