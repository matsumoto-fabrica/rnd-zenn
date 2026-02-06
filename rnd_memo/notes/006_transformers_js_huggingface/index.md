---
created: 2026-02-06 15:30
status: idea
prev: [[../004_ml5js_browser_ml/index.md]]
next:
---

# Transformers.js - HuggingFace モデルをブラウザで実行

## 仮説

Transformers.js を使えば、HuggingFace の豊富なモデル（NLP、Vision、Audio）をブラウザ内で直接実行でき、Python の transformers ライブラリと同等の体験をJavaScriptで実現できるはず。

## 評価基準

| 指標 | 目標値 | 現在値 |
|------|--------|--------|
| モデルロード | < 30秒 | - |
| 推論速度 | 実用レベル | - |
| 対応タスク | NLP + Vision 両方動作 | - |
| API使いやすさ | Python版と同等 | - |

**成功条件**: 複数のタスク（テキスト生成、画像分類、音声認識等）がブラウザ内で動作すること

**失敗条件**: モデルが大きすぎてロードできない、または推論が遅すぎて使えない

## リサーチ

### Transformers.js とは

- **HuggingFace の transformers ライブラリのJavaScript版**
- ONNX Runtime を内部で使用
- 1,200+の互換モデル（HuggingFace Hub 上のONNX変換済みモデル）
- GitHub: ⭐15,300 / npm: ~285,000 DL/週
- **2025年成長率: 約3倍**（240K→719K月間DL）

### 主な特徴

| 特徴 | 詳細 |
|------|------|
| **NLPタスク** | テキスト生成、要約、翻訳、Q&A、感情分析、埋め込み |
| **Visionタスク** | 画像分類、物体検出、セグメンテーション、深度推定 |
| **Audioタスク** | 音声認識(Whisper)、テキスト読み上げ、音声分類 |
| **マルチモーダル** | 画像キャプション、Visual Q&A |
| **バックエンド** | ONNX Runtime (WebGPU / WASM / WebGL) |
| **API** | Python transformers とほぼ同じ pipeline() API |

### セットアップ

```bash
npm install @huggingface/transformers
```

```javascript
import { pipeline } from "@huggingface/transformers";

// テキスト感情分析
const classifier = await pipeline("sentiment-analysis");
const result = await classifier("I love this library!");
// [{ label: "POSITIVE", score: 0.9998 }]

// 画像分類
const imageClassifier = await pipeline("image-classification");
const result = await imageClassifier("https://example.com/cat.jpg");

// 音声認識（Whisper）
const transcriber = await pipeline("automatic-speech-recognition", "Xenova/whisper-tiny");
const result = await transcriber(audioBlob);
```

### v3 の新機能

- **WebGPU バックエンド対応** - 大幅な高速化
- パッケージ名が `@xenova/transformers` → `@huggingface/transformers` に移行
- マルチモーダルモデル対応強化
- ストリーミングテキスト生成

### 対応タスク一覧

| カテゴリ | タスク |
|----------|--------|
| **テキスト** | text-generation, summarization, translation, fill-mask, question-answering, text-classification, token-classification, zero-shot-classification, feature-extraction |
| **画像** | image-classification, object-detection, image-segmentation, depth-estimation, image-to-text |
| **音声** | automatic-speech-recognition, text-to-speech, audio-classification |
| **マルチモーダル** | visual-question-answering, document-question-answering |

### ユースケース候補

- ブラウザ内リアルタイム翻訳ツール
- Whisper でオフライン音声文字起こし
- 画像キャプション自動生成
- テキスト埋め込み → クライアントサイド意味検索
- ゼロショット分類でカスタムカテゴリ分類

### 参考資料

- [Transformers.js ドキュメント](https://huggingface.co/docs/transformers.js/en/index)
- [Transformers.js GitHub](https://github.com/huggingface/transformers.js)
- [v3 アナウンス](https://huggingface.co/blog/transformersjs-v3)
- [npm: @huggingface/transformers](https://www.npmjs.com/package/@huggingface/transformers)

## 実験ログ

_(実験開始後にログを追記)_

## 結論

_(実験完了後に記載)_
