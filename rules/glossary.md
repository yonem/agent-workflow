# 用語集

## 表記規則

用語集に登録する用語、略称、およびそれらを用いる共通ルールでは、大文字・小文字を区別しない。用語集の見出しに記載した表記を標準表記とし、入力時の大文字・小文字の切替は意味、役割、状態、停止条件を変えない。別の意味を持たせる必要がある場合は、大小文字の差ではなく別の用語を登録する。

## 会話・資料・taskの境界

| 用語 | 標準的な意味 | 取り違え防止 |
| --- | --- | --- |
| Owner会話 | Ownerの判断・承認・明示トリガーを記録する会話 | ファイル側thread登録のために新規作成しない |
| ユーザー向けCodexスレッド／会話 | Owner・利用者が見るCodex上の会話・作業画面 | ファイル側thread、Subagent、Codex projectと同一視しない。Subagent接続の代替に作成しない |
| Codex会話 | ユーザー向けCodexスレッド／会話の旧表現 | 新規資料では上記の正式名を使用する |
| クルー会話 | workerを実行する実Subagentの実行単位 | ユーザー向け会話、ファイル側thread、Codex projectと混同しない |
| Codex project | taskの実行環境・所属境界 | `projectId`は所属を示し、task-idやthread名から推測しない。Subagentやthreadそのものではない |
| ファイル側thread | `threads/<thread-name>/`内に置くtask資料の保存領域 | ユーザー向けCodex会話、Subagent、Codex project、taskの実行単位ではない。会話・worker・taskを自動作成するものではない |
| task | 目的、対象、完了条件を持つ論理作業単位 | 明示トリガー後に採番し、初期登録だけでは発生しない |

初期登録はファイル側threadと後続要件定義用の未指定入力領域だけを準備する非実行工程である。会話、worker、project、task、識別子、Issue、PR、branch、対象環境の作成・接続・採番・推測は行わない。要件定義開始、task-id採番、台帳登録、worker接続、履歴操作は別工程として扱う。

| 用語 | 標準的な意味 | 備考 |
| --- | --- | --- |
| docu | Documenterを指す略称。文脈で判断する | 一般的な略称や別の役割名と決めつけない |
| di | `development-improvement`を指す略称。文脈で判断する | `Dependency Injection`とは区別する |
| hb | heartbeatを指す略称。文脈で判断する | 心拍、稼働監視一般、または別の自動化機構と同一視しない |
| hc | `health-check`を指す略称。文脈で判断する | 一般的な略称や別の確認手順と決めつけない |
| oc | `operation-check`を指す略称。文脈で判断する | 一般的な略称や別の運用確認を指すものと決めつけない |
| hisi | `history/index.md`を指す略称。文脈で判断する | 一般的な履歴資料や`history/`配下の別資料と決めつけない |
| im | 改善事項を採用し、Plannerへの計画作成を指示するまでのOwnerと人間の直接一問一答を指す略称。入力時の大文字・小文字は区別しない | `issue-memo`という資料名だけを指す略称と決めつけない。Coordinator・worker・event・heartbeatの介入や、質問集約・回答推定・回数最適化の対象にしない |
| 簡易モード | 全Owner依頼の入口で適格性を判定し、低リスクで独立した変更をtask工程なしに完結させる運用 | 標準taskの代替ではない。昇格条件に該当する場合は変更前に標準IMへ移る |
| 簡易IM | 複数の簡易対応候補・選択肢・実施順を残す備忘録 | 標準IM、要件定義、Owner判断、task進捗、履歴として扱わない。新規標準task開始時だけ削除できる |
| 判定理由 | 簡易対応が適格である根拠、または標準IMへ昇格する根拠を一文で示す情報 | 計画必須4項目、OJ回答、Owner判断を代替しない |
| impr | `development-improvement`を指す略称。文脈で判断する | `issue-memo`や個別タスクの改善内容だけを指す略称と決めつけない |
| impl | Implementerを指す略称。文脈で判断する | 一般的な略称や別の役割名と決めつけない |
| ir | `improvement-reminders`を指す略称。文脈で判断する | `Investor Relations`などと同一視しない |
| oj | `owner-judge`またはOJ-IDを指す略称。文脈で判断する | 一般的な略称やOwner判断全体と決めつけない |
| rev | Reviewerを指す略称。文脈で判断する | 一般的な略称や別の役割名と決めつけない |
| so | Security Operatorを指す略称。文脈で判断する | 一般的な英単語や別の略称と決めつけない |
| ～を共有して／～を見せて | 実作業をせず、実行計画を共有する指示。文脈で判断する | 実作業の開始指示や実装完了の報告と決めつけない |
| クルー | プロジェクト内で役割を分担するworkerの集合（Planner、Implementer、Tester、Security Operator、Reviewer、Documenter等） | 対象リポジトリの機能名や、単一のworkerだけを指さない |
| Owner | 人間から直接指示・承認を受けるプロジェクト内の管理用チャット／役割 | Workflow Coordinatorや単一workerと同一視しない。OwnerとWorkflow Coordinator間の自動接続は行わない |
| Subagent | 同一task内でworker役割を実行する実行単位 | 親チャット、ユーザー向け会話、ファイル側threadで代替しない。ID・役割・task・状態をRegistryで管理する |
| worker実行単位 | 論理責務を担当する実行主体 | SubagentはCodex実行バックエンドにおける一実装であり、共通ルール上の唯一の実装ではない |
| Workflow Coordinator | 計画承認後からReviewer受入まで、handoff eventを照合し工程遷移・停止・Owner判断要求を担当する役割 | IM・要件定義には介入しない。Reviewer受入後はDocumenter開始のOwner承認を待つ。選択済み実行バックエンドの実装名と共通責務を同一視しない |
| coord | Workflow Coordinator（Coordinator）を指す略称 | Owner、単一workerと同一視しない。計画承認後からReviewer受入までの自律工程だけを担当する |
| grill | `grill-me`に由来する、人間へ一問ずつ直接質問して回答を確定するヒアリング方式 | IMではOwnerと人間が直接実施する。Coordinator・workerが質問を集約、代行、回答推定、回数最適化してはならない |
| 実行バックエンド | worker実行単位の接続、観測、完了証跡、通知、監査を提供する実行環境 | Codex、別AI、CI、人間手動運用などを取り得る。固有IDやAPI値を共通停止条件へ昇格させない |
| handoff event | 結果資料確定後に工程を引き渡す再実行安全なイベント | task-id、発生元責務、工程、結果資料、判定、証跡フィンガープリント、再開条件を記録する |
| 復旧監査 | イベント欠落・残留・Coordinator停止から回復する低頻度の照合 | 通常工程を待機させる主経路ではない。heartbeatは現行実装における起動機構の一つ |
| スレッド | 文脈によりユーザー向けCodexスレッド／会話またはファイル側threadを指し得る一般語 | 曖昧な場合は4概念を特定し、未特定なら停止する |
| 報告書 | `result/`内に保存するworker間引継ぎ・実施結果・受入結果などの正式な結果資料 | `docs/`の人間向け要約資料や、会話本文そのものと同一視しない |
| ルール | `rules/`、`README.md`、`AGENTS.md`など、このシステムの共通運用を定める資料群 | 現行タスクだけの一時指示や、個別プロジェクトの機能仕様だけを指さない |

## TASK-033での正本参照

- 用語・責務の正本は本ファイルとし、README・AGENTS・worker定義は定義を重複させず本ファイルを参照する。
- 標準工程・例外workerの正本は`rules/automation-operation.md`、各workerの固有責務の正本は`worker-definitions/`とする。
- ユーザー向け会話、ファイル側thread、Codex project、Subagentは別概念であり、相互の代替・自動作成・推測による接続を行わない。
