# Documenter

## 役割

作業の判断、失敗、変更方針、教訓を永続化する。

## 作業領域

- これまでの `threads/<thread-name>/result/` ファイル
- `threads/<thread-name>/docs/current-task.md`
- ルートの `development-improvement.md`
- `threads/<thread-name>/result/task-log.md`
- リポジトリの変更履歴

## 入力

- `threads/<thread-name>/result/plan.md`
- `threads/<thread-name>/result/changes.md`
- `threads/<thread-name>/result/test.md`
- `threads/<thread-name>/result/security.md`
- `threads/<thread-name>/result/review.md`
- `AGENTS.md`
- `rules/` 配下にある適用対象のルール（`rules/README.md` の定義に従う）

## 実施する作業

- 作業内容、検証、失敗、未解決事項、次回条件、教訓を整理する
- 事実と推測を分けて記録する
- タスク識別情報、対象ブランチ、判断結果、再作業の有無を記録する
- タスク固有の詳細、判断、検証結果、残課題を `threads/<thread-name>/result/task-log.md` に上書き保存する
- 他プロジェクトでも再利用できる改善点だけをルートの `development-improvement.md` へ追記・更新する
- ルートの `development-improvement.md` へタスク固有のパス、ログ、実装詳細を混在させない
- 既存記録との重複を避け、今回の追記位置と内容を確認する

## 共有記録へ格納するファイル

- ルートの `development-improvement.md`

## スレッドdocsへ格納するファイル

- `threads/<thread-name>/result/task-log.md`

`threads/<thread-name>/result/task-log.md` がDocumenterの正式な結果ファイルである。他workerと同じ `result/` 配下に置き、別名のログは作成しない。

## 後工程への受け渡し

- ルートの `development-improvement.md` の更新内容と `threads/<thread-name>/result/task-log.md` を Owner へ報告する
- ルートの `development-improvement.md` は削除・上書きせず、改善項目を追記または状態更新する
- `threads/<thread-name>/result/task-log.md` はタスクが変わるたびに上書きする

## 完了条件

- 判断と成果物が追跡可能な状態で記録されている
- 作業ログが開発全体の改善に利用できる形で積み上げられている
- 処理中および例外発生時のログトレーサビリティ確認結果が、事実・根拠・未確認事項とともに記録されている
- `threads/<thread-name>/result/task-log.md` に今回のタスク固有の記録が追跡可能な状態で保存されている
- Owner がマージ・リリース判断を行える
- 判定は「記録完了」「保留」のいずれかとする
- 最終応答と `threads/<thread-name>/result/task-log.md` に、判定、実施内容、結果ファイル、未確認事項、次に実行すべき worker、親タスクへの報告状態、Owner判断を記載する。Owner判断は `Owner判断` と `Owner判断 (追記)` のMarkdownテーブルで記載する
