# 開発ルール

## 作業方針

- 変更前に既存ファイルと目的を確認する
- 実装前に対象、対象外、完了条件、リスクを整理する
- 計画外の変更を行わない
- 変更内容と検証結果を記録する
- 既存の手動変更を勝手に上書きしない
- コミットは1目的にまとめ、コミットメッセージ規約に従う
- workerの役割、作業領域、入出力は `worker-definitions/` の定義に従う
- worker間の正式な引き継ぎは会話ではなく、対象スレッドの `threads/<thread-name>/result/` の結果ファイルで行う
- 各workerは完了時に親タスクへ判定、結果ファイル、未確認事項、次のworkerを報告し、親タスクは確認後に次工程へ接続する
- 各workerは完了時に `Owner判断` と `Owner判断 (追記)` をMarkdownテーブルで報告し、再検証時は同様の内容を重複させず差分を追記する。最終報告では `Owner判断 (追記)` を `なし` とする

## 禁止事項

- 秘密情報、個人情報、顧客固有情報の追加
- Ownerの明示承認がないファイルの削除
- 破壊的なGit操作
- 本番環境への接続
- 認証情報を含む外部サービスへの書き込み
- ライセンスが確認できない資料やコードの転載

## 検証

- 変更後は可能な範囲でテスト、静的解析、形式チェックを実行する
- 検証は推測で成功と判定せず、実行結果に基づいて報告する
- Public公開前に秘密情報と著作権・ライセンスを確認する
- コミット前に変更内容、Prefix、秘密情報の混入を確認する

## コミットメッセージ

- 形式は `<prefix>: <What>。<Why>` とする
- Prefixの詳細は `rules/commit-convention.md` を参照する
- 計画外の変更を同じコミットへ混在させない
- Codexはコミット前にメッセージ案を提示し、人間の確認を受ける

## 作業領域

- `README.md`: ワークフロー全体と運用方法
- `threads/<thread-name>/docs/current-task.md`: 各スレッドでPlannerへ渡す現在のタスク
- `threads/<thread-name>/result/task-log.md`: Documenterがスレッド固有の判断・結果・残課題を記録するファイル。タスクごとに上書きする
- `development-improvement.md`: Documenterが全スレッド共通の開発サイクル改善を一覧で積み上げる永続記録
- `worker-definitions/`: workerごとの役割、作業領域、入出力、完了条件
- `threads/<thread-name>/result/`: workerが同じスレッドの後続工程へ渡すタスク単位の結果ファイル。worker別のサブフォルダは作成せず、各タスクで上書きする
- `rules/`: コミット規約やworkerタスク設定などの恒久的な開発ルール

## workerの流れ

```text
threads/<thread-name>/docs/current-task.md
  ↓
Planner
  ↓ Owner承認
Implementer
  ↓
Tester / Security Operator
  ↓
Reviewer
  ↓
Documenter
  ↓ Ownerがマージ・リリースを判断
```

- `threads/<thread-name>/result/plan.md`、`changes.md`、`test.md`、`security.md`、`review.md` はworkerごとに固定し、各タスクで上書きする
- ルートの `development-improvement.md` は改善項目を追記・更新し、タスク固有の詳細は記録しない
- `threads/<thread-name>/result/task-log.md` はDocumenterのタスク固有ログとして、タスクが変わるたびに上書きする
- 1つのスレッドでは1タスクだけを扱い、同一スレッドで複数タスクを並行して実行しない
- Ownerは作業対象のスレッド名とタスクを `threads/<thread-name>/docs/current-task.md` に記載し、workerはその記載から対象スレッドを認識する
- Ownerは `current-task.md` に `スレッド名`、`CodexプロジェクトID`、`Codex実行ディレクトリ`、`対象リポジトリ`、`ベースブランチ`、`作業ブランチ` を必ず記載する
- workerは作業開始前に自身のCodexプロジェクトID、Codex実行ディレクトリ、対象リポジトリへのアクセス可否を確認する。Codex実行ディレクトリは対象リポジトリと一致しなくてもよく、projectIdまたは対象リポジトリが一致しない場合は作業を開始せず親タスクへ報告する
- 1スレッドと1つのCodexプロジェクトを1対1で対応させ、1つのプロジェクトを複数スレッドで共有しない
- 新規スレッドでは専用プロジェクトを作成し、projectIdをcurrent-task.mdへ記載して全workerの所属を照合してからPlannerを起動する
- 同一タスクの再検証では結果ファイルへ差分を追記し、新規タスクでは結果ファイルを新しい内容で上書きする。Documenterの正式な結果ファイルは `threads/<thread-name>/result/task-log.md` とする
- Implementer以降は処理中、とくに例外発生時のログトレーサビリティ確認結果を完了条件として記録する
- スレッドの分離と追加は `rules/thread-operation.md` に従う
- `rules/worker-evidence.md` はworker接続前の入力ゲート、証跡、再確認、外部連携、データ、仕様書の共通ルールとする
- `rules/review-request-format.md` はレビュー依頼の固定形式と返信先を定義する
- `rules/development-improvement-record.md` は改善記録の対象、ステータス、形式、更新ルールを定義する
- `rules/README.md` は `rules/` 配下の適用範囲、状態、優先順位を定義するルールの入口とする
- worker タスクのモデルは `gpt-5.6-luna` に統一し、推論レベルは `rules/worker-task-settings.md` に従う
- Plannerは計画承認前に実装を開始しない
- 計画外の変更が必要になった場合は作業を停止して報告する
- Security Operatorは秘密情報、個人情報、危険な外部操作、公開上の懸念を確認する
- Reviewerが修正依頼と判定した場合は、Implementer、Tester、Security Operator、Reviewerの確認サイクルを再実行する
- Reviewerの受入後にDocumenterへ接続し、Documenter完了後に親タスクがOwnerへ最終報告する

## 停止条件

- 計画外の変更が必要になった
- 同じ検証に2回連続で失敗した
- 要件の解釈が複数に分かれた
- 承認されていない削除、上書き、公開など不可逆な操作が必要になった
- 外部サービスの認証や権限が必要になった
- 公開してよいか判断できない情報が見つかった
