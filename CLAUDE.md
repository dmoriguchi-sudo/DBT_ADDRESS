# CLAUDE.md - DBT_ADDRESS

## プロジェクト概要
BigQueryに地理・競合分析用のテーブルを作成する初期セットアップスクリプト。
関東圏の飲食店・競合小売店・地価データを組み合わせた競合スコアテーブルを構築する。
**手動実行のみ（一度だけ実行するもの）**。

## 技術スタック
- Python 3.11
- Google Cloud BigQuery（プロジェクト: `logistics-449115`）
- GitHub Actions（手動実行のみ）

## 使用シークレット
| シークレット名 | 内容 |
|---|---|
| `GCP_SA_KEY` | GCPサービスアカウントのJSONキー |

## 主要ファイル
| ファイル | 役割 |
|---|---|
| `.github/workflows/create-bigquery-tables-once.yml` | 手動実行ワークフロー |

## 作成されるBigQueryテーブル（logistics-449115.geography）
| テーブル名 | 内容 |
|---|---|
| `stg_foursquare_places_kanto` | 関東圏のFoursquareスポットデータ |
| `stg_competitors_retail` | 競合小売店データ（競合ウェイト付き） |
| `stg_customers_restaurants` | 顧客飲食店データ |
| `stg_land_prices_kanto` | 関東圏地価データ |
| `mart_restaurant_competition_score` | 飲食店競合スコア（最終マート） |

## リスクレベル判定ロジック
- 超都心・都心 かつ 競合スコア < 2.0 → 低リスク
- 郊外・遠郊外 かつ ディスカウント店2件以上 → 高リスク
- 競合スコア ≥ 4.0 → 高リスク
- 競合スコア ≥ 2.0 → 中リスク

## 実行方法
GitHub ActionsのワークフローページでRun workflowボタンから手動実行。
または：
```bash
export GCP_SA_KEY='（サービスアカウントJSONの内容）'
pip install pandas-gbq google-cloud-bigquery
# ワークフロー内のPythonコードを直接実行
```

## 注意事項
- `mart_restaurant_competition_score` の作成は数分かかる（ST_DWITHIN の空間結合処理）
- 一度だけ実行するもの。再実行時は CREATE OR REPLACE で上書きされる
- dmoriguchi-sudo側のActionsが有効（vegekul側は無効）
