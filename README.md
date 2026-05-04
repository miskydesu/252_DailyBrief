# 252 Daily Brief

ストリートスマート社員向けの日次レポート自動配信システム。
**Claude Code クラウドRoutines** で実装。

## ✨ 何ができる

毎日 22:00 に、その日のミーティングを5カテゴリ（戦略 / 個人面談 / 社内 / 社外 / オペレーション）で整理して、Slack Canvas に長文レポート＋DM通知を自動配信する。

さらに、2日後のミーティング向けに前回議事録の要点と次回アジェンダ案を毎朝 8:00 にDMで届ける。

## 🏗 アーキテクチャ

- **1 Claudeアカウント = 1人** の構成
- 全員共通のロジックは `CLAUDE.md` と `skills/` に
- 個人差は `users/{name}.yaml` に
- 実行は Anthropic クラウド（PC非依存）

詳細は [CLAUDE.md](./CLAUDE.md) 参照。

## 📂 リポジトリ構造

```
252_DailyBrief/
├── CLAUDE.md                        # Routine 動作定義（共通ロジック）
├── README.md                        # このファイル
├── skills/                          # 切り出された処理スキル
│   ├── classify-meetings.md         # カテゴリ分類
│   ├── extract-actions.md           # アクション項目抽出
│   └── format-slack-message.md      # Slack出力フォーマット
├── users/                           # ユーザーごとの設定
│   ├── _template.yaml               # 新規追加用テンプレート
│   ├── matsubayashi.yaml            # 松林さん
│   └── nakagawa.yaml                # 中川さん
├── docs/                            # 運用ドキュメント
│   └── setup-guide.md               # セットアップ手順
└── logs/                            # Routine実行ログ（自動生成）
```

## 🚀 ユーザーの追加方法

1. `users/_template.yaml` をコピーして `users/{自分の名前}.yaml` を作成
2. Slack User ID・Calendar ID・Drive フォルダIDを記入
3. 自分のClaudeアカウントでこのリポジトリをClaude Code Projectとしてclone
4. 各種Connector（Calendar / Drive / Gmail / Slack）を認証
5. claude.ai/code/scheduled で2つのRoutineを登録

詳細は [docs/setup-guide.md](./docs/setup-guide.md) を参照。

## 🎯 マイルストーン

| 日付 | ターゲット |
|---|---|
| 5/5 | 松林さん・中川さんの実データでデモ |
| 5/7 | Anthropicプレゼン |
| 5/19 | レビューモード終了、本番運用へ |
