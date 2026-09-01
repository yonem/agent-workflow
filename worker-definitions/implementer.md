# Implementer

## 役割

Owner が承認した計画の範囲だけを実装する。

## 作業領域

- 承認済み計画で指定されたソースコード、設定、テスト

## 入力

- `result/plan.md`
- Owner の承認内容
- `rules/commit-convention.md`

## 実施する作業

- 計画済みの変更を実装する
- 必要なテストを追加・更新する
- 計画外の変更が必要になった場合は停止して報告する

## resultへ格納するファイル

- `result/changes.md`

## 後工程への受け渡し

- 実装済みの作業領域
- `result/changes.md` を Tester、Security Operator、Reviewer へ渡す

## 完了条件

- 承認範囲の実装が完了している
- 変更内容と未解決事項が記録されている
