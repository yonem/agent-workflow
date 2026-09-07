---
scope: all-threads
status: active
---

# current-task.md テンプレート

各スレッドの `threads/<thread-name>/result/current-task.md` は、次の形式を基本とする。これはPlanner接続用の特別な入力資料であり、通常のworker result本文自由化の対象外とする。

このファイルは、タスクの識別情報・Task Definition・Task Lifecycleの入力スキーマを正本として定義する。進行中タスクの共通台帳は次タスク以降`threads/<thread-name>/docs/task-progress.md`とし、実行環境との照合は `rules/workflow-consistency-check.md`、計画関連Owner回答の記録は `rules/plan-approval-required-info.md`、履歴と復旧の手順は `rules/thread-operation.md`に従う。

```markdown
# Current Task

## Task Context

- スレッド名：
- CodexプロジェクトID：
- 対象リポジトリ：
- Codex実行ディレクトリ：
- ベースブランチ：
- 作業ブランチ：記述ルールに従い新規作成

## Task Definition

- タスクID：
- タスク名：
- 目的：
- 対象：
- 対象外：
- 完了条件：
- リスク：
- 検証方法：

## Task Lifecycle

- 論理タスクID：
- 開始日時（Asia/Tokyo）：
- 完了日時（未完了時は空欄）：
- 退避元CodexプロジェクトID：
- 退避元スレッド名：
```

## 記載ルール

- `Task Context`の6項目は必須とする
- `CodexプロジェクトID`、スレッド名、対象リポジトリは、実際の所属・対象と一致させる
- `Codex実行ディレクトリ`はworkerの実行場所であり、対象リポジトリと異なっていてよい
- `作業ブランチ`は、既存ブランチを使用する場合を除き `記述ルールに従い新規作成` とする
- 新規作成した作業ブランチ名は、作成後に `current-task.md` と担当結果ファイルへ記録する
- `Task Definition`はPlannerが計画を作成できる具体性で記載する
- `Task Lifecycle`はタスクの開始、完了、退避、復旧を追跡するために記載する。履歴へ退避しない場合も、未実施理由を結果ファイルへ記録する
- `current-task.md`は識別情報・対象・Task Lifecycleの正本とし、共通進捗は`task-progress.md`、承認済み計画は`result/plan.md`、Implementer結果は`result/changes.md`、Reviewer判定は`result/review.md`を正本とする。`history/index.md`は要約・候補の参照であり、これらのresultを代替しない
- 各資料の更新責任と更新境界は`rules/thread-operation.md`の資料マップに従う。正本候補が複数、参照切れ、責務重複、更新境界不明の場合は推測で補正せず停止する
- `タスクID`は新規タスクの論理識別子として必須とし、Task DefinitionとTask Lifecycleで同じ値を記録する。タスク名だけで別タスクを統合しない
- 新規運用ではhistory-key/run-idを記載しない。旧履歴に残る値は読み取り専用のlegacy情報としてのみ扱い、新規資料へ複製しない
- 新規履歴は`history/<task-id>/`へ保存し、同じtask-idの重複履歴は上書きせずOwnerへ報告する
- 旧timestamp形式historyを整理する場合は、`task-legacy-history-backup`をバックアップ領域の管理識別子として記載し、旧タスク個別へ論理task-idを遡及付与しない。旧配置対応はmanifestと`history/index.md`へ記録する
- task-idは候補資料の識別子、ファイル名、日時だけを根拠にせず、実際に作成・確定された論理タスク台帳から連番で採番する。既存TASK-001、TASK-002、TASK-003の欠落・誤採番・重複・未確定は推測で変更せずOwnerへ報告する
- 新規タスクへ切り替える場合は、旧current-task、task-progress、現行result、退避manifest、history/index行の対応を確認し、旧資料の退避完了前にtask-progressや現行resultを新task-idへ変更しない
- タスク切替時のOwner判断は、既承認・継承可能、範囲変更による再判断、未回答、新規に分類する。既承認・継承可能な判断は再掲せず、同一論点は既存OJ-IDを継続し、別論点だけ新規OJ-IDを採番する
- 欠落、不一致、解釈不能な記載がある場合、Plannerは作業を開始せずOwnerへ報告する
