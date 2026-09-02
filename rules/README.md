# Rules運用

## 適用範囲

- `rules/` 配下のMarkdownファイルは、原則として共通ルールである
- workerは作業開始前に `rules/README.md` を確認し、適用対象のルールを読み取る
- 適用範囲が明記されていないルールは、全worker・全プロジェクトへ適用する
- 新しい共通ルールを追加するとき、worker定義の入力欄を更新する必要はない

## 適用対象の判断

各ルールに適用範囲と状態が明記されている場合は、それに従う。

```text
scope: all-workers
scope: planner
scope: implementer
scope: tester
scope: security-operator
scope: reviewer
scope: documenter
status: active
status: deprecated
status: inactive
```

- `status: active` または状態の記載がないルールを適用する
- `deprecated` と `inactive` のルールは適用しない
- 特定のworker向けルールは、該当workerが適用する
- タスクで明示的に対象外とされたルールは適用しない
- 適用範囲や状態が不明で判断できない場合は、作業を開始せず親タスクへ報告する

## 優先順位

1. `AGENTS.md`
2. Ownerが承認した `docs/current-task.md`
3. `rules/` 配下の共通ルール
4. `worker-definitions/` のworker固有手順

同じ優先順位でルールが矛盾する場合は、作業を開始せず親タスクへ報告する。新しいルールを追加する場合は、既存ルールとの重複・矛盾・適用範囲を確認する。

## ルールファイルの記載例

```markdown
---
scope: all-workers
status: active
---

# ルール名
```
