---
name: rnd-seed
description: Collect R&D seeds - trending topics, followed accounts posts from X, Zenn, Note.com. Use when user wants to "collect seeds", "seed collection", "check trends", "weekly summary", "シード収集", "トレンドチェック", "週次サマリー", or "promote seed".
---

# R&D シード収集エージェント

X のフォローアカウント、Zenn・Note.com のトレンドから情報を収集し、R&D の種（seed）をメモとして蓄積する。

## 基本原則

1. **削除禁止、追記のみ**: rnd-memo と同じ原則を seeds にも適用
2. **ベストエフォート収集**: X は API 不使用のため完全な網羅は不可能
3. **スコアリングで優先度付け**: 複数ソースで言及 = 高注目度
4. **R&D への昇格パス**: seed → rnd_memo/notes/ への導線を維持

---

## 操作モード

| モード | トリガー | 動作 |
|--------|---------|------|
| daily | "シード収集", "トレンドチェック", "最新情報", "seed collection" | X/Zenn/Note.com から当日分を収集 |
| weekly | "週次サマリー", "weekly summary" | 当週の daily を集約・トレンド分析 |
| promote | "シード昇格", "R&Dにする", "promote seed" | seed を rnd_memo/notes/ のノートに昇格 |

---

## ディレクトリ構成

```
rnd_memo/seeds/
├── config.yml          # 監視設定（アカウント、キーワード、フィルタ）
├── daily/              # 日次収集（YYYY-MM-DD.md）
└── weekly/             # 週次サマリー（YYYY-WNN.md）
```

---

## Daily 収集フロー

ユーザーが「シード収集」「トレンドチェック」「最新情報」と言ったとき:

### 手順

1. `rnd_memo/seeds/config.yml` を読み込む
2. 以下のソースから並行して情報収集:

   **X (Twitter)**:
   - 各アカウントの最新投稿: `WebSearch("site:x.com/{username} {topic}")` で検索
   - キーワード横断: `WebSearch("{keyword} site:x.com")` で検索
   - ※ X API 未使用のためベストエフォート（Brave Search 経由）

   **Zenn**:
   - トレンド記事: `WebFetch("https://zenn.dev/api/articles?order=trending&count=20")` で取得
   - トピック別: `WebFetch("https://zenn.dev/api/articles?topicname={topic}&order=trending&count=10")` で取得
   - ユーザー別: `WebFetch("https://zenn.dev/api/articles?username={user}&order=latest&count=5")` で取得

   **Note.com**:
   - カテゴリ別: `WebFetch("https://note.com/api/v1/categories/{slug}")` で取得

   **キーワード横断検索**:
   - `WebSearch("{keyword} site:zenn.dev OR site:note.com")` で検索

3. 収集結果を統合:
   - 重複排除（URL ベース）
   - スコアリング（後述）
   - `config.yml` の `filters.exclude_keywords` でフィルタリング

4. `rnd_memo/seeds/daily/YYYY-MM-DD.md` に書き出し

### スコアリング基準

| 基準 | スコア |
|------|--------|
| 複数ソースで言及 | ★★★ |
| config のキーワードに合致 | ★★ |
| いいね数が `min_liked_count` 以上 | ★★ |
| フォローアカウントの投稿 | ★★ |
| 単一ソースのみ | ★ |

注目度は ★★★（高）、★★（中）、★（低）の3段階。

### API レスポンスの処理

**Zenn API が JSON を返す場合**: articles 配列から title, path, liked_count, user.username を抽出。
**Zenn API が HTML を返す場合**: WebSearch にフォールバックして `"zenn.dev トレンド {topic}"` で検索。
**Note.com API が失敗する場合**: WebSearch にフォールバックして `"note.com {category} 人気"` で検索。

---

## Weekly サマリーフロー

ユーザーが「週次サマリー」「weekly summary」と言ったとき:

### 手順

1. 当週（月曜〜日曜）の daily ファイルを全て読み込み
2. 分析:
   - トピック出現頻度をカウント
   - クロスソース分析（複数ソースで言及されたトピック）
   - 先週との比較（先週の weekly があれば）
3. `rnd_memo/seeds/weekly/YYYY-WNN.md` に生成

### 週番号の決定

ISO 8601 準拠: `date +%G-W%V` 形式（例: `2026-W06`）

---

## Promote フロー

ユーザーが「シード昇格」「R&Dにする」「promote seed」と言ったとき:

### 手順

1. ユーザーに昇格する Seed ID を確認（例: Z-003）
2. 該当する daily ファイルから Seed 情報を取得
3. `rnd_memo/notes/` の次のシーケンス番号を確認
4. `rnd_memo/notes/NNN_title/index.md` を新規作成:
   - rnd-memo スキルのノートテンプレートに従う
   - Seed の要約 → 仮説セクション
   - Seed のリンク → 参考資料セクション
   - Seed のタグ → 評価基準のヒント
5. `rnd_memo/index.md` のノート一覧テーブルに新しいエントリを追加
6. daily ファイルの該当 Seed に `[PROMOTED → NNN_title]` を追記

### Promote 時のノート命名

- タイトルは英語の snake_case（パスに日本語禁止）
- Seed の内容から適切な英語タイトルを生成

---

## テンプレート

### Daily メモテンプレート（rnd_memo/seeds/daily/YYYY-MM-DD.md）

<!-- BEGIN: DAILY_SEED_TEMPLATE -->
```markdown
---
date: YYYY-MM-DD
total_seeds: N
---

# Seeds: YYYY-MM-DD

## 収集サマリー

| ソース | 件数 | 注目度高 |
|--------|------|----------|
| X | N | N |
| Zenn | N | N |
| Note.com | N | N |

---

## X (Twitter)

### X-001: [タイトル](URL)
- **アカウント**: @username
- **要約**: 1-2行の要約
- **キーワード**: tag1, tag2
- **注目度**: ★★★/★★/★

---

## Zenn

### Z-001: [タイトル](URL)
- **著者**: username / **いいね**: N
- **要約**: 1-2行の要約
- **キーワード**: tag1, tag2
- **注目度**: ★★★/★★/★

---

## Note.com

### N-001: [タイトル](URL)
- **著者**: nickname / **いいね**: N
- **要約**: 1-2行の要約
- **キーワード**: tag1, tag2
- **注目度**: ★★★/★★/★

---

## 本日の注目 Seeds (Top 5)

| # | ID | タイトル | ソース | 注目度 | キーワード |
|---|-----|---------|--------|--------|-----------|
| 1 | Z-001 | ... | Zenn | ★★★ | ... |
| 2 | X-002 | ... | X | ★★★ | ... |
| 3 | ... | ... | ... | ★★ | ... |
| 4 | ... | ... | ... | ★★ | ... |
| 5 | ... | ... | ... | ★★ | ... |

## R&D 候補メモ

- **Z-001**: （この Seed が R&D になり得る理由・コメント）
- **X-002**: （この Seed が R&D になり得る理由・コメント）
```
<!-- END: DAILY_SEED_TEMPLATE -->

### Weekly メモテンプレート（rnd_memo/seeds/weekly/YYYY-WNN.md）

<!-- BEGIN: WEEKLY_SEED_TEMPLATE -->
```markdown
---
week: YYYY-WNN
period: YYYY-MM-DD 〜 YYYY-MM-DD
total_seeds: N
---

# Weekly Seeds Summary: YYYY-WNN

## トレンド分析

### トピック出現頻度

| トピック | 今週 | 先週 | 変化 |
|---------|------|------|------|
| AI | 12 | 8 | ↑ |
| Rust | 5 | 6 | ↓ |
| ... | ... | ... | ... |

### クロスソース分析

複数ソースで言及されたトピック:

| トピック | X | Zenn | Note | 合計 |
|---------|---|------|------|------|
| AI agent | 3 | 5 | 2 | 10 |
| ... | ... | ... | ... | ... |

## Top 10 Seeds ランキング

| # | ID | タイトル | ソース | 日付 | 注目度 |
|---|-----|---------|--------|------|--------|
| 1 | Z-001 | ... | Zenn | MM-DD | ★★★ |
| ... | ... | ... | ... | ... | ... |

## R&D 昇格候補

### 候補1: [タイトル](元URL)
- **元 Seed**: YYYY-MM-DD / Z-001
- **理由**: なぜ R&D として深掘りする価値があるか
- **想定スコープ**: 何を調査・実験するか

### 候補2: ...

## 監視アカウント活動サマリー

| アカウント | 投稿数 | 注目投稿 |
|-----------|--------|---------|
| @sdaifuku | N | X-001, X-005 |

## 来週の注目ポイント

- トレンドから予測される来週の注目トピック
- 継続監視すべきキーワード
```
<!-- END: WEEKLY_SEED_TEMPLATE -->

---

## 重要ルール

- **既存ファイルの内容を削除・置換しない**: 訂正は ~~取り消し線~~ で対応
- **daily ファイルは日付ごとに1つ**: 同日に再実行した場合は既存ファイルに追記
- **Seed ID はソース別プレフィックス**: X-NNN, Z-NNN, N-NNN（日ごとにリセット）
- **promote 後も daily の Seed は残す**: `[PROMOTED → NNN_title]` を追記するのみ
- **config.yml の変更はユーザーに確認**: 勝手にアカウントやキーワードを追加しない
- **API 失敗時はフォールバック**: WebFetch 失敗 → WebSearch で代替検索
- **ファイル名は英語のみ**: daily/weekly のファイル名に日本語を含めない
