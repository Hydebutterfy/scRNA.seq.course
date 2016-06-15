---
# knit: bookdown::preview_chapter
output: html_document
---

# DE expression in a real dataset



```r
library(scRNA.seq.funcs)
library(M3Drop)
library(limma)
set.seed(1)
```

One of the key differences between single-cell RNASeq and bulk RNASeq
is the large number of dropouts (zero expression) for genes with even
moderately high levels of average expression. These zeros violate
assumptions made by many statistical tools used for bulk RNASeq,
eg. Negative Binomial expression used by DESeq2, normality assumptions
of correlation methods, or assumptions of few tied-ranks for many
non-parametric tests. Some recent scRNASeq methods model a different
dropout rate for each gene (eg. [MAST](https://github.com/RGLab/MAST),
[BASiCS](https://github.com/catavallejos/TutorialBASiCS)) while other
methods try to fit the relationship between expression level and
dropout rate across genes for a specific application (eg. SCDE,
ZIFA). We will be using our new package Michaelis-Menten Modelling of
Dropouts (M3Drop) which specifically focuses on modelling and gaining
biological insights from dropouts.

For this section we will be working with the [Usoskin et
al](http://www.nature.com/neuro/journal/v18/n1/full/nn.3881.html)
data. It contains 4 cell types: NP = non-peptidergic nociceptors, PEP
= peptidergic nociceptors, NF = neurofilament containing and TH =
tyrosine hydroxylase containing


```r
usoskin1 <- readRDS("usoskin/usoskin1.rds")
dim(usoskin1)
```

```
## [1] 25334   622
```

```r
table(colnames(usoskin1))
```

```
## 
##  NF  NP PEP  TH 
## 139 169  81 233
```

## Fitting the models

First we must normalize & QC the dataset. M3Drop contains a built-in
function for this which removes cells with few detected genes, removes
undetected genes, and converts raw counts to CPM.


```r
uso_list <- M3Drop::M3Drop_Clean_Data(
    usoskin1,
    labels = colnames(usoskin1),
    min_detected_genes = 2000,
    is.counts = TRUE
)
```

__Exercise__: How many cells & genes have been removed by this filtering? Do you agree with the 2000 detected genes threshold?

## The Michaelis-Menten Equation

The Michaelis-Menten (MM) Equation is the standard model of enzyme
kinetics. We use it to model dropouts in single-cell RNASeq because
most dropouts occur as a result of failing to be reverse-transcribed
to sufficient levels. The details are omitted here, but in this model
the probability that a transcript will not be found is given by

$$P_{dropout} = K/(K + S)$$

where $S$ is the mRNA concentration in the cell (we will estimate this as average expression) and $K$ is the Michaelis-Menten constant. 


Other models that have been proposed are:

* $P=Logistic(\log(S))$ used by [SCDE](http://hms-dbmi.github.io/scde/) (determining differential expression) and
* $P=e^{-\lambda*S^2}$ used by [ZIFA (zero-inflated PCA)](http://genomebiology.biomedcentral.com/articles/10.1186/s13059-015-0805-z).

Now we will fit all three models to the normalized Usoskin data:

```r
models <- M3Drop::M3Drop_Dropout_Models(uso_list$data)
title(main = "Usoskin")
```

<img src="20-dropouts_files/figure-html/unnamed-chunk-5-1.png" width="816" style="display: block; margin: auto;" />

## Right outliers

There are many outliers to the right of the fitted MM curve. Genes
which are expressed at different levels in subpopulations of our cells
will be shifted to the right of the curve. This happens because the MM
curve is a convex function, whereas averaging dropout rate and
expression is a linear function.

We use M3Drop to identify significant outliers to the right of the MM
curve. We also apply 1% FDR multiple testing correction:


```r
DE_genes <- M3Drop::M3Drop_Differential_Expression(
    uso_list$data,
    mt_method = "fdr",
    mt_threshold = 0.01
) 
title(main = "Usoskin")
```

<img src="20-dropouts_files/figure-html/unnamed-chunk-6-1.png" width="672" style="display: block; margin: auto;" />

Check which of the known neuron markers are identified as DE:

```r
uso_markers <- 
    c("Nefh", "Tac1", "Mrgprd", "Th", "Vim", "B2m", 
      "Col6a2", "Ntrk1", "Calca", "P2rx3", "Pvalb")
rbind(uso_markers, uso_markers %in% DE_genes$Gene) 
```

```
##             [,1]   [,2]   [,3]     [,4]   [,5]   [,6]    [,7]     [,8]   
## uso_markers "Nefh" "Tac1" "Mrgprd" "Th"   "Vim"  "B2m"   "Col6a2" "Ntrk1"
##             "TRUE" "TRUE" "TRUE"   "TRUE" "TRUE" "FALSE" "FALSE"  "TRUE" 
##             [,9]    [,10]   [,11]  
## uso_markers "Calca" "P2rx3" "Pvalb"
##             "TRUE"  "TRUE"  "TRUE"
```

## Validation of DE results

We can also plot the expression levels of these genes to check they really are DE genes.

```r
M3Drop::M3Drop_Expression_Heatmap(
    DE_genes$Gene,
    uso_list$data,
    cell_labels = uso_list$labels,
    key_genes = uso_markers
)
```

<img src="20-dropouts_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />

## Comparing M3Drop to other methods

We can compare the genes identified as DE using M3Drop to those
identified using other methods. Running differential expression
methods which compare two groups at a time is slow for this dataset (6
possible pairs of groups x 15,708 genes) thus we have provided you
with the output for DESeq2 as \code{DESeq_table}. We can also use the
Brennecke method introduced earlier to identify highly variable genes
by using the entire dataset as spike-ins.



```r
Brennecke_HVG <- M3Drop::Brennecke_getVariableGenes(
    uso_list$data,
    fdr = 0.01,
    minBiolDisp = 0.5
)
```

<img src="20-dropouts_files/figure-html/unnamed-chunk-9-1.png" width="672" style="display: block; margin: auto;" />

```r
length(Brennecke_HVG)
```

```
## [1] 1595
```

```r
DESeq_table <- readRDS("usoskin/DESeq_table.rds")
length(unique(DESeq_table$Gene))
```

```
## [1] 2604
```

__Exercise__: Plot a heatmap of the expression of the HVGs and DESeq DE genes. Do they look differentially expressed?

__Exercise__: How many of the known markers are identified by Brennecke & DESeq?

Finally, we can compare the overlaps between these three dataset.


```r
all.genes <- unique(
    c(
        as.character(DESeq_table$Gene),
        Brennecke_HVG,
        as.character(DE_genes$Gene)
    )
)
venn.diag <- vennCounts(
    cbind(
        all.genes %in% as.character(DESeq_table$Gene),
        all.genes %in% Brennecke_HVG,
        all.genes %in% as.character(DE_genes$Gene)
    )
)
limma::vennDiagram(
    venn.diag,
    names = c("DESeq", "HVG", "M3Drop"),
    circle.col = c("magenta", "blue", "green")
)
```

<img src="20-dropouts_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />
