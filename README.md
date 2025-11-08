# GWASサンプルSNP推定結果 解析レポート

## 目次
1. [解析の概要](#1-解析の概要)
2. [データの読み込みと前処理](#2-データの読み込みと前処理)
3. [品質管理：リードカウントによる選抜](#3-品質管理リードカウントによる選抜)
4. [Genotype一致性の評価](#4-genotype一致性の評価)
5. [プレートマップによる空間的分布](#5-プレートマップによる空間的分布)
6. [圃場マップによる時空間分析](#6-圃場マップによる時空間分析)
7. [クロス集計による詳細解析](#7-クロス集計による詳細解析)
8. [KOB084系統の特殊事例](#8-kob084系統の特殊事例)
9. [一度も一致しなかった系統](#9-一度も一致しなかった系統)
10. [結論と推奨事項](#10-結論と推奨事項)

---

## 1. 解析の概要

### 1.1 目的
RNA-seqデータから得られたSNP情報を用いて、GWASサンプル（2018-2020年の3年分）の系統（Genotype）を推定し、記録されている系統情報との一致性を検証する。

### 1.2 データソース
- **対象期間**: 2018年、2019年、2020年の3年間
- **サンプル総数**: 3年分を統合
- **比較対象**: 記録上のGenotype vs SNPから推定したGenotype_panel

### 1.3 解析フロー
```
データ読み込み (2018-2020)
    ↓
SNP推定結果の統合
    ↓
KOB054サンプルの除外
    ↓
品質フィルタリング (total.rawcnt ≥ 10^5.5)
    ↓
一致性評価・可視化
    ↓
空間分析 (プレート・圃場)
    ↓
詳細クロス集計
```

---

## 2. データの読み込みと前処理

### 2.1 データ統合
- 3年分のAttributeデータ（`all.at.GWAS.2018/2019/2020`）を読み込み
- 各年のgenotype_match.tsvから最良マッチ（Discordance最小）を抽出
- Sample_Indexをキーにして統合

### 2.2 KOB054の除外
**理由**: 特定の系統（KOB054）が多数のサンプルで誤推定される傾向が確認されたため、解析から除外

**除外対象**:
- 記録上のGenotype = "KOB054"
- 推定されたGenotype_panel = "KOB084" (注: "084"の誤記と思われる)

### 2.3 データクリーニング
- 2020年のSample_Indexから`OsaAN_`の`_`を削除し、フォーマットを統一

**出力ファイル**:
- `results/uncrit/all_samples_with_genotype_match_uncrit.csv` (選抜前)
- `results/all_samples_with_genotype_match.csv` (選抜後)

---

## 3. 品質管理：リードカウントによる選抜

### 3.1 選抜基準
**閾値**: `total.rawcnt ≥ 10^5.5` (約316,228 reads)

この閾値は、SNP推定の信頼性を担保するために設定された。

### 3.2 選抜前後の比較

#### 図1: Total Raw Count分布（一致 vs 不一致）
**ファイル**: `results/totalread_histogram/totalread_histogram_all_years.png`

**主要な知見**:
1. **横軸（log10スケール）**: Total Raw Count（総リード数）
2. **色分け**:
   - 青色: 記録と推定が一致したサンプル
   - 赤色: 記録と推定が不一致のサンプル
3. **黒い縦線**: 10^5.5の選抜閾値

**解釈**:
- リード数が少ないサンプル（10^5.5以下）では不一致率が高い傾向
- リード数が多いサンプルでは一致率が向上
- 10^5.5の閾値設定は妥当であることを示唆

#### 年別サマリー
各年のリード数分布と一致率は年別に比較可能（`results/totalread_histogram/totalread_count_summary.csv`参照）。

### 3.3 選抜効果
選抜により、以下の改善が期待される:
- SNP推定の信頼性向上
- ノイズの多いサンプルの除外
- 系統同定精度の向上

**出力ファイル**:
- `results/totalread_histogram/` 配下の各種PNG・CSV

---

## 4. Genotype一致性の評価

### 4.1 年別一致性

#### 図2: 年別 Genotype記録とSNP推定の一致性
**ファイル**: 
- 選抜前: `results/uncrit/Genotype_match_by_year.png`
- 選抜後: `results/Genotype_match_by_year.png`

**内容**:
- 各年のサンプル数を一致/不一致で積み上げ棒グラフ表示
- 青色: 一致、赤色: 不一致

**解釈**:
- 各年での一致率の傾向を把握
- 年次間での品質のバラつきを評価
- 選抜後は一致率が改善

### 4.2 PlateNo別一致性

#### 図3: PlateNo・年別 Genotype記録との一致率（ヒートマップ）
**ファイル**: 
- 選抜前: `results/uncrit/Genotype_match_by_year_plateNo.png`
- 選抜後: `results/Genotype_match_by_year_plateNo.png`

**内容**:
- X軸: PlateNo、Y軸: Year
- 各セルに一致サンプル数と一致率（%）を表示
- 色: 赤（低一致率）→ 黄色 → 青（高一致率）

**解釈**:
- プレート間での品質のバラつきを可視化
- 特定のプレートで一致率が低い場合、そのプレートの処理に問題があった可能性
- 問題のあるプレートを特定し、原因究明に役立つ

#### 図4: PlateNo別 Genotype記録との一致性（年別ファセット）
**ファイル**: 
- 選抜前: `results/uncrit/Genotype_match_by_plateNo_faceted.png`
- 選抜後: `results/Genotype_match_by_plateNo_faceted.png`

**内容**:
- 年別にファセット分割
- PlateNo別に一致/不一致のサンプル数を並列棒グラフで表示

**解釈**:
- より詳細なプレート別の傾向を把握
- 年ごとのプレート数や品質の違いを比較

**出力ファイル**:
- `results/Genotype_match_summary_by_year_plateNo.csv`

### 4.3 DiscordanceとSites_comparedの分布

#### 図5: 比較サイト数の分布
**ファイル**: `results/uncrit/Sites_compared_histogram.png`

**内容**:
- X軸: 比較したSNPサイト数
- Y軸: サンプル数
- 色: 一致（青）vs 不一致（赤）

**解釈**:
- 比較サイト数が多いほど推定の信頼性が高い
- 不一致サンプルで比較サイト数が少ない場合、データ不足が原因の可能性

#### 図6: Discordance（不一致率）の分布
**ファイル**: `results/uncrit/Discordance_histogram.png`

**内容**:
- X軸: Discordance（最良マッチとの不一致率）
- Y軸: サンプル数

**解釈**:
- Discordanceが低い = 推定系統とのSNP一致度が高い
- 一致サンプルは低Discordance、不一致サンプルは高Discordanceを示す傾向

#### 図7: Discordance vs Sites_compared散布図
**ファイル**: `results/uncrit/Discordance_vs_Sites_compared_scatter.png`

**内容**:
- X軸: Sites_compared、Y軸: Discordance
- 点の色: 一致（青）vs 不一致（赤）

**解釈**:
- 比較サイト数とDiscordanceの関係を視覚化
- 理想的には、Sites_comparedが多く、Discordanceが低いサンプルが多いことが望ましい

---

## 5. プレートマップによる空間的分布

### 5.1 プレートマップの概要
96wellプレート形式でサンプルの一致/不一致状況を可視化。

**ディレクトリ**: `results/plate_maps/`

### 5.2 マップの構成
- **行（Row）**: A-H（通常の96wellプレート）
- **列（Column）**: 1-12
- **各セルの表示内容**:
  - 上段: 記録上のGenotype
  - 下段: SNP推定のGenotype_panel
- **色分け**:
  - 青色: 一致
  - 赤色: 不一致
  - 白色: 10^5.5以下（選抜前のみ）
  - 灰色: SNPデータなし

### 5.3 ファイル命名規則
`{Year}_PlateNo{PlateNo}.png`
例: `2018_PlateNo1.png`, `2019_PlateNo5.png`

### 5.4 活用方法
1. **コンタミネーション検出**: 隣接ウェル間での系統の混在パターンを確認
2. **プレート端効果**: プレートの端（A列、H列、1番、12番）での品質低下を確認
3. **ピペッティングエラー**: 特定のパターン（例: 列または行全体）での不一致

---

## 6. 圃場マップによる時空間分析

### 6.1 圃場マップの原理
**座標の決定**: LineNameの後ろ3文字を圃場座標として使用
例: `LineName = "Koshihikari001"` → 圃場座標 = 1

### 6.2 代表推定系統の概念
各記録系統について、最も多く推定された系統を「代表推定系統」として計算。

**表示ロジック**:
1. 代表推定系統 = 記録系統 → 系統名のみ表示（青色）
2. 代表推定系統 ≠ 記録系統 → `{代表推定} <> {記録}` 形式で表示（赤色）
3. 常に一致する系統 → 系統名のみ表示（青色）

### 6.3 年別圃場マップ

#### 図8: 圃場マップ（年別）
**ファイル**: 
- 選抜前: `results/field_maps/field_map_{Year}.png`
- 選抜後: `results/field_maps_filtered/field_map_{Year}_filtered.png`

**内容**:
- X軸: 圃場座標（LineName後ろ3文字）
- 各タイルに系統名または不一致情報を表示
- 色: 青（一致）vs 赤（不一致）

**解釈**:
- 圃場内での系統取り違えのパターンを空間的に把握
- 隣接する圃場位置での混入を検出
- サンプリング時のエラーを推定

### 6.4 全年統合圃場マップ

#### 図9: 圃場マップ（全年統合）
**ファイル**: 
- 選抜前: `results/field_maps/field_map_all_years.png`
- 選抜後: `results/field_maps_filtered/field_map_all_years_filtered.png`

**内容**:
- Y軸に年を追加し、3年分を1枚にまとめて表示

**解釈**:
- 年次間での圃場利用パターンの比較
- 特定の圃場位置で毎年問題が発生していないかを確認

**出力ファイル**:
- `results/field_maps/field_map_data.csv`
- `results/field_maps/genotype_representative_summary_{Year}.csv`

---

## 7. クロス集計による詳細解析

### 7.1 Genotypeクロス集計

#### 図10: Genotype記録 vs SNP推定ヒートマップ
**ディレクトリ**: `results/genotype_cross/`

**ファイル形式**: 
- PNG: `genotype_cross_heatmap_{Year}.png` / `genotype_cross_heatmap_all_years.png`
- PDF: `genotype_cross_heatmap_{Year}.pdf` / `genotype_cross_heatmap_all_years.pdf`

**内容**:
- X軸: 記録上のGenotype
- Y軸: SNP推定のGenotype_panel
- 各セル: サンプル数（log10スケール）
- 色: 赤（対角線＝一致）、青（非対角線＝不一致）

**解釈**:
1. **対角線上（赤色）**: 記録と推定が一致
2. **対角線外（青色）**: 記録と推定が不一致
3. **濃い色**: サンプル数が多い組み合わせ
4. **パターン認識**: 
   - 特定の系統が別の系統に頻繁に誤推定される場合、その組み合わせのセルが濃くなる
   - 遺伝的に近い系統間での混同の可能性

**出力ファイル**:
- `results/genotype_cross/genotype_cross_table_{Year}.csv`
- 不一致TOP10リストなど

### 7.2 Genotypeクロス集計（不一致系統のみ）

#### 図11: Genotype記録 vs SNP推定（不一致系統のみ）
**ディレクトリ**: `results/genotype_cross_filtered/`

**内容**:
- 完全一致系統（記録と推定が常に100%一致する系統）を除外
- 問題のある系統のみに焦点を当てた解析

**解釈**:
- より詳細な不一致パターンの把握
- 対策が必要な系統の特定

**出力ファイル**:
- `results/genotype_cross_filtered/perfect_match_genotypes.csv`: 完全一致系統リスト

### 7.3 LineNameクロス集計

#### 図12: LineName記録 vs SNP推定ヒートマップ
**ディレクトリ**: `results/linename_cross/`

**内容**:
- Genotype名をLineName（系統名）に変換してクロス集計
- `Pnael2LineName.csv`を用いて変換

**解釈**:
- 系統レベル（LineName）での一致性評価
- 複数のGenotypeが同じLineNameに対応する場合の集約効果

### 7.4 LineNameクロス集計（不一致系統のみ）

#### 図13: LineName記録 vs SNP推定（不一致系統のみ）
**ディレクトリ**: 
- 選抜前: `results/linename_cross_filtered/`
- 選抜後（10^5.5フィルタ）: `results/linename_cross_filtered_selected/`

**内容**:
- 完全一致系統を除外したLineName解析
- 10^5.5フィルタ後のデータでも同様の解析を実施

**解釈**:
- 品質選抜後の不一致パターンの変化を評価
- 高品質データのみでの系統間混同パターン

**出力ファイル**:
- `results/linename_cross_filtered/perfect_match_linenames.csv`

---

## 8. KOB084系統の特殊事例

### 8.1 KOB084問題の発見
多数のサンプルが系統"KOB084"と誤推定される現象が確認された。

### 8.2 KOB084推定サンプルの分析

#### 図14: Total Raw Count分布（KOB084推定 vs その他）
**ファイル**: `results/kob084_histogram/kob084_histogram_by_year.png`

**内容**:
- オレンジ色: KOB084と推定されたサンプル
- 灰色: その他のサンプル
- 年別にファセット分割

**解釈**:
- KOB084と推定されたサンプルのリード数分布
- 低リード数サンプルでKOB084が誤推定されやすい可能性

#### 図15: 不一致サンプルのみでのKOB084推定
**ファイル**: `results/kob084_histogram/kob084_mismatch_histogram_by_year.png`

**内容**:
- 記録と推定が不一致のサンプルのみに限定
- その中でKOB084と推定されたサンプルの割合

**解釈**:
- 不一致の原因としてKOB084誤推定がどの程度寄与しているか
- KOB084除外の妥当性を評価

### 8.3 KOB084の除外理由
上記の解析結果から、KOB084は信頼性の低い推定結果として扱い、解析から除外することが決定された。

**出力ファイル**:
- `results/kob084_histogram/kob084_estimated_samples.csv`: KOB084推定サンプル詳細
- `results/kob084_histogram/kob084_mismatch_estimated_samples.csv`: 不一致中のKOB084推定サンプル

---

## 9. 一度も一致しなかった系統

### 9.1 問題系統の抽出
記録系統の中で、全サンプルで推定結果と一度も一致しなかった系統を特定。

**出力ファイル**: `results/never_matched_genotypes.csv`

### 9.2 想定される原因
1. **系統名の誤記**: 記録上の系統名が誤っている
2. **系統の取り違え**: サンプリング時に別の系統を採取
3. **参照パネルの問題**: SNP参照パネルにその系統が含まれていない
4. **ラベル貼り間違い**: サンプル管理時のヒューマンエラー

### 9.3 対応策
1. 元の記録を再確認
2. サンプリング手順の見直し
3. 参照パネルへの系統追加を検討
4. 該当サンプルの再サンプリング・再シーケンス

---

## 10. 結論と推奨事項

### 10.1 主要な知見

#### 1. 品質管理の重要性
- **10^5.5（約316,000 reads）の閾値**: SNP推定の信頼性を担保するために効果的
- リード数が少ないサンプルでは不一致率が高く、品質選抜の必要性が明確

#### 2. 系統一致性
- 選抜後のデータでは一致率が向上
- ただし、選抜後も一定数の不一致が残存

#### 3. 空間的パターン
- プレートマップ・圃場マップにより、系統的なエラーパターンを検出可能
- 特定のプレートや圃場位置での問題を特定

#### 4. KOB084問題
- 特定の系統（KOB084）が頻繁に誤推定される
- 参照パネルの見直しまたは該当系統の除外が必要

#### 5. 完全不一致系統
- 一度も一致しなかった系統が存在
- 記録の正確性やサンプリング手順の見直しが必要

### 10.2 推奨事項

#### 短期的対策
1. **品質フィルタリングの適用**: 10^5.5閾値を標準として採用
2. **KOB084の除外**: 信頼性の低い推定結果として扱う
3. **不一致サンプルの精査**: 
   - プレートマップ・圃場マップから系統的エラーを特定
   - 原因究明と再サンプリング

#### 中期的対策
1. **サンプリングプロトコルの改善**:
   - ダブルチェック体制の導入
   - バーコード管理の強化
2. **参照パネルの見直し**:
   - 頻出系統の追加
   - 遺伝的に近い系統の識別性向上
3. **プレート処理の標準化**:
   - プレート間のバラつきを減少させる手順の確立

#### 長期的対策
1. **自動化の導入**: ヒューマンエラーを減らすための自動サンプリング・ラベリングシステム
2. **リアルタイムQC**: シーケンス直後の品質チェックと即時フィードバック
3. **統合データベース**: サンプル情報とシーケンス結果の一元管理

### 10.3 データの信頼性評価

#### 信頼性の高いデータ
- `is.5.5 == TRUE`（10^5.5以上のリード数）
- 記録と推定が一致するサンプル
- Discordanceが低いサンプル

#### 注意が必要なデータ
- リード数が閾値以下のサンプル
- KOB084と推定されたサンプル
- 一度も一致しなかった系統のサンプル
- 特定のプレートや圃場位置のサンプル

### 10.4 今後の解析への活用

この解析で得られた知見は、以下の研究に活用可能:
1. **GWAS解析**: 信頼性の高いサンプルのみを使用
2. **遺伝的多様性解析**: 系統同定精度の向上
3. **品質管理基準**: 他のプロジェクトへの応用
4. **サンプル追跡**: 問題のあるサンプルの原因究明

---

## 補足資料

### A. 主要な出力ファイル一覧

#### 統合データ
- `results/all_samples_with_genotype_match.csv`: 全サンプルの統合データ（10^5.5選抜後）
- `results/uncrit/all_samples_with_genotype_match_uncrit.csv`: 全サンプルの統合データ（選抜前）

#### サマリー
- `results/Genotype_match_summary_by_year_plateNo.csv`: 年・PlateNo別一致性サマリー
- `results/totalread_histogram/totalread_count_summary.csv`: リードカウントサマリー
- `results/never_matched_genotypes.csv`: 一度も一致しなかった系統リスト

#### 詳細解析
- `results/genotype_cross/genotype_cross_table_{Year}.csv`: 年別Genotypeクロス集計
- `results/linename_cross/linename_cross_table_{Year}.csv`: 年別LineNameクロス集計
- `results/field_maps/field_map_data.csv`: 圃場マップデータ
- `results/kob084_histogram/kob084_estimated_samples.csv`: KOB084推定サンプル詳細

### B. 図表ディレクトリ構成

```
results/
├── Genotype_match_by_year.png                    # 図2
├── Genotype_match_by_year_plateNo.png            # 図3
├── Genotype_match_by_plateNo_faceted.png         # 図4
├── uncrit/
│   ├── Sites_compared_histogram.png              # 図5
│   ├── Discordance_histogram.png                 # 図6
│   └── Discordance_vs_Sites_compared_scatter.png # 図7
├── totalread_histogram/
│   └── totalread_histogram_all_years.png         # 図1
├── plate_maps/
│   └── {Year}_PlateNo{PlateNo}.png              # プレートマップ
├── field_maps/
│   ├── field_map_{Year}.png                      # 図8
│   └── field_map_all_years.png                   # 図9
├── genotype_cross/
│   └── genotype_cross_heatmap_{Year}.png         # 図10
├── genotype_cross_filtered/
│   └── genotype_cross_filtered_{Year}.png        # 図11
├── linename_cross/
│   └── linename_cross_heatmap_{Year}.png         # 図12
├── linename_cross_filtered/
│   └── linename_cross_filtered_{Year}.png        # 図13
└── kob084_histogram/
    ├── kob084_histogram_by_year.png              # 図14
    └── kob084_mismatch_histogram_by_year.png     # 図15
```

### C. 用語集

- **Genotype**: 記録上の系統名
- **Genotype_panel**: SNPから推定された系統名
- **LineName**: 系統の別名（品種名など）
- **Discordance**: SNP推定における不一致率（低いほど推定精度が高い）
- **Sites_compared**: 比較したSNPサイト数（多いほど信頼性が高い）
- **total.rawcnt**: 総リード数（RNA-seqの測定量）
- **is.5.5**: total.rawcnt ≥ 10^5.5 のフラグ
- **PlateNo**: 96wellプレートの番号
- **Position**: プレート内のウェル位置（例: A1, B3）

---

## 改訂履歴
- 2025-XX-XX: 初版作成


