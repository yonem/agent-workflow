# 用語集

## 会話・資料・taskの境界

| 用語 | 標準的な意味 | 取り違え防止 |
| --- | --- | --- |
| Owner会話 | Ownerの判断・承認・明示トリガーを記録する会話 | ファイル側thread登録のために新規作成しない |
| ユーザー向けCodexスレッド／会話 | Owner・利用者が見るCodex上の会話・作業画面 | ファイル側thread、Subagent、Codex projectと同一視しない。Subagent接続の代替に作成しない |
| Codex会話 | ユーザー向けCodexスレッド／会話の旧表現 | 新規資料では上記の正式名を使用する |
| クルー会話 | workerを実行する実Subagentの実行単位 | ユーザー向け会話、ファイル側thread、Codex projectと混同しない |
| Codex project | taskの実行環境・所属境界 | `projectId`は所属を示し、task-idやthread名から推測しない。Subagentやthreadそのものではない |
| ファイル側thread | `threads/<thread-name>/`の資料保存領域 | 会話・worker・taskを自動作成するものではない |
| task | 目的、対象、完了条件を持つ論理作業単位 | 明示トリガー後に採番し、初期登録だけでは発生しない |

初期登録はファイル側threadと後続要件定義用の未指定入力領域だけを準備する非実行工程である。会話、worker、project、task、識別子、Issue、PR、branch、対象環境の作成・接続・採番・推測は行わない。要件定義開始、task-id採番、台帳登録、worker接続、履歴操作は別工程として扱う。

| 用語 | 標準的な意味 | 備考 |
| --- | --- | --- |
| Owner Agent | Orchestratorの旧称 | 人間のOwnerや親sessionを指さない。新規のルール・資料では`Orchestrator`を使用する |
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
| Orchestrator | タスク進行・クルー接続・正本照合・承認要求を担当するプロジェクト内の実行チャット／Agent | 人間のOwnerとは別の役割。`Owner Agent`は旧称としてのみ扱う。人間への承認要求はOrchestratorチャット上で提示する |
| orch | Orchestratorの短縮語 | Ownerやworkerと混同せず、タスク進行を担当するAgent／チャットを指す |
| ファイル側thread | Codexプロジェクトと対になる`threads/`内のtask資料保存領域 | ユーザー向けCodex会話、Subagent、Codex project、taskの実行単位ではない |
| Subagent | 同一task内でworker役割を実行する実行単位 | 親チャット、ユーザー向け会話、ファイル側threadで代替しない。ID・役割・task・状態をRegistryで管理する |
| スレッド | 文脈によりユーザー向けCodexスレッド／会話またはファイル側threadを指し得る一般語 | 曖昧な場合は4概念を特定し、未特定なら停止する |
| 報告書 | `result/`内に保存するworker間引継ぎ・実施結果・受入結果などの正式な結果資料 | `docs/`の人間向け要約資料や、会話本文そのものと同一視しない |
| ルール | `rules/`、`README.md`、`AGENTS.md`など、このシステムの共通運用を定める資料群 | 現行タスクだけの一時指示や、個別プロジェクトの機能仕様だけを指さない |

## TASK-033での正本参照

- 用語・責務の正本は本ファイルとし、README・AGENTS・worker定義は定義を重複させず本ファイルを参照する。
- 標準工程・例外workerの正本は`rules/automation-operation.md`、各workerの固有責務の正本は`worker-definitions/`とする。
- ユーザー向け会話、ファイル側thread、Codex project、Subagentは別概念であり、相互の代替・自動作成・推測による接続を行わない。
