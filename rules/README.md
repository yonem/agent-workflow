# Rules運用

## 適用範囲

- `rules/` 配下のMarkdownファイルは、原則として共通ルールである
- workerは作業開始前に `rules/README.md` を確認し、適用対象のルールを読み取る
- 適用範囲が明記されていないルールは、全worker・全プロジェクトへ適用する
- 新しい共通ルールを追加するとき、worker定義の入力欄を更新する必要はない
- 計画関連Owner向け回答の対象・実行環境・ブランチ必須情報は `rules/plan-approval-required-info.md` を参照する
- Plannerへ指示する前の要件定義提案フォーマットと接続ゲートは `rules/requirement-definition-format.md` を参照する
- 実装媒体をルール・プロンプト・Markdownに限定する場合は `rules/implementation-medium.md` を参照する
- 新規運用開始時のローカル`history/`と`history/index.md`の初期化は `rules/history-initialization.md` を参照する
- 人間向け資料、`owner-jadge.md`、`task-progress.md`、`issue-memo.md`の共通形式・責務・作成・復元・移行手順は `rules/human-facing-documentation.md` を参照する
- 用語・標準的な意味・取り違え防止の正本は `rules/glossary.md` とし、用語そのものを対応キーとして参照する
- 次タスク以降の進行中タスク台帳は `threads/<thread-name>/docs/task-progress.md` を正本とし、worker固有結果は `threads/<thread-name>/result/`へ記録する。配置・履歴境界は `rules/thread-operation.md`を参照する
- `docs/issue-memo.md`と`docs/task-progress.md`は人間向けに整理した要約・判断・進捗とし、関連会話・状態変化の都度、既存項目を更新・統合する。AI・worker向けの詳細な引継ぎ・証跡は`result/`へ記録し、会話本文や`result/`本文をdocsへ機械的に複製しない。共通の形式・責任・作成・復元・移行境界は `rules/human-facing-documentation.md`を参照する
- 新機能追加、新規task作成、要件定義開始、承認済み計画へのtask追加の明示宣言だけで新規要件定義を発火する。相談・補足・確認・意見交換では管理資料を変更せず、曖昧な場合は確認まで停止する。新規taskではOwnerが現行taskの中断・終了または別project・別thread等での継続を選択するまで、要件記録・task-id採番・新規memo作成を開始しない。切替時はhistory退避、manifest・`history/index.md`・退避内容の照合後に旧docs/resultを初期化し、同一projectのactive重複・判定不能・部分成功は停止する
- Workerの状態確認結果はOwner明示トリガー時、または作業中のworker接続失敗を契機に切り替えた時だけ `threads/<thread-name>/docs/health-check.md`へ記録する。用途と境界は `rules/worker-health-check.md`を参照する
- 前回health-checkと現在worker一覧の差分確認は `rules/worker-connection-diff.md`を参照し、比較可能な2回目以降だけ専用resultへ記録する
- 履歴概要は `history/index.md`、詳細は各履歴の`manifest.md`、`docs/`、`result/`を参照する
- 改善事項の発見から効果確認、再評価、記録更新、移行先有効化までの共通サイクルは `rules/development-improvement-record.md` を正本とする
- 共通rules、worker定義、テンプレート、README、移行確認手順を変更する場合の影響確認は `rules/workflow-consistency-check.md`、`rules/workflow-integrity-check.md`、`rules/operation-check-report.md` を順に参照する

## 用語・共通認識の参照ゲート

- 用語の標準的な意味と取り違え防止の意味は、正本である`rules/glossary.md`を参照し、個別rules・README・worker定義へ同じ定義を重複掲載しない。
- 文脈指定がない場合は用語集の標準的な意味をデフォルト解釈とする。適用文脈が明示されている場合はその文脈を優先する。
- 未登録語、同音異義語、造語、略語、表記衝突、意味衝突、デフォルト解釈不能がある場合は、既存語へ推測置換せず、対象・候補・影響・Owner確認事項を記録して停止する。
- 用語集の初期作成・復元・移行先有効化は`rules/glossary.md`の手順に従い、配置、3列表、基準語4語、更新責任、参照先、停止条件を確認できるまで有効化完了としない。

## ルール追加時の永続性・移行性ゲート

- ルールや機能を追加するときは、現行タスクだけで有効になる一時対応ではなく、タスク完了後も残り続けるシステム機能として設計する。
- 他プロジェクトへagent-workflowを移行した時点から有効になるよう、共通ルール、worker定義、テンプレート、導入手順、記録先、適用条件を必要な範囲で整備する。
- 現行プロジェクト固有の接続設定、タスク固有の判断、特定の一時資料の存在を、移行先で機能が有効になるための前提にしない。
- 移行後の有効化条件、担当worker、入力、出力、完了条件、停止条件を確認できない場合は、ルール追加を完了扱いにせずOwnerへ報告する。
- このゲートは、個別タスクの完了判定とは別に、システム機能として継続利用できることを確認するために適用する。

## ルール変更の影響確認ゲート

共通資料を変更する場合は、変更前に直接対象、参照対象、記録対象、移行先導入対象を分類し、正本・参照先・更新責任・更新境界・旧表現・停止条件を確認する。変更後は同じ一覧を再確認し、旧表現、参照切れ、重複記載、相反する停止条件、共通仕様と固有設定の混在を判定する。

影響が特定できない、正本・責任・境界が不明、資料間に矛盾がある、または判定不能な場合は、推測で補正せず `changes.md` に期待値、実際値、根拠、影響、停止理由、再開条件を記録して停止する。履歴原本、legacy、backupは読み取り保全し、現行資料の変更対象へ自動的に含めない。

## タスク統合候補の粒度判定ゲート

タスク統合候補は、`history/index.md`の「新ルール統合判定台帳（候補判定の正本）」だけを通常の候補抽出元とし、task-id、タスク名、目的、対象リポジトリ、ローカルパス、ベースブランチ、機能・レイヤー、タスク概要の8項目だけを、対象リポジトリ一致、ローカルパス完全一致、ベースブランチ一致、機能・レイヤー分類、タスク概要比較の順に判定する。task-idは候補行の識別子であり、タスク名やtask-idの一致だけで統合しない。`状態`と`最終更新`、Owner判断、効果確認状態は統合判定の条件に使用せず、必要な場合は別途証跡として確認する。Owner承認に統合先、共通範囲、固有範囲、完了条件、履歴の扱い、競合時の優先方針がない場合は実統合へ進まない。

判定結果は`継続`、`統合候補`、`関連のみ`、`別タスク`、`判定不能`に分類する。`統合候補`はOwner承認前に確定せず、task-id、対象範囲、完了条件、履歴の扱いを変更しない。必須情報の欠落、矛盾、分類不能、Owner判断不明、history本体の横断検索や原本操作が必要な場合は停止し、影響と再開条件を記録する。

## 適用対象の判断

各ルールに適用範囲と状態が明記されている場合は、それに従う。

```text
scope: all-workers
scope: all-threads
scope: planner
scope: implementer
scope: tester
scope: security-operator
scope: reviewer
scope: documenter
status: active
status: deprecated
status: inactive
```

- `status: active` または状態の記載がないルールを適用する
- `deprecated` と `inactive` のルールは適用しない
- 特定のworker向けルールは、該当workerが適用する
- タスクで明示的に対象外とされたルールは適用しない
- 適用範囲や状態が不明で判断できない場合は、作業を開始せず親タスクへ報告する

## 優先順位

1. `AGENTS.md`
2. Ownerが承認した対象スレッドの `threads/<thread-name>/result/current-task.md`
3. `rules/` 配下の共通ルール
4. `worker-definitions/` のworker固有手順

同じ優先順位でルールが矛盾する場合は、作業を開始せず親タスクへ報告する。新しいルールを追加する場合は、既存ルールとの重複・矛盾・適用範囲を確認する。

## ルールファイルの記載例

```markdown
---
scope: all-workers
status: active
---

# ルール名
```
