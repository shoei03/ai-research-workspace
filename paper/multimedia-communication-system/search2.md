# マルチメディアコミュニケーションシステム 課題2 調査メモ

執筆時の参考用。**文章はユーザが書く。** 各情報の `cite キー` は `refs.bib` に対応。

**一次情報主義**：以下の事実はすべて RFC（IETF）または Opus 公式（Xiph.Org / opus-codec.org）、または対応サービスの公式ヘルプから直接確認した内容のみを記載。確認できなかった項目は「公式記載なし」と明記する。

### 📌 このメモの読み方（検証手順）

各引用ブロックの直下に、以下を必ず併記する：
- **🔗 直リンク**：その節までジャンプできる URL（RFC は `#section-X.Y` アンカーで章節へ直接飛べる）
- **🔍 Ctrl+F**：そのページ内で1発検索できる**短く一意な英語キーワード**

ブラウザで URL を開いた直後に `Ctrl+F`（Mac は `Cmd+F`）でキーワードを貼り付ければ、該当箇所がハイライトされる。コピペは「`...`」内の英語だけにすると確実（日本語/全角記号を含めると検索失敗の原因になる）。

---

## 課題の要件

> 身の回りで実際に使用されている Audio CODEC を1つ（講義で紹介したものを除く）あげ、
> 1. どのような場面で利用されているCODECか
> 2. 標本化周波数，量子化ビット数はどれくらいか
> 3. そのCODECは具体的にどのようなアルゴリズムを採用しているか
> について調べる。
>
> 講義既出：G.711 / G.722.2 (AMR-WB) / G.723.1 / G.726 / G.729 / EVS / CELP

---

## 選定：Opus

- 公式サイト：https://opus-codec.org/
- 規格本体：IETF RFC 6716（2012 年 9 月、Proposed Standard） `opus-rfc6716`
- 更新：RFC 8251（2017 年 10 月） `opus-rfc8251`
- RTP ペイロード仕様：RFC 7587（2015 年 6 月） `opus-rfc7587`
- 講義で紹介された CODEC（G.711 等）に含まれない、現代のリアルタイム音声・音楽配信で広く使われる汎用コーデック。

---

## 1. 利用場面

### (a) RFC 6716 Abstract（規格自身が想定する場面）

> "This document defines the Opus interactive speech and audio codec. Opus is designed to handle a wide range of interactive audio applications, including Voice over IP, videoconferencing, in-game chat, and even live, distributed music performances."
> — RFC 6716 Abstract（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716（ページ冒頭 Abstract）
- 🔍 Ctrl+F: `interactive speech and audio codec`

opus-codec.org も同一の利用場面を公式に掲げている：
> "Voice over IP, videoconferencing, in-game chat, and even remote live music performances"
> — opus-codec.org トップページ（`opus-codec-site`）

- 🔗 https://opus-codec.org/
- 🔍 Ctrl+F: `remote live music performances`

### (b) WebRTC で「実装必須（REQUIRED）」（身の回りでの普及の根拠）

RFC 7874 "WebRTC Audio Codec and Processing Requirements"（2016 年 5 月、IETF）§3 において、Opus は WebRTC エンドポイントに実装が **REQUIRED**（実装必須）と規定されている：

> "WebRTC endpoints are REQUIRED to implement the following audio codecs: o Opus [RFC6716] with the payload format specified in [RFC7587]."
> — RFC 7874 §3（`webrtc-rfc7874`）

- 🔗 https://www.rfc-editor.org/rfc/rfc7874#section-3
- 🔍 Ctrl+F: `REQUIRED to implement the following audio codecs`

⇒ 現代のブラウザ（Chrome / Firefox / Safari / Edge）で動作する Web 会議系サービス（WebRTC 経由）では、音声コーデックとして Opus が標準的に利用される。

### (c) YouTube 推奨アップロード音声コーデック

Google 公式の YouTube ヘルプ「動画のアップロード推奨エンコード設定」で、推奨音声コーデックとして Opus が明記されている：

> "Audio codec: AAC-LC or Opus or Eclipsa Audio"
> — Google 公式ヘルプ（`youtube-recommended-settings`）

- 🔗 https://support.google.com/youtube/answer/1722171
- 🔍 Ctrl+F: `AAC-LC or Opus`
- 補足：英語表示のページで上記英文がヒットする。日本語表示の場合は `Opus` か `音声コーデック` で検索

### (d) その他のサービス（公式裏取りできず）

- Discord / WhatsApp / Zoom / Microsoft Teams 等についても Opus 採用が広く言われているが、本調査で**公式の一次資料で明言を確認できなかった**ため本メモには記載しない（必要なら別途公式技術ドキュメント・公式ブログを再調査の上で追加）。

---

## 2. 標本化周波数・量子化ビット数

### 標本化周波数（RFC 6716 §2 Table 1）

| 略称 | 名称 | オーディオ帯域 | サンプリング周波数 |
|---|---|---|---|
| NB | narrowband | 4 kHz | **8 kHz** |
| MB | medium-band | 6 kHz | **12 kHz** |
| WB | wideband | 8 kHz | **16 kHz** |
| SWB | super-wideband | 12 kHz | **24 kHz** |
| FB | fullband | 20 kHz | **48 kHz** |

- 出典：RFC 6716 §2（`opus-rfc6716`）
- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2
- 🔍 Ctrl+F: `super-wideband` で Table 1 にジャンプ可（他に出現箇所がほぼない一意な語）
- 副典：opus-codec.org トップ「Sampling rates from 8 kHz (narrowband) to 48 kHz (fullband)」（`opus-codec-site`）
  - 🔗 https://opus-codec.org/
  - 🔍 Ctrl+F: `Sampling rates from 8 kHz`

### ビットレート

> "Opus supports all bitrates from 6 kbit/s to 510 kbit/s."
> — RFC 6716 §2.1.1（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2.1.1
- 🔍 Ctrl+F: `6 kbit/s to 510 kbit/s`

opus-codec.org も同値：「Bitrates from 6 kb/s to 510 kb/s」（`opus-codec-site`）。
- 🔗 https://opus-codec.org/
- 🔍 Ctrl+F: `6 kb/s to 510 kb/s`

### フレーム長

> "Opus can encode frames of 2.5, 5, 10, 20, 40, or 60 ms. It can also combine multiple frames into packets of up to 120 ms."
> — RFC 6716 §2.1.4（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2.1.4
- 🔍 Ctrl+F: `2.5, 5, 10, 20, 40, or 60 ms`

### チャネル

> "Opus can transmit either mono or stereo frames within a single stream"
> — RFC 6716 §2.1.2（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2.1.2
- 🔍 Ctrl+F: `mono or stereo frames`

### 量子化ビット数（入力 PCM 側）

RFC 6716 本文では入力 PCM のビット数を1値で固定する記述はない。一方で、Opus の **公式リファレンス実装（libopus）の C API** は次のとおり 16-bit 符号付き PCM (`opus_int16`) を主要入力として規定している：

```c
opus_int32 opus_encode(OpusEncoder *st,
                       const opus_int16 *pcm,
                       int frame_size,
                       unsigned char *data,
                       opus_int32 max_data_bytes);
```
> "Input signal (interleaved if 2 channels). length is `frame_size*channels*sizeof(opus_int16)`"
> — Opus 公式 API ドキュメント `opus_encode()` の `pcm` 引数説明（`opus-api-encoder`）

- 🔗 https://opus-codec.org/docs/opus_api-1.5/group__opus__encoder.html
- 🔍 Ctrl+F: `opus_int16` （または `Input signal (interleaved`）

浮動小数版 `opus_encode_float()`（±1.0 正規化の `float`）も同ページにある。

⇒ 報告では「**入力標本は通常 16 bit 符号付き PCM（公式 API 仕様）**」と書くのが安全。

---

## 3. 採用アルゴリズム

### ハイブリッド構造：SILK（LP）+ CELT（MDCT）

Opus はリニア予測符号化に基づく **SILK 層** と、変換符号化に基づく **CELT 層** を組み合わせたハイブリッド構造。

> "The LP layer is based on the SILK codec [SILK]. The version of SILK used in Opus is substantially modified from, and not compatible with, the stand-alone SILK codec previously deployed by Skype."
> — RFC 6716 §2（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2
- 🔍 Ctrl+F: `previously deployed by Skype`

> "The MDCT layer is based on the Constrained-Energy Lapped Transform (CELT) codec [CELT]."
> — RFC 6716 §2（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2
- 🔍 Ctrl+F: `Constrained-Energy Lapped Transform`

### 3 つの動作モード（RFC 6716 §3.1 Table 2）

| モード | Config 番号 | 対応帯域 | フレーム長 | 主用途 |
|---|---|---|---|---|
| **SILK-only** | 0–11 | NB / MB / WB | 10, 20, 40, 60 ms | 音声（低～中ビットレート） |
| **Hybrid（SILK + CELT）** | 12–15 | SWB / FB | 10, 20 ms | 広帯域音声 |
| **CELT-only** | 16–31 | NB / WB / SWB / FB | 2.5, 5, 10, 20 ms | 音楽・低遅延用途 |

- 出典：RFC 6716 §3.1 Table 2（`opus-rfc6716`）
- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-3.1
- 🔍 Ctrl+F: `Table 2: TOC byte configuration parameters`

### ハイブリッドモードの帯域分担

> "The cutoff between the two lies at 8 kHz, the maximum WB audio bandwidth. In the MDCT layer, all bands below 8 kHz are discarded, so there is no coding redundancy between the two layers."
> — RFC 6716 §2（`opus-rfc6716`）

- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2
- 🔍 Ctrl+F: `no coding redundancy`

⇒ 8 kHz 以下を SILK（LP）、8 kHz 以上を CELT（MDCT）が担当し、冗長性を回避。

### SILK 層の構成要素（RFC 6716 §4.2 SILK Decoder）

| 要素 | 該当節 | 直リンク・Ctrl+F |
|---|---|---|
| LPC 合成フィルタ | §4.2.7.9.2 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.2.7.9.2 ／ 🔍 `LPC Synthesis Filter` |
| LTP（長期予測）フィルタ | §4.2.7.6 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.2.7.6 ／ 🔍 `Long-Term Prediction` |
| ノイズシェイピング（複雑度議論） | §2.1.5 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-2.1.5 ／ 🔍 `noise shaping` |
| ピッチ予測・励振量子化・サブフレーム | §4.2 全体 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.2 ／ 🔍 `SILK Decoder` |

### CELT 層の構成要素（RFC 6716 §4.3 CELT Decoder）

| 要素 | 該当節 | 直リンク・Ctrl+F |
|---|---|---|
| MDCT に基づく変換符号化 | §4.3 冒頭 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.3 ／ 🔍 `CELT Decoder` |
| 帯域単位の Bit Allocation | §4.3.3 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.3.3 ／ 🔍 `Bit Allocation` |
| 球面ベクトル量子化（PVQ）：形状デコード | §4.3.4 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.3.4 ／ 🔍 `Shape Decoder` |
| PVQ：エンコーダ側 | §5.3.8 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-5.3.8 ／ 🔍 `Spherical Vector Quantization` |
| Anti-collapse 処理 | §4.3.5 | 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-4.3.5 ／ 🔍 `Anti-Collapse` |

### 符号化／パケット構成（RFC 6716 §3）

- 1〜複数フレームを1パケットに束ね、TOC（Table of Contents）バイトでモード／帯域／フレーム数を表現。
- 🔗 https://www.rfc-editor.org/rfc/rfc6716#section-3.1
- 🔍 Ctrl+F: `TOC byte`

---

## 4. ライセンス／開発主体

- ライセンス：**BSD 3-clause**（特許含めロイヤリティフリーで配布）。opus-codec.org トップページ：「Opus is a totally open, royalty-free, highly versatile audio codec.」（`opus-codec-site`）
  - 🔗 https://opus-codec.org/
  - 🔍 Ctrl+F: `totally open, royalty-free`
- 著者（RFC 6716）：JM. Valin（Mozilla）、K. Vos（Skype）、T. Terriberry（Mozilla）。
  - 🔗 https://www.rfc-editor.org/rfc/rfc6716（ページ最上部の "Authors:" 行）
  - 🔍 Ctrl+F: `J-M. Valin` または `Mozilla Corporation`

---

## 参考文献キー早見表

| cite キー | 内容 | 種別 | 直リンク |
|---|---|---|---|
| `opus-rfc6716` | RFC 6716 Definition of the Opus Audio Codec | IETF Standards Track | https://www.rfc-editor.org/rfc/rfc6716 |
| `opus-rfc8251` | RFC 8251 Updates to the Opus Audio Codec | IETF Standards Track | https://www.rfc-editor.org/rfc/rfc8251 |
| `opus-rfc7587` | RFC 7587 RTP Payload Format for the Opus Speech and Audio Codec | IETF Standards Track | https://www.rfc-editor.org/rfc/rfc7587 |
| `webrtc-rfc7874` | RFC 7874 WebRTC Audio Codec and Processing Requirements | IETF Standards Track | https://www.rfc-editor.org/rfc/rfc7874 |
| `opus-codec-site` | Opus Codec 公式トップページ | Xiph.Org 公式 | https://opus-codec.org/ |
| `opus-api-encoder` | Opus 1.5 公式 API ドキュメント（opus_encoder） | Xiph.Org 公式 | https://opus-codec.org/docs/opus_api-1.5/group__opus__encoder.html |
| `youtube-recommended-settings` | YouTube ヘルプ「動画のアップロード推奨エンコード設定」 | Google 公式ヘルプ | https://support.google.com/youtube/answer/1722171 |

---

## refs.bib への反映状況

上記7エントリは **すでに `paper/multimedia-communication-system/refs.bib` に追記済み**（既存の課題1エントリは保持）。BibTeX 本体は同ファイルを参照。

---

## 補足：本調査で**確認できなかった**事項（記載しない）

- Discord / Zoom / Microsoft Teams / WhatsApp 等が Opus を採用しているとの公式技術文書は本調査では一次情報として確定できなかった。「使われている」と書く場合は、それぞれの公式技術ブログ・公式仕様ページを別途確認のうえで追記すること。
- RFC 6716 §6（Conformance）には入力 PCM ビット深度の規定がない。「16 bit」と書く場合は **公式 C API（opus_int16）** を根拠とするのが正確。
