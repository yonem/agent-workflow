# Workerタスク設定

各 worker の Codex タスクは、モデルを `gpt-5.6-luna`、推論レベルを `low` に統一する。設定はSubagentの生成時または再接続時に適用し、実行中Subagentへ途中変更を行わない。

| Worker | モデル | 推論レベル | 主な用途 |
| --- | --- | --- | --- |
| Planner | `gpt-5.6-luna` | `low` | 現状調査、要件分解、実装計画 |
| Implementer | `gpt-5.6-luna` | `low` | 承認済み計画の実装 |
| Tester | `gpt-5.6-luna` | `low` | 定型テスト、Lint、静的解析、ビルドの実行と結果整理 |
| Security Operator | `gpt-5.6-luna` | `low` | 秘密情報、外部操作、安全性の確認 |
| Reviewer | `gpt-5.6-luna` | `low` | 要件、設計、検証、安全性の独立レビュー |
| Documenter | `gpt-5.6-luna` | `low` | 判断、教訓、ライフサイクル改善の記録 |

## 運用ルール

- 役割ごとの標準モデル・推論レベルは本表を正本とする。ただし、適用条件が完全一致し、共通rulesの保護対象と競合しない`rules/local/`のOwner確認済みルールは、対象taskのモデル・推論レベルを具体化できる。適用結果とRegistryとの照合をresultへ記録し、切替を確認できない場合は作業を開始しない
- Testerがテスト設計、失敗原因分析、複数資料の整合性判断を要する場合は、設定を変更せずReviewerへ引き継ぐ
- 推論レベルを変更する場合は、変更理由をルートの `development-improvement.md` に記録する
- タスク作成時は該当する `worker-definitions/*.md` とこの設定を入力として確認する
- 設定変更後に生成または再接続するSubagentから新しい設定を適用する。適用値と実測値はhealth-checkまたは担当resultへ記録し、取得できない値は`未確認`とする。

## 標準worker接続サイクル

標準サイクルは次の順序とする。各workerの固有責務・結果ファイル・判定値は `worker-definitions/`、入力ゲートと証跡は `rules/worker-evidence.md`、報告形式は `rules/worker-report-template.md`、履歴とtask境界は `rules/thread-operation.md`を正本とする。

```text
Delivery（Planner） → Owner承認 → Delivery（Implementer）
  → 独立Reviewer → Delivery（Documenter） → Ownerの完了判断
```

- 標準SubagentはDelivery 1件と独立Reviewer 1件の最大2件とする。Deliveryは同一task内でclose/resumeしてPlanner、Implementer、Documenterの論理責務を順番に担当する。
- Subagentの同時稼働は1件だけとし、各工程の結果記録・入力照合・closeを完了してから次工程を接続する。詳細は`rules/automation-operation.md`を正本とする。
- TesterとSecurity Operatorは標準工程に含めない。外部接続、高リスク変更、または独立検証が計画に明記され、Ownerが例外承認した場合だけ、必要な方を順番に接続する。
- PlannerはOwner承認前に実装を開始しない。各workerは接続前に `rules/workflow-consistency-check.md` の入力ゲートを通る
- 接続したTesterまたはSecurity Operatorの結果が必要な場合だけ、当該結果を入力としてReviewerへ接続する
- Reviewerが修正依頼と判定した場合は、確認可能な指摘を1回の総合報告へ集約し、対象、根拠、影響、対応責任、再確認条件を全件提示する。Implementerは全件を一括修正し、親タスクは全正本を横断照合してから必要なTester、Security Operator、Reviewerの確認を再実行する。修正中に新しい重大不整合が発生した場合だけ追加指摘を許可する
- 上記の総合報告後、親タスクが全正本の照合表を記録するまでReviewerを再接続しない。初回確認漏れや資料間未同期を都度指摘として扱わず、同一原因の修正依頼が2回連続した場合はリトライを停止してOwnerの継続承認またはルール修正を待つ。
- Reviewerの受入後にDocumenterへ接続し、Documenterは `operation-check.md`、`task-log.md`、汎用改善記録を更新してOwnerへ報告する。`operation-check.md`が存在しない、または固定構造を満たさない場合は、Documenter記録完了・Owner完了確認・次タスク切替へ進めない
- Reviewer受入後にDeliveryがDocumenter責務として`operation-check.md`、`task-log.md`、汎用改善記録を更新し、Owner完了判断を要求する。任意の運用観測や承認範囲外の是正は、task完了を止めずOwner管理IRまたは新規im候補へ分離する。
- Documenterを移行先で省略する場合は、導入時に記録責任者と記録先を明示する。責任または記録先が不明な状態では有効化完了としない
- 停止条件、外部操作、承認、判断本文のOJ表示は各正本rulesに従う。プロジェクト固有のworker省略はこの共通サイクルへ追加しない
