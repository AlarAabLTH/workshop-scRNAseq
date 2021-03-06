---
title: "Seurat: Quality control"
author: "Åsa Björklund  &  Paulo Czarnewski"
date: "Sept 13, 2019"
output:
  html_document:
    self_contained: true
    highlight: tango
    df_print: paged
    toc: yes
    toc_float:
      collapsed: false
      smooth_scroll: true
    toc_depth: 3
    keep_md: yes
    fig_caption: true
  html_notebook:
    self_contained: true
    highlight: tango
    df_print: paged
    toc: yes
    toc_float:
      collapsed: false
      smooth_scroll: true
    toc_depth: 3
---


<style>
h1, .h1, h2, .h2, h3, .h3, h4, .h4 { margin-top: 50px }
p.caption {font-size: 0.9em;font-style: italic;color: grey;margin-right: 10%;margin-left: 10%;text-align: justify}
</style>

***
# Get data

In this tutorial, we will be using 3 publicly available dataset downloaded from 10X Genomics repository. They can be downloaded using the following bash commands. Simply create a folder called `data` and then use `curl` to pull the data from the 10X database.


```bash
mkdir data
curl -o data/pbmc_1k_v2_filtered_feature_bc_matrix.h5 -O http://cf.10xgenomics.com/samples/cell-exp/3.0.0/pbmc_1k_v2/pbmc_1k_v2_filtered_feature_bc_matrix.h5
curl -o data/pbmc_1k_v3_filtered_feature_bc_matrix.h5 -O http://cf.10xgenomics.com/samples/cell-exp/3.0.0/pbmc_1k_v3/pbmc_1k_v3_filtered_feature_bc_matrix.h5
curl -o data/pbmc_1k_protein_v3_filtered_feature_bc_matrix.h5 -O http://cf.10xgenomics.com/samples/cell-exp/3.0.0/pbmc_1k_protein_v3/pbmc_1k_protein_v3_filtered_feature_bc_matrix.h5
```

With data in place, now we can start loading libraries we will use in this tutorial.


```r
suppressMessages(require(Seurat))
suppressMessages(require(Matrix))
```

We can first load the data individually by reading directly from HDF5 file format (.h5). Note that among those , the dataset p3.1k actually has both gene expression and CITE-seq data, so we will use only the `Gene Expression` here.


```r
v3.1k <- Read10X_h5("data/pbmc_1k_v3_filtered_feature_bc_matrix.h5", use.names = T)
v2.1k <- Read10X_h5("data/pbmc_1k_v2_filtered_feature_bc_matrix.h5", use.names = T)
p3.1k <- Read10X_h5("data/pbmc_1k_protein_v3_filtered_feature_bc_matrix.h5", use.names = T)
```

```
## Genome matrix has multiple modalities, returning a list of matrices for this genome
```

```r
p3.1k <- p3.1k$`Gene Expression`
```

***
# Create one merged object

We can now load the expression matricies into objects and then merge them into a single merged object. Each analysis workflow (Seurat, Scater, Scranpy, etc) has its own way of storing data. We will add dataset labels as cell.ids just in case you have overlapping barcodes between the datasets. After that we add a column `Chemistry` in the metadata for plotting later on.


```r
sdata.v2.1k <- CreateSeuratObject(v2.1k, project = "v2.1k")
sdata.v3.1k <- CreateSeuratObject(v3.1k, project = "v3.1k")
sdata.p3.1k <- CreateSeuratObject(p3.1k, project = "p3.1k")

# Merge datasets into one single seurat object
alldata <- merge(sdata.v2.1k, c(sdata.v3.1k,sdata.p3.1k), add.cell.ids=c("v2.1k","v3.1k","p3.1k"))

# Add in a metadata column that indicates v2 vs v3 chemistry
alldata$Chemistry <- ifelse(alldata$orig.ident == "v2.1k","v2","v3")
```

 Here it is how the count matrix and the metatada look like for every cell.


```r
as.data.frame(alldata@assays$RNA@counts[1:10,1:2])
head(alldata@meta.data,10)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["v2.1k_AAACCTGAGCGCTCCA-1"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["v2.1k_AAACCTGGTGATAAAC-1"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"0","_rn_":"MIR1302-2HG"},{"1":"0","2":"0","_rn_":"FAM138A"},{"1":"0","2":"0","_rn_":"OR4F5"},{"1":"0","2":"0","_rn_":"AL627309.1"},{"1":"0","2":"0","_rn_":"AL627309.3"},{"1":"0","2":"0","_rn_":"AL627309.2"},{"1":"0","2":"0","_rn_":"AL627309.4"},{"1":"0","2":"0","_rn_":"AL732372.1"},{"1":"0","2":"0","_rn_":"OR4F29"},{"1":"0","2":"0","_rn_":"AC114498.1"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div><div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["orig.ident"],"name":[1],"type":["chr"],"align":["left"]},{"label":["nCount_RNA"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["nFeature_RNA"],"name":[3],"type":["int"],"align":["right"]},{"label":["Chemistry"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"v2.1k","2":"6631","3":"2029","4":"v2","_rn_":"v2.1k_AAACCTGAGCGCTCCA-1"},{"1":"v2.1k","2":"2196","3":"881","4":"v2","_rn_":"v2.1k_AAACCTGGTGATAAAC-1"},{"1":"v2.1k","2":"2700","3":"791","4":"v2","_rn_":"v2.1k_AAACGGGGTTTGTGTG-1"},{"1":"v2.1k","2":"3551","3":"1183","4":"v2","_rn_":"v2.1k_AAAGATGAGTACTTGC-1"},{"1":"v2.1k","2":"3080","3":"1333","4":"v2","_rn_":"v2.1k_AAAGCAAGTCTCTTAT-1"},{"1":"v2.1k","2":"5769","3":"1556","4":"v2","_rn_":"v2.1k_AAAGCAATCCACGAAT-1"},{"1":"v2.1k","2":"4655","3":"1221","4":"v2","_rn_":"v2.1k_AAAGTAGGTAGCAAAT-1"},{"1":"v2.1k","2":"5230","3":"1803","4":"v2","_rn_":"v2.1k_AAATGCCGTCTAGAGG-1"},{"1":"v2.1k","2":"3735","3":"1129","4":"v2","_rn_":"v2.1k_AACACGTCACCTCGGA-1"},{"1":"v2.1k","2":"3988","3":"1347","4":"v2","_rn_":"v2.1k_AACACGTCATGGTTGT-1"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


***
# Calculate QC

Having the data in a suitable format, we can start calculating some quality metrics. We can for example calculate the percentage of mitocondrial and ribosomal genes per cell and add to the metadata. This will be helpfull to visualize them across different metadata parameteres (i.e. datasetID and chemistry version). There are several ways of doing this, and here manually calculate the proportion of mitochondrial reads and add to the metadata table.

Citing from "Simple Single Cell" workflows (Lun, McCarthy & Marioni, 2017): "High proportions are indicative of poor-quality cells (Islam et al. 2014; Ilicic et al. 2016), possibly because of loss of cytoplasmic RNA from perforated cells. The reasoning is that mitochondria are larger than individual transcript molecules and less likely to escape through tears in the cell membrane."


```r
# Way1: Doing it using Seurat function
alldata <- PercentageFeatureSet(alldata, "^MT-", col.name = "percent_mito")

# Way2: Doing it manually
total_counts_per_cell <- colSums(  alldata@assays$RNA@counts  )
mito_genes <- rownames(alldata)[grep("^MT-",rownames(alldata))]
alldata$percent_mito <- colSums(  alldata@assays$RNA@counts[mito_genes,]  ) / total_counts_per_cell

head(mito_genes,10)
```

```
##  [1] "MT-ND1"  "MT-ND2"  "MT-CO1"  "MT-CO2"  "MT-ATP8" "MT-ATP6" "MT-CO3" 
##  [8] "MT-ND3"  "MT-ND4L" "MT-ND4"
```

In the same manner we will calculate the proportion gene expression that comes from ribosomal proteins.


```r
# Way1: Doing it using Seurat function
alldata <- PercentageFeatureSet(alldata, "^RP[SL]", col.name = "percent_ribo")

# Way2: Doing it manually
ribo_genes <- rownames(alldata)[grep("^RP[SL]",rownames(alldata))]
head(ribo_genes,10)
alldata$percent_ribo <- colSums(  alldata@assays$RNA@counts[ribo_genes,]  ) / total_counts_per_cell
```

```
##  [1] "RPL22"   "RPL11"   "RPS6KA1" "RPS8"    "RPL5"    "RPS27"   "RPS6KC1"
##  [8] "RPS7"    "RPS27A"  "RPL31"
```

***
# Plot QC

Now we can plot some of the QC-features as violin plots.


```r
feats <- c("nFeature_RNA","nCount_RNA","percent_mito","percent_ribo")
VlnPlot(alldata, group.by= "orig.ident", features = feats, pt.size = 0.1,ncol = 4) + NoLegend()
```

![](seurat_01_qc_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

As you can see, the v2 chemistry gives lower gene detection, but higher detection of ribosomal proteins. As the ribosomal proteins are highly expressed they will make up a larger proportion of the transcriptional landscape when fewer of the lowly expressed genes are detected. And we can plot the different QC-measures as scatter plots.


```r
cowplot::plot_grid(ncol = 4,
  FeatureScatter(alldata, "nCount_RNA"  , "nFeature_RNA", group.by = "orig.ident", pt.size = .5),
  FeatureScatter(alldata, "percent_mito", "nFeature_RNA", group.by = "orig.ident", pt.size = .5),
  FeatureScatter(alldata, "percent_ribo", "nFeature_RNA", group.by = "orig.ident", pt.size = .5),
  FeatureScatter(alldata, "percent_ribo", "percent_mito", group.by = "orig.ident", pt.size = .5)
)
```

![](seurat_01_qc_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

***
# Filtering

## Detection-based filtering

A standard approach is to filter cells with low amount of reads as well as genes that are present in at least a certain amount of cells. Here we will only consider cells with at least 200 detected genes and genes need to be expressed in at least 3 cells. Please note that those values are highly dependent on the library preparation method used.


```r
selected_c <- WhichCells(alldata, expression = nFeature_RNA > 200)
selected_f <- rownames(alldata)[ Matrix::rowSums(alldata) > 3]

data.filt <- subset(alldata, features=selected_f, cells=selected_c)
dim(data.filt)
```

```
## [1] 16157  2869
```

 Extremely high number of detected genes could indicate doublets. However, depending on the cell type composition in your sample, you may have cells with higher number of genes (and also higher counts) from one cell type. <br>In these datasets, there is also a clear difference between the v2 vs v3 10x chemistry with regards to gene detection, so it may not be fair to apply the same cutoffs to all of them. Also, in the protein assay data there is a lot of cells with few detected genes giving a bimodal distribution. This type of distribution is not seen in the other 2 datasets. Considering that they are all PBMC datasets it makes sense to regard this distribution as low quality libraries. Filter the cells with high gene detection (putative doublets) with cutoffs 4100 for v3 chemistry and 2000 for v2. <br>Here, we will filter the cells with low gene detection (low quality libraries) with less than 1000 genes for v2 and < 500 for v2.


```r
#start with cells with many genes detected.
high.det.v3 <- WhichCells(data.filt, expression = nFeature_RNA > 4100)
high.det.v2 <- WhichCells(data.filt, expression = nFeature_RNA > 2000 & orig.ident == "v2.1k")

# remove these cells
data.filt <- subset(data.filt, cells=setdiff(WhichCells(data.filt),c(high.det.v2,high.det.v3)))

# check number of cells
ncol(data.filt)
```

```
## [1] 2797
```

Additionally, we can also see which genes contribute the most to such reads. We can for instance plot the percentage of counts per gene.


```r
#Compute the relative expression of each gene per cell
rel_expression <- t( t(data.filt@assays$RNA@counts) / Matrix::colSums(data.filt@assays$RNA@counts)) * 100
most_expressed <- sort(Matrix::rowSums( rel_expression ),T)[20:1] / ncol(data.filt)

par(mfrow=c(1,2),mar=c(4,6,1,1))
boxplot( as.matrix(t(rel_expression[names(most_expressed),])),cex=.1, las=1, xlab="% total count per cell",col=scales::hue_pal()(20)[20:1],horizontal=TRUE)
```

![](seurat_01_qc_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

As you can see, MALAT1 constitutes up to 30% of the UMIs from a single cell and the other top genes are mitochondrial and ribosomal genes. It is quite common that nuclear lincRNAs have correlation with quality and mitochondrial reads, so high detection of MALAT1 may be a technical issue. Let us assemble some information about such genes, which are important for quality control and downstream filtering.

## Mito/Ribo filtering

We also have quite a lot of cells with high proportion of mitochondrial and low proportion ofribosomal reads. It could be wise to remove those cells, if we have enough cells left after filtering. <br>Another option would be to either remove all mitochondrial reads from the dataset and hope that the remaining genes still have enough biological signal. <br>A third option would be to just regress out the `percent_mito` variable during scaling. In this case we had as much as 99.7% mitochondrial reads in some of the cells, so it is quite unlikely that there is much cell type signature left in those. <br>Looking at the plots, make reasonable decisions on where to draw the cutoff. In this case, the bulk of the cells are below 25% mitochondrial reads and that will be used as a cutoff. We will also remove cells with less than 5% ribosomal reads. 


```r
selected_mito <- WhichCells(data.filt, expression = percent_mito < 0.25)
selected_ribo <- WhichCells(data.filt, expression = percent_ribo > 0.05)

# and subset the object to only keep those cells
data.filt <- subset(data.filt, cells = selected_mito)
data.filt <- subset(data.filt, cells = selected_ribo)
dim(data.filt)
```

```
## [1] 16157  2599
```

As you can see, there is still quite a lot of variation in `percent_mito`, so it will have to be dealt with in the data analysis step. We can also notice that the `percent_ribo` are also highly variable, but that is expected since different cell types have different proportions of ribosomal content, according to their function.

## Plot filtered QC

Lets plot the same QC-stats another time.


```r
feats <- c("nFeature_RNA","nCount_RNA","percent_mito","percent_ribo")
cowplot::plot_grid(ncol = 1,
VlnPlot(data.filt, group.by= "orig.ident", features = feats, pt.size = 0.1,ncol = 4) + NoLegend())
```

![](seurat_01_qc_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

# Calculate cell-cycle scores

We here perform cell cycle scoring. To score a gene list, the algorithm calculates the difference of mean expression of the given list and the mean expression of reference genes. To build the reference, the function randomly chooses a bunch of genes matching the distribution of the expression of the given list. Cell cycle scoring adds three slots in data, a score for S phase, a score for G2M phase and the predicted cell cycle phase.


```r
data.filt <- CellCycleScoring(object = data.filt,
                              g2m.features = cc.genes$g2m.genes,
                              s.features = cc.genes$s.genes)
```

We can now plot a violin plot for the cell cycle scores as well.


```r
VlnPlot(data.filt, features = c("S.Score","G2M.Score"), group.by= "orig.ident",ncol = 4, pt.size = .1)
```

![](seurat_01_qc_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

In this case it looks like we only have a few cycling cells in the datasets.
# Save data 
Finally, lets save the QC-filtered data for further analysis.

#CELLCYCLE_ALL4:


```r
saveRDS(data.filt,"data/3pbmc_qc.rds")
```

### Session Info
***


```r
sessionInfo()
```

```
## R version 3.5.1 (2018-07-02)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## Running under: macOS  10.15
## 
## Matrix products: default
## BLAS/LAPACK: /Users/asbj/miniconda3/envs/sc_course/lib/R/lib/libRblas.dylib
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] Matrix_1.2-17   Seurat_3.0.1    RJSONIO_1.3-1.2 optparse_1.6.4 
## 
## loaded via a namespace (and not attached):
##  [1] tsne_0.1-3          nlme_3.1-141        bitops_1.0-6       
##  [4] bit64_0.9-7         RColorBrewer_1.1-2  httr_1.4.1         
##  [7] sctransform_0.2.0   tools_3.5.1         backports_1.1.5    
## [10] R6_2.4.1            irlba_2.3.3         KernSmooth_2.23-15 
## [13] lazyeval_0.2.2      colorspace_1.4-1    withr_2.1.2        
## [16] npsurv_0.4-0        gridExtra_2.3       tidyselect_0.2.5   
## [19] bit_1.1-14          compiler_3.5.1      hdf5r_1.2.0        
## [22] plotly_4.9.1        labeling_0.3        caTools_1.17.1.2   
## [25] scales_1.0.0        lmtest_0.9-37       ggridges_0.5.1     
## [28] pbapply_1.4-2       stringr_1.4.0       digest_0.6.23      
## [31] rmarkdown_1.17      R.utils_2.9.0       pkgconfig_2.0.3    
## [34] htmltools_0.4.0     bibtex_0.4.2        htmlwidgets_1.5.1  
## [37] rlang_0.4.2         zoo_1.8-6           jsonlite_1.6       
## [40] ica_1.0-2           gtools_3.8.1        dplyr_0.8.3        
## [43] R.oo_1.23.0         magrittr_1.5        Rcpp_1.0.3         
## [46] munsell_0.5.0       reticulate_1.13     ape_5.3            
## [49] lifecycle_0.1.0     R.methodsS3_1.7.1   stringi_1.4.3      
## [52] yaml_2.2.0          gbRd_0.4-11         MASS_7.3-51.4      
## [55] gplots_3.0.1.1      Rtsne_0.15          plyr_1.8.4         
## [58] grid_3.5.1          parallel_3.5.1      gdata_2.18.0       
## [61] listenv_0.7.0       ggrepel_0.8.1       crayon_1.3.4       
## [64] lattice_0.20-38     cowplot_1.0.0       splines_3.5.1      
## [67] SDMTools_1.1-221.1  zeallot_0.1.0       knitr_1.26         
## [70] pillar_1.4.2        igraph_1.2.4.1      reshape2_1.4.3     
## [73] future.apply_1.3.0  codetools_0.2-16    glue_1.3.1         
## [76] evaluate_0.14       lsei_1.2-0          metap_1.1          
## [79] data.table_1.11.6   vctrs_0.2.0         png_0.1-7          
## [82] Rdpack_0.11-0       gtable_0.3.0        getopt_1.20.3      
## [85] RANN_2.6.1          purrr_0.3.3         tidyr_1.0.0        
## [88] future_1.15.1       assertthat_0.2.1    ggplot2_3.2.1      
## [91] xfun_0.11           rsvd_1.0.2          survival_2.44-1.1  
## [94] viridisLite_0.3.0   tibble_2.1.3        cluster_2.1.0      
## [97] globals_0.12.4      fitdistrplus_1.0-14 ROCR_1.0-7
```




