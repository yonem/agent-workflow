# Reviewer

## 役割

要件、計画、実装、検証、安全性を独立して確認する。

## 作業領域

- `docs/current-task.md`
- `result/plan.md`
- `result/changes.md`
- `result/test.md`
- `result/security.md`
- 実装差分

## 入力

- 上記の計画、実装、検証、安全性の結果ファイル
- `rules/commit-convention.md`

## 実施する作業

- 要件と完了条件への適合を確認する
- 指摘事項を重要度付きで記録する
- 合格、条件付き承認、修正依頼、保留を判定する

## resultへ格納するファイル

- `result/review.md`

## 後工程への受け渡し

- `result/review.md` を Documenter と Owner へ渡す

## 完了条件

- 指摘、対応方針、残課題、判定が記録されている
