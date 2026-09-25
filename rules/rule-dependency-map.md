---
scope: all-common-rules
status: active
---

# ルール依存関係一覧

## 目的

共通ルールの変更時に、正本と意味を共有する参照先を一意に特定し、更新漏れ・相反条件・責務重複を防ぐ。本書は値や手順の正本ではなく、影響確認に使う依存関係の正本である。各行の正本が優先し、本書へ本文を複製しない。

## 利用手順

1. 変更する要件に対応する行を選び、正本と全参照先を影響一覧へ記録する。
2. 正本と参照先の意味、責任、発火条件、停止条件、記録先を変更前後で照合する。
3. 参照先に変更が不要な場合も、不要である根拠を記録する。
4. 新しい共通要件または参照関係を追加・変更・削除する場合は、同じ変更で本書を更新する。依存関係が特定できない場合は、共通ルールの変更を完了扱いにしない。

個別task資料、履歴、ローカルルール、選択済みバックエンドの実体値は一覧の恒久的な依存先に含めない。バックエンド固有の操作は対応するアダプター規約だけに記録する。

## 共通工程・責務

| 要件 | 正本 | 参照先 | 更新責任 | 変更時の確認 |
| --- | --- | --- | --- | --- |
| IMはOwnerと人間の直接一問一答であり、自律工程の対象外 | `rules/automation-operation.md` | `AGENTS.md`、`README.md`、`rules/glossary.md`、`rules/im-template.md` | 共通ルール変更担当 | Coordinator・event・workerがIMへ介入しないこと |
| 自律工程は計画承認からReviewer受入まで | `rules/automation-operation.md` | `AGENTS.md`、`README.md`、`rules/execution-portability.md`、`rules/workflow-state-summary.md` | 共通ルール変更担当 | 終点と停止状態が一致すること |
| Reviewer受入後のDocumenter開始はOwner明示承認が必要 | `rules/automation-operation.md` | `rules/handoff-event-contract.md`、`rules/worker-task-settings.md`、`rules/worker-evidence.md`、`rules/workflow-integrity-check.md`、`rules/workflow-state-summary.md`、`worker-definitions/reviewer.md`、`worker-definitions/documenter.md`、`AGENTS.md`、`README.md` | 共通ルール変更担当 | Owner承認前にDocumenter接続可とする記載がないこと |
| task完了はReviewer受入、Documenter開始承認、Documenter記録、Owner完了判断を分離して確認する | `rules/thread-operation.md` | `rules/development-improvement-record.md`、`rules/workflow-integrity-check.md`、`rules/workflow-state-summary.md`、`rules/worker-report-template.md`、`worker-definitions/reviewer.md`、`AGENTS.md`、`README.md` | 共通ルール変更担当 | 完了根拠の欠落・順序逆転がないこと |
| 全依頼を簡易モードで入口判定し、昇格条件なら標準taskへ移す | `rules/quick-operation.md` | `rules/im-template.md`、`rules/task-initialization-and-requirement-gate.md`、`rules/automation-operation.md`、`rules/human-facing-documentation.md`、`rules/glossary.md`、`README.md` | 共通ルール変更担当 | 簡易対応が標準taskの保護対象を緩和しないこと |
| タスク外の共通ルール更新は、影響範囲を閉じた即時反映として扱う | `rules/quick-operation.md` | `rules/rule-refresh.md`、`rules/idempotency.md`、本書、`README.md` | 共通ルール変更担当 | 正本・直接参照先・再読込・旧表現・参照切れ・競合・冪等性を確認し、工程・承認・実行契約の変更を混在させないこと |
| 簡易判定の理由表示、複数対応の要約、標準IMへの最小引継ぎ、確認選定 | `rules/quick-operation.md` | `rules/requirement-definition-format.md`、`README.md`、`rules/glossary.md` | 共通ルール変更担当 | 計画必須4項目・OJ回答形式を代替せず、未確認の推測・不要な全体確認・重複記録がないこと |
| 複数の簡易対応候補だけを簡易IMへ記録し、新規標準task開始時に削除できる | `rules/quick-operation.md` | `rules/quick-im-template.md`、`rules/task-initialization-and-requirement-gate.md`、`rules/human-facing-documentation.md`、`rules/glossary.md` | 共通ルール変更担当 | 簡易IMを標準IM・task・履歴へ混在させず、削除条件を確認できること |

## 指摘・再試行・輻輳

| 要件 | 正本 | 参照先 | 更新責任 | 変更時の確認 |
| --- | --- | --- | --- | --- |
| 指摘を品質修正、接続同期差分、工程境界不備へ分類する | `rules/automation-operation.md` | `rules/worker-evidence.md`、`rules/issue-classification-cases.md`、`rules/worker-task-settings.md`、`worker-definitions/reviewer.md` | 共通ルール変更担当 | 結果パス不一致を接続同期差分へ含めず、同一事実が複数分類へ固定されないこと |
| 品質修正だけがImplementer→Reviewerの修正ラウンドと予算対象 | `rules/worker-evidence.md` | `rules/automation-operation.md`、`rules/worker-task-settings.md`、`worker-definitions/reviewer.md` | 共通ルール変更担当 | 接続同期差分・工程境界不備が予算を消費しないこと |
| 完了通知、担当result、handoff eventの証跡役割を分離する | `rules/handoff-event-contract.md` | `rules/automation-operation.md`、`rules/execution-portability.md`、`rules/workflow-consistency-check.md`、`rules/current-task-template.md`、`rules/worker-health-check.md`、選択バックエンド規約 | 共通ルール変更担当 | 完了通知がresultの内容・判定を上書きせず、通知だけで次工程を開始しないこと |
| 接続同期差分は一度の同期と同一工程境界の再照合だけ | `rules/automation-operation.md` | `rules/handoff-event-contract.md`、`rules/workflow-consistency-check.md`、`rules/current-task-template.md`、`rules/worker-evidence.md`、`rules/issue-classification-cases.md`、選択バックエンド規約 | 共通ルール変更担当 | task ID・担当責務・結果パス・内容フィンガープリントが一致する場合だけ同期し、再接続、再レビュー、event再発行を伴わないこと |
| 工程境界不備はeventをblockedにし、固定した再開条件とOwner判断へ分離する | `rules/handoff-event-contract.md` | `rules/automation-operation.md`、`rules/workflow-consistency-check.md`、`rules/current-task-template.md`、`rules/worker-evidence.md`、`rules/issue-classification-cases.md`、選択バックエンド規約 | 共通ルール変更担当 | 訂正責任者・訂正対象・再照合資料・Owner再判断要否を記録し、自動再接続・品質修正化・状態巻戻しがないこと |
| 同一原因の修正・再レビューには上限と集約エスカレーションを適用する | `rules/worker-evidence.md` | `rules/automation-operation.md`、`rules/worker-task-settings.md`、選択バックエンド規約 | 共通ルール変更担当 | 同一原因をattempt番号だけで新規扱いしないこと |

## event・実行バックエンド・移植性

| 要件 | 正本 | 参照先 | 更新責任 | 変更時の確認 |
| --- | --- | --- | --- | --- |
| handoff eventの状態、claim、重複抑止、復旧境界、既存台帳の移行 | `rules/handoff-event-contract.md` | `rules/automation-operation.md`、`rules/execution-portability.md`、`rules/workflow-consistency-check.md`、`rules/current-task-template.md`、`rules/worker-evidence.md`、選択バックエンド規約 | 共通ルール変更担当 | 同一eventの二重接続・状態巻戻しがなく、既存eventを削除・上書き・再発行せずに補完または限定停止できること |
| 共通工程は実行バックエンド中立で、結果eventを主経路とする | `rules/execution-portability.md` | `rules/automation-operation.md`、`rules/handoff-event-contract.md`、`rules/worker-task-settings.md`、`README.md` | 共通ルール変更担当 | 特定製品・API・常駐プログラムを共通前提にしないこと |
| バックエンド選択、能力不足時の停止、移植導入の確認 | `rules/execution-portability.md` | `rules/local-rules.md`、`rules/worker-task-settings.md`、`rules/workflow-integrity-check.md`、選択バックエンド規約 | 共通ルール変更担当 | 共通rulesへ固有実体値を固定しないこと |
| 定期監視は復旧監査だけであり、通常経路を代替しない | `rules/handoff-event-contract.md` | `rules/execution-portability.md`、`rules/automation-operation.md`、選択バックエンド規約 | 共通ルール変更担当 | 監視停止だけで通常event経路を停止しないこと |

## 整合性・再実行安全性

| 要件 | 正本 | 参照先 | 更新責任 | 変更時の確認 |
| --- | --- | --- | --- | --- |
| 正本、参照先、更新責任、更新境界の整合性確認 | `rules/workflow-integrity-check.md` | `rules/workflow-consistency-check.md`、`rules/operation-check-report.md`、`rules/rule-refresh.md`、`rules/README.md` | 共通ルール変更担当 | 参照切れ、複数正本、更新責任重複がないこと |
| 共通ルールの冪等性・再実行安全性 | `rules/idempotency.md` | `rules/rule-refresh.md`、`rules/worker-evidence.md`、`rules/handoff-event-contract.md`、`rules/issue-classification-cases.md` | 共通ルール変更担当 | 固定実体値、重複作成、状態巻戻し、不要操作がないこと |
| ルール更新後の再読込と影響確認 | `rules/rule-refresh.md` | `rules/README.md`、`AGENTS.md`、`rules/operation-check-report.md`、本書 | 共通ルール変更担当 | 本書の正本・参照先を影響一覧へ反映したこと |

## 維持管理

- 共通rules、README、AGENTS.md、worker定義、テンプレートを変更する担当は、変更前後に本書の該当行を確認する。
- 本書にない新しい共通要件は、依存関係を追加するまで恒久ルールとして完了扱いにしない。
- 参照先の削除・名称変更・統合時は、本書を先に更新せず、変更前後の対応関係と旧表現の扱いを影響記録へ残してから更新する。
- 本書を変更しても、個別task、履歴、ローカルルール、選択バックエンドの現行状態を自動変更しない。
