# iOS Vision Framework セグメンテーションAPI比較検証

**日付**: 2026-02-12
**プロジェクト**: EventCardMaker (ios-cardMaker)
**タグ**: #iOS #Vision #セグメンテーション #人物切り抜き #パフォーマンス

## 概要

iOSのVision frameworkで利用可能な人物セグメンテーションAPIを、リアルタイムカメラプレビューで実機比較検証した。

## 検証環境

- **端末**: iPhone SE 第3世代（A15 Bionic）
- **iOS**: 17+
- **計測方法**: アプリ内FPSカウンター（1秒間のフレーム数を実測）

## 検証したAPI

### 1. VNGeneratePersonSegmentationRequest (iOS 15+)

人物専用のセグメンテーション。3段階の精度モードあり。グレースケールマスク（0.0〜1.0の確信度）を返す。

### 2. VNGenerateForegroundInstanceMaskRequest (iOS 17+)

WWDC23で発表。写真アプリの「被写体を持ち上げ」と同じ技術。人物以外の被写体も対応。インスタンス単位で分離可能。

## 結果

### パフォーマンス比較（iPhone SE3 / A15 Bionic）

| API | モード | FPS | 精度 |
|-----|--------|-----|------|
| PersonSegmentation | `.fast` | 30+ | △ 誤検出あり（人物以外も拾う） |
| PersonSegmentation | `.balanced` | **26** | ○ 実用的 |
| PersonSegmentation | `.accurate` | **11** | ◎ 高精度 |
| **ForegroundInstanceMask** | — | **23** | **◎◎ 最高精度** |

### 精度の詳細

#### PersonSegmentation
- `.fast`: 人物以外のオブジェクトも切り抜いてしまうことがある
- `.balanced`: 人物認識は安定するが、髪の毛のエッジがやや甘い
- `.accurate`: 髪の毛のラインも比較的綺麗だが、FPS 11はリアルタイム用途では厳しい場面あり

#### ForegroundInstanceMask
- 精度はPersonSegmentation `.accurate` 以上
- 髪の毛・エッジの処理が非常に綺麗
- **マスクがほぼバイナリ（ON/OFF）で返る**ため、閾値調整の必要がない
- Neural Engineに最適化されており、新しいAPIにも関わらず高速

### 閾値（Threshold）について

PersonSegmentationはグレースケールマスクのため、閾値でエッジの厳しさを調整可能：

| 閾値 | 効果 |
|------|------|
| 0.5 | 甘め、髪の毛の先まで残る、誤検出も残りやすい |
| 0.9 | 厳しめ、確実な部分だけ、エッジがシャープ |
| 0.95 | かなり厳しめでも精度良好（推奨値） |

**ForegroundInstanceMaskは閾値不要**（自動最適化済み）。

### シャッター撮影時の注意

- リアルタイム処理と撮影処理を**同一キューで実行するとフリーズする**
- 撮影処理は別のDispatchQueueに分離すべき
- 特にaccurateモードでは処理が重く、カメラフレーム更新が止まる

## 結論

**ForegroundInstanceMaskが全面的に優秀**:
- 精度: 最高（PersonSegmentation accurate以上）
- 速度: 23fps（balanced 26fpsに迫る）
- 使いやすさ: 閾値調整不要
- 対応: iOS 17+が必須だが、イベント用途なら端末指定可能

**実用推奨構成**:
- プレビュー: `ForegroundInstanceMaskRequest`（23fps、最高精度）
- 撮影時: `ForegroundInstanceMaskRequest`（同一API、1枚処理）

PersonSegmentationの出番は、iOS 15-16対応が必要な場合のみ。

## 備考

- iPhone 15 Pro (A17 Pro) ではさらに高速が期待される
- ForegroundInstanceMaskは人物以外（物体）も切り抜ける汎用性がある
- リアルタイムプレビューで23fps出るのは十分スムーズ（映像感）

## 次のステップ

- [ ] iPhone 15 Proでの実測比較
- [ ] 複数人同時切り抜きのパフォーマンス
- [ ] 照明条件による精度変化の検証（暗所、逆光）
- [ ] カード合成時の色味なじませ処理の最適化

---

*このメモは研究開発の記録です。記事化の素材として使用できます。*
