---
scope: all-threads
status: active
---

# current-task.md テンプレート

ファイル側threadの初期登録だけでは`current-task.md`をtask入力として確定しない。要件定義開始が明示され、task-id、対象、対象外、完了条件、停止条件、実行環境が確定した後に作成・更新する。初期登録段階で必要な入力は未指定または空として扱い、会話・worker・project・識別子を推測しない。

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
- 作業ブランチ：Planner接続時は未決定可。Implementer接続前に確定・実体照合

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

## Worker Registry

Worker Registryは、接続前の指定設定と、接続後に取得できる実測値・状態を分けて記録する正本である。ユーザー向けCodexスレッド／会話、ファイル側thread、同一task内のSubagent、Codex projectを混同しない。接続前の指定設定が確認できない場合は接続しない。指定済みSubagentの実測値を取得できない場合は`未確認`として記録するが、それだけで自律オーケストレーション・受入・完了を停止せず、親チャット、会話、ファイル側threadを実Subagentの代替にしない。

| 論理責務 | Subagent種別 | 実Subagent ID | 親Orchestrator session | 所属Codex project ID | 指定モデル・推論 | 実測モデル・推論 | 実行ディレクトリ | 状態 | 確認日時・証跡 | 更新責任者 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Planner | Delivery |  |  |  | `gpt-5.6-luna` / `low` |  |  | 準備中 |  | Orchestrator |
| Implementer | Delivery（Planner完了後に明示resume） |  |  |  | `gpt-5.6-luna` / `low` |  |  | 準備中 |  | Orchestrator |
| Documenter | Delivery（Reviewer受入後に明示resume） |  |  |  | `gpt-5.6-luna` / `low` |  |  | 準備中 |  | Orchestrator |
| Reviewer | 独立Reviewer Subagent |  |  |  | `gpt-5.6-luna` / `low` |  |  | 準備中 |  | Orchestrator |
| Tester | 例外Subagent |  |  |  | `gpt-5.6-luna` / `low` |  |  | 対象外 |  | Orchestrator |
| Security Operator | 例外Subagent |  |  |  | `gpt-5.6-luna` / `low` |  |  | 対象外 |  | Orchestrator |

### 親session返却先

| 項目 | 値 |
| --- | --- |
| 親Orchestrator session |  |
| 報告対象 | 親Orchestrator session。Owner判断はOrchestratorがOwnerへ提示する |

workerは親Orchestrator sessionの実Subagentとして接続されたことを確認する。標準SubagentはDelivery 1件と独立Reviewer 1件であり、Deliveryは同一task内で明示resumeしてPlanner、Implementer、Documenterの論理責務を順次担当する。Reviewerは独立した実Subagentである。ユーザー向けCodexスレッド／会話ID、ファイル側thread名、Codex project ID、親session IDは実Subagent IDの代替に使用しない。親Orchestrator sessionまたは接続前の指定設定が確認不能なら接続を開始しない。実測値が取得不能なら未確認として証跡を記録するが、それだけで自律オーケストレーション・受入・完了を停止しない。TesterとSecurity Operatorは、承認済み計画に外部接続、高リスク変更、または独立検証が明記され、Owner例外承認がある場合だけ`対象外`から更新する。
```

## 記載ルール

- `Task Context`の6項目は必須とする
- `CodexプロジェクトID`、スレッド名、対象リポジトリは、実際の所属・対象と一致させる
- `Codex実行ディレクトリ`はworkerの実行場所であり、対象リポジトリと異なっていてよい
- Planner接続時の`作業ブランチ`は未決定または候補を許容する。Implementer接続前にOwner承認済み計画と実体を照合し、確定値を記録する。
- Worker Registryの状態は`準備中`、`接続済み`、`完了・close済み`、`失敗`、`中断`、`対象外`を区別する。不明・重複候補・確定前は再作成せずOwner確認へ停止する。
- 新規作成した作業ブランチ名は、作成後に `current-task.md` と担当結果ファイルへ記録する
- `Task Definition`はPlannerが計画を作成できる具体性で記載する
- `Task Lifecycle`はタスクの開始、完了、退避、復旧を追跡するために記載する。履歴へ退避しない場合も、未実施理由を結果ファイルへ記録する
- Worker RegistryはOrchestratorがDelivery、Reviewer、例外workerの接続・再利用方針を承認済み計画と照合して更新する。Deliveryの論理責務行は同一実Subagent IDを共有してよい。各Subagentは開始前に指定モデル・推論を確定して接続ツールへ渡し、確定不能なら接続・作業を停止する。接続後に実測値が取得不能またはworker報告と異なる場合は、接続ツールの実測値を正本へ同期し、差分を履歴注記として記録して継続する。接続ツール結果自体のproject・親session・実行ディレクトリ不一致は停止する。
- Reviewerの実Subagent IDはレビューAttempt単位で記録する。再接続後は現行AttemptのIDをWorker Registryと現行`review.md`へ記録し、過去AttemptのIDは履歴として保持する。過去AttemptとのID差分、またはID変更だけを理由に工程状態を不一致・未受入へ戻さない。
- `current-task.md`は識別情報・対象・Task Lifecycleの正本とし、共通進捗は`task-progress.md`、承認済み計画は`result/plan.md`、Implementer結果は`result/changes.md`、Reviewer判定は`result/review.md`を正本とする。`history/index.md`は要約・候補の参照であり、これらのresultを代替しない
- 各資料の更新責任と更新境界は`rules/thread-operation.md`の資料マップに従う。正本候補が複数、参照切れ、責務重複、更新境界不明の場合は推測で補正せず停止する
- `タスクID`は新規タスクの論理識別子として必須とし、Task DefinitionとTask Lifecycleで同じ値を記録する。タスク名だけで別タスクを統合しない
- 新規運用ではhistory-key/run-idを記載しない。旧履歴に残る値は読み取り専用のlegacy情報としてのみ扱い、新規資料へ複製しない
- 新規履歴は`history/<task-id>/`へ保存し、同じtask-idの重複履歴は上書きせずOwnerへ報告する
- 旧timestamp形式historyを整理する場合は、`task-legacy-history-backup`をバックアップ領域の管理識別子として記載し、旧タスク個別へ論理task-idを遡及付与しない。旧配置対応はmanifestと`history/index.md`へ記録する
- task-idは候補資料の識別子、ファイル名、日時だけを根拠にせず、実際に作成・確定された論理タスク台帳から連番で採番する。台帳上の欠落・誤採番・重複・未確定は推測で変更せずOwnerへ報告する
- 新規タスクへ切り替える場合は、旧current-task、task-progress、現行result、退避manifest、history/index行の対応を確認し、旧資料の退避完了前にtask-progressや現行resultを新task-idへ変更しない
- タスク切替時のOwner判断は、既承認・継承可能、範囲変更による再判断、未回答、新規に分類する。既承認・継承可能な判断は再掲せず、同一論点は既存OJ-IDを継続し、別論点だけ新規OJ-IDを採番する
- 欠落、不一致、解釈不能な記載がある場合、Plannerは作業を開始せずOwnerへ報告する

新規taskへ切り替えた`current-task.md`には、可能な範囲で旧taskの状態・再開条件、Ownerの扱い選択、退避manifestと`history/index.md`の参照先、退避内容の照合結果、新規task-idの一意性確認を記録する。切替前の旧資料が退避・照合済みでない場合は、新しいtaskの入力へ変更せず停止する。同一projectのactive重複、照合不能、部分成功は未確認として扱い、再開条件が満たされるまでPlannerへ接続しない。
