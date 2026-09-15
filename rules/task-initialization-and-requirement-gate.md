# タスク初期化・要件定義実施ゲート

scope: all-workers

## 目的

ルールを読んだだけで作業した扱いにせず、タスク初期化、要件定義の記録、正本照合を実施証跡で確認する。確認できない場合は、Planner接続、計画作成、後続worker接続、完了、履歴操作を停止する。

## 初期登録時（要件定義開始前）

- Ownerがファイル側threadの登録だけを明示した場合は、`threads/<thread-name>/`と未指定または空の後続入力領域だけを準備する。
- この段階ではCodex会話、クルー会話、worker、Codexプロジェクト、task、TASK-ID、OJ-ID、Issue、PR、branch、対象環境、history台帳を作成・接続・採番・推測しない。
- 要件定義開始が明示されていない補足・相談・確認では、task資料・task-id・historyを作成または変更しない。
- 初期登録と要件定義開始を区別できない、対象領域が未確認、既存active taskとの境界が不明な場合は停止する。

初期登録の完了はtask開始やworker接続の許可を意味しない。要件定義開始後は、以下のtask初期化・Owner判断・履歴・project境界のゲートを改めて実施する。

## 要件定義開始時

要件定義資料は`rules/im-template.md`を入口テンプレートとして作成する。`issue-memo.md`、必須10観点、現状・要求状態比較、質問ごとの更新記録、Planner接続前チェックを分散資料で代替してはならない。

1. `docs/issue-memo.md`を作成または現行task用に初期化する。
2. `docs/owner-jadge.md`を作成または現行im用に初期化し、OJの正本を確保する。
3. task-id、目的、対象、対象外、開始理由、未決定事項、停止条件を記録する。
4. `rules/human-facing-documentation.md`の固定7列で、`issue-memo.md`へ現状・要求状態比較表を作成する。未確認項目は`未確認`と記載し、表を省略しない。
5. 会話で決定・変更された事項は、次の計画・worker接続・Owner回答の前に、既存項目と比較表の関連行へ統合して`issue-memo.md`を更新する。会話本文の機械的な追記は禁止する。
6. OJの提示・回答ごとに、`owner-jadge.md`、`issue-memo.md`、更新記録の反映結果を照合する。
7. `issue-memo.md`、`owner-jadge.md`、または現状・要求状態比較表が存在しない、更新されていない、列順が不一致、OJが不一致、または現行taskと不一致の場合は、次の質問、計画、Planner接続を停止する。

## タスク切替・初期化

要件定義中の会話は、会話履歴だけを正本としない。Owner回答、worker報告、停止・再開、履歴操作、ルール変更、次工程判断が発生した直後に、`issue-memo.md`、`owner-jadge.md`、`task-log.md`、`operation-check.md`のうち該当する資料を更新し、更新成功を確認する。更新が完了するまで次の質問、接続、完了、履歴操作へ進まない。会話が中断・消失しても、最新の正本資料だけで現状、判断、次工程、停止理由を復元できる状態を維持する。

1. 現行taskの`docs/`・`result/`を初期化する前に、history、manifest、`history/index.md`、退避内容を照合する。同じtask-idの再開前スナップショットがある場合は、`rules/history-initialization.md`の追記スナップショットを作成・照合する。hisi更新とdocs/result初期化を同一処理として扱わない。
2. 照合成功後かつOwnerが削除対象を明示承認した場合だけ、現行taskの`docs/`・`result/`内の対象ファイルを削除する。ディレクトリは残してよい。承認がない場合は削除せず停止する。
3. 新taskの`result/current-task.md`を最初に作成し、Planner入力の正本とする。
4. task-id確定後、`rules/history-initialization.md`に従いhisiのサマリー・詳細・統合判定用パートへ同じtask-idを登録し、サマリー・詳細の状態を`対応中`として3表の一致・昇順を照合する。
5. 退避した旧taskのIR全件から対応済みIRだけを除外し、未完了IR全件を新taskの`docs/`へ継承する。未完了IRが0件なら`improvement-reminders.md`を作成しない。未選択を理由に未完了IRを継承から除外しない。
6. 必要な`docs/`・`result/`資料だけを新規作成する。
7. 作成後、全ファイルについてtask-id、タスク名、目的、対象、状態、Owner判断、未完了事項、次工程、停止条件を照合する。
8. 旧task-id、旧目的、旧完了条件、旧進捗、旧worker結果が残っている場合は、旧history参照として明示されている場合を除き停止する。
9. hisi更新前後の既存行保持・追記結果、docs/result初期化前後の一覧、初期化結果、IR継承結果、照合結果を`operation-check.md`と`result/task-log.md`へ記録する。

## 接続ゲート

要件定義中は、必須10観点の各質問・回答・実測後に`issue-memo.md`の更新成功を確認する。未解決、未確認、不一致、更新失敗、Owner判断残件が一つでもある場合はPlanner接続を停止し、6項目（期待値、実際値、根拠、影響、停止理由、再開条件）を記録する。

Planner接続では作業ブランチ未決定を許可する。Owner承認済み計画を入力としてImplementerへ接続する直前に限り、作業ブランチを決定し、実体、`current-task.md`、承認済み`plan.md`を照合する。照合不能・不一致なら接続を停止する。

Plannerまたは後続workerへ接続するには、次の全項目を実施済みとして結果資料に記録する。

```text
[ ] history・manifest・history/index.mdの照合（hisi更新とdocs/result初期化を別工程として確認）
[ ] Ownerの明示承認を確認した対象だけ、docs/・result/の旧ファイルを削除
[ ] result/current-task.mdの作成
[ ] task-id確定直後のhisi対応中登録と3表照合
[ ] 退避IR全件と対応済みIRの除外を照合し、未完了IR全件を継承（0件ならIRファイルなし）
[ ] 必要資料の新規作成
[ ] 全資料のtask-id・目的・状態・Owner判断の照合
[ ] issue-memo.mdの最新決定事項反映
[ ] owner-jadge.mdの作成・OJ反映・issue-memoとの一致
[ ] issue-memo.mdの現状・要求状態比較表（固定7列）
[ ] operation-check.mdの対応前後比較・根拠・未確認区分
[ ] 初期化ゲートの実施結果・確認者・確認日時
```

未実施、未確認、報告のみ、workerの自己申告だけでは接続条件を満たさない。不一致があれば、期待値、実際値、根拠、影響、停止理由、再開条件を記録して停止する。

## 完了条件

初期化・要件定義の完了は、資料が存在することではなく、上記チェック項目の実施結果が正本資料に記録され、`current-task.md`、`issue-memo.md`、`task-progress.md`、`operation-check.md`、担当resultの内容が一致していることとする。

## Owner完了判断後のクローズ・初期化ゲート

Ownerがタスクの作業完了を判断した場合、次タスクの開始を待たず、同一の完了処理として次を実施する。

1. `current-task.md`へ完了日時と最終状態を記録する。後続改善候補はOwner管理IRまたは新規im候補へ分離する。
2. 現行`docs/`と`result/`の一覧、task-id、projectId、thread、状態、Owner判断、未確認事項、次工程を照合する。
3. `history/<task-id>/`が存在しないことを確認し、現行`docs/`と`result/`をコピーしてmanifestを作成する。同一task-idの履歴が存在する場合は上書きせず停止する。
4. 退避先、manifest、`history/index.md`、退避ファイルの内容と件数を相互照合する。部分成功、不一致、欠落、読取不能の場合は現行領域を初期化しない。
5. 照合成功とOwner承認後、現行`docs/`と`result/`のファイルを削除し、ディレクトリだけを残す。クローズ済みの現行領域へ旧taskの資料、OJ、状態、worker結果を残してはならない。
6. 引継ぎ、効果確認、再評価条件は履歴manifestおよび履歴スナップショットを正本とする。現行領域へ複製しない。
7. 操作前後の一覧、退避先、manifest、照合結果、初期化結果を記録し、次タスクは現行領域が空であることを確認してから開始する。

完了判断後に現行`docs/`または`result/`が残っている、完了日時が空欄、履歴manifestがない、または履歴とindexの照合が未完了の場合は、タスクを完全なクローズと扱わず、次タスク接続・worker接続・履歴消費を停止する。
