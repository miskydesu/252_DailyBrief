# Skill: Slackメッセージフォーマット

`users/{name}.yaml` の `output.slack.mode` によって出力方式が変わる。

| mode | レポート本体 | DM |
|---|---|---|
| `dm`（デフォルト） | Block Kit でDMに直送 | レポート = DM本体 |
| `canvas` | プライベートチャンネルにCanvas作成 | 「公開しました + リンク」のみ |

## Block Kit の基本ルール

- Slack API の `chat.postMessage` に `blocks` 配列として渡す
- `text` フィールドも必ず設定する（通知プレビュー用）
- mrkdwn は `"type": "mrkdwn"` フィールド内でのみ有効
- 1メッセージのブロック数上限は50。超える場合は分割して連続送信する

## Routine A: daily-report の Block Kit 構成

### 1. ヘッダー

```json
{"type": "header", "text": {"type": "plain_text", "text": "📅 2026-05-05 のデイリーブリーフ"}}
```

### 2. サマリー行

```json
{"type": "section", "text": {"type": "mrkdwn", "text": "📋 ミーティング: *5件*　✅ アクション: *3件*　💬 重要相談: *2件*"}}
```

### 3. divider

```json
{"type": "divider"}
```

### 4. カテゴリ見出し

```json
{"type": "section", "text": {"type": "mrkdwn", "text": "*🎯 戦略*"}}
```

### 5. ミーティング1件

```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*[09:00-10:00] 〇〇プロジェクト戦略会議*\n👤 参加: 〇〇、△△\n📝 要点: ...\n📎 <議事録URL|議事録>　🎥 <録画URL|録画>　📅 <カレンダーURL|カレンダー>"
  }
}
```

- 録画/議事録が存在しない場合はそのリンクだけ省略（他は残す）
- カレンダーリンクは常に付ける

### 6. アクション一覧

Slack DM では Markdown テーブルがレンダリングされないため、箇条書きで代替：

```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*✅ アクション*\n• 松林 ／ XXX案を整理 ／ 5/8 ／ <MTGリンク|〇〇MTG>\n• 藤田 ／ YYY を中川に展開 ／ 5/10 ／ <MTGリンク|△△MTG>"
  }
}
```

### 7. 重要相談・意思決定

```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*💬 重要相談・意思決定*\n• *XXX について*：〇〇案で進める方向で合意（<リンク|出典>）\n• *YYY について*：△△さんから相談あり、来週判断（<リンク|出典>）"
  }
}
```

### 8. 来週までにやること

```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*📅 来週までにやること*\n• AAA を BBB さんに確認\n• CCC のドラフトレビュー"
  }
}
```

### 9. 先だけど重要なこと

```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*🔭 先だけど重要なこと*\n• DDD（〇月）：今のうちから準備が必要\n• EEE（〇月）：意思決定だけ早めに"
  }
}
```

## canvasモード（mode: canvas）

### Canvas本体（Markdown）

`output.slack.canvas_channel` で指定したプライベートチャンネルに作成する。
Canvas名: `📅 YYYY-MM-DD {user.name} デイリーブリーフ`

本文はdmモードと同じ構成をMarkdownで記述：

```markdown
# 📅 2026-05-05 のデイリーブリーフ

📋 ミーティング: 5件 / ✅ アクション: 3件 / 💬 重要相談: 2件

---

## 🎯 戦略

**[09:00-10:00] 〇〇プロジェクト戦略会議**
- 参加: 〇〇、△△
- 要点: ...
- 📎 [議事録](link) 　🎥 [録画](link)　📅 [カレンダー](link)

...（以下同構成）

## ✅ アクション
| 担当 | 内容 | 期限 | 出典 |
|---|---|---|---|
| 松林 | XXX案を整理 | 5/8 | [〇〇MTG](link) |

## 💬 重要相談・意思決定
## 📅 来週までにやること
## 🔭 先だけど重要なこと
```

### DM通知（canvasモード）

Canvasを作成した後、本人DMにリンクのみ送信：

```
📊 今日のレポート公開しました
👉 <Canvas URL|本日のデイリーブリーフ>

📋 ミーティング: {N}件 / ✅ アクション: {M}件 / 💬 重要相談: {K}件
```

---

## Routine B: meeting-prep の出力形式

`output.slack.mode` の値に関わらず、**1日1Canvas**（全会議まとめて）＋ **DM通知1件** で固定。

### Canvas 本体（Markdown）

Canvas タイトル: `🗓 MM/DD MTG準備（YYYY-MM-DD 作成）`

```markdown
# 🗓 MM/DD のMTG準備

---

## HH:MM 〇〇定例　📅 [カレンダー](link)

### 📝 前回（M/D）の要点
- 要点1
- 要点2

### ✋ 持ち越し宿題
- 担当者: XXX → 状況は？
- 担当者: YYY → 完了？

### 💡 次回アジェンダ案（たたき台）
1. 前回宿題の確認
2. ...
3. ...

📎 [前回議事録](link)

---

## HH:MM △△MTG　📅 [カレンダー](link)

...（以下同構成）
```

### DM通知（Block Kit）

```json
[
  {
    "type": "section",
    "text": {
      "type": "mrkdwn",
      "text": "🗓 *MM/DD のMTG準備ができました*\n👉 <canvas_url|🗓 MM/DD MTG準備>\n\n• HH:MM 〇〇定例\n• HH:MM △△MTG"
    }
  }
]
```

## エラー通知のDM

```json
[
  {"type": "header", "text": {"type": "plain_text", "text": "⚠️ Daily Brief 実行エラー"}},
  {
    "type": "section",
    "text": {
      "type": "mrkdwn",
      "text": "*ルーチン*: daily-report\n*時刻*: 2026-05-05 22:00 JST\n*エラー*: {エラー概要}"
    }
  }
]
```

`review_recipients` にも同じメッセージを送る（レビューモード期間中）。

## 文体・トーン

- 敬語は使わない（本人へのDMなので）
- 絵文字は構造マーカーとしてのみ使う（装飾としては使わない）
- 数字や固有名詞は省略しない
- 推測した内容には必ず `(推定)` `(要確認)` のマーカーを付ける
