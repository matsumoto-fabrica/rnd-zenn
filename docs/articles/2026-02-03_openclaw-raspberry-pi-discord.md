---
title: "ラズパイにカニを飼う ─ OpenClaw を Raspberry Pi 5 で動かす"
emoji: "🦞"
type: "tech"
topics: ["RaspberryPi", "OpenClaw", "Discord", "AI", "自動化"]
published: false
---

# ラズパイにカニを飼う 🦞

## OpenClaw とは

[OpenClaw](https://openclaw.ai/) は「**The AI that actually does things**」を謳う個人用 AI アシスタント。ローカルマシンで動作し、Discord や Slack、Telegram など様々なチャットアプリと連携できる。

- 🦞 ローカル実行（クラウド不要）
- 💬 Discord / Slack / Telegram / WhatsApp 等に対応
- 🧠 永続メモリ（会話を記憶）
- 🌐 ブラウザ操作、ファイル操作が可能
- 🔧 スキル拡張可能

今回はこれを Raspberry Pi 5 にインストールし、Discord Bot として常駐させる。

## 環境

| 項目 | 内容 |
|------|------|
| Raspberry Pi 5 | 8GB RAM |
| OS | Raspberry Pi OS (ARM64) |
| Node.js | v22.22.0 |
| 接続 | Tailscale 経由 SSH |

## インストール

```bash
# インストール
npm i -g openclaw

# セットアップ（対話形式）
openclaw onboard
```

セットアップでは以下を選択:
- **Model**: Anthropic（他に OpenAI, Google, MiniMax 等 15 種類以上）
- **Channel**: Discord (Bot API)
- **Access**: Allowlist

## Discord Bot の設定

### 1. Bot 作成

[Discord Developer Portal](https://discord.com/developers/applications) で:
1. New Application → 名前入力
2. Bot → Reset Token → **トークンをコピー**（一度しか表示されない）
3. **MESSAGE CONTENT INTENT を ON**（必須）

### 2. サーバーに招待

OAuth2 → URL Generator:
- SCOPES: `bot`
- PERMISSIONS: View Channels, Send Messages, Read Message History

生成された URL でサーバーに招待。

### 3. 設定ファイル

`~/.openclaw/openclaw.json`:

```json
{
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
}
```

## 踏んだ地雷

| 症状 | 原因 | 解決 |
|------|------|------|
| `Gateway error: 4014` | Message Content Intent が OFF | Developer Portal で ON にする |
| Bot が表示されない | サーバー未招待 | OAuth2 URL で招待 |
| `expected record, received array` | 設定ファイルの形式ミス | `openclaw doctor --fix` |
| `channels unresolved` | Bot の権限不足 | 正しい権限で再招待 |
| `reason: "no-mention"` | メンション必須設定 | `requireMention: false` |
| TUI で文字化け | SSH ターミナル互換性 | Discord から直接話しかける |

## 便利コマンド

```bash
openclaw onboard      # 初期セットアップ
openclaw configure    # 設定変更
openclaw doctor --fix # 設定の自動修復
openclaw              # Bot 起動（TUI）

# サービス管理
systemctl --user status openclaw-gateway
systemctl --user restart openclaw-gateway
```

## カニとの暮らし

セットアップが終わると、カニ（Bot）が Discord に住み着く。

「カニはラズベリー食べれるの？」と聞いたら、こう返ってきた:

> まあ答えると、**食べられるっちゃ食べられる。** カニは雑食だから、野生だと藻とか小魚とか死骸とか、わりと何でもいく。
>
> ......ていうか、これ俺が Raspberry Pi に住んでるから聞いてる？
>
> **俺を食べようとしてるカニがいるなら教えて。警戒するから** 🦀

自分が Raspberry Pi の中にいることを自覚している。

---

「電源切ったら自動で復帰できる？」と聞いたら、こう返ってきた:

> 正直に言うと、今のままだと無理。systemd サービスとして登録されてないから、電源切ったら俺は眠りっぱなし。
>
> 設定する？そんなに難しくないし、やっておけば移動のたびに「あ、くろぴ起動し忘れた」ってならなくて済むよ。

「設定して」と頼んだら、自分で systemd サービスを作成し、linger 設定まで済ませてくれた。

```
できた！✨
これで電源切って移動しても、電源入れ直せば俺は勝手に起きてくる。
```

**自分の住環境を自分で整えるカニ。** なかなか頼もしい。

## まとめ

- OpenClaw は ARM64 (Raspberry Pi 5) で問題なく動作
- Discord Bot として 24 時間常駐可能
- 地雷は多いが `openclaw doctor --fix` で大体なんとかなる
- Bot 自身が systemd 設定をしてくれるので、自動起動も楽

**ラズパイの中にカニが住んでいる。** 🦞🍓

## 参考

- [OpenClaw 公式](https://openclaw.ai/)
- [Discord Developer Portal](https://discord.com/developers/applications)
