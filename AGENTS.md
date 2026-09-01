# 開発ルール

## 作業方針

- 変更前に既存ファイルと目的を確認する
- 実装前に対象、対象外、完了条件、リスクを整理する
- 計画外の変更を行わない
- 変更内容と検証結果を記録する
- 既存の手動変更を勝手に上書きしない
- コミットは1目的にまとめ、コミットメッセージ規約に従う
- workerの役割、作業領域、入出力は `worker-definitions/` の定義に従う
- worker間の正式な引き継ぎは会話ではなく `result/` の結果ファイルで行う

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
- `docs/current-task.md`: Plannerへ渡す現在のタスク
- `docs/development-improvement.md`: Documenterが開発全体の改善に向けて積み上げる記録
- `worker-definitions/`: workerごとの役割、作業領域、入出力、完了条件
- `result/`: workerが後続工程へ渡すタスク単位の結果ファイル。worker別のサブフォルダは作成せず、各タスクで上書きする
- `rules/`: コミット規約などの恒久的な開発ルール

## workerの流れ

```text
docs/current-task.md
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

- `result/plan.md`、`changes.md`、`test.md`、`security.md`、`review.md` はworkerごとに固定し、各タスクで上書きする
- `docs/development-improvement.md` は上書きせず、開発全体の改善記録として追記する
- Plannerは計画承認前に実装を開始しない
- 計画外の変更が必要になった場合は作業を停止して報告する
- Security Operatorは秘密情報、個人情報、危険な外部操作、公開上の懸念を確認する

## 停止条件

- 計画外の変更が必要になった
- 同じ検証に2回連続で失敗した
- 要件の解釈が複数に分かれた
- 承認されていない削除、上書き、公開など不可逆な操作が必要になった
- 外部サービスの認証や権限が必要になった
- 公開してよいか判断できない情報が見つかった
