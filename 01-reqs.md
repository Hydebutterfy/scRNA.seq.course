
# Technical requirements

We tried to make this course purely [R-based](https://www.r-project.org/). However, one of the methods that we describe ([SNN-Cliq](http://bioinfo.uncc.edu/SNNCliq/)) is only partly R-based. It makes a simple _python_ call from R and requires a user to have rights to write to the current directory. You also need to download [this file](http://bioinfo.uncc.edu/SNNCliq/Cliq.txt) and put it in `~/snn-cliq/` directory.

Other than that, before running the course exercises, all you need to do is to install the following packages in your R:

[devtools](https://cran.r-project.org/web/packages/devtools/index.html) for installing packages from GitHub:

```r
install.packages("devtools")
```
[scRNA.seq.funcs](https://github.com/hemberg-lab/scRNA.seq.funcs) - R package containing our own functions used in this course:

```r
devtools::install_github("hemberg-lab/scRNA.seq.funcs")
```

[mvoutlier](https://cran.r-project.org/web/packages/mvoutlier/index.html) - for an automatic outlier detection in the [scater](https://github.com/davismcc/scater) package.

```r
install.packages("mvoutlier")
```

[M3D](https://github.com/tallulandrews/M3D) for identification of important and DE genes, developed by [Tallulah Andrews](http://www.sanger.ac.uk/people/directory/andrews-tallulah-s):

```r
devtools::install_github("tallulandrews/M3D", ref = "release")
```

[RUVSeq](https://bioconductor.org/packages/release/bioc/html/RUVSeq.html) for normalization using ERCC controls:

```r
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("RUVSeq")
```

[ROCR](https://cran.r-project.org/web/packages/ROCR/index.html) for performance estimations:

```r
install.packages("ROCR")
```

[limma](https://bioconductor.org/packages/release/bioc/html/limma.html) for plotting Venn diagrams:

```r
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("limma")
```

[DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) for calculation of DE genes:

```r
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("DESeq2")
```

[scde](http://hms-dbmi.github.io/scde/) for calculation of DE genes:

```r
devtools::install_github("hms-dbmi/scde", build_vignettes = FALSE)
```

* Installation on Mac OS X may require [this additional gfortran library](http://thecoatlessprofessor.com/programming/rcpp-rcpparmadillo-and-os-x-mavericks-lgfortran-and-lquadmath-error/):
```
curl -O http://r.research.att.com/libs/gfortran-4.8.2-darwin13.tar.bz2
sudo tar fvxz gfortran-4.8.2-darwin13.tar.bz2 -C /
```

* See the [help page](http://hms-dbmi.github.io/scde/help.html) for additional support.

[pheatmap](https://cran.r-project.org/web/packages/pheatmap/index.html) for plotting good looking heatmaps:

```r
install.packages("pheatmap")
```

[pcaMethods](http://bioconductor.org/packages/release/bioc/html/pcaMethods.html) required by __pcaReduce__ package below for unsupervised clustering of scRNA-seq data:

```r
## try http:// if https:// URLs are not supported
source("https://bioconductor.org/biocLite.R")
biocLite("pcaMethods")
```

[pcaReduce](https://github.com/JustinaZ/pcaReduce) for unsupervised clustering of scRNA-seq data ([bioRxiv](http://biorxiv.org/content/early/2015/09/08/026385)):

```r
devtools::install_github("JustinaZ/pcaReduce")
```

[Rtsne](https://cran.r-project.org/web/packages/pheatmap/index.html) for unsupervised clustering of scRNA-seq data:

```r
install.packages("Rtsne")
```

[SC3](http://bioconductor.org/packages/devel/bioc/html/SC3.html) for unsupervised clustering of scRNA-seq data ([bioRxiv](http://biorxiv.org/content/early/2016/01/13/036558)):

```r
devtools::install_github("hemberg-lab/SC3", ref = "R-old")
```

* Before running __SC3__ for the first time __only__, please start R and enter:

```r
RSelenium::checkForServer()
```


