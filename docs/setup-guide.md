# セットアップガイド

新しいユーザーが Daily Brief を使い始めるまでの手順。所要時間は **約30分**。

## 前提条件

- Claudeアカウント（Pro / Max / Team / Enterpriseのいずれか）
- Google Workspaceアカウント（カレンダー・Driveが使えること）
- Slackワークスペースのメンバーであること
- GitHubアカウント（リポジトリにアクセスできること）

## Step 1: GitHub連携

1. claude.ai/code にアクセス
2. GitHubアカウントを連携
3. `miskydesu/252_DailyBrief` リポジトリへのアクセスを許可

## Step 2: Connector認証

claude.ai の Settings → Connectors で以下4つを認証：

- [ ] **Google Calendar** — 自分のWorkspaceアカウントでログイン
- [ ] **Google Drive** — 同上
- [ ] **Gmail** — 同上（オプション、データソースとして使う場合）
- [ ] **Slack** — ストリートスマートのワークスペースを選択

## Step 3: ユーザーYAMLの作成

1. リポジトリをcloneしてブランチ作成
2. `users/_template.yaml` をコピーして `users/{自分の名前}.yaml` 作成
3. 以下を記入：
   - `slack_user_id`: SlackのプロフィールからCopy member ID で取得
   - `email`: 会社メアド
   - `data_sources.google_calendar.calendar_id`: 通常は会社メアドと同じ
   - `data_sources.google_drive.meet_recordings_folder_id`: Drive上の「Meet Recordings」フォルダのURLから抽出
   - `data_sources.google_drive.claude_meeting_notes_folder_id`: Claudeで議事録を保存しているフォルダ
   - `output.slack.canvas_channel`: 個人専用のプライベートチャンネル名
   - `review.review_recipients`: 品質モニタする人のSlack User ID
4. Pull Request を出してマージ

## Step 4: Slack設定

1. 個人専用のSlackプライベートチャンネルを作成（例: `#daily-brief-{自分の名前}`）
2. Claude のSlack Bot をそのチャンネルに招待
3. Bot に Canvas作成権限があることを確認

## Step 5: Routineの作成

claude.ai/code/scheduled で以下2つを登録：

### Routine A: daily-report

- **名前**: `daily-report-{自分の名前}`
- **スケジュール**: `0 22 * * *` (cron形式、JST 22:00)
- **タイムゾーン**: Asia/Tokyo
- **プロンプト**:
```
リポジトリ miskydesu/252_DailyBrief の CLAUDE.md に従って、Routine A (daily-report) を実行してください。
対象ユーザー設定: users/{自分の名前}.yaml
```

### Routine B: meeting-prep

- **名前**: `meeting-prep-{自分の名前}`
- **スケジュール**: `0 8 * * *` (JST 8:00)
- **タイムゾーン**: Asia/Tokyo
- **プロンプト**:
```
リポジトリ miskydesu/252_DailyBrief の CLAUDE.md に従って、Routine B (meeting-prep) を実行してください。
対象ユーザー設定: users/{自分の名前}.yaml
```

## Step 6: 手動実行で動作確認

各Routineで「Run now」を押して手動実行。

確認項目：
- [ ] エラーなく完了する
- [ ] Slack Canvas が作成される
- [ ] Slack DM に通知が届く
- [ ] レビューモード期間中なら、品質モニタにもDMが届く
- [ ] レポート内容が妥当（カテゴリ分類・アクション抽出・リンク埋め込み）

## Step 7: 微調整

最初の数日は手動でも動かしながら、レポートの質を確認：

- カテゴリ分類が違うミーティングがあれば → `skills/classify-meetings.md` のルールを追加 PR
- アクション抽出の漏れ・誤りがあれば → `skills/extract-actions.md` を調整
- 出力フォーマットの好み → `CLAUDE.md` または `skills/format-slack-message.md` を調整

## トラブルシュート

### Routineがエラーで止まる

- claude.ai/code の Routines 画面でログ確認
- Connectorのトークン切れ → 再認証
- Drive内のフォルダIDが間違っている → YAMLを修正

### レポートにミーティングが出てこない

- Calendar Connectorで該当日のイベントが見えているか確認
- `only_accepted: true` になっている場合、辞退・未返信のイベントは除外される

### Canvas作成権限エラー

- Slack Botがチャンネルに招待されているか確認
- ワークスペース管理者にClaudeアプリのCanvas権限を確認
