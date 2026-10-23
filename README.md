# DBRE-workspace

DBRE（Database Reliability Engineering）に関連した学習をするための作業場。

DBA としての経験をベースに、SLI/SLO の設計・監視・障害対応・IaC といった「DB を信頼性の観点から運用する」スキルを身につけることを目的とする。

学習の進め方と現在地は [`docs/roadmap.md`](./docs/roadmap.md) を参照。

---

## このリポジトリの位置づけ

実際の監視の実装は、個人開発中の Web サービス **[PhysiFun](https://github.com/Physicalfunding/PhysiFun)** のリポジトリで行う。このリポジトリには、サービスに依存しない学習・実験・演習の記録を置く。

### 棲み分けの判断基準

> **「PhysiFun がなくなっても価値があるか？」**
>
> - ない → PhysiFun に置く（そのサービスを運用するためのもの）
> - ある → このリポジトリに置く（DBRE としての知識・経験そのもの）

もうひとつの基準として、**PhysiFun は「壊してはいけない場所」、このリポジトリは「壊していい場所」** とする。負荷試験や障害注入などの実験はすべてこちらで行う。

| 置くもの | PhysiFun | DBRE-workspace |
| --- | --- | --- |
| 計測コード（OTel / カスタムスパン） | ✅ | 検証用の最小構成のみ |
| Supabase メトリクスの Collector 設定 | ✅ | — |
| SLO | 確定版 | 検討過程・却下した案 |
| アラート・ダッシュボード定義 | ✅ | 実験用 |
| Runbook | ✅ | — |
| ポストモーテム | 実際のインシデント | 演習（架空のシナリオ） |
| 学習ノート | — | ✅ |
| 負荷試験・障害注入の環境 | — | ✅ |
| AWS（Aurora + CDK）の演習 | — | ✅ |

### 2 つのリポジトリの回し方

1. **このリポジトリで試す**（実験・検討）
2. **固まったら PhysiFun に PR を出す**（検討ログから PR へリンクを貼る）
3. **PhysiFun で運用して気づいたことを、このリポジトリに振り返りとして書く**

---

## ディレクトリ構成

```
DBRE-workspace/
├── README.md
├── docs/
│   ├── roadmap.md        # 学習ロードマップと進捗
│   ├── notes/            # 学習ノート（書籍・記事の要点、用語整理）
│   ├── design-logs/      # 設計の検討ログ（なぜそうしたか、比較した案）
│   ├── drills/           # 障害演習の記録
│   └── postmortems/      # 演習のポストモーテム
├── sandbox/              # ローカル実験環境（Docker）
└── aws-cdk/              # Aurora + CloudWatch の構築と演習
```

---

## 使うツール（無料の範囲）

| ツール | 用途 | 無料の範囲（2026年10月時点） |
| --- | --- | --- |
| New Relic | トレース・メトリクスの可視化、サービスレベル、アラート | 取り込み 100GB/月、フル機能ユーザー 1 人。超過時は翌月まで取り込み停止 |
| Supabase Metrics API | PhysiFun の Postgres メトリクス（約 200 種）の取得 | 全プロジェクトで利用可 |
| OpenTelemetry Collector | メトリクスの収集・New Relic への転送 | OSS |
| Docker | ローカル実験環境（Postgres + pgbench など） | 無料 |
| AWS Free plan | Aurora PostgreSQL Serverless・CloudWatch の演習 | 新規アカウントのみ。最大 6 か月、クレジット $100 + 最大 $100 |
| Claude | 設計レビュー、障害演習の出題、コードレビュー | — |

---

## ルール

- **シークレットは絶対にコミットしない。** New Relic のライセンスキー、Supabase の secret キー、AWS の認証情報は `.env` に置き、`.gitignore` で除外する（このリポジトリは public）
- **New Relic のデータは区別する。** 実験のデータは `service.name` と `deployment.environment` タグで PhysiFun のデータと混ざらないようにする
- **AWS は使い終わったら消す。** `cdk destroy` を習慣にし、AWS Budgets でアラートを設定しておく
- **検討ログには結論だけでなく理由を書く。** 「何を選んだか」より「なぜそれを選び、何を捨てたか」を残す

---

## Claude の使い方

- **SLO 設計のレビュー**：設計ログを渡して、観点の漏れや目標値の妥当性を指摘してもらう
- **障害演習の出題役**：「レプリカ遅延が急増した。メトリクスはこれ」のようなシナリオを出してもらい、調査手順を回答してレビューを受ける
- **実行計画・クエリのレビュー**：`EXPLAIN ANALYZE` の結果を渡して、自分の読みとすり合わせる
- **IaC・設定のレビュー**：CDK コードや Collector の設定を渡してレビューしてもらう
