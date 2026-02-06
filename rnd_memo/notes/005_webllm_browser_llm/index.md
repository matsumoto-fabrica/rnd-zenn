---
created: 2026-02-06 15:30
status: idea
prev: [[../004_ml5js_browser_ml/index.md]]
next:
---

# WebLLM - ブラウザ内でLLMを動かす

## 仮説

WebLLM を使えば、サーバーなしでブラウザ内で完結するLLMチャット体験やAIアプリケーションが構築できる。WebGPU の恩恵でパフォーマンスも実用レベルに達しているはず。

## 評価基準

| 指標 | 目標値 | 現在値 |
|------|--------|--------|
| モデルロード | < 60秒 | - |
| 応答速度 | > 10 tokens/sec | - |
| ブラウザ互換性 | Chrome/Edge/Firefox | - |
| メモリ使用量 | < 4GB VRAM | - |

**成功条件**: ブラウザ上でローカルLLMとの対話デモが安定動作すること

**失敗条件**: WebGPU未対応でそもそも動かない、またはメモリ不足で実用不能

## リサーチ

### WebLLM とは

- **ブラウザ内で動くLLM推論エンジン**（MLC AI プロジェクト）
- WebGPU を使ってGPUアクセラレーションされたLLM推論を実行
- OpenAI 互換API（drop-in replacement）
- GitHub: ⭐17,000 / npm: ~103,000 DL/週
- **2025年成長率: +340%** - ブラウザMLライブラリで最速の成長

### 主な特徴

| 特徴 | 詳細 |
|------|------|
| **対応モデル** | Llama 3.x, Phi-3/3.5, Gemma 2, Mistral, Qwen 2.5 等数十種 |
| **API** | OpenAI Chat Completions 互換（streaming対応） |
| **バックエンド** | WebGPU（必須） |
| **プライバシー** | 完全クライアントサイド - データがデバイスから出ない |
| **機能** | ストリーミング、JSON mode、Function calling |
| **実行環境** | ブラウザ、Web Worker、Service Worker |

### セットアップ

```bash
npm install @mlc-ai/web-llm
```

```javascript
import { CreateMLCEngine } from "@mlc-ai/web-llm";

const engine = await CreateMLCEngine("Llama-3.1-8B-Instruct-q4f32_1-MLC");

const reply = await engine.chat.completions.create({
  messages: [{ role: "user", content: "Hello!" }],
  stream: true,
});

for await (const chunk of reply) {
  console.log(chunk.choices[0]?.delta?.content || "");
}
```

### 技術要件

- **必須**: WebGPU対応ブラウザ（Chrome 113+, Edge 113+, Firefox 141+, Safari 26+）
- **推奨**: 4GB+ VRAM のGPU
- **モデルサイズ**: 量子化済みで数百MB〜数GB（初回ダウンロード、キャッシュされる）

### ユースケース候補

- プライバシー重視のAIチャットボット
- オフラインで動くAIアシスタント
- クリエイティブコーディング × 自然言語（p5.jsと組み合わせ？）
- エッジでのテキスト要約・翻訳

### 参考資料

- [WebLLM GitHub](https://github.com/mlc-ai/web-llm)
- [WebLLM ドキュメント](https://webllm.mlc.ai/)
- [WebLLM 論文 (arXiv)](https://arxiv.org/abs/2412.15803)
- [Mozilla Blog: 3W for in-browser AI](https://blog.mozilla.ai/3w-for-in-browser-ai-webllm-wasm-webworkers/)

## 実験ログ

_(実験開始後にログを追記)_

## 結論

_(実験完了後に記載)_
