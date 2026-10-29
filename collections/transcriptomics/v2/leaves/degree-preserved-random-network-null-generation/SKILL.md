---
name: degree-preserved-random-network-null-generation
description: Use when benchmarking whether an inferred GRN models disease gene sets
  (e.g., DisGeNET Immune, Metabolic, Neuronal categories) better than chance, and
  a degree-matched random network null is needed to compute performance gain.
when_to_use_negative:
- When the question is network construction quality itself (e.g., perturb-seq perturbation
  target prediction or held-out cosine-similarity goodness of fit) rather than disease
  subnetwork modeling relative to degree-matched nulls
- When the input is a raw scRNA-seq expression matrix rather than an already-inferred
  GRN; network inference (e.g., SCING BuildNetwork.ipynb) must happen first
edam_operation: http://edamontology.org/operation_3948
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SCING
  role: Source GRN being evaluated; its performance gain over degree-preserved random
    networks is benchmarked against GRNBOOST2, ppcor, and PIDC
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Benchmark GRN inference method whose disease subnetwork performance gain over
    degree-preserved nulls is compared with SCING's
- name: ppcor
  role: Benchmark GRN inference method (partial correlation) evaluated with the same
    random-walk null framework
- name: PIDC
  role: Benchmark GRN inference method (conditional mutual information) evaluated
    with the same random-walk null framework
- name: conda
  role: Environment manager used to install the SCING software environment containing
    the random-walk disease subnetwork evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:12249e14b3f6beef31d1c077b09c1eee0acd5d42be79d28717f509a1019bbf4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - conda
  - ppcor
  edam_operation: http://edamontology.org/operation_3948
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# degree-preserved-random-network-null-generation

## Summary

Generates random GRN nulls with node degree (and similar node attributes) matched to an inferred GRN, so that a real GRN's ability to model disease subnetworks via the Huang et al. random walk framework can be quantified as a performance gain over chance. This is SCING's biological validation scheme for disease modeling accuracy.

## When to use

Use when benchmarking whether an inferred GRN models disease gene sets (e.g., DisGeNET Immune, Metabolic, Neuronal categories) better than chance, and a degree-matched random network null is needed to compute performance gain.

## When NOT to use

- When the question is network construction quality itself (e.g., perturb-seq perturbation target prediction or held-out cosine-similarity goodness of fit) rather than disease subnetwork modeling relative to degree-matched nulls
- When the input is a raw scRNA-seq expression matrix rather than an already-inferred GRN; network inference (e.g., SCING BuildNetwork.ipynb) must happen first

## Inputs

- Inferred gene regulatory network (edge list/adjacency) from SCING, GRNBOOST2, ppcor, or PIDC on scRNA-seq data
- Disease gene sets from DisGeNET (e.g., Immune, Metabolic, Neuronal disease categories)
- Node degree distribution / node attributes of the real GRN

## Outputs

- Degree-preserved random GRN nulls with similar node attributes
- Random-walk performance gain of the real GRN over the nulls, per disease category
- Cell-type-by-disease performance-gain matrix (e.g., disease_subnetwork_performance_gain.csv)

## How to apply

For each inferred GRN (e.g., SCING, GRNBOOST2, ppcor, or PIDC networks built from mouse cell atlas scRNA-seq for bone marrow, brain, and liver), run the random walk framework from Huang et al. on the disease gene set of interest (Immune, Metabolic, or Neuronal DisGeNET gene sets). Generate random GRN nulls that preserve the real network's node degree characteristics (and similar node attributes), then compute the performance gain of the real GRN's disease subnetwork modeling over these nulls. Repeat per cell type, per disease category, and aggregate the gains into a cell-type-by-disease performance-gain matrix (e.g., disease_subnetwork_performance_gain.csv). A method is judged better if its random-walk performance gain over the degree-preserved nulls exceeds that of competing methods across tissues and disease categories.

## Related tools

- **SCING** (Source GRN being evaluated; its performance gain over degree-preserved random networks is benchmarked against GRNBOOST2, ppcor, and PIDC) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Benchmark GRN inference method whose disease subnetwork performance gain over degree-preserved nulls is compared with SCING's)
- **ppcor** (Benchmark GRN inference method (partial correlation) evaluated with the same random-walk null framework)
- **PIDC** (Benchmark GRN inference method (conditional mutual information) evaluated with the same random-walk null framework)
- **conda** (Environment manager used to install the SCING software environment containing the random-walk disease subnetwork evaluation)

## Evaluation signals

- The random nulls must match the real GRN's node degree characteristics and have similar node attributes, not be fully random Erdos-Renyi networks
- The output performance-gain matrix should show SCING achieving a higher random-walk performance gain than GRNBOOST2, ppcor, and PIDC across Immune, Metabolic, and Neuronal disease categories
- The gain should hold consistently across all evaluated tissues (bone marrow, brain, liver) and across the full mouse cell atlas (33 tissues, 106 cell types)
- Each real-vs-null comparison should yield a scalar performance gain per (cell type, disease category) pair that aggregates cleanly into disease_subnetwork_performance_gain.csv

## Limitations

- Inferred GRN links are not necessarily causal or directional, so disease subnetwork gains reflect association structure rather than causal mechanisms
- Hyperparameter selection for the underlying GRN inference can be dataset dependent, which propagates into the null comparisons
- Null quality depends on the choice of preserved node attributes; the framework compares only against degree-preserved (similar node attribute) random GRNs, not against other null models

## Evidence

- [methods] Huang random walk null design: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs"
- [readme] README null attributes: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs with similar node attributes"
- [abstract] DisGeNET validation claim: "the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability of SCING"
- [intro] Benchmark scope: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [methods] MCA application scale: "and to the mouse cell atlas in 33 tissues and 106 cell types"
