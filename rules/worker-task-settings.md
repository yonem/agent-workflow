# Workerタスク設定

各 worker の Codex タスクは、モデルを `gpt-5.6-luna` に統一する。推論レベルは役割に応じて次のとおりとする。

| Worker | モデル | 推論レベル | 主な用途 |
| --- | --- | --- | --- |
| Planner | `gpt-5.6-luna` | `high` | 現状調査、要件分解、実装計画 |
| Implementer | `gpt-5.6-luna` | `high` | 承認済み計画の実装 |
| Tester | `gpt-5.6-luna` | `medium` | テスト、Lint、静的解析、ビルドの検証 |
| Security Operator | `gpt-5.6-luna` | `high` | 秘密情報、外部操作、安全性の確認 |
| Reviewer | `gpt-5.6-luna` | `high` | 要件、設計、検証、安全性の独立レビュー |
| Documenter | `gpt-5.6-luna` | `medium` | 判断、教訓、ライフサイクル改善の記録 |

## 運用ルール

- worker 間でモデルを変更しない
- 推論レベルを変更する場合は、変更理由をルートの `development-improvement.md` に記録する
- タスク作成時は該当する `worker-definitions/*.md` とこの設定を入力として確認する
- 設定変更後に作成するタスクから新しい設定を適用する

## 標準worker接続サイクル

標準サイクルは次の順序とする。各workerの固有責務・結果ファイル・判定値は `worker-definitions/`、入力ゲートと証跡は `rules/worker-evidence.md`、報告形式は `rules/worker-report-template.md`、履歴とtask境界は `rules/thread-operation.md`を正本とする。

```text
Planner → Owner承認 → Implementer → Tester / Security Operator（並行可）
        → Reviewer → Documenter → Ownerのマージ・リリース判断
```

- PlannerはOwner承認前に実装を開始しない。各workerは接続前に `rules/workflow-consistency-check.md` の入力ゲートを通る
- TesterとSecurity Operatorの両方の完了後にReviewerへ接続する
- Reviewerが修正依頼と判定した場合は、指摘をImplementerへ戻し、必要なTester、Security Operator、Reviewerの確認を再実行する
- Reviewerの受入後にDocumenterへ接続し、Documenterは `task-log.md` と汎用改善記録を更新してOwnerへ報告する
- Reviewer受入と実運用後の効果確認は別判定とする。Documenterは受入後に、効果確認の状態、証跡、確認担当、次回確認日、継続条件、再評価条件を記録する。効果不足や承認範囲外の是正がある場合は、`rules/development-improvement-record.md` と `rules/thread-operation.md` に従って再評価または新しいTASK-xxxを開始する
- Documenterを移行先で省略する場合は、導入時に記録責任者と記録先を明示する。責任または記録先が不明な状態では有効化完了としない
- 停止条件、外部操作、承認、判断本文のOJ表示は各正本rulesに従う。プロジェクト固有のworker省略はこの共通サイクルへ追加しない
