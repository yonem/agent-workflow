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

## 完了報告と次工程への接続

- 各 worker は作業完了時に、親タスク向けの完了報告を必ず自身の最終応答に記載し、担当の結果ファイルにも同じ内容を記録する
- 完了報告は次の固定項目で作成する：`判定`、`実施内容`、`結果ファイル`、`未確認事項`、`次に実行すべき worker`
- 親タスクは子タスクの完了状態、最終応答、結果ファイルの3点を確認してから、次の worker に作業を依頼する
- 子タスクから親タスクへの自動通知が利用できない場合でも、親タスクは完了状態を確認し、結果ファイルを読み取って報告を補完する
- Tester と Security Operator は並行して実行できるが、両方の完了報告を受けてから Reviewer に接続する
- Reviewer が「修正依頼」と判定した場合は、指摘内容を Implementer に戻し、修正後に Tester、Security Operator、Reviewer の確認サイクルを再実行する
- Reviewer が受入判定した場合は、Documenter に接続する
- Documenter はタスク固有の結果を `result/task-log.md` に上書きし、汎用的な改善点だけを `docs/development-improvement.md` に1項目1行で追記・更新する
- Documenter の完了報告と `result/task-log.md`、`docs/development-improvement.md` の更新を確認した後、親タスクは Owner に最終結果を報告する
- worker が停止条件に該当した場合は、完了扱いにせず、理由と Owner に求める判断を親タスクへ報告する
