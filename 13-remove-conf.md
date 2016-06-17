---
knit: bookdown::preview_chapter
---

# Remove confounders using controls

## Introduction

In the previous chapter we normalized for library size, effectively removing it as a confounder. Now we will consider removing other less well defined confounders from our data using the ERCC spike-in controls. Technical confounders (aka batch effects) can arise from difference in reagents, isolation methods, the lab/experimenter who performed the experiment, even which day/time the experiment was performed. 

Since the same amount of ERCC spike-in was added to each cell in our experiment we know that all the variablity we observe for these genes is due to technical noise; whereas endogenous genes are affected by both technical noise and biological variability. Technical noise can be removed by fitting a model to the spike-ins and "substracting" this from the endogenous genes. There are several methods available based on this premise (eg. [BASiCS](https://github.com/catavallejos/BASiCS), [scLVM](https://github.com/PMBio/scLVM), [RUV](http://bioconductor.org/packages/release/bioc/html/RUVSeq.html)); each using different noise models and different fitting procedures. Alternatively, one can identify genes which exhibit significant variation beyond technical noise (eg. Distance to median, [Highly variable genes](http://www.nature.com/nmeth/journal/v10/n11/full/nmeth.2645.html))




```r
library(scRNA.seq.funcs)
library(RUVSeq)
library(scater, quietly = TRUE)
library(scran)
options(stringsAsFactors = FALSE)
umi <- readRDS("blischak/umi.rds")
umi.qc <- umi[fData(umi)$use, pData(umi)$use]
endog_genes <- !fData(umi.qc)$is_feature_control
erccs <- fData(umi.qc)$is_feature_control
```

## Remove Unwanted Variation

Factors contributing to technical noise frequently appear as "batch
effects" where cells processed on different days or by different
technicians systematically vary from one another. Removing technical
noise and correcting for batch effects can frequently be performed
using the same tool or slight variants on it. We will be considering
the [Remove Unwanted Variation (RUV)](http://bioconductor.org/packages/release/bioc/html/RUVSeq.html)
method which uses [singular value decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition)
(similar to PCA) on subsets of the dataset which should be invariant
(e.g. [ERCC spike-ins](https://www.thermofisher.com/order/catalog/product/4456740)). Then the method removes the identified unwanted factors.


```r
qclust <- scran::quickCluster(umi.qc, min.size = 30)
umi.qc <- scran::computeSumFactors(umi.qc, sizes = 15, clusters = qclust)
umi.qc <- scater::normalize(umi.qc)
assayData(umi.qc)$ruv_counts <- RUVSeq::RUVg(
    round(exprs(umi.qc)),
    erccs,
    k = 1)$normalizedCounts
```

## Effectiveness 1

We evaluate the effectiveness of the normalization by inspecting the
PCA plot where shape corresponds the technical replicate and colour
corresponds to different biological samples (individuals from whom the
iPSC lines where derived). Separation of biological samples and
interspersed batches indicates that technical variation has been
removed. 


```r
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "exprs")
```

<div class="figure" style="text-align: center">
<img src="13-remove-conf_files/figure-html/rm-conf-pca-rle-1.png" alt="(\#fig:rm-conf-pca-rle)PCA plot of the blischak data after RLE normalisation" width="672" />
<p class="caption">(\#fig:rm-conf-pca-rle)PCA plot of the blischak data after RLE normalisation</p>
</div>


```r
scater::plotPCA(umi.qc[endog_genes, ],
                colour_by = "batch",
                size_by = "total_features",
                shape_by = "individual",
                exprs_values = "ruv_counts")
```

<div class="figure" style="text-align: center">
<img src="13-remove-conf_files/figure-html/rm-conf-pca-rle-ruv-1.png" alt="(\#fig:rm-conf-pca-rle-ruv)PCA plot of the blischak data after RLE and RUV normalisations" width="672" />
<p class="caption">(\#fig:rm-conf-pca-rle-ruv)PCA plot of the blischak data after RLE and RUV normalisations</p>
</div>

## Effectiveness 2

We can also examine the relative log expression (RLE) across cells to
confirm technical noise has been removed from the dataset.


```r
boxplot(list(scRNA.seq.funcs::calc_cell_RLE(exprs(umi.qc), erccs),
             scRNA.seq.funcs::calc_cell_RLE(assayData(umi.qc)$ruv_counts, erccs)))
```

<div class="figure" style="text-align: center">
<img src="13-remove-conf_files/figure-html/rm-conf-rle-comp-1.png" alt="(\#fig:rm-conf-rle-comp)Comparison of the relative log expression of the blischak data before and after the RUV normalisation" width="672" />
<p class="caption">(\#fig:rm-conf-rle-comp)Comparison of the relative log expression of the blischak data before and after the RUV normalisation</p>
</div>

## Exercise

Perform the same analysis with read counts of the Blischak data. Use `blischak/reads.rds` file to load the reads SCESet object. Once you have finished please compare your results to ours (next chapter). Additionally, experiment with other combinations of normalizations and compare the results.
