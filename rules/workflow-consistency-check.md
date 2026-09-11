---
scope: all-threads
status: active
---

# ワークフロー整合性チェック

worker接続前および作業開始前に、`current-task.md`、`docs/task-progress.md`、スレッドの結果ファイル、workerの実行環境が同じタスクを指していることを確認するための共通手順。

本書は記載値と実環境の照合手順を正本とする。用語の標準的な意味と取り違え防止は`rules/glossary.md`を正本とする。6項目とTask Definition/Task Lifecycleの記載形式は `rules/current-task-template.md`、計画関連Owner回答の記録は `rules/plan-approval-required-info.md`、project/thread/taskのライフサイクルは `rules/thread-operation.md`に従う。共通台帳の正本は次タスク以降`docs/task-progress.md`とする。

## 資料の正本・責務・更新境界

| 資料 | 正本として扱う情報 | 更新責任 | 更新境界・参照先 |
| --- | --- | --- | --- |
| `docs/issue-memo.md` | 要件、決定事項、現状・要求状態比較 | 要件定義担当／Owner | 要件定義中。詳細計画は`result/plan.md`へ参照する |
| `result/current-task.md` | task識別、Task Lifecycle、Registryの期待値 | Ownerまたは指定作成責任者 | Planner接続入力。実測状態を上書きしない |
| `docs/task-progress.md` | 現行taskの状態、工程、停止・再開条件 | 工程担当 | 要約のみ。詳細証跡は`result/`へ置く |
| `result/plan.md` | 承認済み計画、対象・対象外 | Planner／Owner | 実装範囲の入力。実装結果で書き換えない |
| `result/changes.md`等 | workerの詳細な実施・確認結果 | 担当worker | 現行taskの結果のみ。履歴原本を統合しない |
| `docs/owner-jadge.md` | Owner判断、OJ-ID、回答 | Owner／関連worker | Owner判断のみ。進捗を複製しない |
| `docs/health-check.md` | workerの照合結果 | OwnerまたはPlanner | 明示トリガー時のみ。Registryを代替しない |
| `history/index.md`・manifest | 履歴台帳メタデータ | Owner | 履歴操作時のみ。history原本は読み取り専用 |

Registryは期待値、一覧・個別読取・Owner確認はそれぞれ独立した確認値として扱い、相互に代替しない。

ファイル側threadの初期登録は資料保存領域の準備だけであり、Codex会話、クルー会話、worker、project、task、識別子、history台帳を発生させない。接続前照合・Worker Registry・active taskの確認は、要件定義開始とtask識別情報が確定した後に行う。初期登録とtask開始を区別できない場合はdocs/result/historyの更新と接続を停止する。

## ルール変更時の影響確認

共通rules、worker定義、報告テンプレート、README、workflow整合性資料、移行先導入手順を変更する前後では、次の順序で影響を確認する。対象を特定できない場合は実装を開始せず、`changes.md`へ停止記録を残す。

1. 変更案、目的、対象、対象外、期待状態、発火契機を確認する。
2. 直接変更対象、参照対象、記録対象、移行先導入対象へ分類し、各資料の正本、参照先、更新責任、更新境界を一覧化する。
3. 旧用語、旧状態、旧パス、旧責務、旧Owner判断形式を列挙し、現行資料での許容・修正対象・保全対象を区別する。
4. 期待値と実際値、正本候補、停止条件、共通仕様とプロジェクト固有設定の相違を、影響あり・影響なし・対象外・未確認・矛盾・変更漏れへ分類する。
5. 承認範囲内の変更だけを実施し、変更後に同じ影響一覧、正本マップ、旧表現、停止条件、移行先有効化条件を再確認する。

### 影響確認の記録項目

```text
変更対象・変更理由:
直接対象:
参照対象:
記録対象:
移行先導入対象:
正本・参照先・更新責任・更新境界:
旧表現と扱い:
判定分類:
変更前の根拠:
変更後の確認結果:
未確認事項:
停止理由・影響・再開条件:
```

履歴原本、legacy、backup、外部サービスは、Owner承認と計画に明示されない限り変更対象へ含めない。task-id、project/thread、Worker Registry、IMP-ID、draftsなど本プロジェクト固有の値を共通仕様の必須条件へ昇格させない。

## 確認対象

```text
threads/<thread-name>/result/current-task.md
threads/<thread-name>/docs/task-progress.md
threads/<thread-name>/result/*.md
worker自身のCodexプロジェクト所属・実行環境
```

用語を判定条件、状態、役割、資料名として扱う場合は、`rules/glossary.md`の登録語・標準的な意味・取り違え防止の意味を照合する。未登録語、同音異義語、造語、略語、意味衝突、デフォルト解釈不能が残る場合は、整合性確認を完了扱いにせずOwnerへ確認を求める。

## project境界・会話種別・Worker Registryの照合

現在のCodex projectId、Ownerが指定したファイル側thread、`current-task.md`のprojectId・対象リポジトリ・実行ディレクトリをこの順で照合する。Owner会話は判断・承認の記録元、クルー会話はworkerの実行単位、ファイル側threadは`docs/`と`result/`を保存する作業領域であり、同じ「スレッド」として扱わない。

Worker Registryは`current-task.md`を期待値の正本、一覧・個別読取・health-check・接続差分を実測の根拠とする。既存候補を先に照合し、役割、threadId、hostId、所属projectId、モデル・推論レベル、実行ディレクトリ、状態を比較する。候補の新規追加は、不足を確認しOwnerが明示指示した場合だけ行う。

projectIdまたは対象ファイル側threadが未確認・不一致・複数候補、会話種別を区別不能、Registryと実測が不一致・重複・未確認の場合は、docs/result/historyの参照・更新、worker接続、履歴操作、task切替を停止し、期待値、実際値、根拠、影響、再開条件を親タスクへ報告する。

## 実装媒体・承認範囲の照合

作業開始前に、リポジトリの実態、taskの変更対象、計画に記載された許可媒体・許可操作、禁止操作、Owner承認状態を照合する。コード、テスト、解析、ビルド、文書、ルールを変更対象ごとに判定し、文書中心・コード含有・混在のいずれでも承認範囲外の操作を許可しない。判定不能は停止とする。

## ファイル上の確認項目

| 確認項目 | 確認内容 |
| --- | --- |
| 必須項目 | `current-task.md`に6項目がすべて記載されている |
| スレッド名 | 記載されたスレッド名と保存先の`<thread-name>`が一致する |
| CodexプロジェクトID | 値が空でなく、対象スレッドの専用プロジェクトを示している |
| 対象リポジトリ | 絶対パスで記載され、対象として参照可能である |
| Codex実行ディレクトリ | 値が記載され、workerがアクセス可能である。対象リポジトリと異なっていてよい |
| ベースブランチ | 値が記載されている |
| 作業ブランチ | ブランチ名、または`記述ルールに従い新規作成`が記載されている |
| Task Lifecycle | 履歴退避・復旧・再検証を扱う場合、開始日時、task-id、退避元project/thread、状態が記載されている。新規運用ではhistory-key/run-idを使用しない |
| 共通台帳 | 次タスク以降、`docs/task-progress.md`が存在し、current-task、result、historyとの役割境界を示している |
| 結果ファイル | Planner以外は前工程と担当workerの固定ファイルが対象スレッドの`result/`配下にある。Plannerの`plan.md`は未作成でも工程上正常とする |

## worker・Ownerが確認する項目

ファイルだけでは確認できない次の項目は、worker自身とOwnerが確認する。

- workerのCodexプロジェクトIDが`current-task.md`と一致している
- workerが対象スレッドの専用Codexプロジェクトに所属している
- workerが対象リポジトリを参照できる
- workerのCodex実行ディレクトリが記載値と一致している
- 作業ブランチの実体が記載値と一致している
- 履歴を扱う場合、通常の履歴は`history/<task-id>/manifest.md`のtask-id、元project/thread、対象リポジトリ、状態と一致している。`history/task-legacy-history-backup/`を扱う場合はbackup manifest、旧配置対応、候補・復旧対象外の記載を確認し、旧値を新規task-idへ遡及利用しない

## 適用ルール確認ゲート

worker接続前、作業開始前、作業完了報告前、Reviewer受入前、Documenter記録前に、適用対象の共通rulesと`rules/local/`を確認する。確認結果は担当workerのresultへ記録し、少なくとも次を明示する。

- 参照した共通rulesとローカルルールの正本パス
- 対象task、リポジトリ、ローカルパス、ブランチ、機能・レイヤー、利用シーンへの適用判定
- 共通rulesの保護対象、ローカルルールとの優先順位、競合の有無
- 実施内容へ適用したルール、証跡パス、未確認事項
- 判定不能・競合・正本欠落時の停止理由と再開条件

上記の記録がない、適用範囲を確定できない、共通rulesの保護対象を確認できない、または競合が解消していない場合は、作業開始、次worker接続、受入、完了、履歴操作を停止する。確認したという報告だけで、対象・判定・証跡を省略してはならない。

## 判定

```text
すべて一致
  → 作業開始可能

1項目でも不一致・確認不能
  → 作業開始不可
  → 期待値と実際の値を親タスクへ報告
  → Ownerの修正・再接続後に再確認
```

不一致時は、誤った結果ファイルを作成・更新しない。projectIdの所属や外部サービスの認証状態を推測で補完してはならない。

## Worker接続前チェックリスト

OwnerはPlannerまたは各workerを接続する前に、次の項目を確認する。再起動後のworker再接続時も同じチェックリストを使用する。

```markdown
# Worker接続前チェック

- 対象スレッド：
- CodexプロジェクトID：
- 対象リポジトリ：
- 作業ブランチ：

## 確認項目

- [ ] Codexプロジェクトが対象スレッド専用である
- [ ] projectIdがcurrent-task.mdと一致している
- [ ] workerが対象プロジェクトに所属している
- [ ] 対象リポジトリへアクセスできる
- [ ] Codex実行ディレクトリを確認した
- [ ] current-task.mdの必須6項目が揃っている
- [ ] 履歴を扱う場合、Task Lifecycle、manifest、history/index.mdの対応を確認した

## 新規要件定義・タスク切替の整合性確認

新規task開始を確認する場合は、候補判定とは別に、次の順序と状態を照合する。

- 発言が明示的な新機能追加、新規task作成、要件定義開始、承認済み計画へのtask追加であること。相談・補足・確認・意見交換、または意図不明の場合は新規taskへ進めない。
- Ownerが現行taskの中断・終了または現行taskを残して別project・別thread等で実施する選択を明示していること。未回答・保留・不明ならtask-id、新規memo、履歴操作、active切替を停止する。
- 切替時は、現行docs/resultの退避先、manifest、`history/index.md`行、退避内容のtask-id・状態・原典・未完了事項を照合し、すべて成功していること。片側確認や部分成功は完了扱いにしない。
- 照合成功後だけ旧docs/resultの初期化と新規memo/current-task/task資料の作成を行い、新旧資料の混在がないことを確認する。初期化後は`docs/`と`result/`の全資料を確認し、旧task-id、旧OJ、旧状態、旧進捗、旧worker結果が残っていれば作業開始・worker接続・次task開始を停止する。
- task-idの一意性と同一projectのactive taskが1件であることを確認する。現行taskを残す選択時に同一projectのactive重複がある場合は停止する。

不一致・確認不能・部分成功・承認範囲外の削除や上書きがある場合は、期待値、実際値、証跡、影響、停止理由、再開条件を記録し、Reviewer接続や次task開始へ進めない。

## タスク統合候補の判定

- 候補抽出元は`history/index.md`に限定し、task-id、タスク名、目的、対象リポジトリ、ローカルパス、ベースブランチ、機能・レイヤー、タスク概要の8項目だけを記録値として比較する。`状態`と`最終更新`、Owner判断、統合候補、履歴パス、正本・根拠は統合判定の比較条件に含めない
- 比較順序は、対象リポジトリ一致、ローカルパス完全一致、ベースブランチ一致、機能・レイヤー分類、タスク概要（目的・対象・未完了事項・完了条件・制約）の詳細比較とする
- 判定は`継続`、`統合候補`、`関連のみ`、`別タスク`、`判定不能`へ分類し、各分類の根拠、影響、次の扱いを記録する
- 初回導入で`history/index.md`が未作成の場合は、`rules/history-initialization.md`に従い空の履歴領域へ初期台帳を作成してから照合する。初期化済みの移行先で`history/index.md`が欠落、読取不能、不一致、必須概要の比較不能、共通範囲と固有範囲の分離不能、Owner判断不明になった場合は推測で補正せず停止する
- Owner承認前の統合確定、history原本の変更、自動削除・移動・改名・上書きは行わない。例外整備が承認された場合も操作前後を照合する
- [ ] 次タスク以降、docs/task-progress.mdを共通台帳の正本として確認した
- [ ] 作業ブランチを確認した
- [ ] 同一スレッドで別タスクを実行していない
- [ ] 既存のresultファイルを確認した
- [ ] 起動するworkerの役割が正しい
- [ ] 適用する共通rulesとローカルルールの正本を確認した
- [ ] 適用範囲、保護対象、優先順位、競合、停止条件を確認した
- [ ] ルール適用結果の記録先を確定した

## 判定

- [ ] 接続可能
- [ ] 作業停止してOwnerへ確認
```

1項目でも確認できない場合は、workerを接続せず、確認できない項目と必要なOwner対応を記録する。

## 記録

確認結果は担当workerの結果ファイルへ記録する。少なくとも次を残す。

- 確認日時
- 確認したファイルと項目
- 期待値
- 実際の値
- 判定
- 不一致または未確認の場合の理由とOwnerへの依頼
- 履歴を扱う場合は、task-id、manifest、現行resultまたは履歴resultのどちらを正本としたか
