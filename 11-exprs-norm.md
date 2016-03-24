---
knit: bookdown::preview_chapter
---

# Normalization for library size

## Introduction

In the previous chapter we identified important confounding factors and explanatory variables. scater allows one to account for these variables in subsequent statistical models or to condition them out using `normaliseExprs()`, if so desired. This can be done by providing a design matrix to `normaliseExprs()`. We are not covering this topic here, but you can try to do it yourself as an exercise.

Instead we will explore how simple size-factor normalisations correcting for library size can remove the effects of some of the confounders and explanatory variables.

## Library size

Library sizes vary because of the various reasons:
* scRNA-seq data is often sequenced on highly multiplexed platforms the total reads which are derived from each cell may differ substantially.
* Protocols may differ in terms of their coverage of each transcript, their bias based on the average content of __A/T__ nucleotides, or their ability to capture short transcripts.

Ideally, we would like to compensate for all of these differences and biases when comparing data from two different experiments.

Many methods to correct for library size have been developped for bulk RNA-seq and can be equally applied to scRNA-seq (eg. UQ, SF, CPM, RPKM, FPKM, TPM). Some quantification methods
(eg. [Cufflinks](http://cole-trapnell-lab.github.io/cufflinks/), [RSEM](http://deweylab.github.io/RSEM/)) incorporated library size when determining gene expression estimates thus do not require this normalization.

We will continue to work with the Blischak data that was used in the previous chapter and show how scater makes it easy to perform different types of size-factor normalizations.




```r
library(scater, quietly = TRUE)
options(stringsAsFactors = FALSE)
umi <- readRDS("blischak/umi.rds")
umi.qc <- umi[fData(umi)$use, pData(umi)$use]
endog_genes <- !fData(umi.qc)$is_feature_control
```

## Normalisations

The simplest way to normalize this data is to convert it to counts per
million (__CPM__) by dividing each column by its total then multiplying by
1,000,000. Note that spike-ins should be excluded from the
calculation of total expression in order to correct for total cell RNA
content, therefore we will only use endogenous genes. scater performs this normalisation by default, you can control it by changing `exprs_values` parameter to `"exprs"`.

Another method is called __TMM__ is the weighted trimmed mean of M-values (to the reference) proposed by [Robinson and Oshlack (2010)](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2010-11-3-r25), where the weights are from the delta method on Binomial data.

Another very popular method __RLE__ is the scaling factor method proposed by [Anders and Huber (2010)](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2010-11-10-r106). We call it "relative log expression", as median library is calculated from the geometric mean of all columns and the median ratio of each sample to the median library is taken as the scale factor.

A similar to __RLE__ the __upperquartile__ is the upper-quartile normalization method of [Bullard et al (2010)](http://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-94), in which the scale factors are calculated from the 75% quantile of the counts for each library, after removing genes which are zero in all libraries. This idea is generalized here to allow scaling by any quantile of the distributions.

Let's compare all these methods.

### Raw

```r
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "counts")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-raw-1.png" alt="(\#fig:norm-pca-raw)PCA plot of the blischak data" width="90%" />
<p class="caption">(\#fig:norm-pca-raw)PCA plot of the blischak data</p>
</div>

### CPM

```r
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "cpm")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-cpm-1.png" alt="(\#fig:norm-pca-cpm)PCA plot of the blischak data after CPM normalisation" width="90%" />
<p class="caption">(\#fig:norm-pca-cpm)PCA plot of the blischak data after CPM normalisation</p>
</div>

### log2(CPM)

```r
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "exprs")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-log2-cpm-1.png" alt="(\#fig:norm-pca-log2-cpm)PCA plot of the blischak data after log2(CPM) normalisation" width="90%" />
<p class="caption">(\#fig:norm-pca-log2-cpm)PCA plot of the blischak data after log2(CPM) normalisation</p>
</div>

### TMM

```r
umi.qc <- 
    scater::normaliseExprs(umi.qc,
                           method = "TMM",
                           feature_set = endog_genes,
                           lib.size = rep(1, ncol(umi.qc)))
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "norm_counts")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-tmm-1.png" alt="(\#fig:norm-pca-tmm)PCA plot of the blischak data after TMM normalisation" width="90%" />
<p class="caption">(\#fig:norm-pca-tmm)PCA plot of the blischak data after TMM normalisation</p>
</div>

### RLE

```r
umi.qc <- 
    scater::normaliseExprs(umi.qc,
                           method = "RLE",
                           feature_set = endog_genes,
                           lib.size = rep(1, ncol(umi.qc)))
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "norm_counts")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-rle-1.png" alt="(\#fig:norm-pca-rle)PCA plot of the blischak data after RLE normalisation" width="90%" />
<p class="caption">(\#fig:norm-pca-rle)PCA plot of the blischak data after RLE normalisation</p>
</div>

### Upperquantile

```r
umi.qc <- 
    scater::normaliseExprs(umi.qc,
                           method = "upperquartile", 
                           feature_set = endog_genes,
                           p = 0.99,
                           lib.size = rep(1, ncol(umi.qc)))
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "norm_counts")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-uq-1.png" alt="(\#fig:norm-pca-uq)PCA plot of the blischak data after UQ normalisation" width="90%" />
<p class="caption">(\#fig:norm-pca-uq)PCA plot of the blischak data after UQ normalisation</p>
</div>

## Interpretation

Clearly, only the CPM normalisation has reduced the effect of the number of detected genes and separated cells by individuals (though very weakly).

## Other methods

Some methods combine library size and fragment/gene length normalization such as:

* __RPKM__ - Reads Per Kilobase Million (for single-end sequencing)
* __FPKM__ - Fragments Per Kilobase Million (same as __RPKM__ but for paired-end sequencing, makes sure that paired ends mapped to the same fragment are not counted twice)
* __TPM__ - Transcripts Per Kilobase Million (same as __RPKM__, but the order of normalizations is reversed - length first and sequencing depth second)

These methods are not applicable to our dataset since the end
of the transcript which contains the UMI was preferentially
sequenced. Furthermore in general these should only be calculated
using appropriate quantification software from aligned BAM files not
from read counts since often only a portion of the entire
gene/transcript is sequenced, not the entire length.

However, here we show how these normalisations can be calculated using scater. First, we need to find the effective transcript length in Kilobases. However, our dataset containes only gene IDs, therefore we will be using the gene lengths instead of transcripts. scater uses the [biomaRt](https://bioconductor.org/packages/release/bioc/html/biomaRt.html) package, which allows one to annotate genes by other attributes:

```r
umi.qc <-
    scater::getBMFeatureAnnos(umi.qc,
                              filters = "ensembl_gene_id", 
                              attributes = c("ensembl_gene_id",
                                             "hgnc_symbol",
                                             "chromosome_name",
                                             "start_position",
                                             "end_position"), 
                              feature_symbol = "hgnc_symbol",
                              feature_id = "ensembl_gene_id",
                              biomart = "ENSEMBL_MART_ENSEMBL", 
                              dataset = "hsapiens_gene_ensembl",
                              host = "www.ensembl.org")

# If you have mouse data, change the arguments based on this example:
# scater::getBMFeatureAnnos(object,
#                           filters = "ensembl_transcript_id", 
#                           attributes = c("ensembl_transcript_id", 
#                                        "ensembl_gene_id", "mgi_symbol", 
#                                        "chromosome_name",
#                                        "transcript_biotype",
#                                        "transcript_start",
#                                        "transcript_end", 
#                                        "transcript_count"), 
#                           feature_symbol = "mgi_symbol",
#                           feature_id = "ensembl_gene_id",
#                           biomart = "ENSEMBL_MART_ENSEMBL", 
#                           dataset = "mmusculus_gene_ensembl",
#                           host = "www.ensembl.org") 
```

Some of the genes were not annotated, therefore we filter them out:

```r
umi.qc.ann <-
    umi.qc[!is.na(fData(umi.qc)$ensembl_gene_id), ]
```

Now we compute the effective gene length in Kilobases by using the `end_position` and `start_position` fields:

```r
eff_length <- abs(fData(umi.qc.ann)$end_position -
                      fData(umi.qc.ann)$start_position)/1000
```

Now we are ready to perform the normalisations:

```r
tpm(umi.qc.ann) <-
    calculateTPM(
        umi.qc.ann,
        eff_length
    )
fpkm(umi.qc.ann) <-
    calculateFPKM(
        umi.qc.ann,
        eff_length
    )
```

Plot the results as a PCA plot:

```r
scater::plotPCA(umi.qc.ann,
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "fpkm")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-pca-fpkm-1.png" alt="(\#fig:norm-pca-fpkm)PCA plot of the blischak data after FPKM normalisation" width="90%" />
<p class="caption">(\#fig:norm-pca-fpkm)PCA plot of the blischak data after FPKM normalisation</p>
</div>

TPM normalisation produce a zero-matrix, we are not sure why, it maybe a bug in scater.

## Visualize genes

Now after the normalisation we are ready to visualise the gene expression:

### Raw

```r
scater::plotExpression(umi.qc.ann,
                       rownames(umi.qc.ann)[1:6],
                       x = "individual",
                       exprs_values = "counts",
                       colour = "batch")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-genes-raw-1.png" alt="(\#fig:norm-genes-raw)Expression of the first 6 genes of the blischak data" width="90%" />
<p class="caption">(\#fig:norm-genes-raw)Expression of the first 6 genes of the blischak data</p>
</div>

### CPM

```r
scater::plotExpression(umi.qc.ann,
                       rownames(umi.qc.ann)[1:6],
                       x = "individual",
                       exprs_values = "cpm",
                       colour = "batch")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-genes-cpm-1.png" alt="(\#fig:norm-genes-cpm)Expression of the first 6 genes of the blischak data after the CPM normalisation" width="90%" />
<p class="caption">(\#fig:norm-genes-cpm)Expression of the first 6 genes of the blischak data after the CPM normalisation</p>
</div>

### log2(CPM)

```r
scater::plotExpression(umi.qc.ann,
                       rownames(umi.qc.ann)[1:6],
                       x = "individual",
                       exprs_values = "exprs",
                       colour = "batch")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-genes-log2-cpm-1.png" alt="(\#fig:norm-genes-log2-cpm)Expression of the first 6 genes of the blischak data after the log2(CPM) normalisation" width="90%" />
<p class="caption">(\#fig:norm-genes-log2-cpm)Expression of the first 6 genes of the blischak data after the log2(CPM) normalisation</p>
</div>

### Upperquantile

```r
scater::plotExpression(umi.qc.ann,
                       rownames(umi.qc.ann)[1:6],
                       x = "individual",
                       exprs_values = "norm_counts",
                       colour = "batch")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-genes-UQ-1.png" alt="(\#fig:norm-genes-UQ)Expression of the first 6 genes of the blischak data after the UQ normalisation" width="90%" />
<p class="caption">(\#fig:norm-genes-UQ)Expression of the first 6 genes of the blischak data after the UQ normalisation</p>
</div>

### FPKM

```r
scater::plotExpression(umi.qc.ann,
                       rownames(umi.qc.ann)[1:6],
                       x = "individual",
                       exprs_values = "fpkm",
                       colour = "batch")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-genes-fpkm-1.png" alt="(\#fig:norm-genes-fpkm)Expression of the first 6 genes of the blischak data after the FPKM normalisation" width="90%" />
<p class="caption">(\#fig:norm-genes-fpkm)Expression of the first 6 genes of the blischak data after the FPKM normalisation</p>
</div>

### TPM

```r
scater::plotExpression(umi.qc.ann,
                       rownames(umi.qc.ann)[1:6],
                       x = "individual",
                       exprs_values = "tpm",
                       colour = "batch")
```

<div class="figure" style="text-align: center">
<img src="11-exprs-norm_files/figure-html/norm-genes-tpm-1.png" alt="(\#fig:norm-genes-tpm)Expression of the first 6 genes of the blischak data after the TPM normalisation" width="90%" />
<p class="caption">(\#fig:norm-genes-tpm)Expression of the first 6 genes of the blischak data after the TPM normalisation</p>
</div>

## Exercise

Perform the same analysis with read counts of the Blischak data. Use `blischak/reads.rds` file to load the reads SCESet object. Once you have finished please compare your results to ours (next chapter).
