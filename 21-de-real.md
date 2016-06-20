---
# knit: bookdown::preview_chapter
output: html_document
---

# DE in a real dataset



```r
library(scRNA.seq.funcs)
library(DESeq2)
library(scde)
library(ROCR)
library(limma)
set.seed(1)
```

## Introduction

The main advantage of using synthetic data is that we have full
control over all aspects of the data, and this facilitates the
interpretation of the results. However, the transcriptional bursting
model is unable to capture the full complexity of a real scRNA-seq
dataset. Next, we are going to analyze the difference between the
transcriptomes of the 2-cell and the 4-cell state of a mouse embryo as
described by [Biase et al](http://genome.cshlp.org/content/24/11/1787.short). For our purposes you need to download the [`biase`](http://hemberg-lab.github.io/scRNA.seq.course/biase/biase.txt) into the `biase` folder in your working directory. We can then look at the data:

```r
biase <- as.matrix(
    read.table(
        "biase/biase_et_al_2cell_4cell_fpkm.tsv"
    )
)
# keep those genes that are expressed in at least 6 cells
biase <- biase[rowSums(biase > 0) > 5, ]
pheatmap::pheatmap(
    log2(biase + 1),
    scale = "column",
    cutree_cols = 2,
    kmeans_k = 100,
    show_rownames = FALSE
)
```

<img src="21-de-real_files/figure-html/de-real-biase-fpkm-1.png" width="672" style="display: block; margin: auto;" />

As you can see, the cells cluster well by their developmental stage.

We can now use the same methods as before to obtain a list of
differentially expressed genes.

Because SCDE is very slow here we will only use a subset of genes. You should not do that with your real dataset, though. Here we do it just for demostration purposes:

```r
biase <- biase[sample(1:nrow(biase), 500), ]
```

## KS-test


```r
pVals <- rep(1, nrow(biase))
for (i in 1:nrow(biase)) {
    res <- ks.test(
        biase[i, 1:20],
        biase[i , 21:40]
    )
    # Bonferroni correction
    pVals[i] <- res$p.value * nrow(biase)
}
```

## DESeq2


```r
cond <- factor(
    c(
        rep("cell2", 20),
        rep("cell4", 20)
    )
)
dds <- DESeq2::DESeqDataSetFromMatrix(
    round(biase) + 1,
    colData = DataFrame(cond),
    design = ~ cond
)
dds <- DESeq2::DESeq(dds)
resDESeq <- DESeq2::results(dds)
pValsDESeq <- resDESeq$padj
```

## SCDE


```r
cnts <- apply(
    biase,
    2,
    function(x) {
        storage.mode(x) <- 'integer'
        return(x)
    }
)
names(cond) <- 1:length(cnts[1, ])
colnames(cnts) <- 1:length(cnts[1, ]) 
o.ifm <- scde::scde.error.models(
    counts = cnts,
    groups = cond,
    n.cores = 1,
    threshold.segmentation = TRUE,
    save.crossfit.plots = FALSE,
    save.model.plots = FALSE,
    verbose = 0,
    min.size.entries = 2
)
priors <- scde::scde.expression.prior(
    models = o.ifm,
    counts = cnts,
    length.out = 400,
    show.plot = FALSE
)
resSCDE <- scde::scde.expression.difference(
    o.ifm,
    cnts,
    priors,
    groups = cond,
    n.randomizations = 100,
    n.cores = 1,
    verbose = 0
)
# Convert Z-scores into 2-tailed p-values
pValsSCDE <- pnorm(abs(resSCDE$cZ), lower.tail = FALSE) * 2 
pValsSCDE <- p.adjust(pValsSCDE, method = "bonferroni")
```

## Comparison of the methods

```r
vennDiagram(
    vennCounts(
        cbind(
            pVals < 0.05,
            pValsDESeq < 0.05,
            pValsSCDE < 0.05
        )
    ),
    names = c("KS-test", "DESeq2", "SCDE"),
    circle.col = c("red", "blue", "green")
)
```

<img src="21-de-real_files/figure-html/de-real-comparison-1.png" width="672" style="display: block; margin: auto;" />

__Exercise 1:__ How does this Venn diagram correspond to what you would expect based on the synthetic data? 

## Visualisation

To further characterize the list of genes, we can calculate the
average fold-changes and compare the ones that were called as
differentially expressed to the ones that were not. 


```r
cell2 <- biase[, 1:20]
cell4 <- biase[, 21:40]
ksGenesChangedInds <- which(pVals<.05)
deSeqGenesChangedInds <- which(pValsDESeq<.05)
scdeGenesChangedInds <- which(pValsSCDE<.05)
ksGenesNotChangedInds <- which(pVals>=.05)
deSeqGenesNotChangedInds <- which(pValsDESeq>=.05)
scdeGenesNotChangedInds <- which(pValsSCDE>=.05)
meanFoldChange <- rowSums(cell2)/rowSums(cell4)
nGenes <- nrow(cell2)

par(mfrow=c(2,1))
hist(log2(meanFoldChange[ksGenesChangedInds]),
     breaks = -50:50,
     freq = FALSE,
     xlab = "# fold-change",
     col = rgb(1, 0, 0, 1/4),
     ylim = c(0, .4),
     xlim = c(-8, 8))
hist(log2(meanFoldChange[deSeqGenesChangedInds]),
     breaks = -50:50,
     freq = FALSE,
     xlab = "# fold-change",
     col = rgb(0, 0, 1, 1/4),
     ylim = c(0, .4),
     xlim = c(-8, 8))
```

<img src="21-de-real_files/figure-html/de-real-biase-hist-1.png" width="672" style="display: block; margin: auto;" />

__Exercise 2:__ Create the histogram of fold-changes for SCDE. Compare
the estimated fold-changes between the different methods? What do the
genes where they differ have in common?

### Volcano plots

A popular method for illustrating the difference between two
conditions is the volcano plot which compares the magnitude of the
change and the significance.


```r
par(mfrow=c(2,1))
plot(log2(meanFoldChange[ksGenesNotChangedInds]),
     -log10(pVals[ksGenesNotChangedInds]/nGenes),
     xlab = "mean expression change",
     ylab = "-log10(P-value), KS-test",
     ylim = c(0, 15),
     xlim = c(-12, 12)) 
points(log2(meanFoldChange[ksGenesChangedInds]),
       -log10(pVals[ksGenesChangedInds]/nGenes),
       col = "red")

plot(log2(meanFoldChange[deSeqGenesNotChangedInds]), 
     -log10(pValsDESeq[deSeqGenesNotChangedInds]),
     xlab = "mean expression change",
     ylab = "-log10(P-value), DESeq2",
     ylim = c(0, 15),
     xlim = c(-12, 12))
points(log2(meanFoldChange[deSeqGenesChangedInds]),
       -log10(pValsDESeq[deSeqGenesChangedInds]),
       col = "blue")
```

<img src="21-de-real_files/figure-html/de-real-biase-volcano-1.png" width="672" style="display: block; margin: auto;" />

### MA-plot

Another popular method for illustrating the difference between two
conditions is the MA-plot, in which the data has been transformed onto the M (log ratios) and A (mean average) scale:

```r
par(mfrow=c(2,1))
plot(log2(rowMeans(cell2[ksGenesNotChangedInds,])),
     log2(meanFoldChange[ksGenesNotChangedInds]),
     ylab = "mean expression change",
     xlab = "mean expression",
     ylim = c(-9, 9)) 
points(log2(rowMeans(cell2[ksGenesChangedInds,])),
       log2(meanFoldChange[ksGenesChangedInds]),
       col = "red")

plot(log2(rowMeans(cell2[deSeqGenesNotChangedInds,])),
     log2(meanFoldChange[deSeqGenesNotChangedInds]),
     ylab = "mean expression change",
     xlab = "mean expression",
     ylim = c(-9, 9))
points(log2(rowMeans(cell2[deSeqGenesChangedInds,])),
       log2(meanFoldChange[deSeqGenesChangedInds]),
       col = "blue")
```

<img src="21-de-real_files/figure-html/de-real-biase-ma-plot-1.png" width="672" style="display: block; margin: auto;" />

__Exercise 3:__ The volcano and MA-plots for the SCDE are missing - can
you generate them? Compare to the synthetic data, what do they tell
you about the properties of the genes that have changed?

### Heatmap of DE genes

Finally, we can plot heatmaps of the genes that were called as DE by
the intersection of the three.


```r
allChangedInds <- intersect(which(pValsDESeq<.05),
                            intersect(which(pValsSCDE<.05),
                                      which(pVals<.05)))
pheatmap::pheatmap(log2(1 + cbind(cell2, cell4)[allChangedInds,]),
                   cutree_cols = 2,
                   show_rownames = FALSE)
```

<img src="21-de-real_files/figure-html/de-real-biase-heatmap-1.png" width="672" style="display: block; margin: auto;" />

__Exercise 4:__ Create heatmaps for the genes that were detected by at least 2/3 methods.

