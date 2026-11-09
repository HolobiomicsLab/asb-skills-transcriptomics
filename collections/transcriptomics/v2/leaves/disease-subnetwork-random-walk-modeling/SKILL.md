---
name: disease-subnetwork-random-walk-modeling
description: Use when a inferred GRN (e.g., from SCING, GRNBOOST2, ppcor, or PIDC)
  needs biological validation by testing how well it models disease gene subnetworks
  from DisGeNET versus degree-preserved random network nulls.
when_to_use_negative:
- Validation target is perturbation target prediction or network overfitting (use
  Perturb-seq AUROC/TPR-at-FPR-0.05 or cosine-similarity held-out gene prediction
  instead)
- No curated disease gene sets (e.g., DisGeNET categories) are available for the phenotype
  of interest
- The network's null comparison requires preserving structure beyond node degree attributes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: SCING
  role: GRN inference method whose networks are evaluated for disease subnetwork modeling
    performance gain
  repo: github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Benchmark GRN inference method (gradient boosting) compared against SCING
    on disease subnetwork modeling
- name: ppcor
  role: Benchmark GRN inference method (partial correlation) compared against SCING
    on disease subnetwork modeling
- name: PIDC
  role: Benchmark GRN inference method (conditional mutual information) compared against
    SCING on disease subnetwork modeling
- name: conda
  role: Environment manager for installing the SCING package and its dependencies
    used in the evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:1c68ed794edcb5a0acf84750911375f5ee7c2b27ed3b9b1485106c4eeb2cf652
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - conda
  - ppcor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# disease-subnetwork-random-walk-modeling

## Summary

Biological validation of inferred gene regulatory networks (GRNs) using the Huang et al. random walk framework to model DisGeNET disease subnetworks, quantifying the performance gain of a real GRN over degree-preserved random GRNs with similar node attributes. Used in SCING to show SCING outperforms GRNBOOST2, ppcor, and PIDC on Immune, Metabolic, and Neuronal disease gene sets.

## When to use

Use when a inferred GRN (e.g., from SCING, GRNBOOST2, ppcor, or PIDC) needs biological validation by testing how well it models disease gene subnetworks from DisGeNET versus degree-preserved random network nulls.

## When NOT to use

- Validation target is perturbation target prediction or network overfitting (use Perturb-seq AUROC/TPR-at-FPR-0.05 or cosine-similarity held-out gene prediction instead)
- No curated disease gene sets (e.g., DisGeNET categories) are available for the phenotype of interest
- The network's null comparison requires preserving structure beyond node degree attributes

## Inputs

- Inferred GRN(s) as edge lists/graphs (e.g., SCING merged and pruned networks per cell type)
- DisGeNET disease gene sets (Immune, Metabolic, Neuronal disease categories)
- Benchmark GRNs from GRNBOOST2, ppcor, and PIDC inferred on the same cells
- Mouse cell atlas scRNA-seq matrices (bone marrow, brain, liver; 33 tissues, 106 cell types)

## Outputs

- Per-GRN performance gain over degree-preserved random null networks for each disease category
- Cell-type-by-disease performance-gain matrix (disease_subnetwork_performance_gain.csv)

## How to apply

For each inferred GRN (e.g., SCING networks built from 100 subsampled networks per tissue, merged and pruned in BuildNetwork.ipynb), run the Huang et al. random walk framework against DisGeNET disease gene sets — here the Immune, Metabolic, and Neuronal categories. Generate degree-preserved random GRNs with similar node attributes as null distributions, then compute the performance gain of the real GRN over these nulls at modeling each disease subnetwork. Benchmark competing GRN inference methods (GRNBOOST2, ppcor, PIDC) on the same cells to obtain comparable gains. To scale the evaluation, extend the SCING pipeline to all 273 mouse cell atlas cell-type networks (33 tissues, 106 cell types) and aggregate per-cell-type, per-disease-category gains into a cell-type-by-disease performance-gain matrix (e.g., disease_subnetwork_performance_gain.csv). The rationale is that a biologically meaningful GRN should recover disease-relevant gene connectivity better than random networks matched on node degree.

## Related tools

- **SCING** (GRN inference method whose networks are evaluated for disease subnetwork modeling performance gain) — github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Benchmark GRN inference method (gradient boosting) compared against SCING on disease subnetwork modeling)
- **ppcor** (Benchmark GRN inference method (partial correlation) compared against SCING on disease subnetwork modeling)
- **PIDC** (Benchmark GRN inference method (conditional mutual information) compared against SCING on disease subnetwork modeling)
- **conda** (Environment manager for installing the SCING package and its dependencies used in the evaluation)

## Evaluation signals

- SCING should show a higher random-walk performance gain than GRNBOOST2, ppcor, and PIDC across Immune, Metabolic, and Neuronal disease categories, consistent with the paper's reported improved disease modeling accuracy
- Performance gains should hold across all tissues tested (bone marrow, brain, liver)
- Random null GRNs must be degree-preserved with similar node attributes to the real GRN before computing gains
- The aggregated matrix (disease_subnetwork_performance_gain.csv) must have rows per cell type (from 273 MCA networks across 33 tissues, 106 cell types) and columns per disease category
- Gains should be positive, indicating the real GRN outperforms degree-matched random networks

## Limitations

- The random walk framework evaluates disease subnetwork modeling relative to degree-preserved random nulls, not absolute predictive accuracy
- SCING hyperparameters require selection and selection can be dataset dependent, which propagates into the downstream disease-modeling evaluation
- Inferred GRN links are not necessarily causal or directional, limiting interpretation of disease subnetwork mechanisms

## Evidence

- [methods] Biological validation via disease subnetwork modeling with a random walk framework: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs"
- [readme] README on disease subnetwork validation design: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs with similar node attributes (d)."
- [intro] Benchmark methods compared: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [methods] Mouse cell atlas scope: "and to the mouse cell atlas in 33 tissues and 106 cell types"
- [abstract] DisGeNET evaluation finding: "Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability of"
