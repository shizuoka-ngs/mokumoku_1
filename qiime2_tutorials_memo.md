# Shizuoka.ngs もくもく会　2019-9-22

## 「qiime2 tutorialsを体験してみる」


## [qiime2をインストールする　https://docs.qiime2.org/2019.7/install/](https://docs.qiime2.org/2019.7/install/)

[Natively installing QIIME2](https://docs.qiime2.org/2019.7/install/native/#natively-installing-qiime-2)で
minicondaでのインストールで問題ないだろうと思います。__インストールするライブラリが多いので少し時間がかかります。__

```sql

```
$ conda update conda
$ conda install wget

$ wget https://data.qiime2.org/distro/core/qiime2-2019.7-py36-osx-conda.yml
$ conda env create -n qiime2-2019.7 --file qiime2-2019.7-py36-osx-conda.yml

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

QCとしてDADA2とDeblurを適用します。

