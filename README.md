# Agent Workflow

新規プロジェクトでAIエージェントを役割ごとに運用し、ファイルを介して作業結果を引き継ぐためのワークフロー定義です。

小規模な作業では1つのセッションで役割を切り替え、大規模または独立性が必要な作業では複数のスレッドに分けます。各スレッドは1つのCodexプロジェクトと1つのタスクに対応します。Orchestrator は任意とし、workerの起動・成果物の状態管理を自動化する場合に利用します。

## 目的

このリポジトリは、CodexなどのAIエージェントを開発プロセスへ組み込むための、汎用的なルール・成果物・プロンプトを提供します。

skillを扱う共通基盤は`rules/skill-framework.md`を正本とし、個別skillの定義には`rules/skill-definition-template.md`を使用します。既存rules、grill、worker分業、正本管理、停止条件、履歴管理、Owner承認ゲートを置換せず、skill単位へ段階的に整理します。

## 導入

本リポジトリは、特定プロジェクトを直接運用するための作業場所ではなく、各プロジェクトへ展開するスターターパックです。共通ルールとworker定義を対象リポジトリへ適用し、対象プロジェクトの構成・技術・実行環境に合わせたタスクと検証計画をPlannerに作成させます。

改善サイクルは、改善事項の発見、汎用性評価、Owner判断、計画、実装、Reviewer受入、実運用後の効果確認、再評価、記録更新、完了または継続の順で運用します。Reviewer受入だけでは改善完了とせず、効果確認の状態・証跡・次回確認日・再評価条件を別に記録します。共通仕様は `rules/development-improvement-record.md` を正本とします。

共通仕様は特定の言語、shell、実行可能プログラム、外部ライブラリ、製品、ベンダー、OS、実行環境を前提にしません。役割、能力、入出力、状態、判定基準、証跡、停止条件で目的を確認し、記録媒体を変更しても正本、責任、状態、証跡、完了条件の意味を維持します。避けられない技術依存は理由、適用範囲・期間、代替可否、移行時影響、Owner判断、停止条件、分離先を例外記録へ残します。

作業リポジトリや利用シーン固有のルールは共通rulesへ混在させず、`rules/local-rules.md` に従ってGit管理外の `rules/local/<rule-name>.md` へ置きます。共通rulesの保護対象を弱めず、適用条件・責任者・競合・証跡を確認できる場合だけ適用します。詳細な命名、最小本文、読込境界、移行・欠落時の扱いは `rules/local-rules.md` を参照してください。

初回導入時は、次の順序で準備します。

1. 本リポジトリの `AGENTS.md`、`rules/`、`worker-definitions/`、`README.md` を対象リポジトリへ展開する
2. `rules/history-initialization.md` に従い、Git追跡対象外のローカル `history/` と、サマリー・詳細・統合判定用8項目を持つ `history/index.md` を初期化する。通常の既存履歴があるのに台帳がない場合は、上書きせず停止する。`history/task-legacy-history-backup/`は今回の特殊な保全領域として初期化判定から除外する
3. `rules/current-task-template.md` を使って対象リポジトリの `threads/<thread-name>/result/current-task.md` を作成し、Task ContextとTask Definitionを記載する。作業ブランチは、既存ブランチを指定しない限り `記述ルールに従い新規作成` とする
4. 初回タスクの既知情報を `history/index.md` へ登録し、`result/current-task.md`との一致を確認する
5. Plannerへ対象リポジトリの内訳確認と `plan.md` の作成を依頼する
6. Plannerの計画をOwnerが確認・承認してから、後続workerを接続する。Planner接続時の作業ブランチ未決定は許容し、Implementer接続前にだけ確定・実体照合する。計画関連Owner向け回答の必須情報は `rules/plan-approval-required-info.md` に従う

`threads/<thread-name>/docs/` と `threads/<thread-name>/result/` は、展開先で生成されるタスク固有の作業領域です。次タスク以降、進行中タスクの共通台帳は`docs/task-progress.md`、worker固有の完了報告は`result/`に分けます。本リポジトリではGit管理対象外とし、共通の仕組みには含めません。共通ルールを更新した場合は、派生先で差分を確認してから必要な内容だけを取り込みます。

Codexの再起動後にworkerの表示やプロジェクト所属が不一致になった場合は、作業を開始せず、Ownerがworkerを正しいプロジェクトへ再作成・再接続します。復旧手順は `rules/thread-operation.md` の「同期失敗時の復旧」を参照してください。

Ownerが「ヘルスチェックを実行して」と明示した場合、または作業中にworkerへ接続できなかった場合は、`rules/worker-health-check.md` に従って同一projectId内のworker一覧、アクセス可否、状態、重複、不一致、不足を確認します。接続失敗時も自動作成へ進まず、Ownerの確認前にworkerの状態を変更しません。不足workerの追加はOwnerが明示的に指示した場合だけ行います。

各スレッドは専用のCodexプロジェクトと1対1で対応させ、通常時は1つのactiveなproject/thread/taskだけを扱います。復旧時の例外を含む分離・履歴・整合性の詳細は `rules/thread-operation.md` と `rules/workflow-consistency-check.md` に従います。

ファイル側threadの初期登録は資料保存領域だけを準備する非実行工程です。Codex会話、クルー会話、worker、project、task、識別子、history台帳は作成・接続・採番せず、要件定義開始後に別ゲートで扱います。作業開始、worker接続、履歴退避、復旧、task切替の前に、現在のCodex projectId、Ownerが指定したファイル側thread、`current-task.md`のprojectId・対象リポジトリ・実行ディレクトリを照合します。Owner会話は判断・承認、Codex会話は作業会話、クルー会話はworker実行、ファイル側threadは資料保存のための別単位です。不一致、未確認、複数候補では資料操作・接続・履歴操作を停止します。

実装媒体はリポジトリの実態と承認済みtask範囲で判定します。文書中心ではMarkdownを中心にし、コードを含む開発リポジトリでは承認済み範囲で既存コード・テスト・解析・ビルドを扱えます。混在リポジトリでは変更対象ごとに判断し、新規依存、外部操作、本番接続、破壊的操作、承認範囲外の変更は停止または個別承認対象です。

既存クルーは`current-task.md`のWorker Registryを期待値として照合し、再利用可否を先に確認します。不足workerの追加は、候補・アーカイブ・個別状態を確認し、Ownerが明示指示した場合だけ行います。各workerは作業開始、計画作成完了、計画外変更、停止・再開、Owner判断発生、完了の各境界で、resultを根拠に親タスクへ報告します。報告済みは承認・受入・次工程接続とは別の状態です。

## 基本ワークフロー

```text
threads/<thread-name>/result/current-task.md
  ↓
Planner：現状調査・実装計画
  ↓ 人間が承認
Implementer：実装・テスト追加
  ↓
Tester：テスト・静的解析・ビルド ─┐
                                     ├─ 両方完了
Security Operator：秘密情報・安全性の確認 ─┘
  ↓
Reviewer：要件・設計・安全性を独立確認
  ↓
Documenter：判断・結果・教訓を記録
  ↓
人間がマージ・リリースを判断
```

各 worker の固有責務、入力、結果ファイル、後工程は `worker-definitions/`、標準接続サイクルとモデル設定は `rules/worker-task-settings.md`、入力ゲートと証跡は `rules/worker-evidence.md`、完了報告の最低記録項目は `rules/worker-report-template.md` を正本とします。通常のresult本文の見出し順・表形式は固定しません。現行result/history境界、復旧、再検証、task-logの更新時点は `rules/thread-operation.md` に従います。

共通rules、worker定義、テンプレート、README、workflow資料、移行手順を変更する場合は、変更前に直接対象・参照対象・記録対象・移行先導入対象を分類し、正本・参照先・更新責任・更新境界・旧表現・停止条件を確認します。変更後は同じ影響一覧を再確認し、影響なし・対象外の根拠、未確認・矛盾・変更漏れの停止理由を`changes.md`へ記録します。手順の正本は `rules/workflow-consistency-check.md` と `rules/workflow-integrity-check.md` です。

動作確認は `rules/operation-check-report.md`、worker状態の確認は `rules/worker-health-check.md`、汎用改善の記録は `rules/development-improvement-record.md` に従います。プロジェクト固有のworker省略は共通文書へ追加しません。

ローカルルールを利用する場合は、タスク開始、worker接続前、作業開始前、ルール参照時、Reviewer受入前、Documenter記録前、移行・復元時に `rules/local-rules.md` の判定を行います。欠落・競合・判定不能時の継続または停止条件も同ルールに従います。

外部連携を利用する場合は、Git管理外の`rules/local/integration-ledger.md`を利用前に確認します。2列の台帳に連携識別子が記載されている場合だけ有効とし、未登録は初回Owner承認、既存識別子確認、申請クルーまたは親による登録、記載確認を経て利用します。同一連携の後続操作は再承認不要です。取消は行削除、台帳欠落・全体読取不能・共通rules競合は全連携停止として扱い、認証情報や接続情報は記録しません。

人間向け資料と管理ファイルの共通形式、正本、責任、初期作成、復元、移行手順は `rules/human-facing-documentation.md` に従います。`owner-jadge.md`、`task-progress.md`、`issue-memo.md`の形式や、サマリーと詳細の昇順・トレース規則も同ルールを参照します。

`docs/issue-memo.md`と`docs/task-progress.md`は人間向けに整理した要約・判断・進捗を記録し、AI・worker向けの詳細な引継ぎ・証跡は`result/`へ記録します。関連会話や状態変化の都度更新しますが、既存項目を統合・更新し、会話本文を機械的に追記しません。

タスク初期化、`issue-memo.md`の実施記録、docs/resultの旧資料削除、新規資料の相互照合、worker接続前の必須チェックは `rules/task-initialization-and-requirement-gate.md` の実施ゲートに従います。ルールを読んだだけでは作業開始・worker接続を許可しません。

共通rules、README、AGENTS.md、worker定義、テンプレート、ローカルルールを追加・更新した場合は、`rules/rule-refresh.md`に従って、指示を待たず次の回答・操作前に更新後の正本を再読込します。再読込・影響確認前の処理は停止します。

用語の正本は`rules/glossary.md`です。用語、標準的な意味、備考（取り違え防止）の3列表を参照し、未登録語・意味衝突・デフォルト解釈不能は推測で補わずOwner確認まで停止します。移行先では、同ファイルの初期作成・復元・有効化手順を確認します。

新機能追加、新規task作成、要件定義開始、承認済み計画へのtask追加が明示された場合だけ新規要件定義を発火します。単なる相談・補足・確認・意見交換では管理資料を変更せず、曖昧な場合は確認まで停止します。新規taskでは、要件記録・task-id採番・新規`issue-memo.md`作成より先にOwnerが現行taskを中断・終了するか、現行taskを残して別project・別thread等で実施するかを選択します。クローズ・終了・中断または切替時はhistory退避、manifest・`history/index.md`・退避内容の照合後に必ず旧docs/resultを初期化し、新規taskへ切り替える場合だけ新規task資料を作成します。同一projectのactive重複、判定不能、部分成功、初期化不完了は停止します。

## 使い方

1. 対象スレッドの `threads/<thread-name>/result/current-task.md` を要件に合わせて記入する
2. Plannerに現状調査と計画作成を依頼する
3. 人間が計画・リスク・完了条件を承認する
4. Implementerに承認済みの範囲だけを実装させる
5. TesterとSecurity Operatorに、それぞれ実測ベースの検証と安全性確認を依頼する
6. 両方の完了報告を確認してからReviewerに独立レビューを依頼する
7. Reviewer受入後に実運用の効果を確認し、Documenterまたは導入時に指定した記録責任者が結果・残課題・次回条件を記録する
8. 効果不足や承認範囲外の是正がある場合は、同一TASKの修正または新しいTASK-xxxの計画をOwnerが判断する
9. 人間が採用、修正、中止、マージ、リリースを判断する

## 移行先での初期有効化確認

他プロジェクトへ展開した後、次を確認するまで共通サイクルを有効化済みとしません。

- `AGENTS.md`、`rules/`、`worker-definitions/`、`README.md`、導入確認手順が配置されている
- `rules/local-rules.md` が配置され、`rules/local/` の正本、Git管理外の扱い、適用条件、責任者、証跡、欠落時の継続・停止条件を確認できる
- `rules/history-initialization.md`に従って、ローカルの`history/`とサマリー・詳細・統合判定用8項目の`history/index.md`を初期化できる
- `current-task.md`、`task-progress.md`、worker結果ファイル、汎用改善記録、タスク固有ログの記録先が確認できる
- Planner、Implementer、Reviewer、Documenterの責務と、Documenterを省略する場合の記録責任者・記録先が確認できる
- Reviewer受入と実運用後の効果確認、継続評価、再評価、新TASK起票、停止条件の境界が確認できる
- 特定の過去task、project/thread、Worker Registry、F-ID、IMP-ID、draftsなしで初回サイクルを開始できる
- 共通資料の正本・参照先・更新責任・更新境界、旧表現の扱い、影響確認と変更後再確認の手順を確認できる
- 移行先固有のproject/thread、Worker Registry、task-id、IMP-ID、現在のブランチ、実行ディレクトリを共通仕様の必須条件にせず、必要な固有設定を導入時に分離して設定できる

不足・責任不明・記録先不明・確認不能がある場合は未有効化として停止し、必要な導入作業を新しいTASK-xxxとして計画します。

## ディレクトリ

```text
.
├── AGENTS.md
├── LICENSE
├── README.md
├── development-improvement.md
├── history/
│   ├── index.md
│   ├── <task-id>/
│       ├── manifest.md
│       ├── docs/
│       └── result/
│   └── task-legacy-history-backup/
│       └── legacy/<old-name>/
├── rules/
│   ├── README.md
│   ├── local-rules.md
│   ├── local/                 # Git管理外。プロジェクト固有ルールの正本
│   └── *.md
├── worker-definitions/
│   ├── planner.md
│   ├── implementer.md
│   ├── tester.md
│   ├── security-operator.md
│   ├── reviewer.md
│   └── documenter.md
├── threads/
│   ├── <thread-name-a>/
│   │   ├── docs/
│   │   │   ├── issue-memo.md
│   │   │   ├── owner-jadge.md
│   │   │   ├── task-progress.md
│   │   │   ├── health-check.md
│   │   │   └── operation-check.md
│   │   └── result/
│   │       ├── current-task.md
│   │       ├── plan.md
│   │       ├── changes.md
│   │       ├── test.md
│   │       ├── security.md
│   │       ├── review.md
│   │       └── task-log.md
│   ├── <thread-name-b>/
│   │   ├── docs/
│   │   └── result/
│   └── <thread-name-c>/
│       ├── docs/
│       └── result/
```

`threads/<thread-name>/` は、Codexプロジェクトと1対1で対応するタスク単位の作業領域です。スレッド名は用途に応じて自由に決定し、新しいスレッドは同じ構成で追加します。`rules/` 配下のMarkdownファイルは共通ルールとして扱い、適用範囲や優先順位は `rules/README.md`、スレッドの分離方法は `rules/thread-operation.md` を参照してください。

## 役割

`Owner` は人間の承認者です。Planner、Implementer、Tester、Security Operator、Reviewer、Documenter が worker として作業します。各 worker の詳細は対応する `worker-definitions/*.md` を参照してください。

## Workerタスク設定

モデル、推論レベル、標準接続サイクルは `rules/worker-task-settings.md` を参照してください。

| 役割 | 主な責任 | コード変更 |
| --- | --- | --- |
| Owner | 目的、優先順位、リスク、最終判断 | 原則なし |
| Planner | 現状調査、要件分解、実装計画 | なし |
| Implementer | 承認済み計画の実装 | あり |
| Tester | 自動検証、再現手順の整理 | なし |
| Security Operator | 秘密情報、外部操作、安全性の確認 | なし |
| Reviewer | 要件・設計・安全性の独立確認 | なし |
| Documenter | 判断・結果・教訓の永続化 | 文書のみ |

## タスク統合候補の判定

タスクの継続・統合候補は、`history/index.md`の新ルール統合判定台帳を候補抽出元として、task-id、タスク名、目的、対象リポジトリ、ローカルパス、ベースブランチ、機能・レイヤー、タスク概要の8項目だけを比較する。task-idは候補行の識別子であり、タスク名やtask-idの一致だけで統合しない。比較順序は対象リポジトリ一致、ローカルパス完全一致、ベースブランチ一致、機能・レイヤー、タスク概要とする。`状態`と`最終更新`、Owner判断、効果確認状態は統合判定の条件に使用せず、必要な場合は別途証跡として確認する。判定結果は`継続`、`統合候補`、`関連のみ`、`別タスク`、`判定不能`へ分類する。

`統合候補`はOwnerの明示承認前に確定せず、history原本の変更、自動削除・移動・改名・上書きを行わない。候補情報の欠落、矛盾、比較不能、Owner判断不明がある場合は、影響・停止理由・再開条件を結果へ記録して停止する。

## 承認ポイント

- タスク定義、調査結果、実装計画：Owner が承認する
- 設計変更、計画外変更、外部サービス操作、本番操作：Owner が判断・承認する
- レビュー結果、マージ、リリース：Owner が採否を判断する

テスト成功だけでは、設計の採用や本番反映を自動承認しません。

## 停止条件

共通の停止条件、安全ゲート、承認ゲートは `AGENTS.md` と適用対象の `rules/` を正本とします。停止時は理由、影響、未確認事項、Ownerに求める判断を結果ファイルへ記録します。

## ファイルの受け渡し

正式な引き継ぎ情報は会話履歴ではなく、対象スレッドの現行結果ファイルまたは指定された履歴スナップショットです。正本指定、manifest、履歴退避、復旧は `rules/thread-operation.md`、証跡とOwner判断は `rules/worker-evidence.md`、承認は `rules/plan-approval-required-info.md` を参照します。

## 安全上の注意

- 実案件のソースコード、顧客情報、個人情報、秘密情報をこのリポジトリへ追加しない
- 実案件で利用する場合は、テンプレートを対象プロジェクトへコピーして固有情報を記載する
- 本番操作、データ削除、外部サービスへの書き込みは人間の承認を必須にする
- Public公開前に、非公開URL、認証情報、社内固有の名称、ライセンス上の問題がないことを確認する
