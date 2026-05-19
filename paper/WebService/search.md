# Webサービス論レポート 調査メモ

執筆時の参考用。文章はユーザが書く。各情報の `cite キー` は `refs.bib` に対応。

---

## Discord

### サービス概要
- テキスト・音声・ビデオチャットを提供するコミュニケーションプラットフォーム
- トップページ：https://discord.com
- 2015年創業。ゲーマー向けとして始まり、現在は一般コミュニティにも広く普及

### 定量情報

※ 一次情報源（公式ブログ・プレスリリース）のみ掲載。アグリゲーターサイト由来の数値は除外。

| 指標 | 値 | cite キー |
|---|---|---|
| DAU | 9,000万人以上（2025年Q4） | `discord-company` |
| MAU | 2億人以上（2025年4月） | `discord-ceo-2025` |
| 月間ゲームプレイ時間 | 15〜19億時間 | `discord-blog-world-plays`, `discord-ceo-2025` |
| WebSocketイベント | 2,600万件/秒 | `discord-elixir-scale` |
| 同時接続ボイスユーザー | 260万人（記事執筆時点） | `discord-voice-webrtc` |
| 音声エグレス帯域 | 220 Gbps / 120 Mpps（記事執筆時点） | `discord-voice-webrtc` |
| APIのRPS | 200万RPS以上（Rust製API） | `discord-rust-elixir` |

**一次情報が見つからなかった指標（記載しない）：**
- 登録ユーザー総数（公式の開示なし）
- 1日のメッセージ数（公式の開示なし）
- 1日の通話時間（公式の開示なし）

### アーキテクチャ・構成

#### インフラ全体
- 全サービスを **Google Cloud Platform (GCP)** 上で稼働
- CDNとDDoS防御に **Cloudflare** を使用 (`discord-cloudflare`)
  - エッジキャッシュから月2ペタバイト以上を配信
  - L3/L4/L7 の DDoS 防御を提供
  - 導入前：HAProxy を大量配置して DNS ロードバランシング

#### リアルタイム通信層（Elixir）
- **Elixir（BEAM VM）** でWebSocketゲートウェイを実装 (`discord-elixir-scale`)
  - 400〜500台のElixirマシンで秒間2,600万WebSocketイベントを処理
  - Erlang VMの並行性モデルと耐障害性を活用

#### パフォーマンスクリティカル層（Rust）
- データ移行スクリプト、NIF関数（メンバーリスト処理）、REST APIサーバをRustで実装 (`discord-rust-elixir`)
  - Rust製SortedSet NIF：最大100万件のセット操作で最良6.5倍、最悪160倍の性能改善
  - Rust製APIサーバ（Axumフレームワーク）：200万RPS以上を安定処理

#### データベース移行：MongoDB → Cassandra → ScyllaDB

**Phase 1：MongoDB（初期）** (`discord-stores-trillions`, `discord-infoq-scylladb`)
- 問題：1億件のメッセージ到達時にデータ＋インデックスがRAMに乗らなくなった
- 結果：読み取りレイテンシが急増し廃止

**Phase 2：Cassandra（2017〜2022）** (`discord-stores-trillions`, `discord-infoq-scylladb`)
- 水平スケール可能な分散DBとして採用
- ピーク時：177ノード、数兆件のメッセージを保存（ノード1台あたり4TB）
- 問題：
  - JVM（Java）のGCポーズによる予測不能なレイテンシスパイク
  - 高トラフィックチャンネルでホットパーティション問題が発生
  - クォーラム一貫性のため、1ノードの遅延がクラスタ全体に波及

**Phase 3：ScyllaDB（2022〜現在）** (`discord-stores-trillions`, `discord-infoq-scylladb`)
- **移行期間：9日間**（当初見積もり3ヶ月）
  - Rustで書いたカスタム移行サービス＋SQLiteチェックポイントで実現
  - 移行速度：秒間320万メッセージ
- **ノード数：177 → 72（60%削減）**（1ノードあたり9TBに拡大）
- **改善効果（p99レイテンシ）**：
  - メッセージ取得：40〜125ms → 15ms
  - メッセージ挿入：5〜70ms → 5ms
- ScyllaDB選定理由：C++製でGCなし、コアごとシャード設計でワークロード分離

#### 音声インフラ（WebRTC） (`discord-voice-webrtc`)
- クライアント-サーバ型（P2Pは大規模チャンネルには不向きなため不採用）
- C++製カスタムSFU（Selective Forwarding Unit）でメディアリレー
- 850台以上の音声サーバを13の地理的リージョンに配置
- ブラウザ：標準WebRTC（SDP/ICE/DTLS/SRTP）
- ネイティブアプリ：WebRTCネイティブライブラリ上のカスタムC++メディアエンジン

#### メッセージ検索インフラ (`discord-indexes-trillions`)
- Elasticsearchを40クラスタ（小規模分散）に分けて運用
- 改善後：検索中央値レイテンシ 500ms → 100ms以下、p99が1s → 500ms以下

---

## GitHub

### サービス概要
- Gitリポジトリのホスティング、コードレビュー、CI/CDを提供するソフトウェア開発プラットフォーム
- トップページ：https://github.com
- 2008年創業。現在はMicrosoftの子会社。世界最大のソースコード管理基盤

### 定量情報

| 指標 | 値 | cite キー |
|---|---|---|
| 登録開発者数 | 1.8億人以上（毎秒1人が新規参加） | `github-octoverse-2025` |
| リポジトリ総数 | 6.3億件 | `github-octoverse-2025` |
| うち公開リポジトリ | 3.95億件 | `github-octoverse-2025` |
| 2025年の新規PR（マージ済み） | 5.19億件 | `github-octoverse-2025` |
| 2025年のコミット数 | 9.86億件（前年比+25%） | `github-octoverse-2025` |
| DBホスト数 | 1,200台以上 | `github-mysql8` |
| 保存データ量 | 300TB超 | `github-mysql8` |
| クエリ数 | 550万クエリ/秒 | `github-mysql8` |
| DBクラスタ数 | 50以上 | `github-mysql8` |

### アーキテクチャ・構成

#### CDN：Fastly (`github-fastly`)
- github.com・GitHub Pages・raw.github.com の静的アセットを Fastly で配信
- Fastly 導入前：1時間ごとに大量リクエストのスパイクが発生
- 導入後：リクエストが平滑化され、バックエンドへの負荷が大幅に軽減

#### ロードバランサ：GLB（GitHub Load Balancer） (`github-haproxy`)
- HAProxy を基盤とするカスタム実装
- **L4（TCP）と L7（アプリ層）を分離した設計**
- ECMP（Equal-Cost Multi-Path）により複数マシンに同一IPを分散
- Git・SSH・MySQLクライアントのリクエストをルーティング
- Consul + Consul-Template + Kube Service Exporter で動的設定管理

#### データベース：MySQL + Vitess によるシャーディング (`github-partitioning-db`, `github-mysql8`)
- 全DBを MySQL で運用（1,200台以上のホスト、50以上のクラスタ）
- スケールのために **Vitess** を導入
  - **垂直分割**：リポジトリ・Issue・PRなどコアテーブル130本を一括移行
  - **水平シャーディング**：無停止でシャード分割・統合が可能
- 主要+レプリカ構成で高可用性を確保
- 2021年時点：クエリ数120万/秒（レプリカ112.5万、プライマリ7.5万）
  - 2019年比で DBホスト負荷を50%削減

#### Spokes：Git分散レプリケーションシステム (`github-spokes-stretching`, `github-spokes-resilience`)
- Gitリポジトリを **3コピー**保持（物理ラック単位で分離し相関障害を防止）
- **読み取り**：最近傍の同期済みレプリカへルーティング（読み書き比 ≈ 100:1）
- **書き込み**：三相コミットプロトコルでアトミック更新
- **整合性保証**：過半数のレプリカへの適用成功を確認後にコミット。失敗した側はリバート
- 障害検出：3リクエスト失敗で unhealthy 判定、修復完了まで除外

#### GitHub Actions のスケール
- **Actions Runner Controller (ARC)**：Kubernetes上でランナーをオートスケール
- **Ephemeral ランナー**：1ジョブ実行ごとにランナーを破棄・再生成
- webhookイベント（`workflow_job`）ドリブンでスケールアウト

#### eBPFによるデプロイ安全性 (`github-ebpf`)
- デプロイ時の循環依存を検出・防止するために eBPF を活用
- `BPF_PROG_TYPE_CGROUP_SKB` でデプロイcGroupのネットワーク出力を監視・制御

---

## 参考文献キー早見表

| cite キー | タイトル（抜粋） | 種別 |
|---|---|---|
| `discord-company` | Discord Company Page（DAU 9,000万+） | Discord公式 |
| `discord-ceo-2025` | Discord Appoints New CEO（MAU 2億+） | Discord公式プレスリリース |
| `discord-blog-world-plays` | Come Build Where the World Plays | Discord公式ブログ |
| `discord-stores-trillions` | How Discord Stores Trillions of Messages | Discord公式ブログ |
| `discord-rust-elixir` | Using Rust to Scale Elixir for 11M Concurrent Users | Discord公式ブログ |
| `discord-indexes-trillions` | How Discord Indexes Trillions of Messages | Discord公式ブログ |
| `discord-voice-webrtc` | How Discord Handles 2.5M Concurrent Voice Users | Discord公式ブログ |
| `discord-elixir-scale` | Real Time Communication at Scale with Elixir | Elixir公式ブログ |
| `discord-cloudflare` | Discord Case Study | Cloudflare（ベンダー） |
| `discord-infoq-scylladb` | Discord Migrated Trillions of Messages to ScyllaDB | InfoQ |
| `github-octoverse-2025` | Octoverse 2025 | GitHub公式 |
| `github-mysql8` | Upgrading GitHub.com to MySQL 8.0 | GitHub Engineeringブログ |
| `github-partitioning-db` | Partitioning GitHub's Relational Databases | GitHub Engineeringブログ |
| `github-spokes-stretching` | Stretching Spokes | GitHub Engineeringブログ |
| `github-spokes-resilience` | Building Resilience in Spokes | GitHub Engineeringブログ |
| `github-ebpf` | How GitHub Uses eBPF to Improve Deployment Safety | GitHub Engineeringブログ |
| `github-fastly` | GitHub Case Study | Fastly（ベンダー） |
| `github-haproxy` | Inside the GitHub Load Balancer | HAProxyConf 2019 |
