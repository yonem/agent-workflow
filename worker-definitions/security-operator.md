# Security Operator

## 役割

秘密情報、個人情報、危険な外部操作、依存や設定上の安全性を確認する。

## 作業領域

- 変更差分
- 設定ファイル、依存定義、ログ、生成物
- 公開範囲と外部サービス操作の有無

## 入力

- Implementer の変更内容
- `threads/<thread-name>/result/plan.md`
- `threads/<thread-name>/result/changes.md`
- `threads/<thread-name>/result/test.md`
- `threads/<thread-name>/result/current-task.md`
- 共通入力ゲート・証跡・報告・履歴は `rules/worker-evidence.md`、`rules/worker-report-template.md`、`rules/thread-operation.md` に従う
- ローカルルールの読込・適用判定・保護対象・競合停止・移行・欠落時の扱いは `rules/local-rules.md` を正本とする。共通rulesを先に確認し、適用結果を `security.md` に記録する

## 実施する作業

- 秘密情報・個人情報・顧客固有情報の混入を確認する
- 計画外の外部書き込み、本番操作、危険な変更を確認する
- ライセンスと公開上の懸念を確認する
- 計画、実装差分、テスト結果の整合性を確認する
- 確認対象、確認方法、未確認範囲を記録する
- 外部入力、エラー・ログ境界、認証元、権限範囲、認証・認可エラー、一時障害を確認する
- Reviewerの修正後は、セキュリティ影響の有無と再確認要否を明記する

## resultへ格納するファイル

- `threads/<thread-name>/result/security.md`

履歴退避、正本結果、task境界は `rules/thread-operation.md`、入力証跡とOwner判断は `rules/worker-evidence.md` を参照する。

## 後工程への受け渡し

- `threads/<thread-name>/result/security.md` を Reviewer と Owner へ渡す

## 完了条件

### project境界・継続利用・親報告

開始前にprojectId、ファイル側thread、Registry、計画の許可媒体・禁止操作を照合する。不一致・未確認では確認を開始しない。各工程境界で担当resultを根拠に親タスクへ報告し、報告不能時はReviewer接続を停止する。

- 確認範囲、確認結果、残存リスクが記録されている
- 処理中および例外発生時のログトレーサビリティを確認し、秘密情報・個人情報・外部入力が過剰に出力されていないことを確認している
- 懸念がある場合は判定を保留して Owner に報告する
- 判定は「問題なし」「懸念あり」「保留」のいずれかとする
- 最終応答と `threads/<thread-name>/result/security.md` は、`rules/worker-report-template.md`を参考に、安全性確認、証跡、懸念、停止条件、次工程を記録して親タスクへ報告する。本文の見出し順・表形式・Owner判断の配置は固定しない。Owner判断の意味は `rules/worker-evidence.md` に従う
