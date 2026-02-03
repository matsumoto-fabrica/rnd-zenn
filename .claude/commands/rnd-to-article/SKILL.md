---
name: rnd-to-article
description: Convert R&D memos into polished technical articles. Use when user wants to "publish", "write article", "create blog post", or "summarize R&D" from rnd_memo notes.
---

# R&D メモから記事生成スキル

`rnd_memo/` の実験ログを読み込み、整理された技術記事を `docs/` に生成する。

## トリガーワード

以下のようなリクエストで発動：
- 「記事にして」「記事を書いて」
- 「ブログ記事にまとめて」
- 「Zennの記事にして」「Qiitaの記事にして」
- 「清書して」「まとめて」
- 「docs/ に出力して」
- 「publish」「summarize R&D」

## ワークフロー

### ステップ1: ソース収集

1. `rnd_memo/index.md` を読み、プロジェクト概要を把握
2. ユーザーが指定したノート、または全ノートを収集
3. 各ノートの `index.md` を読み込み

```bash
# 例: 全ノートのリスト取得
ls rnd_memo/notes/
```

### ステップ2: コンテンツ分析

各ノートから以下を抽出：

| 抽出項目 | ソース | 記事での扱い |
|----------|--------|--------------|
| 仮説 | `## 仮説` | 導入・課題設定 |
| 成功条件 | `## 評価基準` | 目標の明確化 |
| 参考資料 | `## リサーチ` | 参考リンク集 |
| 実験ログ | `## 実験ログ` | 手順・結果 |
| 結論 | `## 結論` | まとめ・学び |

**重要**:
- 失敗した実験（~~取り消し線~~）は「失敗から学んだこと」として価値を持つ
- 時系列の実験ログは、整理された「手順」に再構成する

### ステップ3: 記事構成の決定

ユーザーに記事スタイルを確認：

```
記事のスタイルを選んでください：
1. チュートリアル形式 - 手順を追って解説
2. 技術解説形式 - 仕組みや概念を説明
3. 体験記形式 - やってみた系の記事
4. 比較検証形式 - 複数のアプローチを比較
```

### ステップ4: 記事生成

#### 出力先
```
docs/articles/
└── YYYY-MM-DD_title.md
```

#### 記事テンプレート

```markdown
---
title: "タイトル"
date: YYYY-MM-DD
tags: [tag1, tag2]
source_notes: [001_xxx, 002_yyy]  # 元になったrnd_memoノート
---

# タイトル

## はじめに

この記事では〜について解説します。

**対象読者**:
- 〜に興味がある方
- 〜を実装したい方

**この記事でわかること**:
- 項目1
- 項目2

## 背景・課題

なぜこのR&Dを行ったか。解決したい課題。

## 使用技術

| 技術 | バージョン | 用途 |
|------|-----------|------|
| xxx  | 1.0.0     | yyy  |

## アーキテクチャ / 全体像

![アーキテクチャ図](./architecture.svg)

システムの概要図。rnd_memoにあればそのまま使用、なければ生成。

## 実装手順

### Step 1: 環境構築

```bash
# コマンド例
```

### Step 2: 基本実装

```python
# コード例
```

### Step 3: 動作確認

結果のスクリーンショットや出力。

## ハマりポイント / Tips

rnd_memo の失敗ログから抽出した注意点。

> **注意**: ~~最初の方法~~ではなく、改善後の方法を使ってください。

## 結果・パフォーマンス

| 指標 | 目標値 | 実測値 |
|------|--------|--------|
| xxx  | < 10ms | 8ms    |

## まとめ

### 達成できたこと

- 項目1
- 項目2

### 今後の課題

- 項目1
- 項目2

## 参考資料

- [リンク1](URL) - 説明
- [リンク2](URL) - 説明

---

**元のR&Dメモ**: `rnd_memo/notes/001_xxx/index.md`
```

### ステップ5: 画像・図の処理

1. rnd_memo 内の画像を `docs/articles/images/` にコピー
2. 相対パスを修正
3. 必要に応じてSVG図解を生成

```
docs/articles/
├── YYYY-MM-DD_title.md
└── images/
    ├── architecture.svg
    └── result_screenshot.png
```

### ステップ6: 最終確認

生成した記事をユーザーに提示し、以下を確認：
- タイトルは適切か
- 内容に過不足はないか
- 公開範囲（社内/社外）に問題ないか

---

## プラットフォーム別フォーマット

### Zenn 形式

```markdown
---
title: "タイトル"
emoji: "🔧"
type: "tech"  # tech or idea
topics: ["Unity", "VisionPro", "XR"]
published: false
---
```

### Qiita 形式

```markdown
---
title: タイトル
tags:
  - Unity
  - VisionPro
  - XR
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
---
```

### Note / ブログ 形式

フロントマターなし、カジュアルな文体。

---

## 棲み分けルール

| 項目 | rnd_memo/ | docs/articles/ |
|------|-----------|----------------|
| 目的 | 実験記録 | 公開用記事 |
| 読者 | 自分 | 他者 |
| 文体 | メモ・箇条書き | 整った文章 |
| 失敗 | そのまま記録 | 学びとして整理 |
| 更新 | 追記のみ | 上書き可 |
| 時系列 | 保持 | 再構成可 |

---

## 使用例

### 例1: 特定のノートから記事生成

```
User: 001_initial_setup を記事にして
Assistant:
1. rnd_memo/notes/001_initial_setup/index.md を読み込み
2. 記事スタイルを確認（チュートリアル/解説/体験記/比較）
3. docs/articles/2026-01-16_styly-netsync-setup.md を生成
```

### 例2: 複数ノートをまとめて記事化

```
User: 001〜003をまとめてZennの記事にして
Assistant:
1. 001, 002, 003 を読み込み
2. 共通テーマを抽出
3. Zenn形式で docs/articles/2026-01-20_visionpro-lbe-guide.md を生成
```

### 例3: R&D全体のサマリー記事

```
User: R&D全体をまとめて
Assistant:
1. rnd_memo/index.md と全ノートを読み込み
2. プロジェクト全体の概要記事を生成
3. 各ノートへのリンク付きダイジェスト
```

---

## 注意事項

1. **機密情報の確認**: 公開前に社内情報、APIキー等がないか確認
2. **著作権**: 参考資料の引用ルールを遵守
3. **画像の権利**: スクリーンショットの公開可否を確認
4. **元データの保持**: rnd_memo/ は編集・削除しない（記事生成後も保持）

---

## index.md 更新

記事生成後、`rnd_memo/index.md` に記録を追加：

```markdown
## 生成した記事

| 日付 | 記事 | 元ノート |
|------|------|----------|
| 2026-01-16 | [STYLY-NetSync セットアップガイド](../docs/articles/2026-01-16_styly-netsync-setup.md) | 001_initial_setup |
```
