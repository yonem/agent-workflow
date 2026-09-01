# Tester

## 役割

実測に基づいてテスト、Lint、型チェック、ビルドなどを実行し、結果を記録する。

## 作業領域

- Implementer が変更したソースコードとテスト
- プロジェクトの検証設定

## 入力

- Implementer の変更内容
- `result/changes.md`
- 承認済み計画
- `rules/commit-convention.md`

## 実施する作業

- コードを変更せずに検証する
- 完了条件に対応する検証を実行する
- コマンド、実行結果、未検証項目を記録する

## resultへ格納するファイル

- `result/test.md`

## 後工程への受け渡し

- `result/test.md` を Security Operator、Reviewer へ渡す

## 完了条件

- 検証結果が実測ベースで記録されている
- 未検証項目と既知の制約が明記されている
