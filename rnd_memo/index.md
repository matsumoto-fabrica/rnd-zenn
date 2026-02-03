# R&D INDEX

## プロジェクト概要

**開始日**: 2026-02-03

### ゴール

OpenClaw を Raspberry Pi 5 にインストールし、Discord Bot として動作させる。

### スコープ

**対象**:
- Raspberry Pi 5 への OpenClaw インストール
- Discord Bot としての連携設定
- 動作確認と基本的な使用テスト

**対象外**:
- 本番運用向けのセキュリティ強化
- 大規模スケーリング
- GUI フロントエンド開発

### 評価基準

| 指標 | 目標値 | 説明 |
|------|--------|------|
| インストール成功 | Yes | OpenClaw が RasPi5 で起動する |
| Discord連携 | Yes | Bot がコマンドに応答する |
| 応答速度 | < 30秒 | ユーザー入力から応答まで |

**成功の定義**: Discord で OpenClaw Bot にメッセージを送り、適切な応答が返ってくること

### 技術的制約

- ハードウェア: Raspberry Pi 5
- OS: Raspberry Pi OS (予定)
- 依存関係: OpenClaw、Discord API
- 環境: ローカルネットワーク

### 優先度

1. [高] OpenClaw のインストールと動作確認
2. [高] Discord Bot の設定と連携
3. [中] 安定運用のためのチューニング

---

## 現在のフォーカス

**アクティブノート**: なし（R&D 完了）

**状況**: 🦞 OpenClaw を Raspberry Pi 5 にインストールし、Discord Bot として動作確認完了！

---

## ノート一覧

| # | タイトル | ステータス | 概要 |
|---|----------|------------|------|
| 001 | [initial_setup](notes/001_initial_setup/index.md) | ✅ done | OpenClaw を RasPi5 にインストール、Discord Bot 動作確認成功 |

---

## 生成した記事

| 日付 | 記事 | 元ノート |
|------|------|----------|
| 2026-02-03 | [Raspberry Pi 5 に OpenClaw をインストールして Discord Bot を動かしてみた](../docs/articles/2026-02-03_openclaw-raspberry-pi-discord.md) | 001_initial_setup |
