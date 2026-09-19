---
scope: all-workers
status: active
---

# Codex実行バックエンド

## 目的

`rules/execution-portability.md`の実行バックエンド能力契約を、現行Codex環境で実現する操作、証跡、停止条件を定義する。これはCodex固有の実装規約であり、共通ワークフローの役割・状態・判定基準を置き換えない。

## 適用と有効化

- `rules/execution-portability.md`に従い、対象taskの明示指定または完全一致するローカルルールによってCodexが選択された場合だけ適用する。選択済みのCodexを計画承認時に再指定・再確認してはならない。
- Workflow Coordinatorは、対象taskを管理するOrchestrator threadとする。worker実行単位は同threadに接続したDeliveryまたはReviewer Subagentとする。
- Coordinator thread、対象task、実行コンテキスト、結果資料の保存先、Owner承認済み範囲を照合できない場合は、バックエンドを有効化せずOwner判断を求める。
- Ownerが計画を承認した時点で、CoordinatorはCodexバックエンドを有効化し、承認済み範囲の最初の未接続工程を開始する。自律サイクル開始、Coordinator再開、またはheartbeat有効化の別途指示を待ってはならない。
- 本規約の追加だけでactive taskのない既存Orchestrator、Subagent、heartbeatを作成・再開・有効化してはならない。

## 能力対応表

| 共通能力 | Codexでの実現 | 正本となる証跡 | 利用不能時 |
| --- | --- | --- | --- |
| 接続 | Coordinatorが承認済みの論理責務を担うSubagentへ接続またはresumeする | 接続ツール結果、Worker Registry、`task-log.md` | 既存実行単位の継続可否を確認し、代替作成はOwner承認なしに行わない |
| 再開 | 同一taskのclose済みDeliveryをresumeし、次の論理責務を明示する | resume結果、担当result、`task-log.md` | resume不能を記録し、Owner本体による継続またはOwner判断へ分岐する |
| 観測 | 接続ツール結果、Codex API観測値、Worker Registry、結果資料を照合する | 取得範囲と代替証跡を含む`task-log.md` | API観測不能は`未確認`。既知の矛盾がなければ通常遷移を止めない |
| 完了証跡 | Subagentの完了報告、担当result、完了・close結果を照合する | 担当result、close結果、`task-log.md` | 結果資料または完了根拠が欠ける場合は停止する |
| handoff event通知 | Coordinatorが完了証跡の確定直後にイベントを記録・claimし、次工程を接続する | `result/handoff-events.md`、接続結果、`task-log.md` | 通知不能時だけ復旧監査の対象にする |
| 復旧監査 | Codex heartbeatが未処理・残留eventだけを照合し、必要時だけCoordinatorを再開する | heartbeat設定、`handoff-events.md`、`task-progress.md` | heartbeatを使えない場合は、Coordinatorの明示再開時に同じ台帳を照合する |

## 通常経路：結果イベントによる次工程接続

Codexバックエンドの通常経路はheartbeatではない。CoordinatorはSubagentの完了通知または結果資料確定を受けたら、同じ処理単位で次を順に実施する。

Codex外の常駐ディスパッチャー、独自スクリプト、追加の言語ランタイムは使用しない。Coordinatorは承認後に継続するCodex task内でSubagentの完了を待機し、完了通知または結果資料確定を受けた同じ実行内でeventを処理する。Coordinatorが中断された場合だけ、Codex heartbeatを復旧監査として使用する。

1. 接続結果、担当result、`task-log.md`、Worker Registry、必要なOwner判断を照合する。
2. task-id、発生元責務、工程、結果資料、内容フィンガープリントからhandoff event IDを確定し、共通契約に従って`handoff-events.md`へ`queued`で記録する。
3. 同一event IDが既に存在する場合は新規接続を行わない。同一工程境界に異なる結果フィンガープリントがある場合は`blocked`として記録し、Ownerへ提示する。矛盾がないeventだけをclaimし、重複・並行処理を行わない。
4. 承認済み範囲と次工程の入力ゲートを満たす場合、追加の開始指示やheartbeat実行を待たず、次のSubagent接続またはresumeを行う。
5. 接続結果と処理結果を記録してeventを`completed`へ更新する。Owner判断、停止、または結果欠落の場合は`blocked`と再開条件を記録し、次工程を接続しない。状態遷移と中断時の照合は`rules/handoff-event-contract.md`に従う。

```text
Subagent完了
  → result / task-log / Registry照合
  → handoff eventをqueuedで記録
  → Coordinatorがprocessingとしてclaim
  → 次工程を接続またはresume
  → 結果記録後にcompleted
```

Coordinatorが結果を受けた後に「heartbeatの次回実行待ち」を理由として次工程接続を延期してはならない。

## Codex heartbeatの役割

Codex heartbeatは、このバックエンド固有の復旧監査起動機構である。通常の接続・resume・Owner判断・close・完了確定の実行者ではない。

- 監査対象は、完了証跡があるのにeventがない場合、`processing`の残留、Coordinator停止後の未処理eventに限る。
- 未処理または残留がなければ、Coordinatorを起動せず無応答で終了する。
- 復旧対象がある場合だけCoordinatorを再開し、通常経路と同じ照合・claim・接続手順へ戻す。
- 同一Coordinatorに対するheartbeat設定は1件だけとし、taskごとの新規作成・複製・削除で回避しない。設定名、対象Coordinator、状態、監視対象、比較結果、最後のエスカレーション、終了理由は`task-progress.md`または`result/heartbeat-events.md`へ記録する。
- Codexでの進捗根拠は、担当resultの内容フィンガープリント、Worker Registryの状態・確認証跡、handoff eventのclaim・処理証跡とする。通常の長時間作業は進捗根拠が更新されている限り継続する。これらが同じevent・同じ原因について連続2回の復旧監査で更新されない場合は、Coordinatorを反復再開せず、eventを`blocked`、heartbeatを`PAUSED`へ更新して6項目エスカレーションを記録する。
- `current-task.md`には、Codexの進捗監査方針として前項の進捗根拠・連続2回の無進捗基準・`blocked`/`PAUSED`時の再開条件を、修正・再レビュー予算として「初回Reviewer判定後は最大2ラウンド、到達時は集約エスカレーション」を記録する。Ownerが追加ラウンドを承認した場合だけ、当該taskの記録を更新して再開する。
- 計画承認前、Documenter記録完了後、active taskなし、または同一原因の再試行が2回に達した後は`PAUSED`とする。
- 同一原因の再試行が2回に達した場合、heartbeatを直ちに`PAUSED`へ更新し、6項目エスカレーションを記録する。Ownerが再開を承認するまで`ACTIVE`へ戻さない。
- heartbeatが利用不能でも、結果資料とevent台帳の整合が取れる限り通常経路を停止しない。復旧監査が必要な状態だけをOwner判断へエスカレーションする。

## Codex固有値の扱い

Subagent ID、親session、Codex project、実測モデル・推論、一覧、アーカイブ情報はCodexバックエンドの補助証跡である。指定値は接続要求の入力として記録し、実測値を取得できた場合だけ照合する。APIからの取得不能は未確認として取得範囲・代替証跡を記録するが、それだけを停止条件にしない。

既知のID不一致、重複、project・親session・実行ディレクトリの不一致、結果資料欠落、承認不足、または代替証跡間の矛盾は停止条件とする。

## Codexバックエンドの受入条件

次を確認できる場合に、Codexバックエンドが当該taskで有効と判定する。

- Coordinator、Subagent、結果資料、event台帳、heartbeat設定の役割と記録先が一意である
- Subagent完了から次工程接続までの通常経路が、heartbeatを待たずに記録される
- eventの台帳、`queued`、`processing`、`completed`、`blocked`とclaim責任を照合できる
- heartbeatが未処理・残留eventの復旧監査に限定され、通常経路を遅延させない
- API観測不能と既知の矛盾・必須証跡欠落を区別している
- 無進捗が連続2回確認された場合にevent `blocked`、heartbeat `PAUSED`、集約エスカレーションとなる
- 初回Reviewer判定後の修正・再レビューが最大2ラウンドで集約・停止され、Owner追加承認なしに無限再接続しない
- 再試行2回到達時のheartbeat `PAUSED`とOwner再開承認の条件を確認できる
