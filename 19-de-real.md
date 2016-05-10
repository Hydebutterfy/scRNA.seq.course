---
knit: bookdown::preview_chapter
---

---
# knit: bookdown::preview_chapter
output: html_document
---

# DE expression in a real dataset



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

<img src="19-de-real_files/figure-html/de-real-biase-fpkm-1.png" title="" alt="" style="display: block; margin: auto;" />

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

Because the number of genes now is 500, which is about 100 times more than the number of genes we used for the synthetic dataset, this calculation will take some time. You do __not__ need to run it during the course, instead just check the results below.


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

Because the number of genes now is 500, which is about 100 times more than the number of genes we used for the synthetic dataset, this calculation will take quite a lot of time. You do __not__ need to run it during the course, instead just check the results below.


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

<img src="19-de-real_files/figure-html/de-real-comparison-1.png" title="" alt="" style="display: block; margin: auto;" />

__Exercise:__ How does this Venn diagram correspond to what you would expect based on the synthetic data? 
