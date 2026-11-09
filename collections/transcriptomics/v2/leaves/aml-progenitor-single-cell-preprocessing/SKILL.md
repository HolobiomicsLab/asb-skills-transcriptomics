---
name: aml-progenitor-single-cell-preprocessing
description: Use when scRNA-seq FASTQ (e.g., AML CD34+ aligned with Cell Ranger to
  hg19) must be split into expression-defined populations plus splice-junction PSI
  subclusters, and decision-level integration is needed to expose splicing-defined
  disease subtypes.
when_to_use_negative:
- When a joint latent-space integration such as Seurat WNN is expected to suffice
  and no modality-specific (splicing) subtypes are of interest — the paper reports
  WNN fails to resolve the splicing-defined subpopulations
- When the input lacks exon-exon junction counts or PSI-quantifiable splicing events
  (fewer than 25 cells per event, delta-PSI <= 0.1), since the NMF splicing modality
  cannot be constructed
- When cells already carry a single trusted annotation and no conflicting clustering
  solutions need to be reconciled
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scTriangulate
  role: Decision-level mix-and-match integration of RNA and splicing cluster solutions;
    concat_rna_and_other, add_annotation, add_umap; version 0.9.0
  repo: https://github.com/frankligy/scTriangulate
- name: Cell Ranger 2.1.1
  role: Aligns AML CD34+ FASTQ to hg19 to produce the gene-expression count matrix
- name: ICGS2 in AltAnalyze
  role: Identifies gene-expression cell populations and runs NMF (k = 3) on PSI values
    for splicing subclusters
- name: MultiPath-PSI (with Pysam)
  role: Quantifies percent-spliced-in values from exon-exon junction counts with the
    reported PSI filters
- name: Seurat-WNN
  role: Competing multimodal integration (50 PCs RNA, 30 PCs PSI) used to demonstrate
    scTriangulate's superiority on splicing-defined subtypes
- name: scanpy 1.7.2
  role: Preprocessing/QC backbone; scanpy_recipe wrapper and AnnData handling for
    scTriangulate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:f2cf4a020b29c84f43db1e468adacd8478b37e8fbef001580c4b32f12aaee4f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - Cell Ranger 2.1.1
  - ICGS2 in AltAnalyze
  - MultiPath-PSI
  - MultiPath-PSI (with Pysam)
  - Seurat-WNN
  - scTriangulate
  - scanpy
  - scanpy 1.7.2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# aml-progenitor-single-cell-preprocessing

## Summary

Preprocesses pediatric AML CD34+ single-cell data into gene-expression clusters (ICGS2) and splicing-defined subclusters (NMF on PSI values), then reconciles them with scTriangulate to resolve splicing-defined leukemia blood progenitor subpopulations that Seurat WNN cannot resolve.

## When to use

Use when scRNA-seq FASTQ (e.g., AML CD34+ aligned with Cell Ranger to hg19) must be split into expression-defined populations plus splice-junction PSI subclusters, and decision-level integration is needed to expose splicing-defined disease subtypes.

## When NOT to use

- When a joint latent-space integration such as Seurat WNN is expected to suffice and no modality-specific (splicing) subtypes are of interest — the paper reports WNN fails to resolve the splicing-defined subpopulations
- When the input lacks exon-exon junction counts or PSI-quantifiable splicing events (fewer than 25 cells per event, delta-PSI <= 0.1), since the NMF splicing modality cannot be constructed
- When cells already carry a single trusted annotation and no conflicting clustering solutions need to be reconciled

## Inputs

- AML CD34+ FASTQ files (10x single-cell RNA-seq)
- Cell Ranger count matrix aligned to hg19
- Exon-exon junction counts for PSI quantification
- PSI event matrix filtered to >= 25 cells, delta-PSI > 0.1, FDR-corrected p < 0.05
- Normalized RNA expression (log CPTT) plus PSI values in an AnnData object with conflicting annotation columns for scTriangulate

## Outputs

- ICGS2/cellHarmony-defined gene-expression cell populations
- NMF (k = 3) splicing subcluster labels per population, under both ignore-missing and median-imputed PSI handling
- scTriangulate decision-level cluster assignments (Shapley-based, win_fraction_cutoff = 0.25)
- Seurat WNN integrated clustering (50 PCs RNA, 30 PCs PSI) as a comparison solution
- UMAP/visualizations showing retention of splicing-defined subpopulations marked by KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events

## How to apply

Align AML CD34+ FASTQ with Cell Ranger 2.1.1 to hg19 and derive gene-expression cell populations using ICGS2 in AltAnalyze with default parameters. Quantify percent-spliced-in values from exon-exon junction counts with Pysam + MultiPath-PSI, retaining events with PSI in >= 25 cells, delta-PSI > 0.1 and empirical Bayes t-test p < 0.05 (limma default, FDR-corrected), augmented with concordant MarkerFinder events. Derive candidate splicing subclusters by non-negative matrix factorization in ICGS2 (k = 3 per cellHarmony-defined population), handling missing PSI values both by ignoring and by median-imputing to test robustness. Concatenate normalized RNA and pre-imputed PSI values into a combined AnnData via scTriangulate.preprocessing.concat_rna_and_other and run scTriangulate with win_fraction_cutoff = 0.25 for the final decision-level assignment, then run Seurat WNN (50 PCs for RNA, 30 PCs for PSI) as the competing integration to benchmark against.

## Related tools

- **scTriangulate** (Decision-level mix-and-match integration of RNA and splicing cluster solutions; concat_rna_and_other, add_annotation, add_umap; version 0.9.0) — https://github.com/frankligy/scTriangulate
- **Cell Ranger 2.1.1** (Aligns AML CD34+ FASTQ to hg19 to produce the gene-expression count matrix)
- **ICGS2 in AltAnalyze** (Identifies gene-expression cell populations and runs NMF (k = 3) on PSI values for splicing subclusters)
- **MultiPath-PSI (with Pysam)** (Quantifies percent-spliced-in values from exon-exon junction counts with the reported PSI filters)
- **Seurat-WNN** (Competing multimodal integration (50 PCs RNA, 30 PCs PSI) used to demonstrate scTriangulate's superiority on splicing-defined subtypes)
- **scanpy 1.7.2** (Preprocessing/QC backbone; scanpy_recipe wrapper and AnnData handling for scTriangulate)

## Examples

```
from sctriangulate import *; from sctriangulate.preprocessing import *; sctri = ScTriangulate(dir='./output', adata=adata, query=['icgs2_rna', 'nmf_psi_k3']); sctri.lazy_run()
```

## Evaluation signals

- Splicing event QC invariant: retained events have PSI measured in >= 25 cells, delta-PSI > 0.1, and empirical Bayes t-test p < 0.05 (limma default, FDR-corrected)
- Acceptance check mirroring Supplementary Fig. 10f,g: the scTriangulate solution must retain splicing-defined subclusters marked by KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events, while the Seurat-WNN solution does not resolve them
- Robustness check: candidate NMF subclusters (k = 3) are concordant whether missing PSI values are ignored or median-imputed
- Pruning invariant: clusters with winning fraction wfc < 0.25 are labeled unstable in the scTriangulate output
- Version pinning: reproducible runs use scTriangulate 0.9.0 with scanpy 1.7.2, matching the paper's methods

## Limitations

- Multimodal integration via Seurat WNN on the same modalities fails to resolve the splicing-defined leukemia progenitor subpopulations, so WNN is not an acceptable substitute
- Integration of dozens of annotation sources significantly increases runtime; scTriangulate automatically substitutes Shapley selection with combined rank-based selection when over 15 annotation sets are provided
- Programmatic default analytical settings may not be optimal for all study designs, and population naming remains largely non-standardized

## Evidence

- [methods] we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function.: "we concatenate both properly normalized RNA and ADT values to a combined AnnData through scTriangulate.preprocessing.concat_rna_and_other function."
- [methods] By default, wfc < 0.25 will be labeled as unstable clusters.: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [readme] scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions.: "scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions."
- [other] retaining events with PSI in >= 25 cells, delta-PSI > 0.1 and empirical Bayes t-test p < 0.05 (limma default, FDR-corrected): [evidence span withheld — anchor status: not_found]
- [methods] All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2.: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
