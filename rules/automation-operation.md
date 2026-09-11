---
scope: all-workers
status: active
---

# 自動化操作・受入要求ルール

## 目的

承認済み計画の対象機能は、タスク対応中に直ちに有効化し、同じタスク内で検証する。計画承認完了を自動発動契機とし、人間の別途開始指示を待たない。Agentは条件達成を検知して承認要求を起動するが、Reviewer受入・Owner承認・不可逆操作を自動確定しない。

## 適用境界

- 記録同期、ゲート照合、エスカレーション、リトライ制御、外部接続、worker作成に適用する。
- 外部接続の承認・検証は既存`integration-ledger`を唯一の正本とする。新しい接続台帳を作成しない。
- 削除、上書き、完了確定、承認の代行は自動化しない。

## 計画承認後の即時有効化

Ownerが計画を承認した時点で、承認範囲内の対象機能を「対応中・有効」として扱い、Agentは実装、設定、記録同期、検証の手順を直ちに開始する。別の開始指示、実運用効果測定、次タスク開始を発動条件にしてはならない。

対象機能を有効化できない場合は、実行基盤の不存在を理由に放置せず、承認済み計画に定義した媒体中立の手順へ切り替える。手順自体が定義されていない場合は、その不足を同一タスク内で是正し、実行可能な発動条件・入力・出力・証跡・停止条件を記録する。

## Owner AgentによるSubagentオーケストレーション

Owner Agentは計画承認後、Delivery Subagentを自身のsessionに属するSubagentとしてresumeし、Implementer責務を接続する。Delivery完了後はcloseして独立Reviewerを接続し、Reviewer受入後はDeliveryをresumeしてDocumenter責務を接続する。Subagentの完了通知を受けた時だけ、担当result、`task-log.md`、Owner判断、入力ゲートを照合する。Worker実行中にOwner Agentが常時待機・定期ポーリングすることは要求しない。

## 最小Subagent運用

workerの役割は責務の区分であり、常設Subagentや事前作成済みの役割枠を意味しない。Owner Agentは、承認済み計画の次工程に必要な役割だけをjust-in-timeで接続する。

- 同時に開いてよいSubagentは1件だけとする。TesterとSecurity Operatorを含め、並行接続を行わない。
- Worker Registryの役割行は接続許可または在庫を示さない。未接続役割は`準備中`、計画上不要な役割は`対象外`として記録し、Subagentを作成しない。
- 標準構成はDelivery Subagent 1件と独立Reviewer Subagent 1件だけとする。Delivery Subagentは同一task内でPlanner、Implementer、Documenterの論理責務を順番に担当し、役割切替時はclose済みの同一Subagentをresumeする。
- TesterとSecurity Operatorは独立Subagentにしない。承認済み計画に外部接続、高リスク変更、または独立検証の必要性が明記され、Ownerが例外を承認した場合だけ、必要な1件を順番に接続する。
- 各Subagentは担当resultとtask-logの記録を完了し、Ownerが次工程の入力照合を終えた直後にcloseする。完了済みSubagentを待機・常設・次task用の予備として保持しない。
- Deliveryは同一task内でPlanner、Implementer、Documenterの論理責務を順次担当するため、Owner承認後のImplementerおよびReviewer受入後のDocumenterへの切替は、閉じた同一Delivery Subagentをresumeして行える。別task、未確認の状態、または例外条件の不足を理由に新規Subagentを追加しない。
- Subagentの作成、resume、closeは、役割、task-id、理由、結果資料、前後のopen件数をtask-logへ記録する。1件上限を照合できない場合は接続せず停止する。
- Subagentの恒久構成は、Delivery 1件と独立Reviewer 1件を上限とする。既存Subagentのresumeまたは親sessionへのsend_inputが明示的に不可能と検証された場合でも、Ownerの個別承認なしに代替Subagentを自動作成してはならない。作成拒否・接続不能時はOwner本体が作業を継続し、必要なら停止理由を記録する。
- 現行Subagent一覧は運用・監査履歴として保持し、一覧を減らすための削除操作を前提にしない。完了済みSubagentはclose状態として記録し、履歴上の存在と現在のopen件数を分けて扱う。

Subagentの完了、担当resultの更新、または未回答Owner判断を検出するため、Owner threadにはheartbeat起動機構を1件だけ設定する。heartbeatはactive taskが存在する間だけ`ACTIVE`にし、active taskがない初期状態では`PAUSED`にする。Ownerが明示的なim開始を受けた時は、要件定義・worker接続より先に既存heartbeatを`ACTIVE`へ更新する。Documenter記録後にOwnerが完了を承認し、履歴退避・hisi更新・docs/result初期化が完了した時は、既存heartbeatを`PAUSED`へ更新する。heartbeatをタスクごとに削除・新規作成・複製してはならない。heartbeatは前回確認結果と現行状態を比較し、新規の完了・差分・判断待ち・不一致がない場合、Ownerの処理を無応答で終了する。要対応事項がある場合だけOwner Agentを再開し、結果を照合して承認済み範囲の次工程へ接続するか、人間へ指定形式のOwner判断を求める。次工程接続、結果記録、またはOwner判断の提示が完了し、新しいエスカレーションがなければ、そのOwner実行を終了する。heartbeatは不可逆操作、承認代行、サイドバーworker・新規チャットの作成を行わない。

heartbeatの設定名、対象Owner thread、状態、監視対象、前回結果、比較結果、最後に処理したエスカレーション、Owner実行の終了理由は、読み取り専用health-checkと現行taskの記録へ残す。`PAUSED`はactive taskなしの待機状態でのみ許可し、active task中の欠落・停止・対象不一致・重複は自動接続を有効と扱わず、Owner Agentが人間へ復旧判断を求める。heartbeatの重複作成は禁止し、既存設定を状態更新して維持する。

## heartbeat輻輳防止ガード

heartbeatは設定の一意性だけでなく、エスカレーションイベントごとの排他制御を行う。正本は現行taskの`result/heartbeat-events.md`とし、Owner Agentだけが更新する。

1. Ownerは処理前に、task-id、発生源（Subagent IDまたは資料）、結果資料、内容フィンガープリントから再実行安全なイベントIDを確定する。日時だけをイベントIDに使用しない。
2. 同一イベントIDが`processing`または`completed`なら、資料更新、通知、worker接続、OJ提示を行わず終了する。
3. 別イベントが`processing`なら、後続イベントを到着順に`queued`として記録し、並行処理・通知・worker接続を行わず終了する。queued化だけを理由に人間判断を求めてはならない。
4. `processing`イベントを`completed`へ更新した後、最も早い`queued`イベントを1件だけ`processing`としてclaimし、自動処理を再開する。
5. 未処理イベントだけを`processing`としてclaimしてから結果照合・次工程接続を行う。処理終了後に`completed`へ更新する。
6. イベントID、台帳、状態、claimのいずれかを照合できない場合は、推測で再実行せず停止する。競合検出時にheartbeatを再作成・複製・停止して回避してはならない。

`heartbeat-events.md`は少なくともイベントID、task-id、発生源、結果資料、内容フィンガープリント、状態、claim責任、処理結果、再開条件を保持する。状態は`queued`、`processing`、`completed`を使用する。`processing`が残る場合は同一イベントを再実行せず、queuedイベントも進めない。台帳不整合または復旧不能だけをOwner判断の停止理由とする。

工程遷移は、Planner→Implementer→必要なTester / Security Operator→Reviewer→Documenter→Owner AgentのIRリマインド・完了判断要求とする。人間はOwner Agentだけへ指示し、個別Subagentへの直接指示・既存サイドバーチャット間の任意送信を行わない。

Subagentの完了通知が取得不能な場合は、Owner Agentの次回実行時に現行resultと`task-log.md`を照合して未接続工程を復旧する。独立チャットへの送信拒否、通知本文の再送失敗、サイドバー表示の有無だけでは工程を停止しない。結果資料の欠落、不一致、停止判定、Owner判断残件は停止条件とする。

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

再通知・承認要求のリトライは最大2回までとする。2回に達しても応答がない場合、または同一原因の失敗が続く場合は、自動継続せず6項目エスカレーションへ移行する。

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
