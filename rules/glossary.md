# 用語集

## 会話・資料・taskの境界

| 用語 | 標準的な意味 | 取り違え防止 |
| --- | --- | --- |
| Owner会話 | Ownerの判断・承認・明示トリガーを記録する会話 | ファイル側thread登録のために新規作成しない |
| Codex会話 | Codex上の作業会話 | ファイル側threadやtaskと同一視せず、必要性確定後に作成・再利用を判断する |
| クルー会話 | workerを実行する会話単位 | Owner会話やファイル側threadと混同しない |
| Codexプロジェクト | 会話・作業環境の所属境界 | `projectId`は所属を示し、task-idやthread名から推測しない |
| ファイル側thread | `threads/<thread-name>/`の資料保存領域 | 会話・worker・taskを自動作成するものではない |
| task | 目的、対象、完了条件を持つ論理作業単位 | 明示トリガー後に採番し、初期登録だけでは発生しない |

初期登録はファイル側threadと後続要件定義用の未指定入力領域だけを準備する非実行工程である。会話、worker、project、task、識別子、Issue、PR、branch、対象環境の作成・接続・採番・推測は行わない。要件定義開始、task-id採番、台帳登録、worker接続、履歴操作は別工程として扱う。

| 用語 | 標準的な意味 | 備考 |
| --- | --- | --- |
| Owner／オーナー／親 | 人間から指示を受ける起点Agent。他のクルーへ指示を出し、IR・OJ・タスク進行を管理する | この会話の人間と同一視しない。一般的な所有者、親スレッド、または他クルーの役割名を指さない |
| docu | Documenterを指す略称。文脈で判断する | 一般的な略称や別の役割名と決めつけない |
| di | `development-improvement`を指す略称。文脈で判断する | `Dependency Injection`とは区別する |
| hb | heartbeatを指す略称。文脈で判断する | 心拍、稼働監視一般、または別の自動化機構と同一視しない |
| hc | `health-check`を指す略称。文脈で判断する | 一般的な略称や別の確認手順と決めつけない |
| oc | `operation-check`を指す略称。文脈で判断する | 一般的な略称や別の運用確認を指すものと決めつけない |
| hisi | `history/index.md`を指す略称。文脈で判断する | 一般的な履歴資料や`history/`配下の別資料と決めつけない |
| im | 要件定義と同意を指す略称。「im（要件定義）して」は要件定義を実施して同意を得る意味として扱う。文脈で判断する | `issue-memo`という資料名だけを指す略称と決めつけない |
| impr | `development-improvement`を指す略称。文脈で判断する | `issue-memo`や個別タスクの改善内容だけを指す略称と決めつけない |
| impl | Implementerを指す略称。文脈で判断する | 一般的な略称や別の役割名と決めつけない |
| ir | `improvement-reminders`を指す略称。文脈で判断する | `Investor Relations`などと同一視しない |
| oj | `owner-jadge`またはOJ-IDを指す略称。文脈で判断する | 一般的な略称やOwner判断全体と決めつけない |
| rev | Reviewerを指す略称。文脈で判断する | 一般的な略称や別の役割名と決めつけない |
| so | Security Operatorを指す略称。文脈で判断する | 一般的な英単語や別の略称と決めつけない |
| ～を共有して／～を見せて | 実作業をせず、実行計画を共有する指示。文脈で判断する | 実作業の開始指示や実装完了の報告と決めつけない |
| クルー | プロジェクト内で役割を分担するworkerの集合（Planner、Implementer、Tester、Security Operator、Reviewer、Documenter等） | 対象リポジトリの機能名や、単一のworkerだけを指さない |
| Owner | 人間から直接指示・承認を受けるプロジェクト内の管理用チャット／役割 | Orchestratorや単一workerと同一視しない。OwnerとOrchestrator間の自動接続は行わない |
| Orchestrator | 旧Owner Agentに相当する、タスク進行・クルー接続・正本照合・承認要求を担当するプロジェクト内の実行チャット／Agent | 人間のOwnerとは別の役割。人間への承認要求はOrchestratorチャット上で提示する |
| orch | Orchestratorの短縮語 | Ownerやworkerと混同せず、タスク進行を担当するAgent／チャットを指す |
| スレッド | Codexプロジェクトと対になる`threads/`内で、1つのタスクを処理する単位 | プロジェクト全体、worker全体、または単一workerだけを指さない、Codexプロジェクト内のスレッド(チャット)を指さない |
| 報告書 | `result/`内に保存するworker間引継ぎ・実施結果・受入結果などの正式な結果資料 | `docs/`の人間向け要約資料や、会話本文そのものと同一視しない |
| ルール | `rules/`、`README.md`、`AGENTS.md`など、このシステムの共通運用を定める資料群 | 現行タスクだけの一時指示や、個別プロジェクトの機能仕様だけを指さない |
