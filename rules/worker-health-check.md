---
scope: all-workers
status: active
---

# workerヘルスチェック

## worker作成状態

Worker Registryでは、作成・接続状態を`準備中`、`ready`、`接続済み`、`失敗`、`中断`に区別する。状態不明、重複候補、確定前は再作成せず、期待値・実際値・根拠・影響・停止理由・再開条件を記録してOwner確認へ停止する。

Ownerが同一Codexプロジェクト内のworkerの所属・アクセス可否・状態を確認するための手順を定義する。これはworkerチャットの運用状態を確認する手順であり、アプリケーションの稼働監視、外部サービス監視、ホスト監視には適用しない。

## `health-check.md`の責務境界

ファイル側threadの初期登録だけではworker確認を開始しない。health-checkはOwnerの明示依頼またはworker接続失敗を契機に、taskとprojectの境界が確定した後に実施する。初期登録からworker、会話、project、taskを推測・作成しない。

- `threads/<thread-name>/docs/health-check.md`は、Ownerがヘルスチェックを明示的に依頼した場合、または作業中のworker接続失敗を契機にヘルスチェックへ切り替えた場合に作成・更新する人間向けの正本である。
- workerの完了報告、`threads/<thread-name>/docs/task-progress.md`、`history/*/manifest.md`の代替や、通常作業で毎回作成する必須報告ではない。
- `docs/task-progress.md`には実施日時、判定、未確認事項、原典パスだけを参照として記録し、Worker Registryや状態一覧を全文複製しない。
- Ownerの明示トリガーまたはworker接続失敗による切替がない場合は、`health-check.md`が存在しないことを未実施・正常・異常のいずれとも推測しない。
- 前タスクの`result/health-check.md`は、対応するhistoryのスナップショットとともに旧資料として保全し、新タスクの状態証跡へ自動流用しない。

## `health-check.md`の固定レイアウト

現行の人間向け`health-check.md`は、見出しと次のSubagent照合表だけで構成する。実施日時、対象task、詳細ログ、回答プロンプト、補足説明はこのファイルへ記載せず、必要な証跡として`task-progress.md`またはworkerの`result/`へ記録する。資料は読み取り専用であり、表からworkerの状態、設定、接続、heartbeatを変更してはならない。

```markdown
# Worker照合

| 論理責務 | Subagent種別 | Subagent ID | 親session | 状態 | モデル | 推論 | 結果資料 | 判定 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Planner | Delivery |  |  |  |  |  |  |  |
| Implementer | Delivery |  |  |  |  |  |  |  |
| Documenter | Delivery |  |  |  |  |  |  |  |
| Reviewer | 独立Reviewer |  |  |  |  |  |  |  |
| Tester | 例外 | 対象外 |  | 対象外 | 対象外 | 対象外 | 該当なし | 対象外 |
| Security Operator | 例外 | 対象外 |  | 対象外 | 対象外 | 対象外 | 該当なし | 対象外 |
```

各列は次の意味で記録する。

- `論理責務`：Planner、Implementer、DocumenterはDeliveryの同一Subagent IDを共有する。Reviewerは独立ReviewerのIDを記録する。例外workerは接続しない限り`対象外`とする。
- `Subagent種別`：Delivery、独立Reviewer、または例外を記録する。
- `Subagent ID`：Owner session配下のSubagent識別子を実測記録する。独立threadの識別子で代用しない。
- `親session`：Owner sessionとの親子関係を`一致`、`不一致`、`未確認`で記録する。
- `状態`：確認できたworker状態をそのまま記録する。取得不能や対象外は`未確認`または`対象外`とし、推測しない。
- `モデル`、`推論`：設定正本との照合結果を実測記録する。取得不能や対象外は`未確認`または`対象外`とする。
- `結果資料`：担当resultおよび必要な`task-log.md`の存在・照合結果を記録する。
- `判定`：アクセス可否、所属一致、不一致、重複、対象外、未確認などの結論を短く記録する。

表の列順、役割行、値の意味を変更しない。値を確定できない場合は正常・不足と推測せず、`未確認`として記録する。

## 起動条件と入力ゲート

- ヘルスチェックは、Ownerが `ヘルスチェックを実行して` と明示するか、workerの所属・アクセス・状態確認を明示的に依頼した場合に開始する。加えて、作業中にworkerへ接続できなかった場合は、自動作成へ進まず、読み取り専用のヘルスチェックへ自動的に切り替える。通常のworker作業や会話から、接続失敗以外を理由に自動開始しない。
- worker接続失敗を契機に開始した場合は、失敗したworker、接続を試みた時点、失敗の事実、実行中タスク、未確認範囲を記録し、接続失敗をworker不足・アーカイブ済み・不一致と推測しない。
- 開始前に、最新の `threads/<thread-name>/result/current-task.md`、`rules/worker-task-settings.md`、`worker-definitions/*.md`、`rules/workflow-consistency-check.md`、既存のWorker Registryを確認する。
- `current-task.md`から対象スレッド名、CodexプロジェクトID、対象リポジトリ、Codex実行ディレクトリ、ブランチ、Worker Registryを取得する。対象projectIdが空、対象リポジトリと一致しない、または参照不能な場合は開始せず停止する。
- `list_projects`等で対象projectIdが対象リポジトリのGitプロジェクトであることを確認する。別projectId、別リポジトリ、別hostの一覧を対象結果へ混在させない。
- ヘルスチェックの実行は読み取り専用で開始する。Ownerの確認前に、worker作成、アーカイブ、削除、復元、移動、置換、その他の状態変更を行わない。

## 期待worker一覧の正本

期待workerは、Deliveryの3論理責務（Planner、Implementer、Documenter）と独立Reviewerである。TesterとSecurity OperatorはOwner例外承認がある場合だけ期待対象へ加える。役割名は `worker-definitions/` のファイル名および見出し、指定モデルと推論レベルは `rules/worker-task-settings.md` を正本とする。本ルールはSubagent ID、親session、状態、モデル、推論、結果資料を読み取り照合する手順だけを定義し、資料から状態変更を行わない。

- 各論理責務のSubagent種別、期待subagentId、親session、projectId、Codex実行ディレクトリ、対象リポジトリ、ブランチは対象 `current-task.md` のWorker RegistryとTask Contextから取得する。Registryに必要な値がない場合は未確認として停止し、推測で補完しない。
- Ownerチャットと初期化用スレッドは期待workerに含めない。補助情報として一覧に記録する場合も、workerの不足数や重複数には算入しない。
- `current-task.md`の論理責務、Subagent種別、subagentId、親sessionと実測結果が一致しない場合は、接続ツール結果を実測正本としてRegistry・担当resultへ同期し、旧値を履歴注記として保持する。接続ツール結果自体のproject・親session・作業ディレクトリ不一致、または作成前のモデル・推論未確定の場合のみ停止する。
- Registry照合では、対象ファイル側thread名・thread IDをworker接続の必須一致条件にしない。対象projectId、対象リポジトリ、実行ディレクトリ、Owner会話、クルー会話は混同しない。Owner会話は期待workerに含めず、クルー候補のprojectId不一致は不足へ置き換えず停止する。

## Subagent照合

次の順序を変えずに、照合範囲と取得失敗を結果へ記録する。

1. 対象projectId、対象スレッド、実行日時、実行者がOwner起点であることを記録する。
2. 対象Owner session配下のDeliveryおよび独立Reviewerについて、Worker RegistryのsubagentId、親session、projectId、役割、状態、指定モデル・推論、担当resultを読み取り照合する。独立チャット、サイドバー表示、別projectの候補は取得・照合対象にしない。
3. DeliveryはPlanner、Implementer、Documenterの論理責務で同じsubagentIdを共有すること、ReviewerはDeliveryとは異なるsubagentIdであることを照合する。例外workerはOwner判断と計画記載がある場合だけ照合対象へ加える。
4. 読み取り結果をRegistryと照合し、取得不能・一部欠落・親session不一致・重複・状態不安定の場合は正常・不足を推測しない。

## 判定基準

一覧掲載、個別読取、`current-task.md`のWorker Registry、Owner確認は別々の確認値である。いずれか一つを他の値の代用にせず、各値を個別に記録する。一致しない場合は不一致または未確認として扱い、正常・不足へ自動変換しない。

Codex APIまたは接続ツールからSubagent ID、親session、状態、モデル・推論、一覧、アーカイブ情報などを取得できない場合は、取得範囲・取得元・失敗理由・未確認範囲を記録し、取得不能だけを理由に正常・不足・不一致を確定しない。既知の不一致、重複、結果資料の欠落、承認不足、対象不一致が確認できた場合は停止条件とする。取得不能になった同一APIを自動再試行せず、Worker Registry、heartbeat-events.md、接続要求・完了・close結果、担当result、task-logなど利用可能な別証跡へ切り替えて照合する。別証跡間に矛盾がなければ継続し、矛盾が確認された場合だけ停止する。

thread単位の状態と、役割単位の判定を分けて記録する。1つの役割に複数の問題がある場合は、役割判定を一つに決めて問題フラグを併記し、理由を省略しない。

### thread単位の状態

- **実行中**：一覧または個別読取で処理中、ターン実行中、または同等の実行状態と確認できる。実行中をアクセス不能・不足とみなさず、アーカイブ・削除・復元・追加を行わない。
- **アーカイブ済み**：アーカイブ一覧で確認できる、または個別状態でアーカイブ済みと確認できる。正常稼働workerの数には算入せず、自動復元・自動追加を行わない。
- **アクセス不能**：個別threadを読めない、hostが利用不能、対象projectへの所属を確認できない、または必要メタデータを取得できない。実行中、不一致、不足と断定しない。
- **確認済み**：個別読取が成功し、アーカイブでなく、実行中ではなく、必要なメタデータを取得できた状態。役割単位の正常判定には追加の照合を必要とする。

### 役割単位の判定

- **正常**：Deliveryまたは独立ReviewerのSubagent ID、親session、projectId、論理責務、指定モデル、推論レベル、結果資料が期待値と一致し、重複でない。
- **不足**：現行一覧、アーカイブ一覧、個別確認を完了したうえで、期待役割に対応する候補が一件もなく、アクセス不能、実行中、アーカイブ済み、不一致の候補を不足と誤認しないことを確認できた状態。確認が不完全な場合は不足と判定しない。
- **不一致**：候補はあるが、Subagent ID、親session、projectId、論理責務、指定モデル、推論レベル、結果資料のいずれかが期待値と異なる状態。自動修正・自動置換・自動追加を行わない。
- **重複**：同一役割に対応する候補が2件以上ある状態。実行中・アーカイブ済み・アクセス不能の候補を含め、重複を解消するまで追加しない。
- **実行中**：期待役割の候補に実行中のthreadがある状態。役割単位の不足やアクセス不能と断定せず、状態が確定するまで変更しない。
- **アーカイブ済み**：期待役割の候補がアーカイブ済みで、正常な現行workerが確認できない状態。自動復元・自動追加を行わず、Ownerへ提示する。
- **アクセス不能**：期待役割の候補を個別確認できず、所属・状態・メタデータを確定できない状態。取得失敗を不足と置き換えず、Ownerへ提示する。
- **対象外候補**：論理責務名が一致しても親session、Subagent ID、または対象projectIdが一致しない候補。対象projectの正常workerや不足workerには算入せず、不一致候補として記録し、追加理由に使用しない。

不一致、重複、アクセス不能、対象外候補がある場合は、それらを除外して不足数を水増ししない。期待Subagent IDまたは論理責務の候補が別projectIdまたは別親sessionにある場合は対象外候補または不一致として扱い、不足workerの追加対象にしない。候補が存在する論理責務の不足判定は、Ownerが照合結果を確認した後も必要情報が揃っていない限り行わない。

## Owner確認と手動整理

- 照合後、Ownerへ論理責務ごとの期待値、実測値、状態、判定、根拠、未確認事項を表で提示する。少なくとも `論理責務`、`期待Subagent ID`、`実測Subagent ID`、`親session`、`projectId`、`指定モデル・推論レベルの正本参照先`、`指定モデル・推論レベルの照合結果`、`Codex実行ディレクトリ`、`結果資料`、`重複`、`判定`、`理由`を含める。正本の固定値を一覧へ複製しない。
- Ownerが一覧内容を確認するまで、workerは状態変更を呼び出さない。Owner確認は、一覧の受領だけでなく、判定と未確認範囲を確認したことが分かる明示的な応答でなければならない。
- アクセス不能Subagentを整理する必要がある場合は、対象Subagent ID、親session、アクセス不能の根拠、closeの影響をOwnerへ提示する。closeはOwner判断後に行い、workerは削除、復元、移動を行わない。
- Ownerの手動アーカイブ完了が確認できない場合、worker追加を含む次の操作へ進まない。手動整理後は、現行一覧・アーカイブ一覧・個別読取を最初から再実行する。

## 不足workerの追加

- 手動整理後の再ヘルスチェックで、なお不足と判定された役割だけを不足一覧としてOwnerへ再提示する。不一致、重複、実行中、アーカイブ済み、アクセス不能を不足役割にすり替えない。
- Ownerが例外workerの接続を明示した場合に限り、再確認済みの例外責務だけを接続する。論理責務、対象projectId、対象リポジトリ、親session、Codex実行ディレクトリ、指定モデル、推論レベル、ブランチなどの必須値が不足している場合は接続せず停止する。
- 追加時は対象projectIdを再確認し、期待役割以外のworker、既存候補の置換、不一致候補の修正、重複解消を目的とした自動作成を行わない。Ownerの指示が「確認」「整理」「再確認」だけの場合は作成しない。
- 追加が一部成功した場合や作成結果を個別確認できない場合は、追加を繰り返さず停止し、成功・失敗・未確認の役割を記録する。
- 追加後は、同じprojectIdを対象に現行一覧、アーカイブ一覧、個別thread読取、全役割の判定を再実行する。追加した役割が正常にならない、別roleとして表示される、重複する、実行中で完了状態を確認できない場合は、再追加せずOwnerへ報告する。

## 禁止事項と停止条件

- Owner確認前のworker・チャットの作成、アーカイブ、削除、復元、移動、置換、状態変更は禁止する。自動アーカイブ、自動削除、自動復元、自動置換、自動不足補完も行わない。
- サイドバー表示や独立チャット一覧に存在しないことを理由に不足と判定しない。親session配下のSubagent情報と結果資料を照合できない場合は未確認として停止する。
- projectId、親session、Subagent ID、対象リポジトリ、Codex実行ディレクトリ、論理責務、指定モデル、推論レベルのいずれかを推測で補完しない。作成前の指定モデル・推論、接続ツール結果のproject・親session・作業ディレクトリが確認できない場合は停止する。接続後のSubagent報告との差分は、接続ツール結果を正本として同期し、報告不整合として記録して継続する。
- ReviewerのSubagent IDはAttempt単位で管理する。再接続によるID変更は正常なAttempt遷移として扱い、過去AttemptのIDを現行不一致としない。現行Attemptの接続ツール結果、`review.md`、Worker Registryだけを照合し、工程状態は別の状態欄で判定する。ファイル側thread名・thread IDは補助記録とし、照合不一致だけで停止しない。
- 一覧取得、アーカイブ一覧取得、個別thread読取、Owner確認、手動整理後の再確認、追加後の再確認のいずれかが失敗・部分成功・タイムアウトした場合、失敗範囲を記録して次の状態変更へ進まない。
- 同じ検証に2回連続で失敗した場合、状態が実行中から変化しない場合、結果が取得ごとに変わり再現できない場合は、理由・影響・未確認事項を記録して停止する。
- 外部サービス、本番環境、アプリケーションの稼働監視、秘密情報・個人情報・顧客固有情報を使う確認は本ルールの対象外とし、必要になった場合はOwnerへ判断を求める。

## 結果記録

ヘルスチェックの実行単位ごとの結果は、対象スレッドの `threads/<thread-name>/docs/health-check.md` に記録する。これは人間向け状態確認の正本であり、Planner、Implementer、Tester、Security Operator、Reviewer、Documenterのworker結果とは別の資料である。旧`result/health-check.md`は変更せず、参照する場合は旧資料であることを明記する。

前回health-checkと現在のworker一覧を比較する場合は、`rules/worker-connection-diff.md` を参照する。比較可能な2回目以降だけ `threads/<thread-name>/result/worker-connection-diff.md` を別管理で作成し、health-check本文を複製しない。比較不能・取得失敗・状態変化時は差分を確定しない。

履歴退避、manifest、正本結果、旧result保全は `rules/thread-operation.md` に従う。health-check固有の判定基準と結果項目は本ルールで定義し、履歴保存手順を再定義しない。

新規taskの開始・切替に伴うヘルスチェックでは、同一projectのactive taskが1件であること、現行taskの扱いにOwner回答があること、退避後のmanifest・`history/index.md`・退避docs/resultの照合が完了していることを確認する。いずれかが未確認、部分成功、不一致の場合は不足workerや正常状態を推測せず、task切替とactive状態変更を停止する。

補足証跡には、次の項目を`task-progress.md`またはworkerの`result/`へ記録する。これらを`health-check.md`へ重複記載しない。

- 実行日時、起動トリガー（Owner明示またはworker接続失敗）、対象スレッド、対象projectId、対象リポジトリ、Codex実行ディレクトリ
- 期待worker一覧の正本、指定モデル・推論レベルの正本参照先、current-taskから取得したSubagent ID・親session
- 現行一覧とアーカイブ一覧の取得範囲、ページ数、個別thread読取の成否、取得失敗・未確認範囲
- Ownerへ提示した表と、役割ごとの理由・重複・不一致
- Owner確認の日時と内容、手動アーカイブの対象・実施者・確認結果、追加指示の有無
- 追加した役割、追加結果、追加後の全件再確認結果、未解決事項、停止理由

Subagent ID、親session、projectIdは照合に必要な識別子として記録する。認証情報、アクセストークン、個人情報、顧客固有情報、生の外部入力、不要な会話本文は記録しない。未確認の値は空欄や正常扱いにせず `未確認` と明記する。
