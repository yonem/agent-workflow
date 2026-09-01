# Agent Workflow

新規プロジェクトでAIエージェントを役割ごとに運用し、ファイルを介して作業結果を引き継ぐためのワークフロー定義です。

小規模な作業では1つのセッションで役割を切り替え、大規模または独立性が必要な作業では複数タスクやワークツリーに分けます。Orchestrator は任意とし、タスク分割・worker の起動・成果物の状態管理を自動化する場合に利用します。

## 目的

このリポジトリは、CodexなどのAIエージェントを開発プロセスへ組み込むための、汎用的なルール・成果物・プロンプトを提供します。

## 基本ワークフロー

```text
docs/current-task.md
  ↓
Planner：現状調査・実装計画
  ↓ 人間が承認
Implementer：実装・テスト追加
  ↓
Tester：テスト・静的解析・ビルド
  ↓
Security Operator：秘密情報・安全性を確認
  ↓
Reviewer：要件・設計・安全性を独立確認
  ↓
Documenter：判断・結果・教訓を記録
  ↓
人間がマージ・リリースを判断
```

各 worker の役割、作業領域、受け取るファイル、結果ファイル、後工程への受け渡しは `worker-definitions/` の Markdown ファイルで定義します。タスク単位の結果は root 直下の `result/` に固定ファイル名で上書き保存し、Documenter の開発全体ログだけは `docs/development-improvement.md` に積み上げます。

各 worker は完了時に親タスクへ、判定・結果ファイル・未確認事項・次の worker を報告します。親タスクは報告と結果ファイルを確認してから次工程へ接続します。Reviewer が修正依頼と判定した場合は Implementer に戻し、Tester、Security Operator、Reviewer の確認を再実行します。詳細は `rules/worker-task-settings.md` を参照してください。

## 使い方

1. `docs/current-task.md` をプロジェクトの要件に合わせて記入する
2. Plannerに現状調査と計画作成を依頼する
3. 人間が計画・リスク・完了条件を承認する
4. Implementerに承認済みの範囲だけを実装させる
5. Testerに実測ベースの検証を依頼する
6. Reviewerに独立レビューを依頼する
7. 人間が採用、修正、中止、マージを判断する

## ディレクトリ

```text
.
├── AGENTS.md
├── LICENSE
├── README.md
├── docs/
│   ├── current-task.md
│   └── development-improvement.md
├── rules/
│   ├── commit-convention.md
│   └── worker-task-settings.md
├── worker-definitions/
│   ├── planner.md
│   ├── implementer.md
│   ├── tester.md
│   ├── security-operator.md
│   ├── reviewer.md
│   └── documenter.md
└── result/
    ├── plan.md
    ├── changes.md
    ├── test.md
    ├── security.md
    └── review.md
```

## 役割

`Owner` は人間の承認者です。Planner、Implementer、Tester、Security Operator、Reviewer、Documenter が worker として作業します。各 worker の詳細は対応する `worker-definitions/*.md` を参照してください。

## Workerタスク設定

各 worker の Codex タスクは `gpt-5.6-luna` を使用します。推論レベルは Planner、Implementer、Security Operator、Reviewer が `high`、Tester、Documenter が `medium` です。詳細は `rules/worker-task-settings.md` を参照してください。

| 役割 | 主な責任 | コード変更 |
| --- | --- | --- |
| Owner | 目的、優先順位、リスク、最終判断 | 原則なし |
| Planner | 現状調査、要件分解、実装計画 | なし |
| Implementer | 承認済み計画の実装 | あり |
| Tester | 自動検証、再現手順の整理 | なし |
| Security Operator | 秘密情報、外部操作、安全性の確認 | なし |
| Reviewer | 要件・設計・安全性の独立確認 | なし |
| Documenter | 判断・結果・教訓の永続化 | 文書のみ |

## 承認ポイント

- タスク定義、調査結果、実装計画：Owner が承認する
- 設計変更、計画外変更、外部サービス操作、本番操作：Owner が判断・承認する
- レビュー結果、マージ、リリース：Owner が採否を判断する

テスト成功だけでは、設計の採用や本番反映を自動承認しません。

## 停止条件

- 計画外の変更が必要になった
- 同じ検証に2回連続で失敗した
- 要件の解釈が複数に分かれた
- 外部サービスの認証や権限が必要になった
- 本番環境や個人情報に触れる必要がある
- 承認されていない削除、移動、上書き、公開が必要になった
- 変更範囲が当初想定を大きく超えた

停止時は、理由、試した対応、推定原因、影響範囲、未確認事項、Owner に求める判断を記録します。

## ファイルの受け渡し

正式な引き継ぎ情報は会話履歴ではなく、`result/` に格納された結果ファイルです。後工程は前工程の結果ファイルを読み取り、自身の固定結果ファイルを `result/` に上書き保存します。Documenter の成果物だけは `docs/development-improvement.md` に追記し、開発全体の振り返りとライフサイクル改善に使用します。計画、外部操作、マージ、リリースなどの承認は Owner が行います。

## 安全上の注意

- 実案件のソースコード、顧客情報、個人情報、秘密情報をこのリポジトリへ追加しない
- 実案件で利用する場合は、テンプレートを対象プロジェクトへコピーして固有情報を記載する
- 本番操作、データ削除、外部サービスへの書き込みは人間の承認を必須にする
- Public公開前に、非公開URL、認証情報、社内固有の名称、ライセンス上の問題がないことを確認する
