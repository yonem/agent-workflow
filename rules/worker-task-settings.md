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
- 推論レベルを変更する場合は、変更理由を `docs/development-improvement.md` に記録する
- タスク作成時は該当する `worker-definitions/*.md` とこの設定を入力として確認する
- 設定変更後に作成するタスクから新しい設定を適用する
