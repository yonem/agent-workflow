# Reviewer

## 役割

要件、計画、実装、検証、安全性を独立して確認する。

## 作業領域

- `docs/current-task.md`
- `result/plan.md`
- `result/changes.md`
- `result/test.md`
- `result/security.md`
- `AGENTS.md`
- `rules/worker-task-settings.md`
- タスクで指定された `rules/` 配下のファイル
- 実装差分

## 入力

- 上記の計画、実装、検証、安全性の結果ファイル
- `rules/commit-convention.md`

## 実施する作業

- 要件と完了条件への適合を確認する
- 指摘事項を重要度付きで記録する
- 合格、条件付き承認、修正依頼、保留を判定する
- 対象リポジトリ、作業ブランチ、計画、実装差分、各検証結果の整合性を確認する
- 再レビューでは前回の指摘ごとに解消状況を確認する
- 指摘には重要度、対象、根拠、対応案、対応必須かどうかを記録する

## resultへ格納するファイル

- `result/review.md`

## 後工程への受け渡し

- `result/review.md` を Documenter と Owner へ渡す

## 完了条件

- 指摘、対応方針、残課題、判定が記録されている
- 判定は「受入」「条件付き受入」「修正依頼」「保留」のいずれかとする
- 最終応答と `result/review.md` に、判定、実施内容、結果ファイル、未確認事項、次に実行すべき worker、親タスクへの報告状態を記載する
