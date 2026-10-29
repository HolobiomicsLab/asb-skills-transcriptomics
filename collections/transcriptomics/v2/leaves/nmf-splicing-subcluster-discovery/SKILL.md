---
name: nmf-splicing-subcluster-discovery
description: Use when single-cell RNA-seq data contains splice-junction evidence suggesting
  transcriptionally similar cells split by exon usage (e.g. pediatric AML blasts),
  and gene-expression clustering alone fails to resolve those subpopulations.
when_to_use_negative:
- When cells lack sufficient exon-exon junction coverage to compute PSI in at least
  25 cells, leaving too few informative splicing events for NMF
- When multimodal integration via Seurat WNN already resolves the subpopulations of
  interest, making decision-level reconciliation unnecessary
- When clusters are defined by surface protein or chromatin modalities rather than
  gene expression plus splicing — the article's application is restricted to RNA/splicing
  modalities
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3518
tools:
- name: ICGS2 in AltAnalyze
  role: Identifies gene-expression cell populations and runs NMF on PSI values to
    derive candidate splicing subclusters
- name: MultiPath-PSI
  role: Computes percent-spliced-in values from exon-exon junction counts together
    with Pysam
- name: scTriangulate
  role: Decision-level integration of RNA and splicing cluster assignments via concat_rna_and_other
    and lazy_run (Shapley-based mix-and-match)
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Preprocessing and QC of the combined AnnData holding normalized RNA and PSI
    values
- name: Seurat-WNN
  role: Competing baseline multimodal integration (50 PCs RNA, 30 PCs PSI) used to
    show WNN cannot resolve the splicing-defined subclusters
- name: Cell Ranger 2.1.1
  role: Aligns AML CD34+ FASTQ to hg19 for gene expression and junction quantification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:c6ca91e29b6661d4b220cb4dc8199ca3216c055eeb02e3c4e2b9faecf1b5f1c4
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# nmf-splicing-subcluster-discovery

## Summary

Discovers splicing-defined subpopulations within gene-expression-defined cell clusters by quantifying per-cell percent-spliced-in (PSI) values, applying non-negative matrix factorization (NMF, k=3 per population), and reconciling the splicing subclusters with RNA-defined clusters via scTriangulate decision-level integration.

## When to use

Use when single-cell RNA-seq data contains splice-junction evidence suggesting transcriptionally similar cells split by exon usage (e.g. pediatric AML blasts), and gene-expression clustering alone fails to resolve those subpopulations.

## When NOT to use

- When cells lack sufficient exon-exon junction coverage to compute PSI in at least 25 cells, leaving too few informative splicing events for NMF
- When multimodal integration via Seurat WNN already resolves the subpopulations of interest, making decision-level reconciliation unnecessary
- When clusters are defined by surface protein or chromatin modalities rather than gene expression plus splicing — the article's application is restricted to RNA/splicing modalities

## Inputs

- Single-cell RNA-seq FASTQ (e.g. AML CD34+ fractions) with splice-junction reads, aligned with Cell Ranger 2.1.1 to hg19
- Gene-expression-defined cluster labels from ICGS2 in AltAnalyze (cellHarmony-defined populations)
- Exon-exon junction count matrix for PSI quantification (Pysam + MultiPath-PSI)
- Normalized RNA expression and PSI matrices concatenated into a combined AnnData object

## Outputs

- Filtered splicing-event PSI matrix (events in >= 25 cells, delta-PSI > 0.1, empirical Bayes p < 0.05 FDR-corrected)
- NMF-derived splicing subclusters (k=3 per gene-expression-defined population)
- scTriangulate decision-level cluster assignments reconciling RNA and splicing solutions
- Marker/splicing-event signatures defining subclusters (e.g. KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events)

## How to apply

Quantify PSI values from exon-exon junction counts using Pysam plus MultiPath-PSI, retaining events detected in >= 25 cells, with delta-PSI > 0.1 and empirical Bayes t-test p < 0.05 (limma default, FDR-corrected), augmented with concordant MarkerFinder events. Within each ICGS2/cellHarmony-defined gene-expression population (derived in AltAnalyze from Cell Ranger-aligned FASTQ, e.g. hg19), run NMF in ICGS2 with k=3 on the PSI matrix, handling missing PSI values both by ignoring them and by median-imputing to test robustness. Normalize the RNA counts and prepend the pre-imputed PSI values into a combined AnnData using scTriangulate.preprocessing.concat_rna_and_other, then run scTriangulate with win_fraction_cutoff = 0.25 to obtain a decision-level cluster assignment. Run Seurat WNN (50 PCs for RNA, 30 PCs for PSI) on the same modalities as the competing integration to demonstrate that the scTriangulate solution resolves subclusters that WNN cannot.

## Related tools

- **ICGS2 in AltAnalyze** (Identifies gene-expression cell populations and runs NMF on PSI values to derive candidate splicing subclusters)
- **MultiPath-PSI** (Computes percent-spliced-in values from exon-exon junction counts together with Pysam)
- **scTriangulate** (Decision-level integration of RNA and splicing cluster assignments via concat_rna_and_other and lazy_run (Shapley-based mix-and-match)) — https://github.com/frankligy/scTriangulate
- **scanpy** (Preprocessing and QC of the combined AnnData holding normalized RNA and PSI values)
- **Seurat-WNN** (Competing baseline multimodal integration (50 PCs RNA, 30 PCs PSI) used to show WNN cannot resolve the splicing-defined subclusters)
- **Cell Ranger 2.1.1** (Aligns AML CD34+ FASTQ to hg19 for gene expression and junction quantification)

## Examples

```
from sctriangulate import ScTriangulate; from sctriangulate.preprocessing import concat_rna_and_other; adata = concat_rna_and_other(rna_adata, psi_df); sctri = ScTriangulate(dir='./output', adata=adata, query=['icgs2_rna','nmf_psi_k3']); sctri.lazy_run(win_fraction_cutoff=0.25)
```

## Evaluation signals

- scTriangulate solution retains splicing-defined subclusters distinguishable by the reported KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 marker/splicing events, whereas Seurat-WNN does not resolve them
- PSI filter criteria reproduce the published thresholds: events detected in >= 25 cells, delta-PSI > 0.1, and empirical Bayes t-test p < 0.05 (limma default, FDR-corrected)
- NMF subcluster results are consistent between the two missing-value strategies (ignoring missing PSI vs median-imputing)
- win_fraction_cutoff = 0.25 in scTriangulate flags unstable clusters (wfc < 0.25) that should be pruned before final assignment
- MarkerFinder events are concordant with the MultiPath-PSI-selected events used for clustering

## Limitations

- Seurat WNN (and similar joint-embedding approaches) may fail to resolve modality-specific subclusters that scTriangulate recovers, so WNN is unsuitable as the primary tool when modality-defined subtypes are the goal
- Integration of cluster predictions from dozens of sources significantly increases scTriangulate runtime; over 15 annotation sets triggers automatic substitution of Shapley selection with combined rank-based selection
- Programmatic default analytical settings may not be optimal for all study designs
- Population naming for newly resolved subclusters remains largely non-standardized

## Evidence

- [methods] PSI filter thresholds: [evidence span withheld — anchor status: not_found]
- [methods] NMF k and missing-value handling: [evidence span withheld — anchor status: not_found]
- [methods] scTriangulate integration parameters: [evidence span withheld — anchor status: not_found]
- [methods] WNN comparison design: [evidence span withheld — anchor status: not_found]
- [abstract] WNN limitation motivating decision-level integration: [evidence span withheld — anchor status: not_found]
- [readme] README: scTriangulate workflow on conflicting annotations: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])
sctri.lazy_run()"
