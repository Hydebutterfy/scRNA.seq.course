---
knit: bookdown::preview_chapter
---

# Clustering analysis



Once we have normalized the data and removed confounders we can carry out analyses that will allow us to interpret the data biologically. The exact nature of the analysis depends on the dataset and the biological question at hand. Nevertheless, there are a few operations which are useful in a wide range of contexts and we will be discussing some of them. We will start with the clustering of scRNA-seq data.

## Introduction

One of the most promising applications of scRNA-seq is the discovery
and annotation of cell-types based on the transcription
profiles. Computationally, this is a hard problem as it amounts to
__unsupervised clustering__. That is, we need to identify groups of
cells based on the similarities of the transcriptomes without any
prior knowledge of the labels. The problem is made more challenging
due to the high level of noise and the large number of dimensions
(i.e. genes). 

## Dimensionality reductions

When working with large datasets, it can often be beneficial to apply
some sort of dimensionality reduction method. By projecting
the data onto a lower-dimensional sub-space, one is often able to
significantly reduce the amount of noise. An additional benefit is
that it is typically much easier to visualize the data in a 2 or
3-dimensional subspace. Here we will introduce some of the popular dimensionality reduction methods.

### PCA {#clust-pca}

[Principal component analysis (PCA)](https://en.wikipedia.org/wiki/Principal_component_analysis) is a statistical procedure that uses a transformation to convert a set of observations into a set of values of linearly uncorrelated variables called principal components (PCs). The number of principal components is less than or equal to the number of original variables.

PCA is defined in such a way that the first principal component accounts for as much of the variability in the data as possible, and each succeeding component in turn has the highest variance possible under the constraint that it is orthogonal to the preceding components.

<div class="figure" style="text-align: center">
<img src="figures/pca.png" alt="(\#fig:clust-pca)Schematic representation of PCA dimensionality reduction (taken from [here](http://www.nlpca.org/pca_principal_component_analysis.html))" width="100%" />
<p class="caption">(\#fig:clust-pca)Schematic representation of PCA dimensionality reduction (taken from [here](http://www.nlpca.org/pca_principal_component_analysis.html))</p>
</div>

### Spectral

Spectral decomposition is the factorization of a matrix into a canonical form, whereby the matrix is represented in terms of its eigenvalues and eigenvectors.

In application to scRNA-seq data, the matrix can be either an input expression matrix, or matrix of distances between the cells. The computed eigenvectors are similar to the projections of the data to PCs (chapter \@ref(clust-pca)).

### tSNE

## Clustering methods

__Unsupervised clustering__ is useful in many different applications and
it has been widely studied in machine learning. Some of the most
popular approaches are discussed below.

### Hierarchical clustering

In [hierarchical clustering](https://en.wikipedia.org/wiki/Hierarchical_clustering), one can use either a bottom-up or a
top-down approach. In the former case, each cell is initially assigned to
its own cluster and pairs of clusters are subsequently merged to
create a hieararchy:

<div class="figure" style="text-align: center">
<img src="figures/hierarchical_clustering1.png" alt="(\#fig:clust-hierarch-raw)Raw data" width="30%" />
<p class="caption">(\#fig:clust-hierarch-raw)Raw data</p>
</div>

<div class="figure" style="text-align: center">
<img src="figures/hierarchical_clustering2.png" alt="(\#fig:clust-hierarch-dendr)The hierarchical clustering dendrogram" width="50%" />
<p class="caption">(\#fig:clust-hierarch-dendr)The hierarchical clustering dendrogram</p>
</div>

With a top-down strategy, one instead starts with
all observations in one cluster and then recursively split each
cluster to form a hierarchy. One of the
advantages of this strategy is that the method is deterministic.

### k-means

In [_k_-means clustering](https://en.wikipedia.org/wiki/K-means_clustering), the goal is to partition _N_ cells into _k_
different clusters. In an iterative manner, cluster centers are
assigned and each cell is assigned to its nearest cluster:

<div class="figure" style="text-align: center">
<img src="figures/k-means.png" alt="(\#fig:clust-k-means)Schematic representation of the _k_-means clustering" width="100%" />
<p class="caption">(\#fig:clust-k-means)Schematic representation of the _k_-means clustering</p>
</div>

Most methods for scRNA-seq analysis includes a _k_-means step at some point.

### Graph-based methods

Over the last two decades there has been a lot of interest in
analyzing networks in various domains. One goal is to identify groups
or modules of nodes in a network. Some of these methods can be applied
to scRNA-seq data and one example is the  method, which is based
on the concept of identifying groups of tightly connected nodes.

## Challenges in clustering

* What is the number of clusters _k_?
* __Scalability__: in the last 2 years the number of cells in scRNA-seq experiments has grown by 2 orders of magnitude from ~$10^2$ to ~$10^4$
* Tools are not user-friendly

## Tools for scRNA-seq data

### [SINCERA](https://research.cchmc.org/pbge/sincera.html)

* Based on hierarchical clustering
* Data is converted to _z_-scores before clustering
* Identify _k_ by finding the first singleton cluster in the hierarchy

### [pcaReduce](https://github.com/JustinaZ/pcaReduce)

* Based on PCA

### [SC3](http://bioconductor.org/packages/SC3/)

* Based on PCA and spectral dimensionality reductions
* Utilises _k_-means
* Additionally performs the consensus clustering

### tSNE+_k_-means

### [SNN-Cliq](http://bioinfo.uncc.edu/SNNCliq/)
