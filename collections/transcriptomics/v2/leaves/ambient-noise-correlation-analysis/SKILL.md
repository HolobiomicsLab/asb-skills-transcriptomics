---
name: ambient-noise-correlation-analysis
description: Use when comparing ADT normalization strategies (CLR vs dsb) for residual
  ambient antibody noise, or when assessing whether a protein cluster's normalized
  values track empty-droplet background levels.
when_to_use_negative:
- Dataset lacks empty droplets or the raw (unfiltered) Cell Ranger output, since the
  98th-percentile empty-droplet background estimate cannot be computed
- Goal is RNA-only analysis with no ADT/protein modality, since ambient antibody noise
  is specific to ADT counts
- Data are flow cytometry, bulk, or plate-based where droplet-encapsulated unbound
  antibody noise does not apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3577
tools:
- name: dsb
  role: Denoises and ambient-corrects ADT counts via DSBNormalizeProtein using empty
    droplets and isotype controls; its residual-free output is the negative control
    showing no empty-droplet correlation
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: Performs CLR normalization (NormalizeData, normalization.method='CLR'), WNN
    joint clustering (FindMultiModalNeighbors) with 30 mRNA PCs, and FindClusters
    (algorithm = 3)
  repo: https://github.com/satijalab/seurat-data
- name: mclust
  role: Implements the per-cell 2-component Gaussian mixture model (EM) used by dsb
    to estimate the background protein mean
- name: limma
  role: Provides removeBatchEffect() used by dsb for robust matrix decomposition of
    the technical noise component
- name: R
  role: Runtime environment; dsb uses prcomp() to compute PC1 of the noise variable
    matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:b8c217c20d14911e2e11d1bd9037bfd6e4814d9f794bba5645409febbc57329d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ambient-noise-correlation-analysis

## Summary

Quantifies ambient antibody noise retained in CLR-normalized ADT values by correlating per-protein median normalized values of a target WNN cluster against the 98th percentile of empty-droplet expression, demonstrating dsb removes this correlation. Grounded in the CD14+ CD86-high monocyte cluster 3 case study from Kotliarov et al. PBMC CITE-seq data.

## When to use

Use when comparing ADT normalization strategies (CLR vs dsb) for residual ambient antibody noise, or when assessing whether a protein cluster's normalized values track empty-droplet background levels.

## When NOT to use

- Dataset lacks empty droplets or the raw (unfiltered) Cell Ranger output, since the 98th-percentile empty-droplet background estimate cannot be computed
- Goal is RNA-only analysis with no ADT/protein modality, since ambient antibody noise is specific to ADT counts
- Data are flow cytometry, bulk, or plate-based where droplet-encapsulated unbound antibody noise does not apply

## Inputs

- Raw and filtered Cell Ranger UMI matrices for RNA and ADT counts from a PBMC CITE-seq experiment (e.g., Kotliarov et al. healthy donor data)
- Empty/background droplet ADT count matrix (raw-output barcodes not in the filtered cell set, with protein reads)
- Isotype control antibody measurements for dsb denoising

## Outputs

- Per-protein median dsb- and CLR-normalized value tables for the target cluster
- Pearson correlation statistic of cluster median CLR values vs 98th-percentile empty-droplet levels (expected R = 0.67, p = 3.1e-10)
- Differential expression gene table for the target cluster (ROC-test)
- Seurat WNN cluster assignments under both normalizations

## How to apply

Load the CITE-seq dataset defining cells as filtered Cell Ranger barcodes and background droplets as remaining raw-output barcodes with protein reads. Run two parallel ADT normalizations: dsb::DSBNormalizeProtein (denoise.counts=TRUE, use.isotype.control=TRUE) and Seurat CLR, then build WNN neighbors (FindMultiModalNeighbors) combining 30 mRNA principal components with each normalized protein matrix, and cluster with FindClusters (algorithm = 3). Identify the target cluster (cluster 3 = CD14+ CD86-high monocytes under dsb) and compute per-protein median normalized values under both normalizations. Compute the Pearson correlation of the cluster's median CLR values against the 98th percentile of empty-droplet protein levels; a significant correlation (R = 0.67, p = 3.1e-10) indicates ambient noise retained in CLR, while its absence under dsb shows ambient noise was accounted for via empty droplets. Optionally run ROC-test differential expression on the cluster to confirm the phenotype (inflammatory/activation genes consistent with CD86-high).

## Related tools

- **dsb** (Denoises and ambient-corrects ADT counts via DSBNormalizeProtein using empty droplets and isotype controls; its residual-free output is the negative control showing no empty-droplet correlation) — https://cran.r-project.org/package=dsb
- **Seurat** (Performs CLR normalization (NormalizeData, normalization.method='CLR'), WNN joint clustering (FindMultiModalNeighbors) with 30 mRNA PCs, and FindClusters (algorithm = 3)) — https://github.com/satijalab/seurat-data
- **mclust** (Implements the per-cell 2-component Gaussian mixture model (EM) used by dsb to estimate the background protein mean)
- **limma** (Provides removeBatchEffect() used by dsb for robust matrix decomposition of the technical noise component)
- **R** (Runtime environment; dsb uses prcomp() to compute PC1 of the noise variable matrix)

## Examples

```
dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, raw_protein_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.control = isotype.controls)
```

## Evaluation signals

- Pearson correlation of cluster-3 median CLR values vs 98th-percentile empty-droplet levels reproduces R = 0.67 (p = 3.1e-10)
- The same correlation computed on dsb median values is absent/non-significant
- Cluster 3 under dsb-WNN is CD14+ with high CD86, and its DE genes (ROC-test) are inflammatory/activation genes consistent with the CD86-high phenotype
- Per-cell dsb output is residual-based: the technical component (PC1 of fitted µ1 + isotype matrix) has been regressed out of ambient-corrected values
- Empty-droplet definition matches the paper's rule: filtered Cell Ranger barcodes = cells; remaining raw barcodes with protein reads = background

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note
- CLR noise assessment is cluster- and dataset-specific; the R = 0.67 statistic was established on the Kotliarov et al. healthy donor PBMC data and may differ elsewhere

## Evidence

- [discussion] ambient noise correlation with empty droplets under CLR: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] background droplet definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [intro] ambient noise estimation basis: [evidence span withheld — anchor status: not_found]
- [methods] dsb ambient step uses empty droplets: "protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] dsb sensitivity to background definition: "defining a reasonable subset of barcodes as background droplets still requires care"
- [readme] package availability: "The dsb R package is available on [**CRAN: latest dsb
release**](https://CRAN.R-project.org/package=dsb)"
