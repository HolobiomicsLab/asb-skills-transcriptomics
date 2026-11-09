---
name: paired-model-comparison-significance-testing
description: Use when comparing a new causal-inference or prediction model against
  baseline configurations on the same candidate peak-gene pairs, and you need per-configuration
  retrieval AUROCs plus significance statistics for the differences.
when_to_use_negative:
- The candidate sets differ between the proposed model and the baselines, making AUROC
  comparisons on eQTL retrieval non-comparable
- There is no external ground-truth validation resource (such as eQTL Catalogue significant
  associations) to define retrieval positives
- 'The goal is method development rather than ablation: comparing configurations of
  the same model on training performance rather than causal-link retrieval accuracy'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: GrID-Net
  role: 'Proposed model configuration: graph neural network Granger inference run
    on the full three-branch DAG via gridnet.run_gridnet; its AUROC is the reference
    against which linear-Granger baselines are tested'
  repo: https://github.com/alexw16/gridnet
- name: Schema
  role: Provides the joint RNA+ATAC cell representations (gridnet.schema_representations)
    used to build the DAGs on which both the model and baselines are evaluated
  repo: https://github.com/rs239/schema
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-61337-5
    title: Unveiling causal regulatory mechanisms through cell-state parallax
  source_content_hash: sha256:e89429dfcf8ce78abcbb1728f93dc9fc5d5d3593c43f61612f139bb291ca3a9f
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# paired-model-comparison-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rigorous ablation methodology for comparing a graph-neural-network Granger method (GrID-Net) against matched baseline configurations (linear Granger on the full SHARE-seq DAG and on individual cortex, medulla, and IRS branches) by scoring each on eQTL-retrieval AUROC and computing significance statistics for the AUROC differences.

## When to use

Use when comparing a new causal-inference or prediction model against baseline configurations on the same candidate peak-gene pairs, and you need per-configuration retrieval AUROCs plus significance statistics for the differences.

## When NOT to use

- The candidate sets differ between the proposed model and the baselines, making AUROC comparisons on eQTL retrieval non-comparable
- There is no external ground-truth validation resource (such as eQTL Catalogue significant associations) to define retrieval positives
- The goal is method development rather than ablation: comparing configurations of the same model on training performance rather than causal-link retrieval accuracy

## Inputs

- Preprocessed RNA-seq and ATAC-seq AnnData objects from a SHARE-seq mouse skin multimodal dataset
- Candidate peak-gene pairs DataFrame (peaks within 1e6 bp of gene TSS) from gridnet.identify_all_peak_gene_link_candidates
- Full three-branch DAG adjacency matrix plus three branch-specific DAGs (cortex, medulla, IRS)
- Significant skin eQTLs (P < 1e-3) from the eQTL Catalogue for retrieval scoring

## Outputs

- AUROC per model configuration (GrID-Net full DAG, linear Granger full DAG, linear Granger cortex, medulla, IRS)
- Significance statistics for AUROC differences between GrID-Net and each linear-Granger baseline
- Comparison table written to dag_vs_branches_auroc.csv

## How to apply

Build all model configurations on the identical candidate set: restrict peak-gene pairs to peaks within 1 Mb (distance_thresh=1e6) of each gene's TSS after preprocessing RNA/ATAC counts with normalization and log(1+x) transformation via gridnet.preprocess_multimodal. Run the full method (gridnet.run_gridnet on the full three-branch DAG built with construct_dag, n_neighbors=15, pseudotime_algo='dpt') and each ablation baseline (linear Granger on the full DAG, then on the cortex, medulla, and inner-root-sheath branch DAGs separately) on the same candidate_XY_pairs. Score every configuration by AUROC of retrieving significant skin eQTLs (P < 1e-3) from the eQTL Catalogue, combined with log genomic distance as in the paper's logistic-regression evaluation. Compare the baseline AUROCs against the full-model AUROC, compute significance statistics for each pairwise difference, and persist the comparison table (e.g., dag_vs_branches_auroc.csv) so the ablation is reproducible.

## Related tools

- **GrID-Net** (Proposed model configuration: graph neural network Granger inference run on the full three-branch DAG via gridnet.run_gridnet; its AUROC is the reference against which linear-Granger baselines are tested) — https://github.com/alexw16/gridnet
- **Schema** (Provides the joint RNA+ATAC cell representations (gridnet.schema_representations) used to build the DAGs on which both the model and baselines are evaluated) — https://github.com/rs239/schema

## Examples

```
results_df = gridnet.run_gridnet(X,Y,X_feature_names,Y_feature_names,candidate_XY_pairs,dag_adjacency_matrix)
```

## Evaluation signals

- GrID-Net on the full DAG achieves a significantly higher eQTL-retrieval AUROC than every linear-Granger configuration (full DAG and each of the cortex, medulla, IRS branches), reproducing the reported Fig. 2c ablation result
- All linear-Granger baselines yield better-than-random eQTL prediction (AUROC above 0.5), confirming the comparison is not against degenerate baselines
- Every configuration is evaluated on the identical candidate peak-gene pair set (distance_thresh=1e6) and the identical eQTL positive set (P < 1e-3)
- The AUROC scoring incorporates log genomic distance identically across configurations, matching the paper's logistic-regression evaluation
- The results table dag_vs_branches_auroc.csv contains one row per configuration with its AUROC and significance statistics

## Limitations

- eQTL cross-referencing only mimics perturbation-based validation; GrID-Net associations remain causal hypotheses requiring perturbational follow-up
- Computational burden of large single-cell datasets may exceed GPU memory capacity, limiting the scale of comparisons
- The evaluation used 5C data as a substitute for SHARE-seq because mouse skin Hi-C data was unavailable, so chromatin-interaction-based validation is imperfect

## Evidence

- [results] GrID-Net achieved significantly higher accuracy than all of these linear-Granger configurations: [evidence span withheld — anchor status: not_found]
- [discussion] DAG construction settings: [evidence span withheld — anchor status: not_found]
