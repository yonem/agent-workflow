---
scope: all-workers
status: active
---

# 自動化操作・受入要求ルール

## TASK-033標準工程の参照境界

標準工程はDelivery（Planner→Implementer→Documenter）と独立Reviewerを基本とし、同時稼働は1件とする。Tester・Security OperatorはOwner承認済み計画に明記された例外の場合だけ接続する。用語は`rules/glossary.md`、固有責務は`worker-definitions/`、完了境界は`rules/development-improvement-record.md`を正本として参照する。

## 目的

承認済み計画の対象機能は、タスク対応中に直ちに有効化し、同じタスク内で検証する。計画承認完了を自動発動契機とし、人間の別途開始指示を待たない。Agentは条件達成を検知して承認要求を起動するが、Reviewer受入・Owner承認・不可逆操作を自動確定しない。

## 適用境界

- 記録同期、ゲート照合、エスカレーション、リトライ制御、外部接続、worker作成に適用する。
- 外部接続の承認・検証は既存`integration-ledger`を唯一の正本とする。新しい接続台帳を作成しない。
- 削除、上書き、完了確定、承認の代行は自動化しない。

## OwnerとOrchestratorの分離

- `Owner`は人間が直接指示・承認を行うチャット／役割である。
- `Orchestrator`は、従来`Owner Agent`と呼んでいたタスク進行担当のチャット／Agentである。
- Orchestratorはクルー接続、result照合、工程遷移、IRリマインド、停止、承認要求を担当する。
- 人間への承認要求・エスカレーションはOrchestratorチャット上で直接提示する。
- OwnerとOrchestratorの間に自動接続、転送、専用heartbeatを設定しない。

## 計画承認後の即時有効化

Ownerが計画を承認した時点で、承認範囲内の対象機能を「対応中・有効」として扱い、Agentは実装、設定、記録同期、検証の手順を直ちに開始する。別の開始指示、実運用効果測定、次タスク開始を発動条件にしてはならない。

対象機能を有効化できない場合は、実行基盤の不存在を理由に放置せず、承認済み計画に定義した媒体中立の手順へ切り替える。手順自体が定義されていない場合は、計画外変更として停止し、不足、影響、実行可能な発動条件・入力・出力・証跡・停止条件を記録してOwnerへ再承認または新規taskを求める。

## OrchestratorによるSubagentオーケストレーション

Ownerが計画を承認し、次工程の実施を指示した場合、Orchestratorは承認済み計画に従って実Subagentを接続し、結果を照合して次工程へ渡す。オーケストレーションを任意扱いにしたり、親チャットの作業・ファイル側thread・ユーザー向けCodex会話で代替したりしてはならない。接続、完了通知、Registry更新、結果照合の各事実は同一taskの`task-log.md`へ直ちに記録する。

計画承認後からDocumenterの記録完了までは、Orchestratorが自律的に工程を継続する。人間へ処理を返す状態は、(1)承認済み範囲では解決できない続行不能事項についてOwner判断を求める状態、または(2)記録完了後にOwnerへ作業完了確認を求める状態だけとする。「次工程待ち」「Reviewer待ち」「再接続待ち」を理由に人間へ続行指示を求めてはならず、必要なSubagent接続、再接続、結果照合、修正サイクルをこのtask内で実行する。各工程の完了報告は次工程接続の入力であり、Ownerへの追加承認要求ではない。

Subagent自身の報告にある「接続機能なし」「実行不可」等の環境主張は、親Orchestratorの接続ツール結果を覆さない。親Orchestratorは、接続ツールが返したSubagent ID、status、submission、close結果を接続事実の正本とする。Subagent報告と接続ツール結果が食い違う場合は「報告不整合」としてtask-logへ記録し、接続ツール結果をWorker Registryと担当resultへ同期して継続する。報告不整合だけでは停止しない。接続ツール結果自体の欠落、project・親session・作業ディレクトリの不一致、または作成前に指定・確定したモデル／推論設定の欠落だけを停止条件とする。

Orchestratorは計画承認後、Delivery Subagentを自身のsessionに属するSubagentとしてresumeし、Implementer責務を接続する。Delivery完了後はcloseして独立Reviewerを接続し、Reviewer受入後はDeliveryをresumeしてDocumenter責務を接続する。Subagentの完了通知を受けた時だけ、担当result、`task-log.md`、Owner判断、入力ゲートを照合する。人間のOwnerとOrchestratorの間に自動接続は作らず、承認要求はOrchestratorチャット上で人間へ提示する。

ここでいうSubagentは実際に接続・再開されたworker実行単位であり、親チャット、ユーザー向けCodexスレッド／会話、ファイル側thread、Codex projectでは代替できない。Deliveryは同一task内でPlanner→Implementer→Documenterの論理責務を担当し、必要な切替は同じDelivery Subagentの明示resumeで行う。Reviewerは独立した実Subagentとして接続する。Planner、Implementer、Reviewer、Documenterはいずれも`rules/worker-task-settings.md`の`gpt-5.6-luna` / `low`を接続前に指定・照合する。指定設定が確認できない場合は接続・作業を停止する。指定済みSubagentの実測値が取得不能な場合は未確認として証跡を記録するが、それだけを理由に自律オーケストレーション、受入、記録完了を停止しない。新規ユーザー向けCodexスレッド／会話の作成をSubagent接続の代替手段にしてはならない。

## 最小Subagent運用

### 接続後報告不整合の継続基準

- Subagent作成前に、モデル・推論設定を指定・確定し、接続ツールへ渡す。作成前に確定できない場合は接続しない。
- 接続後にSubagent報告のID、状態、結果パスが接続ツール結果と異なる場合、接続ツール結果を実測正本としてWorker Registry、担当result、task-logへ同期する。
- 上記の報告差分だけでは停止しない。接続ツール結果自体の欠落、project・親session・実行ディレクトリの不一致、作成前モデル・推論の未確定だけを停止条件とする。
- Reviewerの実Subagent IDはレビューAttempt単位の値とする。再接続で新しいIDが発行されることは正常であり、過去AttemptのIDを現行資料の不一致として扱わない。現行AttemptのIDだけを接続ツール結果・現行`review.md`・Worker Registryへ同期し、過去AttemptのIDは履歴として保持する。
- タスク工程状態（次worker、Documenter接続可否、受入状態）はReviewer IDと別に管理する。ID変更だけでは工程状態を未同期・不受入と判定しない。

workerの役割は責務の区分であり、常設Subagentや事前作成済みの役割枠を意味しない。Orchestratorは、承認済み計画の次工程に必要な役割だけをjust-in-timeで接続する。

- 同時に開いてよいSubagentは1件だけとする。TesterとSecurity Operatorを含め、並行接続を行わない。
- Worker Registryの役割行は接続許可または在庫を示さない。未接続役割は`準備中`、計画上不要な役割は`対象外`として記録し、Subagentを作成しない。
- 標準構成はDelivery Subagent 1件と独立Reviewer Subagent 1件だけとする。Delivery Subagentは同一task内でPlanner、Implementer、Documenterの論理責務を順番に担当し、役割切替時はclose済みの同一Subagentをresumeする。
- TesterとSecurity Operatorは独立Subagentにしない。承認済み計画に外部接続、高リスク変更、または独立検証の必要性が明記され、Ownerが例外を承認した場合だけ、必要な1件を順番に接続する。
- 各Subagentは担当resultとtask-logの記録を完了し、Ownerが次工程の入力照合を終えた直後にcloseする。完了済みSubagentを待機・常設・次task用の予備として保持しない。
- Deliveryは同一task内でPlanner、Implementer、Documenterの論理責務を順次担当するため、Owner承認後のImplementerおよびReviewer受入後のDocumenterへの切替は、閉じた同一Delivery Subagentをresumeして行える。別task、未確認の状態、または例外条件の不足を理由に新規Subagentを追加しない。
- Subagentの作成、resume、closeは、役割、task-id、理由、結果資料、前後のopen件数をtask-logへ記録する。1件上限を照合できない場合は接続せず停止する。
- Subagentの恒久構成は、Delivery 1件と独立Reviewer 1件を上限とする。既存Subagentのresumeまたは親sessionへのsend_inputが明示的に不可能と検証された場合でも、Ownerの個別承認なしに代替Subagentを自動作成してはならない。作成拒否・接続不能時はOwner本体が作業を継続し、必要なら停止理由を記録する。
- 現行Subagent一覧は運用・監査履歴として保持し、一覧を減らすための削除操作を前提にしない。完了済みSubagentはclose状態として記録し、履歴上の存在と現在のopen件数を分けて扱う。

Subagentの完了、担当resultの更新、または未回答Owner判断を検出するため、Orchestrator threadにはheartbeat起動機構を1件だけ設定する。heartbeatは計画承認前のim・Planner工程では`PAUSED`にする。Ownerが計画を承認した後、Implementerへ接続する直前に既存heartbeatを`ACTIVE`へ更新する。監視対象はImplementer接続からReviewer受入、Documenter記録完了までとし、Documenter記録完了を確認した時点で既存heartbeatを`PAUSED`へ更新する。Owner完了判断、履歴退避、hisi更新、docs/result初期化はheartbeat停止後の別工程とする。heartbeatをタスクごとに削除・新規作成・複製してはならない。heartbeatは前回確認結果と現行状態を比較し、新規の完了・差分・判断待ち・不一致がない場合、Orchestratorの処理を無応答で終了する。要対応事項がある場合だけOrchestratorを再開し、結果を照合して承認済み範囲の次工程へ接続するか、人間へ指定形式のOwner判断を求める。次工程接続、結果記録、またはOwner判断の提示が完了し、新しいエスカレーションがなければ、そのOrchestrator実行を終了する。heartbeatは不可逆操作、承認代行、サイドバーworker・新規チャットの作成を行わない。

heartbeatの設定名、対象Orchestrator thread、状態、監視対象、前回結果、比較結果、最後に処理したエスカレーション、Orchestrator実行の終了理由は、読み取り専用health-checkと現行taskの記録へ残す。`PAUSED`は計画承認前、Documenter記録完了後、active taskなしの待機状態、または同一原因の再試行が最大2回に達した後の停止状態で使用する。計画承認後かつDocumenter記録完了前の欠落・停止・対象不一致・重複は自動接続を有効と扱わず、Orchestratorが人間へ復旧判断を求める。同一原因の再試行が最大2回に達した場合は、自動継続を停止し、heartbeat設定を直ちに`PAUSED`へ更新したうえで、6項目エスカレーションを記録する。heartbeatの重複作成は禁止し、既存設定を状態更新して維持する。

## heartbeat輻輳防止ガード

heartbeatは設定の一意性だけでなく、エスカレーションイベントごとの排他制御を行う。正本は現行taskの`result/heartbeat-events.md`とし、Orchestratorだけが更新する。

1. Orchestratorは処理前に、task-id、発生源（Subagent IDまたは資料）、結果資料、内容フィンガープリントから再実行安全なイベントIDを確定する。日時だけをイベントIDに使用しない。
2. 同一イベントIDが`processing`または`completed`なら、資料更新、通知、worker接続、OJ提示を行わず終了する。
3. 別イベントが`processing`なら、後続イベントを到着順に`queued`として記録し、並行処理・通知・worker接続を行わず終了する。queued化だけを理由に人間判断を求めてはならない。
4. `processing`イベントを`completed`へ更新した後、最も早い`queued`イベントを1件だけ`processing`としてclaimし、自動処理を再開する。
5. 未処理イベントだけを`processing`としてclaimしてから結果照合・次工程接続を行う。処理終了後に`completed`へ更新する。
6. イベントID、台帳、状態、claimのいずれかを照合できない場合は、推測で再実行せず停止する。競合検出時にheartbeatを再作成・複製・停止して回避してはならない。

`heartbeat-events.md`は少なくともイベントID、task-id、発生源、結果資料、内容フィンガープリント、状態、claim責任、処理結果、再開条件を保持する。状態は`queued`、`processing`、`completed`を使用する。`processing`が残る場合は同一イベントを再実行せず、queuedイベントも進めない。台帳不整合または復旧不能だけをOwner判断の停止理由とする。

工程遷移は、Planner→Implementer→必要なTester / Security Operator→Reviewer→Documenter→OrchestratorのIRリマインド・完了判断要求とする。人間はOrchestratorへ指示し、個別Subagentへの直接指示・既存サイドバーチャット間の任意送信を行わない。次工程の接続前に、直前workerの実Subagent ID、完了状態、結果ファイル、Owner判断、Registry更新を照合し、いずれかが欠けた場合は欠落を記録して次工程へ進めない。停止・再開の扱いはOwner判断または記録完了の工程境界でのみ確定する。

Subagentの完了通知が取得不能な場合は、Orchestratorの次回実行時に現行resultと`task-log.md`を照合して未接続工程を復旧する。独立チャットへの送信拒否、通知本文の再送失敗、サイドバー表示の有無だけでは工程を停止しない。結果資料の欠落、不一致、停止判定、Owner判断残件は停止条件とする。

## 自動受入要求の発動条件

Agentは次の条件をすべて確認した時点で、Reviewer受入要求を自動起動する。

1. 対象task、project、thread、repository、branchが一致している。
2. 承認済みplanとOwner判断の範囲内である。
3. 実行対象、入力、権限、依存、重複の確認が完了している。
4. 実行結果、監査証跡、停止条件、復旧方法が記録されている。
5. 未確認、不一致、計画外変更、失敗、復旧不能が残っていない。

条件未達の場合は受入要求を起動せず、6項目（期待値、実際値、根拠、影響、停止理由、再開条件）で停止・エスカレーションする。

## 方式切替

1. 計画承認完了を検知し、承認済み範囲の機能を有効化して承認付き実行を開始する。
2. Agentが受入条件の達成を検知し、Reviewer受入要求を起動する。
3. Reviewer受入後、AgentがOwner承認要求を起動する。
4. Owner承認後に限定自動実行へ切り替える。

承認要求の起動から一定時間応答がない場合も、Agentは自動切替せず、未承認として停止状態を維持し、再通知またはエスカレーションを記録する。

再通知・承認要求のリトライは最大2回までとする。2回に達しても応答がない場合、または同一原因の失敗が続く場合は、自動継続せず、heartbeat設定を直ちに`PAUSED`へ更新してから6項目エスカレーションへ移行する。復旧条件を満たしOwnerが再開を承認するまで、heartbeatを`ACTIVE`へ戻してはならない。

## 外部接続・worker作成

- 外部接続は`integration-ledger`の承認済み・有効・期限内・対象一致を確認してから実行要求を起動する。
- worker作成はproject、role、権限、重複、実行環境、失敗時停止を確認してから実行要求を起動する。
- 接続または作成に失敗した場合、同一原因の自動リトライを繰り返さず停止し、6項目で報告する。

## 監査証跡

各自動化イベントへ、task-id、実行者、対象、入力、承認状態、開始・終了時刻、結果、失敗、停止、復旧、次の承認要求、証跡パスを記録する。Agentの「要求起動」とReviewer・Ownerの「受入・承認」を別イベントとして扱う。

### 記録同期・リトライ監査項目

記録同期とゲート照合では、正本、参照先、更新責任、更新境界、競合の確認結果を記録する。再通知・承認要求・同一原因の再試行は原因単位で試行番号（初回、1回目、2回目）、対象、時刻、結果を記録し、2回目で停止する。原因不明、権限逸脱、対象不一致、復旧不能は試行せず直ちに停止する。

Agentの受入要求起動、Reviewer受入、Owner承認、限定自動実行は別イベントとして記録し、前イベントの成功を次イベントの発動条件とする。承認・受入の未応答は成功とみなさない。

## 再開条件

停止後は、原因、影響、修正、再発防止、承認範囲、復旧方法を確認し、Reviewer再確認とOwner承認が揃うまで自動実行を再開しない。
