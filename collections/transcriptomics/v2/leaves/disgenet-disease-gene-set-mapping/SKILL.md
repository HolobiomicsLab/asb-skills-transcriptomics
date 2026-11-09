---
name: disgenet-disease-gene-set-mapping
description: 'Use when evaluating whether an inferred single-cell GRN captures disease
  biology: you have one or more inferred networks and want a quantitative, per-disease-category
  score of how well the network models disease gene subnetworks versus matched random
  nulls.'
when_to_use_negative:
- You lack DisGeNET disease gene sets or a mapping between network genes and disease
  genes — the gain score cannot be computed
- The network was built on a pre-selected subset of genes or cells rather than a full
  transcriptome context, so degree-preserved nulls would not be comparable in spirit
- Your goal is direct perturbation-target validation from Perturb-seq data rather
  than disease subnetwork modeling — SCING evaluates those as separate benchmarks
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: GRN inference method under evaluation; its networks (built from ~100 subsampled
    networks per tissue merged into a final pruned GRN via BuildNetwork.ipynb) are
    scored for disease subnetwork performance gain
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Benchmark GRN inference method (gradient boosting) whose networks are evaluated
    against the same DisGeNET disease gene sets for comparison
- name: ppcor
  role: Benchmark GRN inference method (partial correlation) evaluated with the same
    random walk disease subnetwork framework
- name: PIDC
  role: Benchmark GRN inference method (conditional mutual information) evaluated
    with the same random walk disease subnetwork framework
- name: conda
  role: Environment manager used to create the scing environment (install/scing.environment.yml)
    required to run the SCING pipeline feeding this evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:8629e5175d86e9a4c5470ceb6e08506c5e60beb97de798e093857602d40e2439
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# disgenet-disease-gene-set-mapping

## Summary

Biological validation of inferred gene regulatory networks (GRNs) by modeling disease subnetworks from DisGeNET disease gene sets (Immune, Metabolic, Neuronal categories) with the Huang et al. random walk framework, scoring each GRN by its performance gain over degree-preserved random GRNs with similar node attributes.

## When to use

Use when evaluating whether an inferred single-cell GRN captures disease biology: you have one or more inferred networks and want a quantitative, per-disease-category score of how well the network models disease gene subnetworks versus matched random nulls.

## When NOT to use

- You lack DisGeNET disease gene sets or a mapping between network genes and disease genes — the gain score cannot be computed
- The network was built on a pre-selected subset of genes or cells rather than a full transcriptome context, so degree-preserved nulls would not be comparable in spirit
- Your goal is direct perturbation-target validation from Perturb-seq data rather than disease subnetwork modeling — SCING evaluates those as separate benchmarks

## Inputs

- Inferred GRN (edge list/adjacency from SCING, GRNBOOST2, ppcor, or PIDC on scRNA-seq data)
- DisGeNET disease gene sets (Immune, Metabolic, Neuronal disease categories)
- Degree-preserved random GRNs with similar node attributes as null distributions
- scRNA-seq source data (e.g., mouse cell atlas bone marrow, brain, liver matrices with supercells)

## Outputs

- Per-GRN random-walk performance gain of disease subnetwork modeling versus degree-preserved random null networks
- Per-cell-type, per-disease-category performance-gain matrix (e.g., disease_subnetwork_performance_gain.csv)

## How to apply

For each inferred GRN (e.g., SCING, GRNBOOST2, ppcor, or PIDC networks built from mouse cell atlas bone marrow, brain, and liver scRNA-seq), run the Huang et al. random walk framework on DisGeNET disease gene sets from the Immune, Metabolic, and Neuronal categories. As nulls, generate random GRNs that preserve the real GRN's node degree characteristics (degree-preserved random networks with similar node attributes), so the comparison is not trivially driven by topology. Compute the performance gain of the real GRN in modeling the disease subnetwork relative to these nulls, and repeat per cell type and disease category. When aggregating, results can be assembled into a cell-type-by-disease performance-gain matrix (e.g., disease_subnetwork_performance_gain.csv) across all 273 mouse cell atlas cell-type networks (33 tissues, 106 cell types).

## Related tools

- **SCING** (GRN inference method under evaluation; its networks (built from ~100 subsampled networks per tissue merged into a final pruned GRN via BuildNetwork.ipynb) are scored for disease subnetwork performance gain) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Benchmark GRN inference method (gradient boosting) whose networks are evaluated against the same DisGeNET disease gene sets for comparison)
- **ppcor** (Benchmark GRN inference method (partial correlation) evaluated with the same random walk disease subnetwork framework)
- **PIDC** (Benchmark GRN inference method (conditional mutual information) evaluated with the same random walk disease subnetwork framework)
- **conda** (Environment manager used to create the scing environment (install/scing.environment.yml) required to run the SCING pipeline feeding this evaluation)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git && cd SCING && conda env create -n scing --file install/scing.environment.yml && conda activate scing && pip install -e .
```

## Evaluation signals

- SCING should achieve a higher random-walk performance gain than GRNBOOST2, ppcor, and PIDC across Immune, Metabolic, and Neuronal disease categories, consistent with the paper's reported improved disease modeling accuracy versus random GRNs
- Null distributions must be degree-preserved random GRNs with node attributes similar to the real GRN — a gain over un-matched random networks would be an invalid comparison
- The output matrix should cover all cell-type × disease-category combinations (e.g., 273 MCA cell-type networks × 3 disease categories) with no missing gain values
- Per-tissue replication: the SCING advantage should hold across bone marrow, brain, and liver networks independently before aggregating

## Limitations

- SCING GRN links are not necessarily causal or directional, so disease subnetwork modeling reflects regulatory association rather than causation
- Many SCING hyperparameters require selection, and while the paper shows robustness across a range of hyperparameters, selection can be dataset dependent
- SCING performance is tied to how nulls are constructed; the metric is gain over degree-preserved random GRNs, not absolute disease-prediction accuracy

## Evidence

- [methods] Biological validation via disease subnetwork modeling with a random walk framework: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs"
- [abstract] Performance evaluation with mouse cell atlas and DisGeNET: "Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability of"
- [intro] Benchmark methods evaluated for disease modeling accuracy: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [readme] Random walk with degree-preserved null GRNs: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs with similar node attributes"
- [methods] Mouse cell atlas scope of application: "We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients"
