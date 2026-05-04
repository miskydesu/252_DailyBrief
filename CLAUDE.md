# 252 Daily Brief — Routine 動作定義

このリポジトリは、ストリートスマート社員向けに **Claude Code クラウド Routines** で日次レポートを自動配信するシステムです。

## 全体方針

- **1 Claudeアカウント = 1人** の構成。各人が自分のアカウントでこのリポジトリをcloneして、自分用Routineを2つ動かす。
- このCLAUDE.mdは**全員共通のロジック**を定義する。個人差は `users/{name}.yaml` に閉じる。
- Routineは Claude Code の**クラウド実行**で動く（PC非依存）。

## 対象ユーザー

`users/` 配下のYAMLファイル単位で1人分の設定を持つ。Routine実行時は環境変数 `USER_CONFIG=users/matsubayashi.yaml` のような形で1ファイルを指定する。

## Routine A: daily-report（毎日 22:00 JST）

### 目的

その日のミーティング・会話・意思決定を、本人の頭の整理用にカテゴリ別でまとめてSlackに届ける。
**本人が能動的に喋ってまとめる必要をなくす**のがゴール。

### 入力ソース

| データ | 取得元 | 用途 |
|---|---|---|
| カレンダー予定 | Google Calendar Connector | 今日（00:00-22:00 JST）のイベント一覧 |
| Meet録画・自動メモ・議事録 | Google Drive Connector | 録画・Geminiメモ・議事録はすべて同じフォルダに格納される |
| 関連メール | Gmail Connector（オプション） | 当日のミーティング関連メール |

### 処理フロー

1. `users/{name}.yaml` を読み込み、対象ユーザーの設定を取得
2. Calendar から今日のイベント一覧を取得（自分が参加者のもの）
3. 各イベントについて：
   - Drive内でMeet自動メモ・録画ファイルを検索（ファイル名・日付・参加者で照合）
   - メモ本文を読む
   - **カテゴリ分類**を実施（5カテゴリ、後述）
   - **アクション項目**抽出（誰が・何を・いつまでに）
   - **重要相談・意思決定**抽出
   - **来週までにやること**抽出
   - **先だけど重要なこと**抽出
4. 全ミーティングを**カテゴリ別に並べる**（カテゴリの中では時系列）
5. `users/{name}.yaml` の `output.slack.mode` に応じて出力：
   - **`dm`モード**：Block Kit でリッチなレポートをDMに直送
   - **`canvas`モード**：プライベートチャンネルにCanvas作成 → DMに「レポート公開しました + リンク」を通知

### カテゴリ分類ルール

`users/{name}.yaml` の `categories` フィールドで定義（順序付き）。デフォルト：

1. **戦略** — 経営方針、事業戦略、中長期計画に関するもの
2. **個人面談** — 1on1、評価面談、メンタリング
3. **社内** — 社内定例、部署横断、社内プロジェクトMTG
4. **社外** — クライアント、パートナー、取引先、対外打合せ
5. **オペレーション** — 日常業務、運用、定常業務、事務的処理

判別が難しい場合は、参加者リスト（社内/社外）と議題から推定する。
複数カテゴリにまたがる場合は**主目的のカテゴリ1つ**に分類。

### 出力フォーマット

詳細は `skills/format-slack-message.md` を参照。

**dmモード**（Block Kit DM直送）構成：
1. **header block** — `📅 YYYY-MM-DD のデイリーブリーフ`
2. **section block** — サマリー行：`📋 N件 / ✅ M件 / 💬 K件`
3. **divider**
4. カテゴリごとに繰り返し：
   - **section block（太字見出し）** — `🎯 戦略` など
   - ミーティング1件ごとに **section block**（時刻・タイトル・参加者・要点・リンク）
   - **divider**
5. アクションテーブル（section block、mrkdwn表形式）
6. **divider**
7. 重要相談・意思決定（section block）
8. **divider**
9. 来週までにやること（section block）
10. **divider**
11. 先だけど重要なこと（section block）

## Routine B: meeting-prep（毎日 08:00 JST）

### 目的

2日後に予定されているミーティングについて、前回議事録と要点を提示し、次回アジェンダ案のたたき台を出す。
**社内外ミーティングの意思決定速度を上げる**のが狙い。

### 処理フロー

1. Calendar から「**今日 + 2日**」のイベントを取得
2. `meeting_prep.target_categories` に含まれるカテゴリの会議のみ対象（最大 `max_per_day` 件）
3. 各イベントについて：
   - 同じ会議体の過去議事録を Drive から検索（タイトル・参加者で照合）
   - 直近回の要点・宿題を抽出
   - 次回アジェンダ案を生成
4. `output.slack.mode` に関わらず **1日分をまとめて1つのCanvas**に出力
5. DM通知を1件送信

### 出力フォーマット

**1日1Canvas**（全会議まとめて）＋ **DM通知1件**

Canvas タイトル: `🗓 MM/DD MTG準備（YYYY-MM-DD 作成）`

Canvas 構成（会議ごとにセクション）:
```
## HH:MM 〇〇定例　📅 [カレンダー](link)

### 📝 前回（M/D）の要点
- ...

### ✋ 持ち越し宿題
- 担当: XXX → 状況は？

### 💡 次回アジェンダ案（たたき台）
1. 前回宿題の確認
2. ...

📎 [前回議事録](link)

---
```

DM通知:
```
🗓 MM/DD のMTG準備ができました
👉 [Canvas タイトル](canvas_url)

• HH:MM 〇〇定例
• HH:MM △△MTG
```

## レビューモード（最初の2週間）

`users/{name}.yaml` に `review_until` が設定されている期間中は、レポートを以下に複製送信する：
- 本人のSlack DM
- `review_recipients` に列挙されたユーザーのSlack DM

`review_until` を過ぎたら自動で本人のみに切り替わる（Routineコード内で日付判定）。

## エラーハンドリング

- Connectorのトークン切れ → Slack DMにエラー通知（本人 + review_recipients）
- Drive内のファイルが見つからない → 該当ミーティングは「録画/メモなし」として処理続行
- 部分的に失敗しても、取得できた範囲でレポートを生成する（黙って欠落させない）

## ログ

各Routine実行時に `logs/{date}-{user}-{routine}.md` に実行ログを残す（GitHubリポジトリにcommit）。
何が成功・失敗したか、何件処理したかを記録。

## 開発・運用

### 各ユーザーのセットアップ手順

1. このリポジトリをClaude CodeのProjectとして読み込み
2. Connector を全部認証（Calendar / Drive / Gmail / Slack）
3. `users/{自分の名前}.yaml` を作成（テンプレートは `users/_template.yaml`）
4. claude.ai/code/scheduled で2つのRoutineを登録：
   - daily-report: 22:00 JST daily / プロンプト「CLAUDE.md の Routine A を実行」
   - meeting-prep: 08:00 JST daily / プロンプト「CLAUDE.md の Routine B を実行」

### 設定変更

`users/{name}.yaml` を編集してcommit。次回Routine実行時から反映。

### Skillの追加

複雑なロジック（カテゴリ判別、アクション抽出など）は `skills/` 配下にmd形式で切り出す。
CLAUDE.mdから `@skills/extract-actions.md` のように参照する。
