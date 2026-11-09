---
name: altanalyze-icgs2-gene-expression-clustering
description: Use when single-cell FASTQ/count data needs gene-expression-defined cell
  population annotations as one modality-specific clustering input to a mix-and-match
  integration, prior to NMF splicing subclustering or scTriangulate reconciliation.
when_to_use_negative:
- When gene-expression population labels already exist (e.g., prior label-transfer
  annotations) — the ICGS2 clustering step would be redundant and only the decision-level
  integration applies
- When the goal is joint embedding-based multimodal integration rather than decision-level
  mix-and-match — the task card shows Seurat WNN failed to resolve the splicing-defined
  subpopulations that ICGS2-based populations support
- When raw counts have not been processed through Cell Ranger to hg19 (or an equivalent
  genome reference), since ICGS2 expects the aligned expression matrix
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0203
tools:
- name: AltAnalyze (ICGS2)
  role: 'Primary tool: identifies gene-expression cell populations from the aligned
    count matrix with default parameters, providing the RNA-defined clustering solution'
- name: Cell Ranger 2.1.1
  role: Aligns AML CD34+ FASTQ reads to hg19 to produce the count matrix input for
    ICGS2
- name: cellHarmony
  role: Supervised/label-transfer step whose population definitions define the k=3
    NMF-PSI subclustering strata derived from the ICGS2 clusters
- name: scTriangulate
  role: Consumes the ICGS2 annotations as one query annotation set, reconciling them
    with splicing-defined subclusters via Shapley-value decision-level integration
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy 1.7.2
  role: Hosts preprocessing and QC (scanpy_recipe) and the AnnData object into which
    ICGS2 annotations and UMAP coordinates are added
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:61058de2e55530d7d603d3469ada61921fd852bafd73ecdc73bdb0ca98f3e7a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - AltAnalyze (ICGS2)
  - Cell Ranger 2.1.1
  - ICGS2 in AltAnalyze
  - MultiPath-PSI
  - cellHarmony
  - scTriangulate
  - scanpy
  - scanpy 1.7.2
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# altanalyze-icgs2-gene-expression-clustering

## Summary

Defines gene-expression cell populations from single-cell RNA-Seq by running ICGS2 in AltAnalyze with default parameters on Cell Ranger-aligned hg19 count data, producing the RNA-defined annotation layer that downstream decision-level integration (scTriangulate) mixes with splicing-defined subclusters.

## When to use

Use when single-cell FASTQ/count data needs gene-expression-defined cell population annotations as one modality-specific clustering input to a mix-and-match integration, prior to NMF splicing subclustering or scTriangulate reconciliation.

## When NOT to use

- When gene-expression population labels already exist (e.g., prior label-transfer annotations) — the ICGS2 clustering step would be redundant and only the decision-level integration applies
- When the goal is joint embedding-based multimodal integration rather than decision-level mix-and-match — the task card shows Seurat WNN failed to resolve the splicing-defined subpopulations that ICGS2-based populations support
- When raw counts have not been processed through Cell Ranger to hg19 (or an equivalent genome reference), since ICGS2 expects the aligned expression matrix

## Inputs

- Single-cell FASTQ files (AML CD34+ pediatric AML blasts)
- Cell Ranger 2.1.1 count matrix aligned to hg19
- AltAnalyze/ICGS2 configuration (default parameters)

## Outputs

- Gene-expression-defined cell population (cluster) annotations per cell from ICGS2
- cellHarmony-defined population labels used to stratify downstream NMF splicing subclustering (k=3)
- AnnData-embedded annotation and UMAP layers for downstream scTriangulate integration

## How to apply

First align the AML CD34+ FASTQ files with Cell Ranger 2.1.1 to the hg19 transcriptome, yielding the per-cell count matrix. Run ICGS2 in AltAnalyze with default parameters on the resulting counts to identify gene-expression cell populations (the leukemia blood progenitor populations in the pediatric AML task). These ICGS2/cellHarmony-defined populations then serve as (a) the parent populations within which NMF-PSI subclustering (k=3) is performed and (b) one annotation set ('player') supplied to scTriangulate alongside splicing-derived subclusters. Annotations and UMAP coordinates are added to an AnnData object via scTriangulate.preprocessing.add_annotation and add_umap before concat_rna_and_other and lazy_run (win_fraction_cutoff = 0.25). The ICGS2 clustering is the gene-expression arm that scTriangulate must reconcile against the PSI-based arm; validation is qualitative, checking that marker-defined populations (e.g., KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events) remain resolvable in the integrated solution.

## Related tools

- **AltAnalyze (ICGS2)** (Primary tool: identifies gene-expression cell populations from the aligned count matrix with default parameters, providing the RNA-defined clustering solution)
- **Cell Ranger 2.1.1** (Aligns AML CD34+ FASTQ reads to hg19 to produce the count matrix input for ICGS2)
- **cellHarmony** (Supervised/label-transfer step whose population definitions define the k=3 NMF-PSI subclustering strata derived from the ICGS2 clusters)
- **scTriangulate** (Consumes the ICGS2 annotations as one query annotation set, reconciling them with splicing-defined subclusters via Shapley-value decision-level integration) — https://github.com/frankligy/scTriangulate
- **scanpy 1.7.2** (Hosts preprocessing and QC (scanpy_recipe) and the AnnData object into which ICGS2 annotations and UMAP coordinates are added)

## Examples

```
cellranger count --id=aml_cd34 --transcriptome=hg19 --fastqs=AML_CD34_FASTQ/   # then ICGS2 in AltAnalyze (default parameters) on the resulting hg19 count matrix
```

## Evaluation signals

- Cell Ranger alignment must use hg19 as reference and version 2.1.1 for faithful reproduction
- ICGS2 must be run with default parameters — deviations make the result non-comparable to the published scTriangulate comparison
- The ICGS2-derived gene-expression populations should correspond to the leukemia blood progenitor populations reported, i.e., they subdivide known cell types
- Marker-defined populations (KLF2/GATA2/GATA1/SPINK2 and LAT2/NUCB2 events) must remain distinguishable after downstream integration with the PSI subclusters via scTriangulate (win_fraction_cutoff = 0.25)
- Qualitative acceptance check: the scTriangulate solution retains splicing-defined subclusters while the competing Seurat WNN solution (50 PCs RNA, 30 PCs PSI) fails to resolve them

## Limitations

- Default analytical settings are programmatic defaults and may not be optimal for all study designs — the article explicitly cautions that reported settings 'will not be optimal for all study designs'
- Validation is a qualitative acceptance check mirroring Supplementary Fig. 10f,g, not a quantitative metric
- Population naming remains largely non-standardized, an unmet challenge acknowledged in the discussion
- ICGS2 output is one input annotation set; integrating dozens of annotation sources significantly increases scTriangulate runtime

## Evidence

- [other] task_007 workflow step 1: [evidence span withheld — anchor status: not_found]
- [discussion] default settings caveat: "While the analytical settings applied in these studies were large programmatic defaults, it is likely these settings will not be optimal for all study designs"
- [readme] supervised clustering support: "Integrate results from both unsupervised and supervised (i.e. cellHarmony, Seurat label transfer) clustering algorithms."
- [discussion] runtime scaling caveat: "the integration of cluster predictions from dozens of sources (e.g., resolutions, modalities, reference annotations), will significantly increase runtime"
