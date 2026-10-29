---
name: single-cell-splicing-psi-quantification
description: Use when single-cell (e.g., Cell Ranger-aligned FASTQ to hg19) exon-exon
  junction reads are available and the goal is to resolve modality-specific splicing-defined
  subpopulations not detectable from gene expression alone.
when_to_use_negative:
- Only a gene-expression count matrix is available with no exon-exon junction reads
  — PSI cannot be quantified and a joint-latent-space multimodal method (e.g., Seurat
  WNN) may be needed instead
- A prior joint embedding is acceptable for the modalities — scTriangulate's decision-level
  PSI integration is aimed at cases where Seurat WNN fails to resolve modality-specific
  subtypes
- More than 15 annotation sets are provided — scTriangulate automatically substitutes
  Shapley selection with combined rank-based selection to reduce runtime
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MultiPath-PSI
  role: Quantifies PSI values from single-cell exon-exon junction counts, together
    with Pysam
- name: ICGS2 in AltAnalyze
  role: Identifies gene-expression cell populations with default parameters and runs
    NMF (k = 3) on PSI values to derive splicing subclusters
- name: scTriangulate
  role: Decision-level integration of RNA and PSI NMF cluster solutions via concat_rna_and_other
    and lazy_run (Shapley-based, win_fraction_cutoff = 0.25)
  repo: https://github.com/frankligy/scTriangulate
- name: Seurat-WNN
  role: Competing multimodal baseline (50 PCs RNA, 30 PCs PSI) expected to fail to
    resolve the splicing-defined subclusters
- name: scanpy 1.7.2
  role: Preprocessing/QC framework into which scTriangulate 0.9.0 integrates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:bf765765f6aa4fb75ec62240ce2f7f83ece399fc2d10829be1ba7562b7cecf7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - Cell Ranger 2.1.1
  - ICGS2 in AltAnalyze
  - MultiPath-PSI
  - Seurat-WNN
  - scTriangulate
  - scanpy
  - scanpy 1.7.2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# single-cell splicing PSI quantification

## Summary

Derives per-cell percent-spliced-in (PSI) values from single-cell exon-exon junction counts using Pysam plus MultiPath-PSI, filters events by cell prevalence and differential splicing, and builds NMF-derived splicing subclusters that can be decision-level integrated with RNA-defined clusters via scTriangulate.

## When to use

Use when single-cell (e.g., Cell Ranger-aligned FASTQ to hg19) exon-exon junction reads are available and the goal is to resolve modality-specific splicing-defined subpopulations not detectable from gene expression alone.

## When NOT to use

- Only a gene-expression count matrix is available with no exon-exon junction reads — PSI cannot be quantified and a joint-latent-space multimodal method (e.g., Seurat WNN) may be needed instead
- A prior joint embedding is acceptable for the modalities — scTriangulate's decision-level PSI integration is aimed at cases where Seurat WNN fails to resolve modality-specific subtypes
- More than 15 annotation sets are provided — scTriangulate automatically substitutes Shapley selection with combined rank-based selection to reduce runtime

## Inputs

- FASTQ from CD34+ single-cell RNA-seq (e.g., pediatric AML blasts) aligned to hg19 with Cell Ranger 2.1.1
- Exon-exon junction count matrix per cell
- Gene-expression-defined cluster annotations (ICGS2 in AltAnalyze / cellHarmony)
- Normalized RNA expression AnnData with X and obsm['X_umap']

## Outputs

- Per-cell PSI matrix filtered to events with PSI in >= 25 cells, delta-PSI > 0.1, FDR-corrected p < 0.05
- NMF (k = 3) splicing subcluster label columns per cellHarmony-defined population
- Concatenated RNA+PSI AnnData and final scTriangulate decision-level cluster assignment (win_fraction_cutoff = 0.25)
- Marker/splicing events (e.g., KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2) distinguishing splicing-defined subpopulations

## How to apply

Align the FASTQ (e.g., Cell Ranger 2.1.1 to hg19) and quantify PSI values from exon-exon junction counts with Pysam + MultiPath-PSI, retaining events detected with PSI in >= 25 cells, delta-PSI > 0.1, and empirical Bayes t-test p < 0.05 (limma default, FDR-corrected), augmented with concordant MarkerFinder events. Within each cellHarmony-defined population, derive candidate splicing subclusters by non-negative matrix factorization in ICGS2 (k = 3 per population), handling missing PSI values both by ignoring them and by median-imputing to check robustness. Concatenate properly normalized RNA and pre-imputed PSI values into a combined AnnData via scTriangulate.preprocessing.concat_rna_and_other, then run scTriangulate (win_fraction_cutoff = 0.25) for the final decision-level assignment, which is feasible because scTriangulate makes no assumption about the distributions of raw underlying values.

## Related tools

- **MultiPath-PSI** (Quantifies PSI values from single-cell exon-exon junction counts, together with Pysam)
- **ICGS2 in AltAnalyze** (Identifies gene-expression cell populations with default parameters and runs NMF (k = 3) on PSI values to derive splicing subclusters)
- **scTriangulate** (Decision-level integration of RNA and PSI NMF cluster solutions via concat_rna_and_other and lazy_run (Shapley-based, win_fraction_cutoff = 0.25)) — https://github.com/frankligy/scTriangulate
- **Seurat-WNN** (Competing multimodal baseline (50 PCs RNA, 30 PCs PSI) expected to fail to resolve the splicing-defined subclusters)
- **scanpy 1.7.2** (Preprocessing/QC framework into which scTriangulate 0.9.0 integrates)

## Examples

```
from sctriangulate import *; from sctriangulate.preprocessing import *; sctri = ScTriangulate(dir='./output', adata=adata, query=['nmf_psi_subclusters','icgs2_rna_labels']); sctri.lazy_run()
```

## Evaluation signals

- Filtered PSI event set shows PSI detected in >= 25 cells, delta-PSI > 0.1, and limma empirical Bayes t-test p < 0.05 (FDR-corrected)
- NMF k = 3 per cellHarmony-defined population yields concordant subclusters under both 'ignore' and 'median-impute' handling of missing PSI values
- scTriangulate solution (win_fraction_cutoff < 0.25 labeled unstable) retains splicing-defined subclusters marked by KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events
- Seurat-WNN on the same modalities (50 PCs RNA, 30 PCs PSI) does NOT resolve those splicing-defined subclusters, mirroring the reported Supplementary Fig. 10f,g comparison
- Final scTriangulate assignment respects the minimum cluster size pruning default of 10 cells

## Limitations

- Missing PSI values require explicit handling (ignore vs. median-impute); different strategies may alter subcluster membership
- Splicing-defined populations must be validated qualitatively via marker/splicing events; population naming remains largely non-standardized
- Programmatic default analytical settings (e.g., win_fraction_cutoff, abs_thresh = 10) may not be optimal for all study designs
- Support for integrating multiple samples of the same modality and across technologies is planned but not yet available

## Evidence

- [methods] PSI filter thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] scTriangulate assumption-free integration rationale: "Unlike existing integration strategies that rely on modality-specific joint embedding or geometric graphs, scTriangulate makes no assumption about the distributions of raw underlying values."
- [methods] concatenating RNA and other modality into AnnData: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
- [methods] win_fraction_cutoff pruning default: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] minimum cluster size pruning default: "the default threshold of 10 produced consistent reliable results for different evaluated datasets (Supplementary Figure 7a)."
- [methods] software versions used for preprocessing: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
