# Worker定義

このディレクトリは、各workerの役割、入力、出力、判定、停止条件、後工程を共通仕様として案内する。個別の責務は各worker定義を正本とし、用語の標準的な意味と取り違え防止は`rules/glossary.md`、入力ゲート・証跡・Owner判断は`rules/worker-evidence.md`、履歴・復元・task境界は`rules/thread-operation.md`を参照する。

## Worker一覧

| worker | 定義 | 現行結果の保存先 |
| --- | --- | --- |
| Planner | `planner.md` | `threads/<thread-name>/result/plan.md` |
| Implementer | `implementer.md` | `threads/<thread-name>/result/changes.md` |
| Tester | `tester.md` | `threads/<thread-name>/result/test.md` |
| Security Operator | `security-operator.md` | `threads/<thread-name>/result/security.md` |
| Reviewer | `reviewer.md` | `threads/<thread-name>/result/review.md` |
| Documenter | `documenter.md` | `threads/<thread-name>/result/task-log.md` |

## 結果資料の扱い

通常のworker result本文は、見出し、項目順、表形式、必須列、末尾配置を固定しない。各workerは、task-id、タスク名、元スレッド名、worker名、開始・完了日時、正本結果パス、判定、実施または確認内容、未確認事項、次工程を対応付けて記録する。Owner判断が発生した場合は、判断ID、回答、ステータス、根拠、未回答時の停止・再開条件を記録する。

`threads/<thread-name>/result/current-task.md`はPlanner接続用の特別な入力資料であり、`rules/current-task-template.md`の形式を維持する。`threads/<thread-name>/docs/health-check.md`は人間向けworker状態確認の正本で、worker resultの代替ではない。旧result、legacy、backup、history原本は、Owner指定なしに現行結果へ移行・統合・上書きしない。

## 接続順序

```text
current-task.md
  ↓
Planner → Owner承認
  ↓
Implementer
  ↓
Tester / Security Operator（接続しない場合はOwner承認済みの標準設定を確認）
  ↓
Reviewer
  ↓
Documenter
```

## 実行モデルと親子連携

人間のタスク進行上の対話窓口はOrchestratorとする。Ownerは即時反映、タスク化しない相談、人間の承認判断を扱う別チャットである。標準SubagentはDeliveryと独立Reviewerだけとし、いずれもOrchestratorのsessionに属する。DeliveryはPlanner、Implementer、Documenterの論理責務を順番に担当する。独立したサイドバーチャットをworker実行単位として新規作成・再利用してはならない。

worker役割は論理的な責務であり、6件のSubagentを常設または事前作成しない。標準は同一taskで再利用するDelivery 1件と独立Reviewer 1件だけである。担当resultとtask-logの照合後は完了済みSubagentをcloseし、同じDeliveryを次の論理責務へresumeする。TesterまたはSecurity Operatorの追加は、計画に必要性を記載しOwnerが例外承認した場合に限る。接続上限・生成・closeの詳細は`rules/automation-operation.md`を正本とする。

各Subagentは担当resultと`task-log.md`へ結果を記録して親sessionへ完了を返す。Orchestratorは完了通知を受けて結果資料・入力ゲートを照合し、問題がなければ次workerを自動接続する。WorkerからOwnerチャットまたは別workerチャットへ任意の可視メッセージを送ることは、正規のエスカレーション経路ではない。

計画承認後は、OrchestratorがImplementer→必要なTester / Security Operator→Reviewer→Documenterを自動で接続する。停止するのは、担当resultの判定、必須証跡、Owner判断、入力ゲートに未確認または不一致がある場合だけとする。通知経路の失敗だけで工程を停止しない。人間への承認要求はOrchestratorチャット上で提示する。

各workerの完了報告は担当resultへ保存し、親sessionへ判定、結果パス、未確認事項、次workerを返す。Owner判断残件、入力不一致、正本不明、参照切れ、履歴操作要求、移行先の責任者・記録先・停止条件の未確認がある場合は次工程へ接続しない。

親sessionへの完了返却は、共通`rules/worker-report-template.md`の安全形式に限定する。判定、結果、実施内容、未確認事項、次工程、エスカレーション要否だけを記載し、絶対パス、Markdownリンク、詳細Owner承認、秘密情報、不要な内部workflow情報を含めない。

各workerは作業開始前に`rules/glossary.md`を参照し、未登録語、同音異義語、造語、略語、意味衝突、デフォルト解釈不能を推測で確定せず、結果へ記録してOwner確認まで停止する。
