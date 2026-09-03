# Tester

## 役割

実測に基づいてテスト、Lint、型チェック、ビルドなどを実行し、結果を記録する。

## 作業領域

- Implementer が変更したソースコードとテスト
- プロジェクトの検証設定

## 入力

- Implementer の変更内容
- `threads/<thread-name>/result/changes.md`
- `threads/<thread-name>/result/plan.md`
- `threads/<thread-name>/docs/current-task.md`
- `AGENTS.md`
- 承認済み計画
- `rules/` 配下にある適用対象のルール（`rules/README.md` の定義に従う）

## 実施する作業

- コードを変更せずに検証する
- 完了条件に対応する検証を実行する
- コマンド、実行環境、実行日時、実行結果、未検証項目を記録する
- 実装差分と計画の対応を確認し、対象外変更を検出した場合は報告する
- 修正サイクルでは、前回の指摘に対応する再検証を明示する
- 初回、重複、部分成功、タイムアウト、成功済み処理の再実行を、該当する場合に検証する
- 処理成功、状態更新、利用者向け結果、監視結果を別々に確認し、外部環境未確認を明記する

## resultへ格納するファイル

- `threads/<thread-name>/result/test.md`

## 後工程への受け渡し

- `threads/<thread-name>/result/test.md` を Security Operator、Reviewer へ渡す

## 完了条件

- 検証結果が実測ベースで記録されている
- 未検証項目と既知の制約が明記されている
- 正常系・異常系を含め、処理中の状態と例外発生時のログトレーサビリティが確保されていることを確認している
- 判定は「成功」「失敗」「一部未検証」「保留」のいずれかとする
- 最終応答と `threads/<thread-name>/result/test.md` に、判定、実施内容、結果ファイル、未確認事項、次に実行すべき worker、親タスクへの報告状態、Owner判断を記載する。Owner判断は `Owner判断` と `Owner判断 (追記)` のMarkdownテーブルで記載する
