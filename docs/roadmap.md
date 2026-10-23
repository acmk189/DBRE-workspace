# DBRE 学習ロードマップ

DBA の経験をベースに、DBRE として必要なスキルを段階的に身につけるためのロードマップ。
各フェーズは「このリポジトリでやること」と「PhysiFun に反映すること」をセットで進める。

PhysiFun 側の実装は親 Issue「SRE を導入する」で管理する。

---

## 全体像

| フェーズ | テーマ                     | 主な場所                  | 費用                        |
| -------- | -------------------------- | ------------------------- | --------------------------- |
| Phase 0  | 準備                       | DBRE-workspace            | 無料                        |
| Phase 1  | SLI/SLO の観点を身につける | DBRE-workspace → PhysiFun | 無料                        |
| Phase 2  | アプリ側の計測             | PhysiFun（検証はこちら）  | 無料                        |
| Phase 3  | DB 側の計測とアラート設計  | 両方                      | 無料                        |
| Phase 4  | AWS（Aurora + CDK）の演習  | DBRE-workspace            | AWS Free plan（6 か月以内） |
| Phase 5  | 継続的な練習               | 両方                      | 無料                        |

---

## Phase 0：準備

**ゴール**：学習を始められる環境が整っている

- [x] ディレクトリ構成と `.gitignore`（`.env` を除外）を作る
- [ ] New Relic のアカウントを作る（PhysiFun 用と分けるかを決める）
- [ ] Docker が動く環境を用意する

---

## Phase 1：SLI/SLO の観点を身につける

**ゴール**：ユーザー視点で SLI/SLO を設計し、その理由を説明できる

### 学ぶ観点

1. **ユーザーの行動から始める**：「誰が何をしようとして、何が起きたら困るか」
2. **SLI の種類**：可用性・レイテンシ・鮮度・正しさ
3. **どこで測るか**：ユーザーに近いほど体験に近いが、ノイズも増える
4. **目標値は現状を測ってから決める**：100% は目指さない
5. **エラーバジェットの使い道**：使い切ったときのルールまで決める
6. **SLO アラートと原因アラートを分ける**：前者は即対応、後者は調査の手がかり

### タスク

- [ ] Google SRE 本「Service Level Objectives」の章を読んでノートにまとめる → `docs/notes/slo-basics.md`
- [ ] SRE Workbook「Implementing SLOs」の章を読んでノートにまとめる
- [ ] 自分用の「SLO 設計チェックリスト」を作る → `docs/notes/slo-checklist.md`
- [ ] PhysiFun のユーザージャーニーを洗い出す（リーダー応募、運営審査、メール送信、cron など）
- [ ] PhysiFun の SLI/SLO を検討する → `docs/design-logs/physifun-slo.md`
  - 応募フォームの送信成功率（可用性）
  - 運営審査画面の p95 レイテンシ（レイテンシ）
  - Outbox のメール送信遅延（鮮度）
  - cron ジョブの成功率
- [ ] 低トラフィックならではの課題（母数の少なさ）への対策を考える
- [ ] Claude に検討ログをレビューしてもらう

**PhysiFun への反映**：SLO v0 を `docs/specs/` に PR

---

## Phase 2：アプリ側の計測

**ゴール**：ユーザーから見た SLI を New Relic で計測できている

- [ ] `sandbox/` に最小の Next.js を作り、`@vercel/otel` から New Relic の OTLP エンドポイントへトレースを送れるか検証する
  - Vercel の公式 New Relic 連携は Pro / Enterprise 向けのため、Hobby プランで OTLP 直送が使えるかを確認する
- [ ] Kysely のクエリにカスタムスパンを付ける方法を検討する（どの粒度で付けるか）
- [ ] New Relic のサービスレベル機能で SLI/SLO を定義する方法を試す
- [ ] 検証結果をまとめる → `docs/design-logs/app-instrumentation.md`

**PhysiFun への反映**：計測コードとサービスレベル定義を PR

---

## Phase 3：DB 側の計測とアラート設計

**ゴール**：DB の状態を可視化し、「本当に困ったときだけ鳴る」アラートを設計できる

### sandbox での実験

- [ ] `sandbox/` に Postgres + pgbench + postgres_exporter + OTel Collector の docker-compose を作る
- [ ] Collector で Prometheus 形式のメトリクスを scrape し、New Relic に送る
- [ ] pgbench で負荷をかけて、接続数・ロック待ち・キャッシュヒット率・レイテンシの動きを観察する
- [ ] 意図的に障害を起こす（長時間ロック、接続枯渇、重いクエリ）

### 設計

- [ ] Supabase Metrics API の約 200 種のメトリクスから、見るべきものを選ぶ → `docs/notes/supabase-metrics.md`
- [ ] アラートを 2 層で設計する → `docs/design-logs/alert-design.md`
  - SLO アラート：ユーザーが困っている（即対応）
  - 原因アラート：予兆・調査の手がかり（通知のみ）
- [ ] 障害注入でアラートが期待どおり鳴るか確認する

**PhysiFun への反映**：Collector 設定、ダッシュボード、アラート定義、Runbook を PR

---

## Phase 4：AWS（Aurora + CDK）の演習

**ゴール**：会社と同じ Aurora の運用を IaC で再現し、障害対応を実測できる

> AWS Free plan は最大 6 か月。期間内に集中して進める。

- [ ] 新規 AWS アカウント（Free plan）を作り、AWS Budgets でアラートを設定する
- [ ] CDK で Aurora PostgreSQL Serverless を構築する → `aws-cdk/`
- [ ] CloudWatch のアラーム・ダッシュボードも CDK で管理する
- [ ] Database Insights で遅いクエリを分析する
- [ ] フェイルオーバー演習を行い、ダウンタイムを実測する
- [ ] スナップショット復元・PITR の演習を行い、RTO / RPO を記録する → `docs/drills/`
- [ ] （余力があれば）同じ構成を Terraform でも書き、CDK と比較する
- [ ] 終わったら `cdk destroy`

---

## Phase 5：継続的な練習

**ゴール**：障害対応と運用改善の経験を積み続ける

- [ ] Claude と障害演習を定期的に行う（月 2 回目安）→ `docs/drills/`
- [ ] ポストモーテムのテンプレートを作る → `docs/postmortems/TEMPLATE.md`
- [ ] マイグレーションのレビュー観点（ロック、破壊的変更、ロールバック）をまとめ、PhysiFun の PR で実践する
- [ ] PhysiFun の運用で気づいたことを振り返りとして残す
- [ ] 四半期ごとにこのロードマップを見直す

---

## 進捗ログ

| 日付       | 内容                                     |
| ---------- | ---------------------------------------- |
| 2026-10-22 | リポジトリ作成、README・ロードマップ作成 |
| 2026-10-23 | ディレクトリ構成と `.gitignore` を作成   |
