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

**アクティブノート**: [005_webllm_browser_llm](notes/005_webllm_browser_llm/index.md)

**状況**: ブラウザML R&D シリーズ進行中（004完了 → 005〜007へ）

---

## ノート一覧

| # | タイトル | ステータス | 概要 |
|---|----------|------------|------|
| 001 | [initial_setup](notes/001_initial_setup/index.md) | ✅ done | OpenClaw を RasPi5 にインストール、Discord Bot 動作確認成功 |
| 002 | [ghostty_overview](notes/002_ghostty_overview/index.md) | active | Ghosttyターミナルの特徴と競合比較調査 |
| 003 | [spatial_computing_xr_comparison](notes/003_spatial_computing_xr_comparison/index.md) | active | Android XR vs Apple Vision Pro 空間コンピューティング方向性比較 (Seed: 2026-02-06/N-003) |
| 004 | [ml5js_browser_ml](notes/004_ml5js_browser_ml/index.md) | ✅ done | ml5.js × p5.js で5モデル対応インタラクティブMLデモ完成（フェイスフィルター+メッシュエフェクト） |
| 005 | [webllm_browser_llm](notes/005_webllm_browser_llm/index.md) | idea | WebLLM - ブラウザ内LLM推論（+340%成長） |
| 006 | [transformers_js_huggingface](notes/006_transformers_js_huggingface/index.md) | idea | Transformers.js - HuggingFace 1,200+モデルをブラウザで（~3倍成長） |
| 007 | [onnx_runtime_web](notes/007_onnx_runtime_web/index.md) | idea | ONNX Runtime Web - WebGPU/WebNN基盤エンジン（+185%成長） |
| 008 | [markdown_cms](notes/008_markdown_cms/index.md) | idea | 自前 Markdown CMS（Hono + DynamoDB + S3 on AWS） |

---

## 生成した記事

| 日付 | 記事 | 元ノート |
|------|------|----------|
| 2026-02-03 | [Raspberry Pi 5 に OpenClaw をインストールして Discord Bot を動かしてみた](../docs/articles/2026-02-03_openclaw-raspberry-pi-discord.md) | 001_initial_setup |
| 2026-02-06 | [ml5.js × p5.js でブラウザ ML デモを作ってみた](../docs/articles/2026-02-06_ml5js-browser-ml-demo.md) | 004_ml5js_browser_ml |
