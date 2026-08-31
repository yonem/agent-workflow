# エージェントワークフロー

## 役割

| 役割 | 主な責任 | 主な成果物 |
| --- | --- | --- |
| Owner | 目的、制約、優先順位、最終判断 | 要件、承認、判断 |
| Planner | 現状調査、要件分解、実装計画 | `implementation-plan.md` |
| Implementer | 承認済み計画の実装 | ソースコード、テスト |
| Tester | 自動検証と実測結果の記録 | `test-result.md` |
| Reviewer | 要件・設計・安全性の独立確認 | `review-result.md` |
| Documenter | 判断・失敗・教訓の永続化 | `task-log.md`、判断記録 |

## 成果物のリレー

```text
current-task.md
  ↓
implementation-plan.md
  ↓ 人間の承認
ソースコード・テスト
  ↓
test-result.md
  ↓
review-result.md
  ↓
task-log.md
```

## 人間の承認ポイント

- 要件と完了条件
- 実装計画と変更範囲
- 計画外の変更
- 外部サービス・本番環境・秘密情報に関する操作
- レビュー指摘の受け入れ
- マージ・リリース

テスト成功だけでは、設計の採用や本番反映を自動承認しません。
