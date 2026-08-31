# 新規PJ導入手順

## 1. 導入前に人間が決めること

次の項目を埋めてからCodexへ作業を依頼する。

| 項目 | 記入内容 |
| --- | --- |
| PJ名 |  |
| 目的 |  |
| 利用者 |  |
| 対象範囲 |  |
| 対象外 |  |
| 技術スタック |  |
| リポジトリ |  |
| 開発基準ブランチ | develop |
| 完了条件 |  |
| 禁止操作 | 本番操作、秘密情報の取得、無断公開など |
| 承認者 |  |
| 期限・優先度 |  |

## 2. リポジトリへ配置するファイル

```text
PJリポジトリ/
├── AGENTS.md
├── README.md
├── docs/
│   ├── project/project-charter.md
│   ├── requirements/v1.md
│   ├── architecture.md
│   ├── onboarding.md
│   ├── initial-instruction.md
│   ├── completion-checklist.md
│   └── commit-convention.md
├── templates/
│   ├── current-task.md
│   ├── implementation-plan.md
│   ├── test-result.md
│   ├── review-result.md
│   └── task-log.md
└── .github/
    └── pull_request_template.md
```

## 3. 実装直前の確認

- PJ定義書の空欄がない
- 要件IDと受入条件がある
- アーキテクチャと責任分担がある
- 最初のタスクが1つに絞られている
- 作業対象ブランチとワークツリーが確定している
- 自動検証コマンドが確認できる
- ハードストップが設定されている
- Ownerが初回計画を承認する準備ができている

## 4. 初回実行

1. Codexに初回指示を投入する
2. Plannerの調査結果と計画を受け取る
3. Ownerが計画、対象、完了条件を承認する
4. Implementerに限定範囲で実装させる
5. Testerが検証結果を記録する
6. Reviewerが独立レビューする
7. Ownerがマージ可否を判断する
