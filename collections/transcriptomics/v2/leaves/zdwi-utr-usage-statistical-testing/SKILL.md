---
name: zdwi-utr-usage-statistical-testing
description: Use when scRNA-seq data has been quantified at 3'UTR isoform level (e.g.
  SingleCellExperiment from scUTRquant with the utrome_hg38_v1 target) and you need
  to test which genes show significant 3'UTR shortening/lengthening per condition
  or perturbation relative to controls.
when_to_use_negative:
- The question is about gene expression abundance rather than 3'UTR isoform length/usage
  — the article shows these are largely independent axes of regulation
- Input is bulk RNA-seq with reads spanning actual 3' cleavage sites, where standard
  APA tools rather than single-cell UTR isoform quantification apply
- The zdWUI matrix already exists and the task is clustering perturbations (walktrap
  rounds k=5/4/3), not statistical testing
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: scUTRboot
  role: Statistical testing package that computes significant dWUI differences in
    3'UTR isoform usage across samples/conditions
- name: scUTRquant
  role: Upstream Snakemake pipeline producing the gene and 3'UTR isoform SingleCellExperiment
    counts that dWUI/zdWUI are computed from
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying pseudoalignment and barcode-correction workflow on which scUTRquant's
    isoform quantification is built
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:017f83a02d46a3da19bcfd086f7235c517f2bceb41c0452217b63261f63ec8f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# zdwi-utr-usage-statistical-testing

## Summary

Statistical testing of 3'UTR isoform usage changes with scUTRboot, computing dWUI (difference in Weighted UTR Isoform usage) and z-scaling it (zdWUI) relative to a control group (e.g. 97 non-targeting guides) to produce a perturbation-by-gene matrix of significant 3'UTR-length responses.

## When to use

Use when scRNA-seq data has been quantified at 3'UTR isoform level (e.g. SingleCellExperiment from scUTRquant with the utrome_hg38_v1 target) and you need to test which genes show significant 3'UTR shortening/lengthening per condition or perturbation relative to controls.

## When NOT to use

- The question is about gene expression abundance rather than 3'UTR isoform length/usage — the article shows these are largely independent axes of regulation
- Input is bulk RNA-seq with reads spanning actual 3' cleavage sites, where standard APA tools rather than single-cell UTR isoform quantification apply
- The zdWUI matrix already exists and the task is clustering perturbations (walktrap rounds k=5/4/3), not statistical testing

## Inputs

- 3'UTR isoform SingleCellExperiment object from scUTRquant (utrome_hg38_v1 target)
- Gene-level SingleCellExperiment object from scUTRquant
- Perturbation metadata mapping guides to cells (e.g. K562 Perturb-seq guide assignments)
- Non-targeting control guide list used as reference (97 guides in the K562 GWPS analysis)

## Outputs

- Perturbation-by-gene matrix of zdWUI z-scores (e.g. 836 perturbations x 883 response genes)
- Per-gene-per-condition dWUI significance calls (3'UTR shortening/lengthening)

## How to apply

Start from scUTRquant gene-level and 3'UTR-isoform SingleCellExperiment objects produced with kallisto bus pseudoalignment, bustools correct barcode correction, and min_umis cell filtering. For each perturbation (or sample group), aggregate single-cell UTR isoform counts to a pseudobulk profile, then use scUTRboot to compute dWUI z-scores relative to the control set — in the K562 Perturb-seq analysis, the 97 non-targeting control guides served as the reference population. This yields a perturbation-by-response-gene matrix of significant 3'UTR-length responses (e.g. 836 perturbations x 883 response genes after effect-size/quality filtering). Rationale: scUTRboot provides the statistical layer that makes 3'UTR quantification usable in standard scRNA-seq downstream analysis, and zdWUI z-scores normalize responses so perturbations with similar UTR-length patterns can be clustered. Decisions at this stage are which controls define the baseline and which perturbations/genes pass effect-size and quality filters before downstream clustering.

## Related tools

- **scUTRboot** (Statistical testing package that computes significant dWUI differences in 3'UTR isoform usage across samples/conditions)
- **scUTRquant** (Upstream Snakemake pipeline producing the gene and 3'UTR isoform SingleCellExperiment counts that dWUI/zdWUI are computed from) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying pseudoalignment and barcode-correction workflow on which scUTRquant's isoform quantification is built)

## Evaluation signals

- The zdWUI matrix has the expected dimensions after filtering (836 perturbation rows x 883 response-gene columns in the K562 GWPS replication)
- Non-targeting control guides used as the z-score reference should be centered (no strong global shortening/lengthening signal in controls)
- Every zdWUI call is relative to the defined control set (97 non-targeting guides), not to an ad-hoc baseline
- Upstream SingleCellExperiment objects passed min_umis cell filtering and were pseudobulked one profile per perturbation before testing
- Downstream walktrap clustering on the matrix yields exactly 18 perturbation clusters as validated against perturbation_clusters.csv

## Limitations

- Most 10x Genomics reads do not span the mRNA 3' end cleavage sites, so 3'UTR usage is resolved against a fixed UTRome annotation rather than exact de novo 3' end mapping
- Transcript annotations should be filtered to protein-coding transcripts with validated 3' ends, excluding mRNA_end_NF-tagged transcripts, before quantification
- Cells below the min_umis threshold must be excluded, and tests require pseudobulk aggregation — single-cell-level dWUI testing is not what the pipeline supports here

## Evidence

- [discussion] Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis: "Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis"
- [intro] and a statistical testing package, called scUTRboot: "and a statistical testing package, called scUTRboot"
- [methods] 'min_umis': minimum number of UMIs per cell; cells below this threshold are excluded: "'min_umis': minimum number of UMIs per cell; cells below this threshold are excluded"
