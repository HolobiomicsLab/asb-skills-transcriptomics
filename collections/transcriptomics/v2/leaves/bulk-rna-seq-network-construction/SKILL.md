---
name: bulk-rna-seq-network-construction
description: Use when bulk RNA-seq expression matrices (e.g., TCGA LUAD tumor vs normal)
  must be transformed into per-sample gene-gene association networks to compare pathway
  connectivity (e.g., EGFR/EGF degrees) at single-sample resolution.
when_to_use_negative:
- The analysis goal is causal network construction or edge direction inference — the
  authors state c-CSN is not designed for causal networks and 'the directions of the
  gene associations cannot be obtained'.
- The question requires group-level differential co-expression only and per-sample
  resolution is unnecessary — a standard population-level correlation network suffices
  and avoids the G-fold computational cost of per-sample CCSNs.
- Input is single-cell scRNA-seq where trajectory/entropy analyses (NFE, CNDM-based
  clustering) rather than bulk per-sample pathway density is the goal.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: c-CSN
  role: Core method that constructs the conditional cell-specific network (CCSN) for
    each sample by testing conditional independence of gene pairs given top-degree
    conditional genes
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:68e3184a9f98b2991563b8755256987f80d57e135cc7f7fb88fa005ab035f97f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bulk-rna-seq-network-construction

## Summary

Applies the c-CSN method to bulk RNA-seq data (e.g., TCGA-LUAD tumor and normal samples) to construct a conditional cell-specific network (CCSN) per individual sample, enabling per-sample gene-gene association analysis such as EGFR pathway connectivity density comparison.

## When to use

Use when bulk RNA-seq expression matrices (e.g., TCGA LUAD tumor vs normal) must be transformed into per-sample gene-gene association networks to compare pathway connectivity (e.g., EGFR/EGF degrees) at single-sample resolution.

## When NOT to use

- The analysis goal is causal network construction or edge direction inference — the authors state c-CSN is not designed for causal networks and 'the directions of the gene associations cannot be obtained'.
- The question requires group-level differential co-expression only and per-sample resolution is unnecessary — a standard population-level correlation network suffices and avoids the G-fold computational cost of per-sample CCSNs.
- Input is single-cell scRNA-seq where trajectory/entropy analyses (NFE, CNDM-based clustering) rather than bulk per-sample pathway density is the goal.

## Inputs

- Bulk RNA-seq normalized gene expression matrix (samples x genes), e.g., TCGA LUAD tumor and normal adjacent tissue counts from GDC
- Significance level α for the conditional independence hypothesis test
- Number of conditional genes G and list of pathway genes (e.g., EGF, EGFR and EGFR pathway members)

## Outputs

- One CCSN (direct gene-gene association network) per individual sample
- Conditional network degree matrix (CNDM) of gene-gene interaction degrees per sample
- Per-sample EGF/EGFR network degrees and EGFR pathway connectivity density values for tumor vs normal comparison

## How to apply

Download and normalize the bulk RNA-seq profiles (e.g., TCGA LUAD tumor and adjacent normal from the GDC portal), then run the c-CSN procedure per sample: select conditional genes as the top-degree 'importance' hub genes from the CSN (G>=1 genes, chosen to include regulators of the pathway of interest such as EGF/EGFR), estimate the conditional independence statistic ρxy|z via scatter-diagram neighborhoods in 3D expression space, and test each gene pair at significance level α against the Nα quantile of the standard normal distribution to build one CCSN per sample. Integrate the per-sample CCSNs into a conditional network degree matrix (CNDM) whose entries are interaction degrees cijk. Extract network degrees of EGF and EGFR from each per-sample CCSN and summarize the EGFR pathway subnetwork degree density, then compare density between tumor and normal samples. Rationale: CSN overestimates associations by including indirect ones, so conditioning on hub genes removes indirect associations and yields direct, robust per-sample networks.

## Related tools

- **c-CSN** (Core method that constructs the conditional cell-specific network (CCSN) for each sample by testing conditional independence of gene pairs given top-degree conditional genes) — https://github.com/LinLi-0909/c-CSN

## Examples

```
git clone https://github.com/LinLi-0909/c-CSN; python cCSN.py --expression luad_tumor_normal_normalized.csv --alpha 0.05 --conditional_genes EGF,EGFR
```

## Evaluation signals

- Every input sample yields exactly one direct-association network (one CCSN per cell/sample), matching the stated property 'One direct association network is generated for one cell'.
- Each gene pair in the CCSN passed the hypothesis test against the Nα quantile of the standard normal distribution at the chosen significance level α.
- The EGFR pathway shows denser connectivity in tumor samples than in benign/normal tissues, with EGF/EGFR CCSN degrees consistently higher across tumor samples (reproducing Figure S8B/S8C findings).
- The integrated output forms an m x n conditional network degree matrix (CNDM) whose entries cijk represent gene-gene interaction degrees for cell/sample k.
- Edges reflect direct associations only: indirect associations present in a plain CSN are eliminated by conditioning on the top-degree hub genes.

## Limitations

- The G-fold computational cost of building per-sample conditional networks is high and no parallel computation scheme is provided; the authors state 'a parallel computation scheme is desired to reduce the computation time'.
- c-CSN does not produce causal networks or edge directions; 'the directions of the gene associations cannot be obtained'.
- Choice of conditional genes (top-degree hub genes, G>=1) directly shapes the resulting network and must be tailored to the pathway of interest (e.g., EGF/EGFR-relevant regulators).

## Evidence

- [abstract] c-CSN method can measure the direct associations between genes by eliminating the indirect associations.: "c-CSN method can measure the direct associations between genes by eliminating the indirect associations."
- [abstract] One network per cell: "1) One direct association network is generated for one cell."
- [methods] Conditional gene selection rule: [evidence span withheld — anchor status: not_found]
- [methods] TCGA LUAD application: "we applied c-CSN to the TCGA lung adenocarcinoma (LUAD) RNA-seq dataset"
- [discussion] Parallelization gap: "a parallel computation scheme is desired to reduce the computation time"
