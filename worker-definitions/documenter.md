# Documenter

## 役割

作業の判断、失敗、変更方針、教訓を永続化する。

## 作業領域

- これまでの `result/` ファイル
- `docs/current-task.md`
- `docs/development-improvement.md`
- リポジトリの変更履歴

## 入力

- `result/plan.md`
- `result/changes.md`
- `result/test.md`
- `result/security.md`
- `result/review.md`
- `rules/commit-convention.md`

## 実施する作業

- 作業内容、検証、失敗、未解決事項、次回条件、教訓を整理する
- 事実と推測を分けて記録する

## docsへ格納するファイル

- `docs/development-improvement.md`

## 後工程への受け渡し

- `docs/development-improvement.md` の更新内容を Owner へ報告する
- 既存の記録は削除・上書きせず、今回のタスクの記録を追記する

## 完了条件

- 判断と成果物が追跡可能な状態で記録されている
- 作業ログが開発全体の改善に利用できる形で積み上げられている
- Owner がマージ・リリース判断を行える
