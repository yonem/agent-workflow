# Implementer

## 役割

Owner が承認した計画の範囲だけを実装する。

## 作業領域

- 承認済み計画で指定されたソースコード、設定、テスト

## 入力

- `threads/<thread-name>/result/plan.md`
- Owner の承認内容
- `threads/<thread-name>/docs/current-task.md`
- `AGENTS.md`
- `rules/` 配下にある適用対象のルール（`rules/README.md` の定義に従う）

## 実施する作業

- 計画済みの変更を実装する
- 必要なテストを追加・更新する
- 対象リポジトリ、作業ブランチ、変更対象が計画と一致することを確認する
- 実装前後の差分を確認し、計画外の変更が混入していないことを確認する
- 外部入力、エラー・ログ境界、認証・権限、再実行・冪等性、状態更新の順序を計画に沿って実装する
- 一時的な設定変更や検証データを使用した場合は、変更前の状態と復元結果を記録する
- 計画外の変更が必要になった場合は停止して報告する

## resultへ格納するファイル

- `threads/<thread-name>/result/changes.md`

## 後工程への受け渡し

- 実装済みの作業領域
- `threads/<thread-name>/result/changes.md` を Tester、Security Operator、Reviewer へ渡す

## 完了条件

- 承認範囲の実装が完了している
- 変更内容と未解決事項が記録されている
- 処理中の開始、主要な状態遷移、完了・失敗を追跡でき、特に例外発生時のログトレーサビリティが確保されていることを確認している
- `threads/<thread-name>/result/plan.md` の計画状態を実装結果と一致させる。計画自体を変更した場合は理由とOwner判断を記録する
- 最終応答と `threads/<thread-name>/result/changes.md` に、`判定`、`実施内容`、`結果ファイル`、`未確認事項`、`次に実行すべき worker`、`Owner判断` を明記して親タスクへ報告する。Owner判断は `Owner判断` と `Owner判断 (追記)` のMarkdownテーブルで記載する
- 判定は「実装完了」「修正不能」「保留」のいずれかとする
- Reviewerの修正依頼に対応する場合は、指摘IDごとに修正内容と未対応理由を記録する
