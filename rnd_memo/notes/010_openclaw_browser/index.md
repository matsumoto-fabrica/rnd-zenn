---
created: 2026-02-12 18:45
status: done
prev: 001_initial_setup
next: []
---

# OpenClaw Browser — Pi5でAIがブラウザを操る

## 仮説

OpenClaw の Browser 機能を使えば、AI エージェントが自律的にWebページを閲覧・操作できる。Raspberry Pi 5（headless）でも動作し、さらにノード経由でMac/Windowsのブラウザもリモート操作できる。

## 評価基準

| 指標 | 目標値 | 現在値 |
|------|--------|--------|
| Pi5で動作 | Yes | ✅ Yes |
| スクショ取得 | Yes | ✅ Yes |
| ページ操作（クリック/入力） | Yes | ✅ Yes |
| ノード経由Mac操作 | Yes | ✅ Yes |

## リサーチ

### OpenClaw Browser とは

**2つのモード:**

1. **openclaw プロファイル（隔離ブラウザ）**
   - OpenClawが管理する専用のChromium
   - エージェントがタブ操作、クリック、入力、スクショ、スナップショット可能
   - 普段使いのブラウザとは完全に分離

2. **chrome プロファイル（Chrome拡張リレー）**
   - 既存のChromeにMV3拡張を入れて、そのタブを操作
   - ツールバーボタンでON/OFF切り替え
   - `chrome.debugger` API経由でCDPメッセージをリレー

### Playwright との関係

**対立するものではなく、内部エンジンとして使っている。**

- **Playwright 単体**: コード書いて動かす自動化ライブラリ（テスト/スクレイピング向け）
- **OpenClaw Browser**: Playwrightの上に「AIエージェント用インターフェース」を被せたもの

具体的な動き:
1. `snapshot` → ページのUI構造をAIが読めるテキストに変換
2. AI が「ref=12のボタンをクリック」と判断
3. 裏で Playwright が実行

**つまり: Playwright = エンジン、OpenClaw Browser = AIドライバー**

Playwrightだけだと自分でコード書く。OpenClaw Browserだとエージェントに「このサイトで○○して」と言うだけ。

### プラットフォーム対応

| プラットフォーム | 対応状況 | 備考 |
|------------------|----------|------|
| macOS | ✅ フル対応 | Chrome/Brave/Edge自動検出 |
| Windows | ✅ 対応 | executablePath指定で動作 |
| Pi5 (ARM64 Linux) | ✅ 条件付き | headless + noSandbox必要 |

**Pi5の条件:**
- `headless: true`（モニタなし環境）
- `noSandbox: true`（ARM64 Linuxのサンドボックス問題回避）
- snap版Chromium NG → apt版Chromiumを使う
- Google Chrome Linux版はamd64のみ → Chromiumが正解

## 実験ログ

### 2026-02-12 18:50 - 環境確認

#### 生データ

- Pi5の `playwright-core@1.58.2` はOpenClawに同梱済み
- フル版 `playwright` は未インストール → `npm install -g playwright@1.58.2` で追加
- Chromium: `/usr/bin/chromium-browser` (v144.0.7559.59, apt版)

### 2026-02-12 18:55 - Pi5ローカルでの動作テスト

#### アクション

1. Gateway configに browser設定を追加:
```json
{
  "browser": {
    "enabled": true,
    "headless": true,
    "noSandbox": true,
    "defaultProfile": "openclaw",
    "executablePath": "/usr/bin/chromium-browser"
  }
}
```
2. Gateway再起動（SIGUSR1）
3. example.com → スナップショット取得成功
4. NHKニュース → スクリーンショット取得成功
5. Yahooニュース → スクリーンショット取得成功

#### 生データ

```
profile: openclaw
enabled: true
running: true
cdpReady: true
headless: true
noSandbox: true
detectedBrowser: custom
executablePath: /usr/bin/chromium-browser
```

スナップショット出力例（example.com）:
```
- heading "Example Domain" [level=1] [ref=e3]
- paragraph [ref=e4]: This domain is for use in...
- link "Learn more" [ref=e6]
```
→ AIが読める構造化テキストに変換されている

### 2026-02-12 19:00 - ノード自動ルーティングの発見

#### 課題

Pi5で動かしたつもりが、fab-macbook-proでブラウザが開いた。

#### 原因

fab-macbook-proがOpenClawノードとして接続中で、`browser.proxy` capabilityを持っていた:
```
caps: ["browser", "system"]
commands: ["browser.proxy", ...]
```

OpenClawのデフォルト動作:
> ブラウザ対応ノードが接続されていると、自動的にそちらにルーティングする

#### 解決

- `target="host"` 指定 → Pi5ローカルで実行される
- `target` 指定なし（デフォルト） → ノードに自動ルーティング

### 2026-02-12 19:10 - デフォルトをPi5に固定

#### アクション

自動ルーティングをオフに設定:
```json
{
  "gateway": {
    "nodes": {
      "browser": {
        "mode": "off"
      }
    }
  }
}
```

#### 結果

- デフォルト → Pi5のheadless Chromium
- Macで見たい時 → `target="node"` で明示指定

## 結論

### 最終結果

**✅ Pi5でOpenClaw Browser完全動作！さらにノード経由でMacのブラウザもリモート操作可能。**

### 最終構成

```
┌─────────────────────────────────────────┐
│  Pi5 (Gateway)                          │
│  ├─ headless Chromium (デフォルト)       │
│  └─ browser tool → AI が自律操作        │
└────────────┬────────────────────────────┘
             │ Tailscale (target="node")
┌────────────▼────────────────────────────┐
│  Mac/Win (ノード)                       │
│  └─ 画面付きブラウザ（必要時のみ）       │
└─────────────────────────────────────────┘
```

### 学び

**技術的な学び:**
- `playwright-core` だけでは不十分、フル版 `playwright` が必要
- Pi5のapt版Chromiumなら問題なし（snap版はNG）
- `headless: true` + `noSandbox: true` がPi5では必須
- ノード自動ルーティングは便利だが、意図しない挙動の原因にもなる
- `gateway.nodes.browser.mode: "off"` で制御可能

**ユースケース:**
- Webスクレイピング（web_fetchで取れないJS描画サイト）
- サイトの目視確認（スクショでDiscordに送信）
- フォーム入力・ログイン操作の自動化
- 定期的なWeb監視

### コマンドまとめ

```bash
# ステータス確認
openclaw browser --browser-profile openclaw status

# URL を開く
openclaw browser --browser-profile openclaw open https://example.com

# スクリーンショット
openclaw browser --browser-profile openclaw screenshot

# スナップショット（AI用テキスト）
openclaw browser --browser-profile openclaw snapshot

# タブ一覧
openclaw browser --browser-profile openclaw tabs
```
