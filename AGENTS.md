# 開発ルール

## 作業方針

- 変更前に既存ファイルと目的を確認する
- 実装前に対象、対象外、完了条件、リスクを整理する
- 計画外の変更を行わない
- 変更内容と検証結果を記録する
- 既存の手動変更を勝手に上書きしない
- ルールや機能の追加は、現行タスクだけの一時対応ではなく、タスク完了後も残り続け、他プロジェクトへの移行時点から有効になるシステム機能として設計する。詳細は `rules/README.md` と適用対象の共通ルールに従う
- 改善事項は、発見・汎用性評価・Owner判断・計画・実装・Reviewer受入・実運用後の効果確認・再評価・記録更新・完了または継続の共通サイクルで扱う。Reviewer受入と効果確認を混同せず、詳細は `rules/development-improvement-record.md` に従う
- 他プロジェクトへ移行した際は、共通rules、worker定義、テンプレート、記録先、責任者、停止条件を確認するまで有効化完了としない。不足時は未有効化として停止する
- 技術非依存性は役割、能力、入出力、状態、判定基準、証跡、停止条件で確認し、媒体変更後も正本・責任・状態・証跡・完了条件の意味を維持する。避けられない技術依存は例外記録なしに共通仕様へ組み込まない
- コミットは1目的にまとめ、コミットメッセージ規約に従う
- workerの役割、作業領域、入出力は `worker-definitions/` の定義に従う
- worker間の正式な引き継ぎは会話ではなく、対象スレッドの `threads/<thread-name>/result/` の結果ファイルで行う
- 各workerは完了時に親タスクへ判定、結果ファイル、未確認事項、次のworkerを報告し、親タスクは確認後に次工程へ接続する
- 各workerの完了報告の項目順・Owner判断の配置は `rules/worker-report-template.md` に従う
- Owner判断の意味、OJ採番、本文表示、IDなし・不明・重複・対象外回答の扱いは `rules/worker-evidence.md` に従う
- 用語の正本、標準的な意味、取り違え防止、未登録語・意味衝突時の停止条件は `rules/glossary.md` を参照する。個別資料へ用語定義を重複掲載しない

## 禁止事項

- 秘密情報、個人情報、顧客固有情報の追加
- 特定のプログラミング言語、shellプログラム、実行可能プログラム、外部ライブラリによる機能実装。詳細は `rules/implementation-medium.md` に従う
- Ownerの明示承認がないファイルの削除
- 破壊的なGit操作
- 本番環境への接続
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
- `threads/<thread-name>/result/current-task.md`: 各スレッドでPlannerへ渡す現在のタスク
- `threads/<thread-name>/result/task-log.md`: Documenterが現行タスクの判断・結果・残課題を記録する正式結果ファイル。履歴退避と更新境界は `rules/thread-operation.md` に従う
- `development-improvement.md`: Documenterが全スレッド共通の開発サイクル改善を一覧で積み上げる永続記録
- `worker-definitions/`: workerごとの役割、作業領域、入出力、完了条件
- `threads/<thread-name>/result/`: workerが同じスレッドの後続工程へ渡す現行タスクの結果ファイル。worker別のサブフォルダは作成せず、退避指示時は`history/yyyyMMddhhmm/`へ先に保存する
- `history/yyyyMMddhhmm/`: Ownerの退避指示時点で保存するタスク・worker結果の読み取り専用スナップショット
- `rules/`: コミット規約やworkerタスク設定などの恒久的な開発ルール
- `rules/current-task-template.md`: 各スレッドの`current-task.md`に記載する識別情報とタスク定義のテンプレート
- `rules/worker-report-template.md`: 各workerの完了報告と結果ファイルの共通テンプレート
- `rules/workflow-consistency-check.md`: current-task、結果ファイル、projectId、実行環境の整合性チェック手順
- `rules/worker-health-check.md`: Owner起点のworkerアクセス・状態確認、不足worker追加前後の手順
- `rules/plan-approval-required-info.md`: 計画関連Owner向け回答の対象・実行環境・ブランチ必須情報
- `rules/requirement-definition-format.md`: Planner接続前の要件定義提案フォーマットと接続ゲート
- `rules/task-initialization-and-requirement-gate.md`: docs/result初期化、issue-memo更新、正本照合、worker接続の実施ゲート
- `rules/rule-refresh.md`: ルール更新後の自動再読込、影響確認、停止ゲート
- `rules/local-rules.md`: ローカルルールの正本配置、命名、適用判定、保護対象、証跡、移行・欠落時の共通ルール
- ファイル側threadの初期登録は資料保存領域だけを準備する非実行工程とし、会話・worker・project・task・識別子の作成・接続・採番は要件定義開始後の別ゲートで行う。詳細は`rules/glossary.md`、`rules/thread-operation.md`、`rules/task-initialization-and-requirement-gate.md`を参照する

## workerの流れ

```text
threads/<thread-name>/result/current-task.md
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

- worker固有の現行結果ファイル、Documenterの`task-log.md`、動作確認記録のパスは `worker-definitions/` と `rules/thread-operation.md` に従う。履歴退避後の更新境界も同ルールに従う
- ルートの `development-improvement.md` は汎用的な改善項目だけを追記・更新し、タスク固有の詳細を混在させない
- 1つのスレッドでは1タスクだけを扱い、対象スレッドとタスクは `current-task.md` で識別する
- `current-task.md` の6項目、Task Definition、Task Lifecycleは `rules/current-task-template.md` に従う
- project/thread/taskの1対1対応、1 active制約、履歴退避・復旧、新規タスク境界は `rules/thread-operation.md` に従う
- worker接続前と作業開始前の整合性確認は `rules/workflow-consistency-check.md` に従う
- タスク初期化、要件定義中のissue-memo更新、Planner・後続worker接続前の実施証跡は `rules/task-initialization-and-requirement-gate.md` に従う。理解確認だけでは接続条件を満たさない
- ルール追加・更新後は、`rules/rule-refresh.md`に従い、指示を待たず次の回答・操作前に更新後の正本を再読込する。再読込と影響確認が未完了なら処理を停止する
- 同一タスクの再検証では結果ファイルへ差分を追記し、新規タスクへの切替時も履歴と旧resultを保全する。詳細は `rules/thread-operation.md` に従う
- Implementerは実装と最低限の変更記録を行い、詳細な検証・整合性確認・受入判定はReviewerへ移譲する。固有責務は `worker-definitions/` に従う
- 共通の入力ゲート、証跡、報告、履歴、接続サイクル、計画回答、動作確認、worker状態は対応する `rules/` の正本を参照する
- `rules/local/` のローカルルールを参照する場合は、共通rulesを先に読み、適用条件・責任者・競合・証跡を `rules/local-rules.md` に従って確認する。共通rulesの保護対象を弱めるローカルルール、判定不能なルール、欠落した正本は適用せず停止する
- 全タスクで`threads/<thread-name>/docs/operation-check.md`を作成または更新し、固定6見出し・7列の対応前後比較・根拠区分・未確認区分をPlanner計画、Reviewer受入、Documenter記録の各境界で確認する。欠落時は受入・完了・履歴操作・次タスク接続を停止する
- Plannerは計画承認前に実装を開始せず、計画外変更・停止条件・安全性懸念は親タスクへ報告する
- Owner判断に未回答・保留・不明・対応不明が残る場合は、次工程、完了、履歴操作を停止し、残件ごとの回答プロンプトを提示する。詳細は `rules/worker-evidence.md` に従う
- Reviewerが修正依頼と判定した場合は `rules/worker-task-settings.md` の再確認サイクルに従い、受入後にDocumenterへ接続する

## 停止条件

- 計画外の変更が必要になった
- 同じ検証に2回連続で失敗した
- 要件の解釈が複数に分かれた
- 承認されていない削除、上書き、公開など不可逆な操作が必要になった
- 外部サービスの認証や権限が必要になった
- 公開してよいか判断できない情報が見つかった
