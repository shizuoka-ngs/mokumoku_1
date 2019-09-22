# Shizuoka.ngs もくもく会　2019-9-22

## 「qiime2 tutorialsを体験してみる」


## [qiime2をインストールする　https://docs.qiime2.org/2019.7/install/](https://docs.qiime2.org/2019.7/install/)

[Natively installing QIIME2](https://docs.qiime2.org/2019.7/install/native/#natively-installing-qiime-2)で
minicondaでのインストールで問題ないだろうと思います。__インストールするライブラリが多いので少し時間がかかります。__



```
$ conda update conda
$ conda install wget

$ wget https://data.qiime2.org/distro/core/qiime2-2019.7-py36-osx-conda.yml
$ conda env create -n qiime2-2019.7 --file qiime2-2019.7-py36-osx-conda.yml
```

```
$ conda info -e
# conda environments:
#
base                  *  /Users/blackdog/miniconda3
qiime2-2019.7            /Users/blackdog/miniconda3/envs/qiime2-2019.7
```
のようにquiime2環境が表示されればインストール成功。

```sql
source activate qiime2-2019.7
```
上記のように環境をactivateし、以後の操作をおこなってください。

## [qiime2 tutorials](https://docs.qiime2.org/2019.7/tutorials/)

### [workflows https://docs.qiime2.org/2019.7/tutorials/overview/#overview-of-qiime-2-plugin-workflows](https://docs.qiime2.org/2019.7/tutorials/overview/#overview-of-qiime-2-plugin-workflows)

### [メタデータ取得](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#sample-metadata)

wgetの場合は

```
wget \
  -O "sample-metadata.tsv" \
  "https://data.qiime2.org/2019.7/tutorials/moving-pictures/sample_metadata.tsv"
```

### [データ取得](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#obtaining-and-importing-data)

tutorialsでは配列のサブセットを使います。以下のコマンドを実行しデータを取得してください。

```
mkdir emp-single-end-sequences
```

```
wget \
  -O "emp-single-end-sequences/barcodes.fastq.gz" \
  "https://data.qiime2.org/2019.7/tutorials/moving-pictures/emp-single-end-sequences/barcodes.fastq.gz"
  
wget \
  -O "emp-single-end-sequences/sequences.fastq.gz" \
  "https://data.qiime2.org/2019.7/tutorials/moving-pictures/emp-single-end-sequences/sequences.fastq.gz"
  
```

## 配列をsamplesにアサイン

```
qiime tools import \
  --type EMPSingleEndSequences \
  --input-path emp-single-end-sequences \
  --output-path emp-single-end-sequences.qza
```

## [Demultiplexing sequences](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#demultiplexing-sequences)

配列をサンプルに関連するようdemultiplexingします。

```
qiime demux emp-single \
  --i-seqs emp-single-end-sequences.qza \
  --m-barcodes-file sample-metadata.tsv \
  --m-barcodes-column barcode-sequence \
  --o-per-sample-sequences demux.qza \
  --o-error-correction-details demux-details.qza
```

```
qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv
```

**demux.qzv**が出力できたら、[qiime2のview https://view.qiime2.org](https://view.qiime2.org)に
ファイルをドラッグして確認してみます。

## [Sequence quality control and feature table construction](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#sequence-quality-control-and-feature-table-construction)

QCとしてDADA2またはDeblurを適用します。

### DADA2を実行する

[DADA2](https://www.ncbi.nlm.nih.gov/pubmed/27214047)は **Illumina amplicon sequence** を検出し、補正するとのこと。

```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left 0 \
  --p-trunc-len 120 \
  --o-representative-sequences rep-seqs-dada2.qza \
  --o-table table-dada2.qza \
  --o-denoising-stats stats-dada2.qza
```

visualizationの出力

```
qiime metadata tabulate \
  --m-input-file stats-dada2.qza \
  --o-visualization stats-dada2.qzv
```

```
mv rep-seqs-dada2.qza rep-seqs.qza
mv table-dada2.qza table.qza
```

QCにDADA2を採用する場合下記を実行し、出力したファイルのファイルネームを変えておく。

```
mv rep-seqs-dada2.qza rep-seqs.qza
mv table-dada2.qza table.qza
```

### Deblurを実行する

```
qiime quality-filter q-score \
 --i-demux demux.qza \
 --o-filtered-sequences demux-filtered.qza \
 --o-filter-stats demux-filter-stats.qza
```

```
qiime deblur denoise-16S \
  --i-demultiplexed-seqs demux-filtered.qza \
  --p-trim-length 120 \
  --o-representative-sequences rep-seqs-deblur.qza \
  --o-table table-deblur.qza \
  --p-sample-stats \
  --o-stats deblur-stats.qza
```
QCにDeblurを採用する場合下記を実行して、ファイルネームを変更する。

```
mv rep-seqs-deblur.qza rep-seqs.qza
mv table-deblur.qza table.qza
```

## [FeatureTable and FeatureData summaries](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#featuretable-and-featuredata-summaries)

それぞれのフィーチャー（クラスターを代表する配列）にどれだけの配列が関連するか要約したテーブルを生成します。

```
qiime feature-table summarize \
  --i-table table.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file sample-metadata.tsv
qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv
```

## [Generate a tree for phylogenetic diversity analyses](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#generate-a-tree-for-phylogenetic-diversity-analyses)

系統樹の生成

```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

## [Alpha and beta diversity analysis](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#alpha-and-beta-diversity-analysis)

サンプル内、サンプル間のdiversityの計算

```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 1103 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir core-metrics-results
```

```
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/faith-pd-group-significance.qzv

qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/evenness_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/evenness-group-significance.qzv
```

## [Alpha rarefaction plotting](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#alpha-rarefaction-plotting)

```
qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 4000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv
```

## [Taxonomic analysis](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#taxonomic-analysis)

classifierをDL

```
wget \
  -O "gg-13-8-99-515-806-nb-classifier.qza" \
  "https://data.qiime2.org/2019.7/common/gg-13-8-99-515-806-nb-classifier.qza"
```

配列を分類器にかける

```
qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```

```
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```

[Differential abundance testing with ANCOM](https://docs.qiime2.org/2019.7/tutorials/moving-pictures/#differential-abundance-testing-with-ancom)

```
qiime feature-table filter-samples \
  --i-table table.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-where "[body-site]='gut'" \
  --o-filtered-table gut-table.qza
```