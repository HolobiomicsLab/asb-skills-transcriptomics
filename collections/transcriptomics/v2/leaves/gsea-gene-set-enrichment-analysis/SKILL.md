---
name: gsea-gene-set-enrichment-analysis
description: 'Use when a cluster from joint mRNA/protein WNN clustering (e.g., Seurat
  FindClusters algorithm=3, resolution=2) needs identity confirmation: rank its transcript
  signature and test enrichment against a curated cell-type gene set (e.g., MAIT-cell
  signature).'
when_to_use_negative:
- When deciding between protein normalization methods itself — use the empty-droplet
  98th-percentile correlation diagnostic (CLR r≈0.8 vs dsb r≈0.13) rather than GSEA
- When no curated gene set signature exists for the candidate cell identity, or the
  cluster is defined only by protein (ADT) signal with no transcript ranking available
- When the input is raw, unnormalized ADT counts — GSEA operates on transcript-level
  enrichment and cannot resolve ambient antibody noise
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment in which GSEA is run on cluster 14 transcripts and results (NES,
    p-value) are written to output files
- name: Seurat
  role: Produces the joint mRNA+ADT WNN clustering and cluster 14 assignment whose
    transcript profile is the GSEA input
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Normalization whose denoised protein values (via DSBNormalizeProtein with
    denoise.counts=TRUE, use.isotype.control=TRUE, and empty-drop background matrix)
    ensure cluster 14 is TCR-Va7.2-exclusive, making the GSEA validation meaningful
  repo: https://github.com/niaid/dsb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:04c13b6de5ea4398f1142e08daaa223a0323752dde48fb5b92c5765736a69321
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-data
  tools:
  - R
  - Seurat
  - dsb
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gsea-gene-set-enrichment-analysis

## Summary

Gene set enrichment analysis (GSEA) applied to transcripts of a transcriptionally defined cluster to confirm its cell identity via a curated signature. In the TEA-seq case study, GSEA of cluster 14 transcripts against a MAIT-cell signature provides orthogonal validation that dsb normalization unmasks the MAIT population misread as conventional T cells under CLR.

## When to use

Use when a cluster from joint mRNA/protein WNN clustering (e.g., Seurat FindClusters algorithm=3, resolution=2) needs identity confirmation: rank its transcript signature and test enrichment against a curated cell-type gene set (e.g., MAIT-cell signature).

## When NOT to use

- When deciding between protein normalization methods itself — use the empty-droplet 98th-percentile correlation diagnostic (CLR r≈0.8 vs dsb r≈0.13) rather than GSEA
- When no curated gene set signature exists for the candidate cell identity, or the cluster is defined only by protein (ADT) signal with no transcript ranking available
- When the input is raw, unnormalized ADT counts — GSEA operates on transcript-level enrichment and cannot resolve ambient antibody noise

## Inputs

- Cluster-level transcript expression / ranked gene list from the joint WNN-clustered Seurat object (e.g., cluster 14 from dsb-normalized TEA-seq data)
- Curated cell-type gene set signature (e.g., MAIT-cell signature)
- Per-cluster protein values under dsb (e.g., TCR-Va7.2) for cross-modal validation

## Outputs

- GSEA results table with normalized enrichment score (NES) and p-value per signature (expected NES≈2.64, p≈1e-10)
- Enrichment results written to output files alongside dsb/CLR normalized value tables and the empty-droplet correlation comparison table

## How to apply

After joint clustering — 30 mRNA principal components combined with dsb- or CLR-normalized ADT values via FindMultiModalNeighbors WNN (dims.list = list(1:30, 1:18)) and FindClusters (algorithm=3, resolution=2) — extract the transcript expression profile of the cluster of interest (cluster 14). Run GSEA on the cluster 14 transcripts against a MAIT-cell signature and record the normalized enrichment score (NES) and p-value. Accept the enrichment as confirmatory only if cluster 14 is also the exclusively TCR-Va7.2-positive cluster under dsb and the empty-droplet correlation check shows CLR, not dsb, tracks ambient noise (r≈0.8 vs r≈0.13). The expected result for this dataset is NES≈2.64 with p≈1e-10; write the GSEA results table alongside the dsb/CLR normalized value tables and the correlation comparison table.

## Related tools

- **R** (Environment in which GSEA is run on cluster 14 transcripts and results (NES, p-value) are written to output files)
- **Seurat** (Produces the joint mRNA+ADT WNN clustering and cluster 14 assignment whose transcript profile is the GSEA input) — https://github.com/satijalab/seurat-data
- **dsb** (Normalization whose denoised protein values (via DSBNormalizeProtein with denoise.counts=TRUE, use.isotype.control=TRUE, and empty-drop background matrix) ensure cluster 14 is TCR-Va7.2-exclusive, making the GSEA validation meaningful) — https://github.com/niaid/dsb

## Evaluation signals

- GSEA of cluster 14 transcripts against the MAIT-cell signature yields NES≈2.64 and p≈1e-10
- Under dsb, cluster 14 is the only TCR-Va7.2-positive cluster at the protein level, corroborating the transcriptomic enrichment
- The ambient-noise control holds: median CLR values (not dsb) correlate with the 98th percentile empty-droplet expression (r≈0.8, p=2.4e-7 for CLR vs r≈0.13, p=0.39 for dsb)
- Reproduction accepted only when cluster 14 is exclusively TCR-Va7.2 positive under dsb and the GSEA enrichment thresholds are met
- GSEA results table, dsb/CLR normalized value tables, and correlation comparison table are all written to output files

## Limitations

- GSEA validation depends on the upstream normalization: under CLR, ambient unbound antibody noise drove cluster 14 protein values so the MAIT population was misinterpreted as conventional T cells; enrichment conclusions inherit this error
- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Assessment details on robustness of noise estimation, empty droplet definition sensitivity, batch effects, and isotype caveats are deferred to the Supplementary Note, not the main text

## Evidence

- [other] GSEA on cluster 14 transcripts against MAIT-cell signature with expected NES and p-value: [evidence span withheld — anchor status: not_found]
- [abstract] dsb improves downstream analyses by unmasking biologically meaningful cell populations: "Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations"
- [discussion] joint clustering of mRNA PCs with dsb or CLR protein data: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [abstract] validation of dsb on TEA-seq data: "analyzing the performance of dsb in eight independent datasets spanning multiple technologies, including CITE-seq, ASAP-seq, and TEA-seq"
- [readme] ambient noise in empty droplets motivating transcriptomic validation of protein-defined clusters: [evidence span withheld — anchor status: external_doc]
