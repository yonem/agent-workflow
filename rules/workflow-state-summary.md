# 状態サマリー運用ルール

## 目的と責務

状態サマリーは、実行プログラムを使わず、現行タスクの状態を要約するためのルールと出力テンプレートである。状態サマリー自身の確認項目は維持するが、通常のworker result本文の項目順・見出し・表形式を固定するものではない。実装媒体の禁止事項は `rules/implementation-medium.md` を正本とする。状態を変更せず、整合性確認の結果を必須入力にしない。固有の未確認事項・要約根拠・次workerは `threads/<thread-name>/result/changes.md` に記録し、進行中タスクの共通台帳は `threads/<thread-name>/docs/task-progress.md` を正本とする。

参照関係は次のとおりとする。

```text
current-task.md
  ├─ rules/thread-operation.md              : active境界・履歴・resultの正本
  ├─ rules/worker-evidence.md               : Owner判断・証跡・未確認の扱い
  ├─ rules/plan-approval-required-info.md   : 対象環境・承認情報
  ├─ rules/development-improvement-record.md : 改善サイクル・効果確認・移行先有効化
  ├─ rules/workflow-integrity-check.md      : 整合性確認とは独立した参照ルール
  └─ result/plan.md, changes.md, task-progress.md, history/*/manifest.md
```

## 入力条件

対象スレッド、実行コンテキストID、対象リポジトリ、実行ディレクトリ、ブランチ、task-id、タスク名、目的が `current-task.md` と一致することを確認する。実行コンテキストIDの形式は選択したバックエンドから取得し、特定製品のproject IDを共通必須値にしない。新規運用ではhistory-key/run-idを入力にしない。入力が欠落・矛盾している場合、要約を `確認済み` として出力しない。

次の資料から、記録値だけを抽出する。

| 入力 | 要約項目 |
| --- | --- |
| `current-task.md` | task-id、タスク名、目的、Task Lifecycle、Worker Registry、Draft Policy |
| `result/plan.md` | 承認済み対象、対象外、次工程、Owner判断 |
| `result/changes.md` | 実施済み変更、未確認事項、次worker |
| `docs/task-progress.md` | 共通進捗、資産区分、受入条件、Owner判断、後続候補 |
| `history/index.md` | 新ルール台帳の10項目（task-id、タスク名、目的、対象リポジトリ、ローカルパス、ベースブランチ、機能・レイヤー、タスク概要、状態、最終更新）。統合判定には状態・最終更新を除く8項目だけを使用 |
| `history/<task-id>/manifest.md` | task-id、タスク名、目的、履歴識別と状態 |
| `history/task-legacy-history-backup/manifest.md` | backup管理識別子、旧配置対応、候補・復旧対象外の状態 |
| `rules/development-improvement-record.md` | 技術非依存性、媒体中立性、例外記録、移行先有効化の要約基準 |

## 正本・参照先・更新責任・更新境界

状態サマリーは要約であり、正本を置き換えない。正本と責任境界は`rules/thread-operation.md`の資料マップを参照し、次を表示する。

- 現行taskの識別正本、共通進捗正本、承認済み計画、Implementer結果、Reviewer判定のパス
- `history/index.md`と`history/<task-id>/manifest.md`の参照関係
- 要約対象の更新責任と、履歴原本・legacy・backupの更新境界
- 正本不明、参照切れ、責務重複、境界不明がある場合の`未確認`または`矛盾`判定

## 改善サイクルの状態集約

Reviewer受入、Reviewer受入結果に対するOwnerのDocumenter開始承認、Documenter記録、Owner完了判断が揃えば対応完了とする。Reviewer受入は自律サイクルの終点であり、Documenter開始はOwner承認を必要とする。運用中の不具合・改善点は現行taskへ混在させず、発生時のOwner依頼として簡易モードを新たに判定する。次の状態を`task-progress.md`と担当resultから要約する。

- `未着手`：対応タスクが開始されていないことを表示する。任意の効果観測の有無は状態判定に使用しない
- `未着手`：改善候補として把握されているが、Owner承認済みの計画・実装・受入に着手していないことを表示する
- `対応中`：Owner承認後の計画・実装・受入を含む対応が進行中であることを表示する。細部のworker工程は`current-task.md`、`task-progress.md`、各resultから別途表示する
- `Reviewer受入・Documenter開始承認待ち`：Reviewerが`受入`または`条件付き受入`を返し、自律サイクルが終了したため、OwnerによるDocumenter開始承認を待っている状態を表示する。この状態でDocumenterを自動接続しない
- `Documenter記録完了・Owner完了確認待ち`：現行taskの記録が完了し、Ownerのclose確認を待っている状態を表示する。後続候補の有無は別欄で表示し、候補だけでtask状態を停止しない
- `operation-check.md`構成欠落：必須6見出しの順序、7列の対応前後比較表、根拠区分、未確認区分、状態台帳との整合の欠落・不一致箇所と、後続Documenterによる是正待ちを表示する
- ルール変更影響確認：直接対象、参照対象、記録対象、移行先導入対象、正本・参照先・更新責任・更新境界、旧表現、判定分類、変更前後の確認結果、未確認事項、停止理由・再開条件を表示する
- IR/IMPはtask本体の必須ゲートではなく、Ownerが管理する後続改善情報として表示する。欠落や未確認だけでtask本体を停止しない
- 新規task開始の終了ゲート：Ownerの明示指示、現行状態更新、manifest、`history/index.md`、次task接続の各段階を表示する。IMP再確認は後続改善の記録であり、task本体の完了ゲートではない

技術非依存性・媒体中立性の要約では、役割、能力、入出力、状態、判定基準、証跡、停止条件の確認結果と、技術依存例外の有無・分離先・Owner判断を表示する。特定技術、媒体、製品、実行環境が必須前提になっている場合や、媒体変更で正本・責任・状態・証跡の意味が失われる場合は`未確認`または`矛盾`として停止する。

Documenter記録後は現行taskの記録状態、Ownerのclose確認または判断の記録元、後続改善候補の管理先を分けて表示する。候補や観測の未確認だけでは`矛盾`とせず、結果資料の欠落・不一致がある場合にだけ停止する。

## 要約項目と照合順序

1. Task Contextとして、task-id、タスク名、目的、対象スレッド、実行コンテキストまたは代替境界証跡、対象リポジトリ、実行ディレクトリ、ブランチ、状態、原典パスを表示する。
2. Worker Registryを論理責務、実行単位種別、実行単位ID、親Coordinatorコンテキスト、Registry上の状態として表示する。Deliveryは3論理責務で同一IDを共有し、Reviewerは独立IDとする。Registry記録は `記録済み` とし、live worker状態は常に `未確認` と分けて表示する。
3. 現行resultの存在、担当worker、次worker、未確認事項を表示する。履歴resultを現行resultの代替として扱わない。
4. Owner判断表から工程影響が`停止`の`要判断`、`未回答`、`保留`、不明、本文不一致を停止残件として、`非停止`判断を後続候補として分けて表示する。
5. 進行中タスクの共通台帳に記録されている対象外・後続候補・安全性制約を短く表示する。
6. `history/index.md`の該当行をcurrent-task、task-progress、現行result、manifestと照合し、task-id、状態、未完了事項、Owner判断、履歴パス、正本・根拠、最終更新の不一致を未確認または矛盾として表示する。これは操作整合性の確認であり、統合候補の類似判定は8項目に限定する。操作前後の片側だけを確認済みとしない。
7. backup・legacyが存在する場合は通常候補・復旧対象外として表示し、backup内部の旧task-id/run-idを現行識別情報へ取り込まない。

## 判定語彙と停止条件

| 判定 | 意味 | 次の扱い |
| --- | --- | --- |
| `確認済み` | 必須入力とresultの存在を確認でき、残件なし | 次worker候補を表示できる |
| `未確認` | live状態、実測OS、結果、未確認事項の一部が未取得 | 未確認理由を表示し、完了扱いにしない |
| `矛盾` | task、正本、worker、Owner判断の記録が不一致 | 推測で要約せず停止 |
| `対象外` | Owner承認で今回扱わないと確定 | 根拠と後続扱いを表示 |

次の場合は、状態サマリーを次worker接続の承認根拠にしない。

- Worker Registryと現行resultの論理責務・実行単位ID・親Coordinatorコンテキスト・実行コンテキストが一致しない
- live状態が取得できないのに「実行中」「完了」「正常」と記録しようとする
- 工程影響が`停止`のOwner判断残件が1件でもある
- current-taskとresult/historyのtask-id、タスク名、目的、project/thread、対象、原典パスが混在している
- 次workerがWorker Registryに存在しない
- `history/index.md`の必須項目が欠落・読取不能・古い、またはcurrent-task、task-progress、現行result、manifestと不一致
- タスク切替の旧資料退避、manifest、index行、task-progress切替が未完了
- task-idの採番根拠が候補資料の識別子、ファイル名、日時だけ、または論理タスク台帳の棚卸しが未確認
- 既承認Owner判断の継承、範囲変更による再判断、未回答、新規の分類がない
- hisiの3状態以外の内部工程名をtask状態へ混入している
- task完了の根拠がReviewer受入、Reviewer受入結果に対するOwnerのDocumenter開始承認、Documenter記録、Owner完了判断以外へ置き換えられている
- サマリーと詳細の不一致、並行更新、正本・証跡・更新責任者の不明、`適用外`・承認範囲外の是正を自動更新で処理している
- ルール変更の直接対象・参照対象・記録対象・移行先導入対象のいずれかが不明、変更前後の確認結果がない、旧表現の扱いが不明、または正本・参照先・責任・境界が不一致である
- 影響確認を`影響なし`・`対象外`とした根拠がなく、`未確認`・`矛盾`・`変更漏れ`を残したまま受入・完了・移行先有効化を進めている
- 承認済み範囲の実装・検証・受入・記録が不足したまま次worker接続・受入・完了へ進めている。IMP判定表や効果確認の未実施だけではtask本体を停止しない
- `対応中`へ変更したのにOwner承認・計画・実装・受入の根拠がない
- Documenter記録完了後のOwner完了確認またはOwner判断がない
- Reviewer受入後にOwnerのDocumenter開始承認がないままDocumenterを接続している
- Owner確認前に完了、履歴退避、次タスク切替、新規IMP採番を確定している
- docs側`operation-check.md`の固定必須見出しの順序、7列の対応前後比較表、根拠区分、未確認区分、状態台帳との整合の欠落・不一致を検出したまま、受入・完了・次worker接続・履歴操作を確定している。これは通常のworker result本文の形式とは別である
- Reviewerの修正依頼、保留、未確認が残る間に`task-progress.md`または`history/index.md`をOwner完了確認待ちへ進めている
- 証跡なしの`対応完了`、別IMPの競合・検証不能を記録せず自動統合する、または承認済み範囲の実装・受入根拠がないまま進めている。効果確認中のIMPをtask状態へ混入させない
- Ownerの新規task開始指示がない、または状態更新→manifest→`history/index.md`→次task接続の順序を飛ばしている。全IMP再確認の未実施だけではtask本体を停止しない

## 合成ケース

| ケース | 入力状態 | 期待判定 | 要約上の扱い |
| --- | --- | --- | --- |
| Registry記録のみ | workerはRegistryにあり、live状態を取得していない | `確認済み` | Registryは`記録済み`、liveは`未確認` |
| Owner残件 | Owner判断に`要判断`または`保留`の`停止`判断がある | `未確認` | 残件を表示し、次worker接続を停止 |
| result欠落 | plan、changes、task-progressのいずれかがない | `未確認` | 欠落パスと影響を表示 |
| task不一致 | current-taskとresultのtask-id、タスク名、目的、project/thread、対象のいずれかが不一致 | `矛盾` | 正本を推測せず停止 |

## 出力フォーマット

MarkdownまたはJSONのいずれか一つで出力し、整合性確認の個別チェック結果を代替しない。

Markdownの最小形式:

```markdown
# 状態サマリー
- 判定: 確認済み / 未確認 / 矛盾 / 対象外
- 読み取り専用: true
- task-id: <task-id>
- タスク名: <task-name>
- 目的: <purpose>
- 対象実行コンテキスト/thread: <execution-context-id>/<thread-name>
- 次worker: <role>
- live worker状態: 未確認（Registryの記録値のみ）

| 役割 | 実行単位ID | 親Coordinatorコンテキスト | Registry | live |
| --- | --- | --- | --- | --- |
| Reviewer | <subagent-id> | <parent-session> | 記録済み | 未確認 |

## 未確認事項
- <理由またはなし>

## Owner判断残件
- <OJ-IDまたはなし>
```

JSONの最小形式:

```json
{
  "readOnly": true,
  "status": "確認済み",
  "taskId": "<task-id>",
  "taskName": "<task-name>",
  "purpose": "<purpose>",
  "executionContext": "<execution-context-id または not-applicable>",
  "thread": "<thread-name>",
  "workers": [
    {
      "role": "Reviewer",
      "executionUnitId": "<execution-unit-id>",
      "parentCoordinatorContext": "<parent-coordinator-context>",
      "registryStatus": "記録済み",
      "liveStatus": "未確認"
    }
  ],
  "nextWorker": "Reviewer",
  "liveWorkerState": "未確認（Registryの記録値のみ）",
  "uncertainties": [],
  "pendingOwnerDecisions": []
}
```

## Owner判断プロンプト

`停止`判断の残件がある場合は、状態サマリーだけで判断を補完せず、未回答・保留・不明な判断と今回新たに発生した増分判断だけを`changes.md`に記録して停止する。`非停止`判断はIRまたは新規im候補の参照先を表示して通常工程を継続する。回答済みの判断は回答プロンプトへ再掲しない。Owner回答ブロックにはOJ行だけを記載し、計画関連の必須4項目は別ブロックへ記載する。

```text
OJ-xxx（要約: 状態サマリーの<残件>）=yes
```

計画関連の必須情報ブロック：

```text
対象リポジトリ: <絶対パス>
実行ディレクトリ: <絶対パス>
ベースブランチ: <branch>
作業ブランチ: <branch>
```

すべての`停止`判断が `判断済み` または `反映確認済み` になるまで、次worker接続、履歴操作、worker状態変更を行わない。`非停止`判断はIRまたは新規im候補へ分離済みであれば通常工程を妨げない。残件がない場合は `Owner判断残件：なし`、`回答プロンプト：なし` と記録する。

## 結果ファイルへの記録

`changes.md` には、要約対象、入力資料、Registry記録値、live状態を未確認とした理由、未確認事項、次worker、停止理由を記録する。`task-progress.md` には、進行中タスクの共通進捗・資産区分・受入条件・Owner判断だけを集約し、Worker Registryのlive状態を推測して追記しない。状態サマリーは読み取り専用であり、入力資料・result/history/rulesを変更しない。

## タスク統合候補の状態表示

タスク統合候補を扱う状態要約には、候補抽出元が`history/index.md`の新ルール統合判定台帳であること、8項目（task-id、タスク名、目的、対象リポジトリ、ローカルパス、ベースブランチ、機能・レイヤー、タスク概要）、5段階の判定順序、分類、Owner判断の状態、未確認事項、停止理由、再開条件を表示する。`状態`と`最終更新`は統合判定の比較項目に含めず、必要な場合は別途証跡として表示する。`統合候補`は統合済みと表示せず、Owner承認前の現行task・history・manifest・indexの不変条件を表示する。実統合を表示する場合は、Owner承認済みの統合先、共通範囲、固有範囲、競合方針、操作後照合の結果を併記する。

候補一覧の欠落、現行資料との不一致、概要比較不能、分類根拠不明、Owner判断不明、history原本操作の要求がある場合は、正常・統合済み・完了とは表示せず、`判定不能`または`停止`として次worker接続と履歴操作を止める。
