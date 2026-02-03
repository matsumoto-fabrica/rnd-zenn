---
created: 2026-02-03 15:30
status: done
prev: null
next: []
---

# OpenClaw を Raspberry Pi 5 にインストール

## 仮説

OpenClaw は Raspberry Pi 5 (ARM64) 上で動作し、Discord Bot として連携できる。

## 評価基準

| 指標 | 目標値 | 現在値 |
|------|--------|--------|
| インストール成功 | Yes | ✅ Yes |
| 起動確認 | Yes | ✅ Yes |
| Discord連携 | Yes | ✅ 動作確認済 |

**成功条件**: Discord でメッセージを送り、OpenClaw が応答を返す

**失敗条件**: ARM64 非対応、メモリ不足、依存関係の問題で動作しない場合

## リサーチ

### 参考資料

- [OpenClaw 公式サイト](https://openclaw.ai/) - メインドキュメント
- Raspberry Pi 5 スペック
- Discord Bot API ドキュメント

### 発見

**OpenClaw とは:**
- 個人用 AI アシスタント（"The AI that actually does things"）
- ローカルマシンで実行、タスク自動化が可能

**対応チャットアプリ:**
- Discord ✅、WhatsApp、Telegram、Slack、Signal、iMessage

**主な機能:**
- 永続的メモリ（ユーザーの好みを記憶）
- ブラウザ制御（ウェブ操作、フォーム入力）
- システムアクセス（ファイル操作、シェルコマンド）
- スキル拡張（カスタム開発可能）

**インストール方法:**
```bash
# ワンライナー（Node.js 含む）
curl -fsSL https://openclaw.ai/install.sh | bash

# npm 経由
npm i -g openclaw

# セットアップ
openclaw onboard
```

**Raspberry Pi 対応:**
- コミュニティで「Raspberry Pi + Cloudflare」での成功報告あり
- ARM64 の明示的サポート記載なし → 要検証

## 実験ログ

### 2026-02-03 15:35 - 公式サイト調査

#### 課題/チャレンジ

OpenClaw の概要と Raspberry Pi 対応状況を確認する。

#### アクション

- 公式サイト https://openclaw.ai/ を調査

#### 生データ（事実のみ、解釈なし）

- インストール: curl または npm
- 対応 OS: macOS, Windows, Linux
- モデル: Anthropic, OpenAI, ローカルモデル対応
- Discord 対応: 明記あり
- RasPi 対応: コミュニティ報告あり（公式明記なし）

#### 分析（解釈、判断）

- Discord 連携は公式サポートされている → 目標達成可能性高い
- Raspberry Pi はコミュニティ実績あり → 試す価値あり
- Node.js ベースなので ARM64 Linux で動く可能性高い

**次のステップ:**
1. Raspberry Pi 5 の現在の状態確認（OS、Node.js）
2. OpenClaw インストール実行
3. Discord Bot 設定

---

### 2026-02-03 18:50 - RasPi5 インストール & Discord Bot 起動成功

#### 課題/チャレンジ

Raspberry Pi 5 に OpenClaw をインストールし、Discord Bot として起動する。

#### アクション

1. SSH 接続（Tailscale 経由: `shudai@100.119.151.128`）
2. Node.js バージョン確認: v22.22.0, npm 10.9.4
3. `npm i -g openclaw` でインストール
4. `openclaw onboard` でセットアップ開始
5. AI プロバイダー: Anthropic (setup-token + API key)
6. チャンネル: Discord (Bot API)
7. Discord Developer Portal で Bot 作成 & サーバー招待
8. チャンネル許可リスト: `open-claw-pi`（プライベートチャンネル）
9. TUI で Bot 起動（Hatch）

#### 生データ（事実のみ、解釈なし）

**システム情報:**
- Node.js: v22.22.0
- npm: 10.9.4
- OS: Raspberry Pi OS (ARM64)

**OpenClaw 設定:**
- Gateway service: systemd でインストール済
- Discord Bot: `@OpenClawPi` (接続 OK, 708ms)
- モデル: anthropic/claude-opus-4-5
- トークン: 0/200k (0%)
- Web UI: http://127.0.0.1:18789/

**Bot 起動確認:**
```
Wake up, my friend!
Hey! I just woke up. ✨
```
→ TUI で Bot が応答、対話開始

#### 分析（解釈、判断）

- **インストール成功**: npm 経由で問題なくインストール完了
- **ARM64 対応**: Raspberry Pi 5 (ARM64) で正常動作
- **Discord 接続**: Bot が接続状態（708ms latency）
- **TUI 動作確認**: Bot がパーソナリティ設定の対話を開始

**残タスク:**
- ~~Discord チャンネル経由での応答テスト~~ ✅ 完了
- Bot のパーソナリティ設定完了

---

### 2026-02-03 19:30 - Discord Bot 応答しない問題の調査・解決

#### 課題/チャレンジ

Discord でメッセージを送っても Bot が応答しない。

#### 生データ（事実のみ、解釈なし）

**エラーログ:**
```
channels unresolved: 1340265206592311336/1468180810975678601
discord: skipping guild message, reason: "no-mention"
Invalid config: expected record, received array
```

#### 分析（解釈、判断）

**発覚した問題:**

1. **設定ファイルの形式エラー**
   - `channels` が配列ではなくオブジェクト（record）である必要があった
   - `openclaw doctor --fix` で自動修復可能

2. **Bot の権限不足**
   - チャンネルを「見る」権限が足りなかった
   - 正しい権限で Bot を再招待する必要があった

3. **メンション必須設定**
   - デフォルトでは Bot へのメンションが必須
   - `requireMention: false` で不要にできる

4. **プライベートチャンネルの問題**
   - Bot をメンバーとして明示的に追加する必要がある
   - パブリックチャンネルの方が設定が簡単

#### 解決策

**1. Bot の再招待（正しい権限で）:**
```
https://discord.com/api/oauth2/authorize?client_id=BOT_CLIENT_ID&permissions=277025508416&scope=bot
```

付与する権限:
- View Channels
- Send Messages
- Read Message History
- Add Reactions
- Use Slash Commands

**2. 設定ファイルの修正:**
```json
"channels": {
  "discord": {
    "enabled": true,
    "guilds": {
      "SERVER_ID": {
        "channels": {
          "CHANNEL_ID": {
            "requireMention": false
          }
        }
      }
    }
  }
}
```

**3. 設定自動修復:**
```bash
openclaw doctor --fix
```

**4. Gateway 再起動:**
```bash
openclaw gateway stop
# systemd が自動再起動
```

---

### トラブルシューティング & Tips

#### 1. SSH 接続でホスト名が解決できない

**問題:**
```
ssh: Could not resolve hostname pi5: nodename nor servname provided, or not known
```

**原因:** Tailscale のホスト名がローカルで解決できない

**解決:** Tailscale IP を直接指定
```bash
ssh shudai@100.119.151.128
```

**Tips:** `~/.ssh/config` に設定を追加すると便利:
```
Host pi5
    HostName 100.119.151.128
    User shudai
```

---

#### 2. Discord Developer Portal「メールアドレスを認証する必要があります」

**問題:** Bot 作成時に「この操作を行うには、メールアドレスを認証する必要があります」と表示

**確認方法:**
1. Discord 設定 → マイアカウント → 「アカウントの状態」タブ
2. 「すべて優良です」と表示されていれば認証済み

**原因:** 実際には認証済みだったが、エラー表示が出た（キャッシュ or 一時的な問題？）

**解決:** ページをリロードして再試行 → 成功

---

#### 3. Discord Bot がサーバーに表示されない

**問題:** Bot をチャンネルに追加しようとしても、Bot が表示されない

**原因:** Bot を Discord サーバーに招待していない

**解決手順:**
1. Discord Developer Portal → OAuth2 → URL Generator
2. SCOPES: `bot` にチェック
3. BOT PERMISSIONS: 必要な権限にチェック
   - View Channels
   - Send Messages
   - Read Message History
   - Embed Links
   - Attach Files
4. 生成された URL をブラウザで開く
5. サーバーを選択して招待

---

#### 4. Discord Bot が応答しない（channels unresolved）

**問題:**
```
discord channels unresolved: SERVER_ID/CHANNEL_ID
discord: skipping guild message, reason: "no-mention"
```

**原因:**
1. Bot の権限不足（チャンネルを見れない）
2. 設定ファイルの形式エラー（配列 vs オブジェクト）
3. メンション必須設定

**解決手順:**

1. **設定自動修復:**
```bash
openclaw doctor --fix
```

2. **Bot を正しい権限で再招待:**
```
https://discord.com/api/oauth2/authorize?client_id=BOT_CLIENT_ID&permissions=277025508416&scope=bot
```

3. **設定ファイルでメンション不要に:**
```json
"channels": {
  "CHANNEL_ID": {
    "requireMention": false
  }
}
```

4. **パブリックチャンネル推奨:** プライベートより設定が簡単

---

#### 5. チャンネル許可リストの形式

**OpenClaw の allowlist 入力形式:**
```
# サーバー名/チャンネル名
My Server/#general

# チャンネル名のみ
#support

# Guild ID / Channel ID（推奨）
1234567890/9876543210
```

**チャンネル ID 取得方法:**
1. Discord 設定 → 詳細設定 → 開発者モード を有効化
2. チャンネルを右クリック → 「チャンネル ID をコピー」

---

### コマンド一覧

#### RasPi5 での実行コマンド

```bash
# Node.js バージョン確認
node --version   # v22.22.0
npm --version    # 10.9.4

# OpenClaw インストール
npm i -g openclaw

# 初期セットアップ（対話形式）
openclaw onboard

# 設定変更（後から）
openclaw configure

# Bot 起動
openclaw

# Web UI トークン付き URL 取得
openclaw dashboard --no-open
```

#### Discord Developer Portal 設定

**Bot 作成:**
1. https://discord.com/developers/applications
2. New Application → 名前入力
3. Bot → Reset Token → トークン取得（重要: 一度しか表示されない）
4. Privileged Gateway Intents → Message Content Intent 有効化

**Bot 招待 URL 生成:**
1. OAuth2 → URL Generator
2. SCOPES: `bot`
3. BOT PERMISSIONS: 必要な権限
4. URL をコピー → ブラウザで開く → サーバー選択

---

### OpenClaw onboard 選択肢メモ

```
Onboarding mode: QuickStart（推奨）

Model/auth provider:
├─ OpenAI
├─ Anthropic (setup-token + API key) ← 選択
├─ MiniMax
├─ Moonshot AI
├─ Google（Gemini 使える！）
├─ OpenRouter
├─ Qwen
├─ Z.AI (GLM 4.7)
├─ Copilot
├─ Vercel AI Gateway
├─ OpenCode Zen
├─ Xiaomi
├─ Synthetic
├─ Venice AI
└─ Skip for now

Channel:
├─ Telegram (Bot API)
├─ WhatsApp (QR link)
├─ Discord (Bot API) ← 選択
├─ Google Chat (Chat API)
├─ Slack (Socket Mode)
├─ Signal (signal-cli)
├─ iMessage (imsg)
├─ Nostr (NIP-04 DMs)
├─ Microsoft Teams (Bot Framework)
├─ Mattermost (plugin)
├─ Nextcloud Talk (self-hosted)
├─ Matrix (plugin)
├─ BlueBubbles (macOS app)
├─ LINE (Messaging API)
├─ Zalo (Bot API / Personal Account)
└─ Tlon (Urbit)

Channels access: Allowlist（推奨）
Skills: Skip for now（後で設定可）
Hooks: Skip for now（後で設定可）
Hatch: TUI（推奨）
```

---

## 結論

### 最終結果

**🦞 成功！Raspberry Pi 5 の中にカニ（OpenClaw）が住んでいる！**

OpenClaw を Raspberry Pi 5 (ARM64) にインストールし、Discord Bot として動作させることに成功した。

| 目標 | 結果 |
|------|------|
| RasPi5 へのインストール | ✅ npm 経由で問題なし |
| ARM64 対応 | ✅ 動作確認済 |
| Discord Bot 連携 | ✅ @OpenClawPi が応答 |
| 常駐動作 | ✅ systemd サービス化済 |

### 学び

**技術的な学び:**
- OpenClaw は Node.js ベースなので ARM64 Linux で問題なく動作する
- Discord Bot の設定では **Message Content Intent** の有効化が必須
- 設定ファイルの `channels` は配列ではなくオブジェクト形式
- `openclaw doctor --fix` で設定エラーを自動修復できる
- パブリックチャンネルの方がプライベートより設定が簡単

**ハマりポイント:**
1. Discord Developer Portal のメール認証エラー（実際は認証済みだった）
2. Bot をサーバーに招待し忘れ
3. Message Content Intent の有効化忘れ → Gateway error 4014
4. 設定ファイルの形式（配列 vs オブジェクト）
5. Bot の権限不足でチャンネルが見えない
6. メンション必須設定（`requireMention: false` で解除）

**プロセスの学び:**
- SSH 経由の TUI はターミナル互換性問題が起きやすい
- ラズパイ上で直接 Claude を立ち上げて調査するのが効率的だった

### 次のステップ

- [ ] Bot のパーソナリティ設定（くろぴ、日本語、皮肉で温かい仲間）
- [ ] Skills の設定（7つ使用可能）
- [ ] Hooks の設定（session-memory など）
- [ ] Cloudflare Tunnel で外部公開（オプション）
- [ ] 記事化して公開
