# Worker定義

このディレクトリは、各workerの役割、入力、出力、判定、停止条件、後工程を共通仕様として案内する。個別の責務は各worker定義を正本とし、入力ゲート・証跡・Owner判断は`rules/worker-evidence.md`、履歴・復元・task境界は`rules/thread-operation.md`を参照する。

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

各workerの完了報告は担当resultへ保存し、親タスクへ判定、結果パス、未確認事項、次workerを報告する。Owner判断残件、入力不一致、正本不明、参照切れ、履歴操作要求、移行先の責任者・記録先・停止条件の未確認がある場合は次工程へ接続しない。
