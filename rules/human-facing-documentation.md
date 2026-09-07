---
scope: all-workers
scope: all-threads
status: active
---

# 人間向け資料と管理ファイルの共通ルール

## 目的

人間の判断・確認に使用する資料と、Planner・workerが参照する管理ファイルを、プロジェクトやタスクが変わっても同じ責務・形式・作成条件・復元条件で運用する。資料の欠落、正本不明、記載重複、判断漏れ、参照切れがある場合は推測で補完せず停止する。

`health-check.md`のWorker照合表は`rules/worker-health-check.md`、`history/index.md`の初期化と台帳は`rules/history-initialization.md`、通常のworker result本文は`rules/worker-report-template.md`を正本とする。本ルールは、それらを含む人間向け資料の共通構造、責任境界、作成・復元・接続手順を定義する。

## 適用対象と正本

| 資料 | 正本 | 主な利用者 | 主な責任 |
| --- | --- | --- | --- |
| 要件定義メモ | `threads/<thread-name>/docs/issue-memo.md` | Owner、Planner | 要件定義担当／Owner |
| Planner入力 | `threads/<thread-name>/result/current-task.md` | Planner | Ownerまたは指定作成責任者 |
| 現行タスク進捗 | `threads/<thread-name>/docs/task-progress.md` | Owner、全worker | Implementerが実装状態、Reviewerが受入状態、Documenterが記録状態を人間向けに要約 |
| Owner判断 | `threads/<thread-name>/docs/owner-jadge.md` | Owner、全worker | Planner・各worker・Owner |
| Worker照合 | `threads/<thread-name>/docs/health-check.md` | Owner、人間 | OwnerまたはPlanner |
| 汎用改善 | `<作業ディレクトリroot>/development-improvement.md` | Owner、全プロジェクト | Documenter／Owner |
| 履歴台帳 | `<作業ディレクトリroot>/history/index.md` | Owner、AI | Owner |

Owner判断の正本は`owner-jadge.md`とする。`owner-jadge.md`はOwner判断だけ、`task-progress.md`は現行タスクの進捗・証跡への参照・停止条件だけを扱い、互いの責務を重複させない。

## docsとresultの使い分け

`docs/`はOwnerや人間が読む要約・判断・進捗の資料、`result/`はworker間の引継ぎ、Planner入力、詳細な実施結果、受入結果、証跡の資料として使い分ける。AIが参照することだけを理由に人間向け資料を`result/`へ移さない。人間向け資料の詳細は人間が読める説明とし、AIだけが必要とする詳細・原典一覧・実施証跡は`result/`へ記録する。

- `issue-memo.md`と`task-progress.md`は、人間向けに整理した資料として`docs/`に置く。
- `result/current-task.md`、`result/plan.md`、`result/changes.md`、`result/review.md`、`result/task-log.md`は、Planner・worker・Documenterが扱う詳細な正式結果として`result/`に置く。
- `docs/`へ`result/`本文を全文複製せず、必要な場合は正本パスと短い要約だけを記載する。
- `result/`へ人間向けサマリーを重複作成せず、Owner判断や進捗の見える化は`docs/`の正本を参照する。

## 人間向け資料の共通構造

人間向け資料は、原則として次の2部構成とする。

1. `サマリー`：人間が短時間で状態、判断、結論を把握するための簡素な表または要約
2. `詳細`：AI・workerが根拠、条件、対応内容、未確認事項を追跡するための情報

サマリーと詳細を持つ資料では、次の規則を必ず適用する。

- 詳細の項目は、対応する識別子（OJ、IMP、task-idなど）の数値部分の昇順で並べる。
- サマリーの識別子と詳細の識別子は完全一致させ、相互に追跡可能にする。
- サマリーにない識別子を詳細へ追加せず、詳細にない識別子をサマリーへ掲載しない。
- 内容が未確認、重複、不一致、対応先不明の場合は並び替えや推測補完で解消せず停止する。
- 対応する識別子がない資料では、日付やファイル名を新たな論理識別子として付与しない。

## `owner-jadge.md`の形式

### 配置と責務

`threads/<thread-name>/docs/owner-jadge.md`をOwner判断の正本とする。Owner判断のサマリー、詳細、回答プロンプトだけを記録し、タスク概要・進捗・証跡・停止条件は`task-progress.md`へ記録する。

### 固定レイアウト

```markdown
# Owner判断

## サマリー

### 判断済み

| OJ | ステータス | OJ概要 |
| --- | --- | --- |

### 未決定

| OJ | ステータス | OJ概要 |
| --- | --- | --- |

### 回答プロンプト

<未決定OJだけをコピー可能な形式で記載。なければ「なし」>

## 詳細

### OJ詳細

#### OJ-001

- ID：`OJ-001`
- ステータス：
- 推奨対応（yes／no）：<yesまたはnoで回答できる質問>
- 対応の詳細：
```

- `判断済み`と`未決定`は別表にする。
- `回答プロンプト`はサマリーの最後に置き、未決定OJだけを記載する。
- 回答値は`=yes`を規定値とし、Ownerが否認・保留・代替案を選ぶ場合だけ変更する。
- 詳細はOJ-ID昇順で並べ、各詳細のIDはサマリーのIDと一致させる。
- 回答済みOJは後続の回答プロンプトへ再掲しない。同じ論点は既存OJ-IDを継続し、別論点だけ新規OJ-IDを採番する。
- Owner判断の回答ブロックと、計画関連必須4項目のブロックを混在させない。
- 判断残件がない場合は、回答プロンプトを作成せず`Owner判断残件：なし`、`回答プロンプト：なし`とする。

## `task-progress.md`の形式

`threads/<thread-name>/docs/task-progress.md`は、現行タスクの共通進捗、正本・証跡、停止・再開条件を人間向けに要約する正本とする。Owner判断の質問本文は`owner-jadge.md`へ置き、同じ内容を重複記載しない。

`task-progress.md`は人間向けの現行状態サマリーであり、AIが確認する詳細証跡の保存先ではない。状態、工程、Owner判断、停止・再開条件に変化があった会話またはworker境界ごとに更新するが、既存の関連行・節を更新し、同じ内容を機械的に追記しない。詳細な実施結果、照合値、原典一覧、受入根拠は対応する`result/`へ記録し、`task-progress.md`には短い要約と参照先だけを残す。

更新時は、変更されたテーマを特定し、同じテーマの既存行・節へ現行値、変更理由、参照先を統合する。既存の項目で表現できない新しいテーマだけを適切な節へ追加し、過去の状態や同一内容を履歴として連続追記しない。過去状態の詳細な経緯が必要な場合は、`result/changes.md`または`result/task-log.md`へ記録し、`task-progress.md`から参照する。

```markdown
# TASK-xxx 共通進捗台帳

## 識別情報

- task-id：`TASK-xxx`
- タスク名：
- 対象スレッド：
- 対象project：
- 対象リポジトリ：
- 作業ブランチ：

## 現在状態

- 判定：
- 次worker：
- Owner判断：
- 終了ゲート：

## 要件要約

## タスク詳細

## 正本・証跡

## 未確認事項

## 停止・再開条件
```

作成時は`result/current-task.md`、`issue-memo.md`、承認済み`plan.md`と照合する。worker工程、変更前後、証跡、停止条件、次工程はここへ集約し、Owner判断の詳細表は複製しない。

## `issue-memo.md`の要件定義運用

### 要件定義中

- `threads/<thread-name>/docs/issue-memo.md`を要件定義中の決定事項、変更理由、未決定事項、懸念、Owner確認事項の正本とする。
- 要件定義、タスク作成、対応案の提示を求める意図を検知した時点で、最初の要件定義案を提示する前に`issue-memo.md`を作成または再開する。要件承認前でtask-idが未確定の場合は、`task-id：未採番（要件定義中）`として記録する。
- 現行タスクの要件変更であれば、現行の`issue-memo.md`を再開して更新する。新規タスクであれば現行タスクの資料へ混在させず、現行タスクの完了・退避境界を確認したうえで新規スレッドの`issue-memo.md`を作成する。凍結済みのissue-memoを別タスクへ流用しない。
- 新規作成時は、少なくとも次の初期情報を記録する。

```markdown
# 要件定義メモ

## 基本情報

- task-id：未採番（要件定義中）
- 状態：要件定義中
- 開始日時：
- 対象スレッド：
- 要件定義の対象：

## 決定事項

## 未決定事項・懸念
```

- 要件定義に関係する会話の都度、決定、変更、未決定、懸念、Owner確認事項を更新する。ただし会話本文をそのまま転載せず、既存の関連項目を更新または同じ章へ統合する。
- 関連性の高い項目は目的、配置、責務、形式、作成・復元、停止条件などのテーマ別にまとめ、項目を点在させない。
- 変更経緯を残す場合も、日付、変更内容、理由、影響を簡潔に記録し、発言単位の議事録にはしない。
- 既存のOJ、IMP、task-idは参照できるが、issue-memo固有の新しい識別子を作成しない。
- 原文要件を保全する場合は、決定事項の整理と混同しない節へ分離する。

要件定義開始時の`issue-memo.md`作成または再開を確認できない場合は、要件定義案の提示、task-idの確定、`result/current-task.md`の作成、Planner接続、後工程接続を停止する。作成失敗や現行タスクとの境界不明は、推測で補完せず、原因と再開条件を結果へ記録する。

### 要件確定とPlanner接続

1. 要件定義担当が`issue-memo.md`へ決定事項を更新・統合する。
2. 対象、対象外、完了条件、リスク、固定レイアウト、正本、責任、停止条件を整理する。
3. 要件定義の終了・凍結を明示し、`issue-memo.md`と`result/current-task.md`を照合する。
4. 凍結内容を`result/current-task.md`へ転記し、Planner入力の正本とする。
5. Plannerが計画を作成する。

凍結後に対象範囲、目的、完了条件、責任、正本、停止条件を変更する場合は、issue-memoを再開して変更内容を追記し、Owner再承認後に`result/current-task.md`と`plan.md`を更新する。承認なしに現行タスクへ混在させない。

## 欠落時の作成と復元

1. 欠落資料の正本、作成責任、適用範囲、参照先を確認する。
2. 本ルールまたは専門ルールの標準テンプレートで新規作成する。
3. 同一taskの現行`docs/`・`result/`、`result/current-task.md`、`task-progress.md`、指定されたhistory manifestの順に既知情報を回収する。
4. 回収した値には原典パスを付け、不明値は`未確認`として記録する。
5. 複数原典の不一致、正本不明、部分復元、履歴原本の変更要求がある場合は停止する。

worker間連携用の`result/`本文は、意思疎通に必要な内容が伝わる限り形式を固定しない。ただし、保存先、担当、task識別、判定、未確認事項、次工程、停止条件の意味は維持する。`result/current-task.md`だけはPlanner入力の専用テンプレートを維持する。

## 移行先での初期有効化

別プロジェクトへ移行した時点から有効化するため、次を確認する。

- 共通`rules/`、worker定義、テンプレート、記録先、責任者、停止条件が存在する。
- `result/current-task.md`、`task-progress.md`、`owner-jadge.md`、`health-check.md`を標準形式で新規作成できる。
- `history/index.md`を`rules/history-initialization.md`の形式で初期化できる。
- `issue-memo.md`から`result/current-task.md`へ要件を確定転記できる。
- Owner判断、Reviewer受入、Documenter記録、効果確認、再評価の責任境界を確認できる。

不足、責任不明、記録先不明、形式不一致、正本不明がある場合は`未有効化`として停止する。

## 検証と更新境界

- 変更前後に正本、参照先、更新責任、作成条件、復元条件、停止条件を照合する。
- 人間向け資料のサマリーと詳細、詳細の昇順、IDトレースを確認する。
- `health-check.md`の表形式は`rules/worker-health-check.md`、`history/index.md`の形式は`rules/history-initialization.md`へ照合する。
- Reviewer受入、Documenter記録、Owner完了確認、効果確認を混同しない。
- 共通ルール、テンプレート、導入手順へ反映されていない要件は、TASK-012の現行資料に記録されていても永続化済みとは判定しない。
