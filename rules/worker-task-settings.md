# Workerタスク設定

各workerの実行バックエンド固有設定は、適用中のローカルルールまたは選択したバックエンドの能力契約から解決する。共通rulesは特定モデル、推論レベル、製品の生成方式を固定しない。設定が必要なバックエンドでは、生成時または再接続時に適用し、実行中のworkerへ途中変更を行わない。

| Worker | モデル | 推論レベル | 主な用途 |
| --- | --- | --- | --- |
| Planner | `<バックエンド指定>` | `<バックエンド指定>` | 現状調査、要件分解、実装計画 |
| Implementer | `<バックエンド指定>` | `<バックエンド指定>` | 承認済み計画の実装 |
| Tester | `<バックエンド指定>` | `<バックエンド指定>` | 定型テスト、Lint、静的解析、ビルドの実行と結果整理 |
| Security Operator | `<バックエンド指定>` | `<バックエンド指定>` | 秘密情報、外部操作、安全性の確認 |
| Reviewer | `<バックエンド指定>` | `<バックエンド指定>` | 要件、設計、検証、安全性の独立レビュー |
| Documenter | `<バックエンド指定>` | `<バックエンド指定>` | 判断、教訓、ライフサイクル改善の記録 |

## 運用ルール

- Planner、Implementer、Reviewer、Documenterの実行主体は、親チャットやファイル側threadではなく、実際に接続・再開されたSubagentとする。Deliveryは同一Subagentを明示resumeして論理責務を切り替え、Reviewerは独立Subagentとする。
- worker実行単位ID、役割、task-id、親実行単位または親Coordinator、指定制約、実測制約、状態、result証跡をWorker Registryへ記録する。ユーザー向け実行会話ID、ファイル側thread名、実行コンテキストIDをworker実行単位IDとして扱わない。
- 接続前に指定モデル・推論を確認できない場合は、接続・resume・作業を開始しない。指定済みSubagentの実測値を取得できない場合は、実測値を`未確認`として根拠をresult/task-logへ記録するが、それだけで自律オーケストレーション・受入・完了を停止しない。親チャット・ファイル側thread・新規ユーザー向けスレッドを代替実行主体にしない。

- 役割ごとの標準モデル・推論レベルは本表を正本とする。ただし、適用条件が完全一致し、共通rulesの保護対象と競合しない`rules/local/`のOwner確認済みルールは、対象taskのモデル・推論レベルを具体化できる。適用結果とRegistryとの照合をresultへ記録し、切替を確認できない場合は作業を開始しない
- Testerがテスト設計、失敗原因分析、複数資料の整合性判断を要する場合は、設定を変更せずReviewerへ引き継ぐ
- 推論レベルを変更する場合は、変更理由をルートの `development-improvement.md` に記録する
- タスク作成時は該当する `worker-definitions/*.md` とこの設定を入力として確認する
- 設定変更後に生成または再接続するSubagentから新しい設定を適用する。適用値と実測値はhealth-checkまたは担当resultへ記録し、取得できない値は`未確認`とする。

## 標準worker接続サイクル

標準サイクルは次の順序とする。各workerの固有責務・結果ファイル・判定値は `worker-definitions/`、入力ゲートと証跡は `rules/worker-evidence.md`、報告形式は `rules/worker-report-template.md`、履歴とtask境界は `rules/thread-operation.md`を正本とする。

```text
Delivery（Planner） → Owner承認 → Delivery（Implementer）
  → 独立Reviewer → OwnerによるReviewer受入結果の承認 → Delivery（Documenter） → Ownerの完了判断
```

- 標準SubagentはDelivery 1件と独立Reviewer 1件の最大2件とする。Deliveryは同一task内でclose/resumeしてPlanner、Implementer、Documenterの論理責務を順番に担当する。
- Subagentの同時稼働は1件だけとし、各工程の結果記録・入力照合・closeを完了してから次工程を接続する。詳細は`rules/automation-operation.md`を正本とする。
- TesterとSecurity Operatorは標準工程に含めない。外部接続、高リスク変更、または独立検証が計画に明記され、Ownerが例外承認した場合だけ、必要な方を順番に接続する。
- PlannerはOwner承認前に実装を開始しない。各workerは接続前に `rules/workflow-consistency-check.md` の入力ゲートを通る
- 接続したTesterまたはSecurity Operatorの結果が必要な場合だけ、当該結果を入力としてReviewerへ接続する
- Reviewerが`品質修正`と分類した修正依頼は、確認可能な指摘を一つの総合報告へ集約し、対象、根拠、影響、対応責任、再確認条件、指摘フィンガープリント、既存指摘との関係を全件提示する。Implementerは全件を一括修正し、親タスクはレビュー基準スナップショットとの差分と工程境界資料を照合してから、必要な確認だけを再実行する。全量横断確認へ戻す条件は`rules/worker-evidence.md`に従う。
- 上記の品質修正の総合報告後、親タスクがスナップショット比較の照合表を記録するまでReviewerを再接続しない。品質修正の再レビューだけを`current-task.md`の予算を消費して管理する。`接続同期差分`は一回の同期と同一境界の入力再確認だけを行い、`工程境界不備`はOwnerが最小同期または別のagent-workflow IMを選ぶまでhandoffを停止する。これらをImplementer修正・Reviewer再接続・修正ラウンド・予算消費へ混在させない。予算到達時は、追加接続ではなく品質指摘・修正・再検証を集約して`blocked`とし、Ownerの追加ラウンド承認または範囲変更を待つ。詳細な判定と記録は`rules/worker-evidence.md`に従う。
- Reviewerの受入後、Coordinatorは受入結果とDocumenter開始用のOwner判断を提示し、Ownerの明示承認までDocumenterへ接続しない。承認後にDocumenterは `operation-check.md`、`task-log.md`、汎用改善記録を更新してOwnerへ報告する。`operation-check.md`が存在しない、または固定構造を満たさない場合は、Documenter記録完了・Owner完了確認・次タスク切替へ進めない
- OwnerがReviewer受入結果を明示承認した後に、DeliveryがDocumenter責務として`operation-check.md`、`task-log.md`、汎用改善記録を更新し、Owner完了判断を要求する。運用中の不具合・改善点や承認範囲外の是正は現行taskへ追加せず、後続のOwner依頼で簡易モードを新たに判定する。
- Documenterを移行先で省略する場合は、導入時に記録責任者と記録先を明示する。責任または記録先が不明な状態では有効化完了としない
- 停止条件、外部操作、承認、判断本文のOJ表示は各正本rulesに従う。プロジェクト固有のworker省略はこの共通サイクルへ追加しない
