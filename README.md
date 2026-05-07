# Mondo Disease Prediction Project

## プロジェクト概要
希少疾患を含む数万件の疾患に対し、単一の統計ソースでは得られない「多角的な関心度」を数値化することで、未知の疾患の患者数や市場性を推計する機械学習モデルの構築を目指す。

## データ収集パイプライン
本プロジェクトでは、28,064件のMondo IDを対象に、以下の3つのフェーズでデータを統合した。

### Phase A: 学術・産業アクティビティ (PubMed & ClinicalTrials.gov)
専門家および企業による投資・研究熱量を可視化。
- PubMed: 疾患名での総論文ヒット数を取得。
- ClinicalTrials.gov: 現在進行中の治験数および目標登録者合計数を抽出。
- インサイト: 研究論文が多い（学術関心）が治験が少ない（産業未着手）疾患の特定が可能。

### Phase B: デジタル・インフルエンス (Wikipedia)
一般社会における認知度と「潜在的な患者・家族」の関心度を可視化。
- Wikipedia Pageviews: 直近1年間の総閲覧数。
- Multilingual Count: 記事が作成されている言語数（国際的な認知指標）。
- インサイト: 論文数は少ないが閲覧数が極端に多い疾患（身近な悩み）の掘り起こし。

### Phase C: 疾患構造解析 (Hierarchical Features)
Mondoのグラフ構造（JSON）をNetworkXで解析し、疾患の「具体性」を数値化。
- Mondo Depth: ルート（Disease）からの階層の深さ。
- Child Count: 疾患の細分化数。
- Leaf Flag: 最末端の具体的疾患かどうかの判定。

## 構築されたデータセットの構成
最終的に生成された `mondo_full_features.csv` は以下のカラムを持つ。

| カラム名 | 説明 | 役割 |
|----------|------|------|
| mondo_id | Mondo疾患識別子 | 主キー |
| pubmed_total_count | 総論文数 | 学術的注目度 |
| ct_active_trials | 進行中の治験数 | 産業的注目度 |
| ct_total_enrollment_target | 治験目標登録者数 | 市場規模(期待値) |
| wiki_annual_pv | 年間Wikipedia閲覧数 | 社会的認知度 |
| wiki_lang_count | 記事の多言語展開数 | グローバル標準化指標 |
| mondo_depth | 階層の深さ | 疾患の希少性・具体性 |
| mondo_child_count | 子ノード数 | カテゴリの大きさ |
| mondo_is_leaf | 末端フラグ | データ粒度の最小単位 |

## 実行ログと特記事項
- 総処理件数: 28,064件
- 総実行時間: 約32時間（APIレート制限を考慮したスリープ込み）
- 安定性確保: `caffeinate` コマンドによるシステムスリープの防止。`tqdm` による進捗可視化。100件ごとのチェックポイント保存（再開機能）を実装。
- 課題: Google Search Result Countについては、レート制限が極めて厳しいため、Wikipedia PVを代替指標として優先活用。