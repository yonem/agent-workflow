# Agent Workflow

新規プロジェクトでAIエージェントを役割ごとに運用し、ファイルを介して作業結果を引き継ぐためのワークフロー定義です。

小規模な作業では1つのセッションで役割を切り替え、大規模または独立性が必要な作業では複数タスクやワークツリーに分けます。Orchestrator は任意とし、タスク分割・worker の起動・成果物の状態管理を自動化する場合に利用します。

## 目的

このリポジトリは、CodexなどのAIエージェントを開発プロセスへ組み込むための、汎用的なルール・成果物・プロンプトを提供します。

## 導入

本リポジトリは、特定プロジェクトを直接運用するための作業場所ではなく、各プロジェクトへ展開するスターターパックです。共通ルールとworker定義を対象リポジトリへ適用し、対象プロジェクトの構成・技術・実行環境に合わせたタスクと検証計画をPlannerに作成させます。

初回導入時は、次の順序で準備します。

1. 本リポジトリの `AGENTS.md`、`rules/`、`worker-definitions/`、`README.md` を対象リポジトリへ展開する
2. 対象リポジトリの `threads/<thread-name>/docs/current-task.md` を作成し、スレッド名、CodexプロジェクトID、Codex実行ディレクトリ、対象リポジトリ、ベースブランチ、作業ブランチを記載する
3. Plannerへ対象リポジトリの内訳確認と `plan.md` の作成を依頼する
4. Plannerの計画をOwnerが確認・承認してから、後続workerを接続する

`threads/<thread-name>/docs/` と `threads/<thread-name>/result/` は、展開先で生成されるタスク固有の作業領域です。本リポジトリではGit管理対象外とし、共通の仕組みには含めません。共通ルールを更新した場合は、派生先で差分を確認してから必要な内容だけを取り込みます。

Codexの再起動後にworkerの表示やプロジェクト所属が不一致になった場合は、作業を開始せず、Ownerがworkerを正しいプロジェクトへ再作成・再接続します。復旧手順は `rules/thread-operation.md` の「同期失敗時の復旧」を参照してください。

各スレッドは専用のCodexプロジェクトと1対1で対応させます。新しいスレッドでは専用プロジェクトを作成し、projectIdを`current-task.md`へ記載して全workerの所属を確認してからPlannerを起動します。

## 基本ワークフロー

```text
threads/<thread-name>/docs/current-task.md
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

各 worker の役割、作業領域、入力、結果ファイル、後工程への受け渡しは `worker-definitions/` の Markdown ファイルで定義します。1つのスレッドでは1タスクだけを扱い、タスク固有の入力と結果はスレッド単位で管理します。結果は `threads/<thread-name>/result/` に固定ファイル名で上書き保存します。Documenterは他workerと同じ `threads/<thread-name>/result/task-log.md` にタスク固有の記録を上書き保存し、全スレッドで再利用できる改善点だけをルートの `development-improvement.md` に追記・更新します。記載ルールは `rules/development-improvement-record.md` を参照してください。

各 worker は完了時に親タスクへ、判定・結果ファイル・未確認事項・次の worker を報告します。親タスクは報告と結果ファイルを確認してから次工程へ接続します。Reviewer が修正依頼と判定した場合は Implementer に戻し、Tester、Security Operator、Reviewer の確認を再実行します。複数の作業ループは `threads/` 配下のスレッドディレクトリで分離して運用します。詳細は `rules/worker-task-settings.md` と `rules/thread-operation.md` を参照してください。

すべての worker は、完了報告と結果ファイルに `Owner判断` と `Owner判断 (追記)` のMarkdownテーブルを記載します。再検証時は同様の内容を重複させず、差分を追記します。最終報告では `Owner判断 (追記)` を `なし` で終えます。詳細は `rules/worker-evidence.md` を参照してください。

同一タスクの再検証では結果ファイルへ追記し、新しいタスクへ切り替える場合は結果ファイルを上書きします。Documenterの正式な結果ファイルは各スレッドの `result/task-log.md` です。Implementer以降は、処理中および例外発生時のログトレーサビリティ確認結果も記録します。

## 使い方

1. 対象スレッドの `threads/<thread-name>/docs/current-task.md` を要件に合わせて記入する
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
├── development-improvement.md
├── rules/
│   ├── README.md
│   └── *.md
├── worker-definitions/
│   ├── planner.md
│   ├── implementer.md
│   ├── tester.md
│   ├── security-operator.md
│   ├── reviewer.md
│   └── documenter.md
├── threads/
│   ├── normal/
│   │   ├── docs/
│   │   │   └── current-task.md
│   │   └── result/
│   │       ├── plan.md
│   │       ├── changes.md
│   │       ├── test.md
│   │       ├── security.md
│   │       ├── review.md
│   │       └── task-log.md
│   ├── emergency/
│   │   ├── docs/
│   │   │   └── current-task.md
│   │   └── result/
│   │       └── *.md
│   └── <new-thread-name>/
│       ├── docs/
│       └── result/
```

`threads/normal/` と `threads/emergency/` はスレッド単位の作業領域です。新しいスレッドは `threads/<new-thread-name>/` として追加します。`rules/` 配下のMarkdownファイルは共通ルールとして扱い、適用範囲や優先順位は `rules/README.md`、スレッドの分離方法は `rules/thread-operation.md` を参照してください。

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

正式な引き継ぎ情報は会話履歴ではなく、対象スレッドの `threads/<thread-name>/result/` に格納された結果ファイルです。後工程は同じスレッドの結果ファイルを読み取り、自身の固定結果ファイルを上書き保存します。Documenterはタスク固有の詳細を `threads/<thread-name>/result/task-log.md` に上書き保存し、汎用的な開発サイクル改善だけをルートの `development-improvement.md` に追記・更新します。計画、外部操作、マージ、リリースなどの承認は Owner が行います。

## 安全上の注意

- 実案件のソースコード、顧客情報、個人情報、秘密情報をこのリポジトリへ追加しない
- 実案件で利用する場合は、テンプレートを対象プロジェクトへコピーして固有情報を記載する
- 本番操作、データ削除、外部サービスへの書き込みは人間の承認を必須にする
- Public公開前に、非公開URL、認証情報、社内固有の名称、ライセンス上の問題がないことを確認する
