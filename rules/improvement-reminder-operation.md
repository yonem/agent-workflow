---
scope: all-workers
status: active
---

# 改善リマインド運用ルール

## ファイルライフサイクル

- `improvement-reminders.md`が存在しない場合、Workflow Coordinatorが受領したIR追加指示、またはクローズ後の初期化直後に未完了IRを継承する場合だけ新規作成する。
- 新規作成時は`## サマリー`と`## 詳細`を作成し、現行ファイル内で未使用のIR-IDを採番する。既存ファイルがない状態で最初に登録する場合だけIR-001とする。
- IRが0件になった場合は、Workflow Coordinatorが`improvement-reminders.md`を削除する。
- ファイル不存在は異常ではなく、IRが0件の状態として扱う。IR追加指示なしに自動作成しない。

## 正本と責任

人間の`Owner`とタスク進行を担う`Workflow Coordinator`を分離する。IRの追加・更新・削除・リマインド判断はWorkflow Coordinatorが行い、判断が必要な場合は人間のOwnerへ直接提示する。OwnerとWorkflow Coordinator間の自動接続は行わない。

- `threads/<thread-name>/docs/improvement-reminders.md`をIR（`improvement-reminders`）の正本とする。
- IR正本は`## サマリー`と`## 詳細`で構成し、サマリーは一覧、詳細はIRごとの根拠・状態・継承・次回条件を記録する。
- サマリーの列順は`IR-ID`、`状態`、`内容`、`継承理由`で固定する。列の追加、削除、並べ替え、別表への置換は禁止する。task-idは対応未定のIRへ付与しないため、サマリーには記載しない。
- サマリーと詳細のIR-IDは数値部分の昇順で並べる。追加・状態変更・継承・除外の更新後も両方を再ソートする。
- サマリーと詳細のIR-ID集合が一致しない、重複する、IDを比較できない場合はIRの更新を完了扱いにせず、Workflow Coordinatorが修正する。task本体は停止しない。
- `improvement-reminders.md`本文には運用ルール、責任分担、停止条件、変更履歴を記載せず、サマリーと詳細だけを記録する。運用ルールは本共通rulesを正本とする。
- IRの追加、更新、状態変更、継承、除外、リマインド判断はWorkflow Coordinatorが担当する。
- Workflow Coordinatorは人間から指示を受ける起点であり、人間と同一視しない。
- DocumenterはIRを管理・編集・照合しない。通常のtask記録だけを担当する。

## IR管理

- IR-IDはWorkflow Coordinatorが連番で付与し、追加前に重複・意味衝突・対象不明を確認する。
- 既存IRの変更は上書きせず、変更理由と変更履歴を残す。
- 対応済みIRは、対応したtaskの現行IRへ状態を残したまま退避対象に含める。クローズ後の初期化直後または次taskのIRを初期化する時点で対応済みIRだけを一覧・詳細から削除し、未完了IRだけを引き継ぐ。IRだけの独立履歴は作成しない。
- クローズ後の初期化直後または次taskの初期化時に引き継ぐ未完了IRが0件の場合は、`improvement-reminders.md`を作成・継承しない。未完了IRがある場合は、クローズ後の初期化直後から`docs/improvement-reminders.md`へ未完了全件を継承する。
- IR-IDの採番は現行の未完了IRに存在する番号だけを対象とし、次taskで削除された対応済みIRの番号を予約しない。
- IRの状態は`未対応`、`対応中`、`対応済み`、`延期`、`別タスク候補`、`Owner除外`で管理する。
- task-idは計画承認後にだけ採番する。対応未定のIRへtask-idを付与しない。
- IRはタスク単位の選択引継ぎ資料ではなく、現行の未完了全件リストとする。task退避時は対応済みIRを含む当該taskの全IRを退避し、旧docs/result初期化直後に対応済みIRを除いた未完了IR全件を現行`docs/improvement-reminders.md`へ継承する。次taskはこの資料を入力として開始する。
- OwnerがIRを除外する場合は対象IR、理由、判断をtask-logへ記録する。未選択を理由に未完了IRを履歴だけへ残してはならない。
- active taskは1つに限定する。

## IR固有の再実行補足

- 共通rules全体への適用は`rules/idempotency.md`に従う。
- IRについては、退避前の全IR集合、次taskへ継承する未完了IR集合、サマリー・詳細の一致、対応済みIRだけが除外されたことを毎回確認する。

## リマインドと停止境界

- Documenterが通常のtask記録を完了した時点で、Workflow Coordinatorは必ずIR正本を確認し、対象IRを人間へリマインドする。Documenter自身はIRを管理・照合・通知しない。
- Workflow CoordinatorはDocumenter記録完了以外の必要な時点でも、追加リマインドを行える。
- リマインド対象は`未対応`、`延期`、`別タスク候補`とする。
- IRの未更新、不一致、未分類、取得失敗はIR処理上の問題であり、task本体の進行・受入・完了を自動停止する条件にしない。
- task本体の停止条件は、既存のtask、worker、Owner承認ルールを優先する。
- IR判断が必要な場合だけ、Workflow Coordinatorが人間へ確認を求める。

## Documenter記録完了時のWorkflow Coordinatorゲート

1. Documenterの通常task記録完了を親タスクが受領する。
2. Workflow Coordinatorが`improvement-reminders.md`の全IRを確認する。
3. `未対応`、`延期`、`別タスク候補`を全件リマインドする。
4. 確認結果と提示したIR-IDを`task-log.md`へ記録する。
5. IRの不一致や未分類があっても、IR処理だけを保留しtask本体は停止しない。

## 事前確認と実運用証跡の境界

- Documenter記録完了前に行うIR正本の存在確認、サマリー・詳細の形式確認、対象件数の把握は、計画・実装・Reviewer照合のための**事前・読み取り専用確認**である。提示件数が0件でも、実運用の発火結果または完了証跡として扱わない。
- 実運用のIR確認・人間への提示・結果記録は、Documenterの通常task記録完了をWorkflow Coordinatorが受領した後に限り、Workflow Coordinatorだけが実施する。Documenterおよび他workerは実運用ゲートを代行しない。
- Workflow Coordinatorは実運用ゲートごとに、Documenter記録完了の根拠、確認日時、IR正本の有無、確認したIR-ID集合、対象IR-ID・状態・内容、提示結果、IR不一致・未分類・取得失敗の有無、IR理由でtask本体を停止しない判定を`task-log.md`へ記録する。IR正本がない場合は、確認したIR-ID集合と対象IRを空集合、提示結果を0件として記録する。

## 移行先の有効化

移行先で次を確認できるまで、IR運用を有効化完了としない。

- IR正本の配置先
- Workflow Coordinatorの更新責任
- DocumenterがIR管理を行わない責任境界
- task-id採番とactive task制約
- IR問題がtask本体を停止しない境界
- Workflow Coordinatorが人間へ判断を求める経路
