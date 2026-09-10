# タスク初期化・要件定義実施ゲート

scope: all-workers

## 目的

ルールを読んだだけで作業した扱いにせず、タスク初期化、要件定義の記録、正本照合を実施証跡で確認する。確認できない場合は、Planner接続、計画作成、後続worker接続、完了、履歴操作を停止する。

## 要件定義開始時

1. `docs/issue-memo.md`を作成または現行task用に初期化する。
2. task-id、目的、対象、対象外、開始理由、未決定事項、停止条件を記録する。
3. `rules/human-facing-documentation.md`の固定7列で、`issue-memo.md`へ現状・要求状態比較表を作成する。未確認項目は`未確認`と記載し、表を省略しない。
4. 会話で決定・変更された事項は、次の計画・worker接続・Owner回答の前に、既存項目と比較表の関連行へ統合して`issue-memo.md`を更新する。会話本文の機械的な追記は禁止する。
5. `issue-memo.md`の更新結果を、更新箇所と確認日時付きで`result/task-log.md`または初期化記録へ記録する。
6. `issue-memo.md`または現状・要求状態比較表が存在しない、更新されていない、列順が不一致、または現行taskと不一致の場合は、計画とPlanner接続を停止する。

## タスク切替・初期化

1. 現行taskの`docs/`・`result/`を削除する前に、history、manifest、`history/index.md`、退避内容を照合する。
2. 照合成功後、現行taskの`docs/`・`result/`内のファイルを削除する。ディレクトリは残してよい。
3. 新taskの`result/current-task.md`を最初に作成し、Planner入力の正本とする。
4. 必要な`docs/`・`result/`資料だけを新規作成する。
5. 作成後、全ファイルについてtask-id、タスク名、目的、対象、状態、Owner判断、未完了事項、次工程、停止条件を照合する。
6. 旧task-id、旧目的、旧完了条件、旧進捗、旧worker結果が残っている場合は、旧history参照として明示されている場合を除き停止する。
7. 初期化前後の一覧、削除結果、作成結果、照合結果を`operation-check.md`と`result/task-log.md`へ記録する。

## 接続ゲート

Plannerまたは後続workerへ接続するには、次の全項目を実施済みとして結果資料に記録する。

```text
[ ] history・manifest・history/index.mdの照合
[ ] docs/・result/の旧ファイル削除
[ ] result/current-task.mdの作成
[ ] 必要資料の新規作成
[ ] 全資料のtask-id・目的・状態・Owner判断の照合
[ ] issue-memo.mdの最新決定事項反映
[ ] issue-memo.mdの現状・要求状態比較表（固定7列）
[ ] operation-check.mdの対応前後比較・根拠・未確認区分
[ ] 初期化ゲートの実施結果・確認者・確認日時
```

未実施、未確認、報告のみ、workerの自己申告だけでは接続条件を満たさない。不一致があれば、期待値、実際値、根拠、影響、停止理由、再開条件を記録して停止する。

## 完了条件

初期化・要件定義の完了は、資料が存在することではなく、上記チェック項目の実施結果が正本資料に記録され、`current-task.md`、`issue-memo.md`、`task-progress.md`、`operation-check.md`、担当resultの内容が一致していることとする。
