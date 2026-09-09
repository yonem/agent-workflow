# Workerタスク設定

各 worker の Codex タスクは、定型的な検証を担うTesterを除き、モデルを `gpt-5.6-terra`、推論レベルを `low` に統一する。Testerは定型的なテスト実行・結果整理を担うため、`gpt-5.6-luna` と `low` を標準設定とする。

| Worker | モデル | 推論レベル | 主な用途 |
| --- | --- | --- | --- |
| Planner | `gpt-5.6-terra` | `low` | 現状調査、要件分解、実装計画 |
| Implementer | `gpt-5.6-terra` | `low` | 承認済み計画の実装 |
| Tester | `gpt-5.6-luna` | `low` | 定型テスト、Lint、静的解析、ビルドの実行と結果整理 |
| Security Operator | `gpt-5.6-terra` | `low` | 秘密情報、外部操作、安全性の確認 |
| Reviewer | `gpt-5.6-terra` | `low` | 要件、設計、検証、安全性の独立レビュー |
| Documenter | `gpt-5.6-terra` | `low` | 判断、教訓、ライフサイクル改善の記録 |

## 運用ルール

- 役割ごとの標準モデル・推論レベルは本表を正本とし、タスク中に無断で変更しない
- Testerがテスト設計、失敗原因分析、複数資料の整合性判断を要する場合は、設定を変更せずReviewerへ引き継ぐ
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
- Reviewerの受入後にDocumenterへ接続し、Documenterは `operation-check.md`、`task-log.md`、汎用改善記録を更新してOwnerへ報告する。`operation-check.md`が存在しない、または固定構造を満たさない場合は、Documenter記録完了・Owner完了確認・次タスク切替へ進めない
- Reviewer受入と実運用後の効果確認は別判定とする。Documenterは受入後に、効果確認の状態、証跡、確認担当、次回確認日、継続条件、再評価条件を記録する。効果不足や承認範囲外の是正がある場合は、`rules/development-improvement-record.md` と `rules/thread-operation.md` に従って再評価または新しいTASK-xxxを開始する
- Documenterを移行先で省略する場合は、導入時に記録責任者と記録先を明示する。責任または記録先が不明な状態では有効化完了としない
- 停止条件、外部操作、承認、判断本文のOJ表示は各正本rulesに従う。プロジェクト固有のworker省略はこの共通サイクルへ追加しない
