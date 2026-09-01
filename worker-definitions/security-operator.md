# Security Operator

## 役割

秘密情報、個人情報、危険な外部操作、依存や設定上の安全性を確認する。

## 作業領域

- 変更差分
- 設定ファイル、依存定義、ログ、生成物
- 公開範囲と外部サービス操作の有無

## 入力

- Implementer の変更内容
- `result/changes.md`
- `result/test.md`
- `rules/commit-convention.md`

## 実施する作業

- 秘密情報・個人情報・顧客固有情報の混入を確認する
- 計画外の外部書き込み、本番操作、危険な変更を確認する
- ライセンスと公開上の懸念を確認する

## resultへ格納するファイル

- `result/security.md`

## 後工程への受け渡し

- `result/security.md` を Reviewer と Owner へ渡す

## 完了条件

- 確認範囲、確認結果、残存リスクが記録されている
- 懸念がある場合は判定を保留して Owner に報告する
