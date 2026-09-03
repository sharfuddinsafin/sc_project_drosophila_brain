# Drosophila Brain Cocaine Response --- Single-Cell RNA-seq Analysis

## Project Overview

This project analyzes single-cell RNA-sequencing data from *Drosophila
melanogaster* brains exposed to cocaine or sucrose in female and male
biological groups. The analysis follows the supplied Project 2 workflow
and evaluates:

1.  Construction of a Drosophila brain single-cell atlas.
2.  Identification of cocaine-responsive cell populations and genes.
3.  Sex-specific differences in the transcriptional response to cocaine.
4.  Biological pathways associated with cocaine-responsive genes.

The complete analysis report is
`sc_analysis_report_drosophila_brain.pdf`.

## Dataset

Eight 10x-style datasets were analyzed, representing two biological
replicates for each sex × treatment combination.

  Sample             Condition             Cells
  ------------------ ------------------ --------
  Female_Cocaine_1   Female + Cocaine     13,072
  Female_Cocaine_2   Female + Cocaine      9,367
  Female_Sucrose_1   Female + Sucrose      9,072
  Female_Sucrose_2   Female + Sucrose     11,693
  Male_Cocaine_1     Male + Cocaine       10,437
  Male_Cocaine_2     Male + Cocaine       11,124
  Male_Sucrose_1     Male + Sucrose       13,193
  Male_Sucrose_2     Male + Sucrose       11,033

**Initial merged dataset:** 88,991 cells × 17,481 genes.

## Analysis Workflow

``` text
Raw 10x data
    ↓
Merge eight samples
    ↓
Quality control and gene filtering
    ↓
Library-size normalization
    ↓
log1p transformation
    ↓
2,000 highly variable genes
    ↓
Scaling / regression
    ↓
PCA
    ↓
Nearest-neighbor graph
    ↓
UMAP
    ↓
Leiden clustering
    ↓
Cluster marker analysis
    ↓
Sex-stratified differential expression
    ↓
Pathway enrichment
```

## Quality Control

The workflow attempted to identify mitochondrial genes using the `mt:`
prefix and ribosomal genes using `RPS/RPL/rps/rpl`.

Filtering criteria:

-   Cells with fewer than 200 detected genes were removed.
-   Genes detected in fewer than 3 cells were removed.
-   Cells with mitochondrial percentage ≥10% were intended to be
    removed.

### QC Outcome

-   Cells before QC: **88,991**
-   Cells after QC: **88,980**
-   Cells removed: **11**
-   Genes before filtering: **17,481**
-   Genes after filtering: **13,141**
-   Genes removed: **4,340**

### Important QC Caveat

The executed output reported **0.0% mitochondrial and 0.0% ribosomal
counts for all cells**. Because the working feature identifiers use
`Dmel_`-prefixed IDs, this is interpreted as a likely
feature-identification failure rather than true biological absence.
Consequently, mitochondrial/ribosomal QC was non-informative in the
current run.

## Normalization and Feature Selection

-   Raw counts were stored in an AnnData layer.
-   Counts were normalized to a target library size of 10,000
    counts/cell.
-   `log1p` transformation was applied.
-   **2,000 highly variable genes** were selected using sample as the
    batch key.
-   Total counts and mitochondrial percentage were regressed.
-   Expression values were scaled with a maximum value of 10.

## Dimensionality Reduction and Clustering

The workflow used:

-   2,000 HVGs
-   30 principal components
-   15-nearest-neighbor graph
-   UMAP
-   Leiden clustering at resolution **0.8**

### Clustering Result

The supplied UMAP contains labels **0--13**, corresponding to **14
visible clusters**.

The reference study reported approximately **36 stable clusters at
resolution 0.8**. Therefore, the current pipeline does not reproduce the
reference atlas at the expected cellular resolution.

## Cell-Type Annotation

Cluster marker genes were ranked using the Wilcoxon test. The notebook
attempted to identify canonical markers including:

`repo`, `elav`, `ey`, `Fas2`, `Gad1`, `VGlut`, `ple`, `SerT`, and
`Tdc2`.

These markers were not recovered in the working feature names/raw
object. Therefore, confident annotation of Kenyon cells, glia,
dopaminergic, cholinergic, GABAergic, glutamatergic, serotonergic, or
octopaminergic populations is **not supported by the current analysis**.

## Differential Expression

Male and female cells were analyzed separately for **Cocaine vs
Sucrose** using Scanpy's Wilcoxon rank-sum procedure.

Significance criteria:

``` text
|log2 fold change| > 1.0
Adjusted P < 0.05
```

### DEG Summary

  Sex        Significant DEGs   Upregulated   Downregulated   Shared
  -------- ------------------ ------------- --------------- --------
  Male                    104            88              16       37
  Female                  183           162              21       37

There were **37 significant genes shared between sexes**.

The current result differs from the strong male-biased DEG response
reported in the reference study. This should not be interpreted as
evidence that females are biologically more cocaine-responsive because
the current workflow differs from the reference analysis and does not
explicitly model biological replicates.

## Pathway Enrichment

Male significant DE genes were analyzed with GSEApy/Enrichr after
stripping the `Dmel_` prefix.

The ranked pathway output included categories associated with:

-   Neurotransmitter receptor complexes
-   Acetylcholine-gated channel complexes
-   Terminal boutons
-   Plasma membrane components
-   Phagocytic vesicles
-   Peroxisomal membrane
-   Mitochondrial ATP synthase
-   AMPA glutamate receptor complex
-   Glycerolipid metabolism
-   Amino-acid catabolism
-   Glutamate metabolism
-   Glucose homeostasis

These categories are compatible with neuronal signaling, synaptic
machinery, and metabolism. However, the displayed terms did **not reach
adjusted P \< 0.05**. They should therefore be treated as ranked
enrichment trends or hypotheses rather than statistically significant
pathway enrichment.

## Key Results

  Metric                        Result
  --------------------------- --------
  Initial cells                 88,991
  Post-QC cells                 88,980
  Initial genes                 17,481
  Retained genes                13,141
  Highly variable genes          2,000
  Leiden clusters                   14
  Male significant DEGs            104
  Female significant DEGs          183
  Male upregulated DEGs             88
  Male downregulated DEGs           16
  Female upregulated DEGs          162
  Female downregulated DEGs         21
  Shared significant DEGs           37

## Main Scientific Conclusion

The supplied Scanpy analysis establishes a single-cell processing
framework for the eight Drosophila brain samples and detects
cocaine-associated transcriptional changes in both sexes.

However, it does not reproduce the reference atlas or its reported
male-biased cocaine response. The most defensible conclusion is that the
current pipeline identifies **sex-specific cocaine-associated
transcriptional remodeling**, while technical and statistical
limitations prevent strong cell-type-specific claims.

In particular, the following need correction before publication-level
interpretation:

-   mitochondrial/ribosomal feature annotation;
-   Drosophila gene-symbol mapping;
-   cluster resolution;
-   cell-type annotation;
-   replicate-aware differential expression;
-   statistically supported pathway analysis.

## Limitations

1.  **Gene identifier mapping:** `Dmel_`-prefixed IDs prevented direct
    canonical marker recovery.
2.  **Mitochondrial/ribosomal QC:** percentages were 0% across cells,
    indicating unsuccessful feature classification.
3.  **Cluster resolution:** 14 clusters were obtained instead of
    approximately 36.
4.  **Differential-expression design:** cell-level Wilcoxon testing does
    not explicitly model the two biological replicates per condition.
5.  **Cell-type-specific response:** current DE was sex-stratified
    rather than cluster-specific.
6.  **Pathway significance:** pathway plots use a forced visualization
    cutoff, so visual prominence does not imply statistical
    significance.

## Recommended Next Steps

1.  Correct mitochondrial and ribosomal gene annotation.
2.  Establish reliable FlyBase/Drosophila gene-symbol mapping.
3.  Re-run QC with validated feature classes.
4.  Investigate the discrepancy between the 14-cluster result and the
    reference 36-cluster solution.
5.  Annotate clusters using validated Drosophila brain markers.
6.  Perform replicate-aware, preferably pseudobulk, differential
    expression.
7.  Perform cocaine-vs-sucrose analysis within biologically defined cell
    types.
8.  Re-run pathway enrichment using statistically supported gene sets.
9.  Report FDR-adjusted pathway statistics explicitly.
10. Reassess sex-specific responses after these corrections.

## Figures

The supplied output inventory contains:

``` text
Figure1_QC_metrics.png
Figure2_UMAP_clusters.png
Figure3_top_cluster_markers.png
Figure4_pathway_dotplot.png
Figure5_pathway_enrichment.png
Figure6_HVG_selection.png
Figure7_PCA_variance.png
```

The report provides interpretation of these figures.

## Reference Study

Baker BM, Mokashi SS, Shankar V, Hatfield JS, Hannah RC, Mackay TFC,
Anholt RRH. **The Drosophila brain on cocaine at single-cell
resolution.** *Genome Research*. 2021;31(10):1927--1938.

The reference study used GEO accession **GSE152495**, reported
approximately 36 clusters, and described stronger cocaine-associated
transcriptional responses in males.

## Project Documentation

Main report:

``` text
sc_analysis_report_drosophila_brain.pdf
```

The report contains the detailed Methods, Results, figure
interpretations, Discussion, Conclusion, References, and AI Usage
Disclosure.

## AI Usage Disclosure

AI assistance was used to organize the supplied Project 2 materials into
a structured academic report, improve scientific wording, and identify
discrepancies between expected project deliverables and supplied
outputs.

**AI model:** ChatGPT (GPT-5.6 Luna)

The numerical findings documented here are derived from the supplied
report and analysis outputs.

## Reproducibility

This README documents the analysis represented in the supplied report.
It is a project-level overview and does not replace the original
Python/Scanpy notebook.

For exact computational reproduction, use the supplied Python notebook
together with the original `.h5ad` datasets and analysis files.
