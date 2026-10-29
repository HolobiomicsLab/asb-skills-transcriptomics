---
name: single-cell-multimodal-trajectory-branch-subsetting
description: Use when a single-cell multimodal dataset forms a tree-like or branching
  differentiation trajectory and you need to ablate topology — testing whether full-trajectory
  DAG inference or individual branch sub-DAGs explain the observed locus-gene signal.
when_to_use_negative:
- The dataset is not a dynamic branching process — GrID-Net relies on multiomic single-cell
  data from matching tissue undergoing a dynamic process, such as differentiation,
  to provide pseudotemporal context
- You only need a single global locus-gene ranking and have no need to compare topology
  configurations; run gridnet.run_gridnet on the full DAG directly
- Cells lack lineage annotations and cannot be partitioned into biologically meaningful
  branches, making a branch-wise ablation undefined
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GrID-Net
  role: Provides construct_dag for building the full and branch-specific cell DAGs
    and run_gridnet for the full-DAG Granger causal inference benchmarked against
    branch configurations
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Learns joint RNA+ATAC cell representations (via gridnet.schema_representations)
    used to infer pseudotime and orient kNN graph edges into DAGs
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:cbb348300c312624607a6ba9ec95bd2f15dd4b1e73f40c103247dad2325f6b2c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/alexw16/gridnet
  repo_url: https://github.com/alexw16/gridnet
  tools:
  - GrID-Net
  - Schema
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-multimodal-trajectory-branch-subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Subset a single-cell multimodal trajectory (e.g., SHARE-seq mouse skin differentiation) into biological lineages/branches, build a separate cell-state DAG per branch with the same construction settings as the full trajectory, and score each branch-specific Granger-causal configuration against the full-DAG GrID-Net result to isolate the contribution of the DAG topology.

## When to use

Use when a single-cell multimodal dataset forms a tree-like or branching differentiation trajectory and you need to ablate topology — testing whether full-trajectory DAG inference or individual branch sub-DAGs explain the observed locus-gene signal.

## When NOT to use

- The dataset is not a dynamic branching process — GrID-Net relies on multiomic single-cell data from matching tissue undergoing a dynamic process, such as differentiation, to provide pseudotemporal context
- You only need a single global locus-gene ranking and have no need to compare topology configurations; run gridnet.run_gridnet on the full DAG directly
- Cells lack lineage annotations and cannot be partitioned into biologically meaningful branches, making a branch-wise ablation undefined

## Inputs

- Unprocessed AnnData objects for paired RNA-seq and ATAC-seq (SHARE-seq mouse skin multimodal dataset) with gene TSS and peak start/end annotations in adata.var
- Branch/lineage cell annotations identifying subpopulations (e.g., cortex, medulla, inner root sheath)
- Candidate peak-gene pair DataFrame from gridnet.identify_all_peak_gene_link_candidates (distance_thresh=1e6)
- Significant eQTLs (P < 1e-3) from the eQTL Catalogue for AUROC evaluation

## Outputs

- Branch-specific DAG adjacency matrices (cortex, medulla, IRS) plus the full three-branch DAG
- Per-configuration peak-gene Granger score tables (linear Granger per branch and gridnet.run_gridnet results DataFrame)
- Comparison table of eQTL-retrieval AUROCs, e.g., written to dag_vs_branches_auroc.csv

## How to apply

First preprocess the paired RNA-seq/ATAC-seq AnnData objects with gridnet.preprocess_multimodal (normalization plus log(1+x) transformation) and assemble candidate peak-gene pairs restricted to peaks within 1 Mb (distance_thresh=1e6) of each gene TSS. Learn joint RNA+ATAC cell representations via gridnet.schema_representations, then build the full three-branch DAG with gridnet.construct_dag using n_neighbors=15 and pseudotime_algo='dpt' (diffusion pseudotime orients the kNN graph edges). Subset cells into each lineage — for SHARE-seq mouse skin these are the cortex, medulla, and inner root sheath (IRS) lineages — and rebuild each branch-specific DAG with identical construct_dag settings so the only varying factor is topology. Apply linear Granger causality to the candidate peak-gene pairs under the four configurations (full DAG, cortex, medulla, IRS) and run gridnet.run_gridnet on the full DAG, then compare AUROC of eQTL retrieval against significant skin eQTLs (P < 1e-3 from the eQTL Catalogue) combined with log genomic distance, as in the paper's logistic-regression evaluation.

## Related tools

- **GrID-Net** (Provides construct_dag for building the full and branch-specific cell DAGs and run_gridnet for the full-DAG Granger causal inference benchmarked against branch configurations) — https://github.com/alexw16/gridnet
- **Schema** (Learns joint RNA+ATAC cell representations (via gridnet.schema_representations) used to infer pseudotime and orient kNN graph edges into DAGs) — https://github.com/rs239/schema

## Examples

```
X_joint = gridnet.schema_representations(rna_adata, atac_adata); dag_full = gridnet.construct_dag(X_joint, iroot, n_neighbors=15, pseudotime_algo='dpt'); dag_cortex = gridnet.construct_dag(X_joint[cortex_idx], iroot_c, n_neighbors=15, pseudotime_algo='dpt')
```

## Evaluation signals

- GrID-Net on the full DAG achieves a significantly higher eQTL-retrieval AUROC than every linear-Granger configuration (full DAG or individual cortex/medulla/IRS branches), reproducing Fig. 2c
- All configurations yield better-than-random eQTL prediction AUROC, so a below-0.5 AUROC indicates a pipeline error
- Branch DAGs must be built with identical settings to the full DAG (n_neighbors=15, pseudotime_algo='dpt') so topology is the only differing factor
- Each branch DAG adjacency matrix must be acyclic and cover exactly the cells assigned to that lineage subset
- Candidate peak-gene pairs for every configuration come from the same DataFrame (peaks within 1 Mb of TSS), so pair sets are comparable across runs

## Limitations

- Large single-cell datasets may exceed GPU memory capacity when running GrID-Net on the full DAG; sketching techniques like Geosketch or Hopper are recommended to retain biological heterogeneity
- GrID-Net infers causal hypotheses from observational data that require perturbation-based validation, which was only mimicked via eQTL cross-referencing
- Mouse skin Hi-C data was unavailable, so 5C data was used as a substitute for the SHARE-seq chromatin interaction evaluation

## Evidence

- [discussion] DAG construction settings in the article: [evidence span withheld — anchor status: not_found]
