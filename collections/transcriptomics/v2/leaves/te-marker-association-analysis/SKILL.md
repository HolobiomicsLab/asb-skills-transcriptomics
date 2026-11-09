---
name: te-marker-association-analysis
description: Use when a single-cell TE count matrix (locus-specific, e.g. from MATES)
  is available and cell populations have been clustered, and you need to test which
  TE loci act as markers of those populations or of co-expressed gene markers.
when_to_use_negative:
- Only subfamily-level TE counts are available and locus-level resolution is required
  — the article's locus-level improvement over subfamily-level (P = 5.48e-7) depends
  on MATES locus quantification
- The TE loci of interest are long autonomously expressed elements such as L1HS or
  identical copies like HERVK isoforms, which MATES cannot reliably distinguish
- Bulk RNA-seq data without per-cell resolution — the marker analysis relies on single-cell
  clustering of individual cells
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATES
  role: Produces the locus-specific single-cell TE count matrix (via bam_processor,
    data_processor, MATES_model, TE_quantifier) on which TE marker associations and
    clustering comparison are performed
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Computes the locus-level TE counts (URi + ceil(alpha_i x MRi) per cell) feeding
    the marker association analysis
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Existing single-cell TE quantification baseline whose limitations motivate
    locus-specific marker discovery
- name: SoloTE
  role: Existing single-cell TE quantification baseline for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:393eee8908da3e8c682961ad2b360b5d37b8ba10b5067bd52b51abc330ea3931
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - SoloTE
  - TE_quantifier
  - bam_processor
  - data_processor
  - scTE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# te-marker-association-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies marker transposable element (TE) loci associated with cell populations identified by clustering MATES locus-specific TE counts in single-cell data, mirroring how the article associated neoplastic markers (EGFR, HUERS-P1-int, HERVK-int) and immune markers (CD74, LTR2B, LTR40A1) in Smart-Seq2 human glioblastoma scRNA-seq.

## When to use

Use when a single-cell TE count matrix (locus-specific, e.g. from MATES) is available and cell populations have been clustered, and you need to test which TE loci act as markers of those populations or of co-expressed gene markers.

## When NOT to use

- Only subfamily-level TE counts are available and locus-level resolution is required — the article's locus-level improvement over subfamily-level (P = 5.48e-7) depends on MATES locus quantification
- The TE loci of interest are long autonomously expressed elements such as L1HS or identical copies like HERVK isoforms, which MATES cannot reliably distinguish
- Bulk RNA-seq data without per-cell resolution — the marker analysis relies on single-cell clustering of individual cells

## Inputs

- Locus-specific TE count matrix per cell (MATES TE_quantifier output, counts = URi + ceil(alpha_i x MRi))
- Gene expression matrix from the same cells (e.g. Smart-seq2 scRNA-seq)
- Cell cluster assignments from TE-only and gene-based clustering
- List of candidate gene markers (e.g. EGFR, CD74) and candidate TE loci (e.g. chr3|104522003|104522491)

## Outputs

- Table of TE marker associations linking TE loci (e.g. HUERS-P1-int, HERVK-int, LTR2B, LTR40A1) to cell populations and gene markers
- Clustering agreement statistics (ARI, NMI) for TE-only vs gene-expression clustering and locus-level vs subfamily-level comparison
- Differential expression assessment of specific locus-gene pairs (e.g. LTR2B locus near CD166)

## How to apply

First run MATES (bam_processor, data_processor, MATES_model, TE_quantifier) to obtain a locus-specific TE count matrix from Smart-Seq2 or 10X single-cell data, with counts computed as URi + ceil(alpha_i x MRi) per cell. Cluster cells on the TE-only count matrix and/or the gene expression matrix, then test TE marker associations for gene markers of the populations — in the glioblastoma case, neoplastic markers (EGFR, HUERS-P1-int, HERVK-int) and immune markers (CD74, LTR2B, LTR40A1). As a decision point, evaluate differential expression of candidate locus-gene pairs, e.g. the LTR2B locus chr3|104522003|104522491 near CD166, and compare locus-level versus subfamily-level clustering agreement (ARI) to justify locus-level resolution (reported P = 5.48e-7). Accept the analysis when the reported statistics (median ARI 0.105, median NMI 0.161 for TE-only vs gene-based clustering) and the marker associations are recovered.

## Related tools

- **MATES** (Produces the locus-specific single-cell TE count matrix (via bam_processor, data_processor, MATES_model, TE_quantifier) on which TE marker associations and clustering comparison are performed) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Computes the locus-level TE counts (URi + ceil(alpha_i x MRi) per cell) feeding the marker association analysis) — https://github.com/mcgilldinglab/MATES
- **scTE** (Existing single-cell TE quantification baseline whose limitations motivate locus-specific marker discovery)
- **SoloTE** (Existing single-cell TE quantification baseline for comparison)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Recover the reported median ARI of 0.105 and median NMI of 0.161 for TE-only vs gene-based cell clustering on the Smart-Seq2 glioblastoma data
- Reproduce the locus-level vs subfamily-level clustering ARI improvement with P = 5.48e-7
- Confirm the expected marker associations: neoplastic markers EGFR with HUERS-P1-int/HERVK-int and immune markers CD74 with LTR2B/LTR40A1
- Verify differential expression signal at the LTR2B locus chr3|104522003|104522491 located near CD166
- Check that the TE count matrix was produced with the expected formula URi + ceil(alpha_i x MRi) per cell and locus

## Limitations

- MATES has limited ability to quantify long autonomously expressed TEs such as L1HS spanning several thousand bases, and estimated multi-mapping rates may be inaccurate for TEs longer than 2001 bp
- Identical TE copies such as HERVK isoforms are hard to distinguish due to high sequence similarity and lack of uniquely mappable flanking regions
- MATES requires more computational resources (GPUs) than scTE and SoloTE; biological discoveries from marker analyses warrant further experimental validation

## Evidence

- [results] Locus vs subfamily clustering difference in glioblastoma: [evidence span withheld — anchor status: not_found]
- [readme] Smart-seq2 downstream TE analysis tutorial in README: [evidence span withheld — anchor status: external_doc]
