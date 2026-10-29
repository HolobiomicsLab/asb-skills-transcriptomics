---
name: benchmark-method-comparison
description: Use when you must objectively compare a new CCI inference method against
  existing ones (e.g. SingleCellSignalR, iTALK, CellPhoneDB, CellChat, CellCall, CytoTalk,
  NATMI) on a shared scRNA-seq dataset with a literature-supported ground-truth interaction
  set.
when_to_use_negative:
- You only need to run a single method for exploratory CCI discovery rather than compare
  methods against a ground truth.
- No literature-curated ground-truth interaction set exists for the cell-type pair
  of interest, so AUC/precision cannot be computed.
- The input is not scRNA-seq expression data with cell-type labels suitable for all
  compared methods.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: DeepCCI
  role: Target GCN-based method under benchmarking; also used for shared preprocessing
    (preprocess.py, Feature.py) and producing ranked CCI predictions (Interaction_inference.py)
  repo: https://github.com/JiangBioLab/DeepCCI
- name: CellChat
  role: One of the seven comparison methods run on the same preprocessed dataset with
    default parameters
  repo: https://github.com/sqjin/CellChat
- name: R
  role: Environment for running R-based comparison/visualization packages during the
    benchmark
- name: Python 3.7.4
  role: Runtime for DeepCCI pipeline (conda env deepcciEnv) used to preprocess, cluster,
    and infer interactions for the target method
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:b8e15ea3e2f5f88240a00795122232cb7aedb8433e880e7f23dbc482c83a211b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - CellChat
  - DeepCCI
  - Python
  - Python 3.7.4
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# benchmark-method-comparison

## Summary

Benchmark a target CCI prediction method (e.g. DeepCCI) head-to-head against competing methods by running all of them on the same preprocessed scRNA-seq dataset and ranking them with literature-curated ground-truth metrics such as AUC and precision.

## When to use

Use when you must objectively compare a new CCI inference method against existing ones (e.g. SingleCellSignalR, iTALK, CellPhoneDB, CellChat, CellCall, CytoTalk, NATMI) on a shared scRNA-seq dataset with a literature-supported ground-truth interaction set.

## When NOT to use

- You only need to run a single method for exploratory CCI discovery rather than compare methods against a ground truth.
- No literature-curated ground-truth interaction set exists for the cell-type pair of interest, so AUC/precision cannot be computed.
- The input is not scRNA-seq expression data with cell-type labels suitable for all compared methods.

## Inputs

- Human testicular scRNA-seq dataset (Wang et al. 2018) from its public GEO accession
- Preprocessed feature file from DeepCCI's preprocessing step (Cluster_model/preprocess.py and Interaction_model/Feature.py)
- Literature-curated ground-truth set of STs-to-SSCs interactions involved in spermatogenesis
- Predicted CCI lists (ranked interaction outfiles) from all eight methods

## Outputs

- Per-method AUC and precision table (cci_benchmark_results.csv)
- Method ranking by AUC and precision
- List of top-ranked predicted CCIs from the target method (47 CCIs from STs to SSCs, e.g. Fig. 4a-b / Supplementary Data S6)

## How to apply

Download the benchmark dataset (e.g. the human testicular scRNA-seq dataset from Wang et al. 2018 via its GEO accession) and preprocess it once with the target method's pipeline (python preprocess.py, then python Feature.py --label_mode True --species Human). Run the target method end-to-end (python Cluster.py with the pretrained model, --pretain False --device cuda, followed by python Interaction_inference.py --device cuda) and run each of the seven comparison methods on the same preprocessed data with default parameters. Collect the ranked predicted-CCI lists from all eight methods and score each against the literature-curated ground-truth set (here: Sertoli-cell-to-spermatogonial-stem-cell interactions involved in spermatogenesis). Compute AUC and precision per method, rank the methods, and inspect the target method's top-ranked predictions (DeepCCI reported 47 CCIs from STs to SSCs, 42/47 i.e. over 89% literature-supported). Write the per-method AUC/precision table to a comparison file such as cci_benchmark_results.csv.

## Related tools

- **DeepCCI** (Target GCN-based method under benchmarking; also used for shared preprocessing (preprocess.py, Feature.py) and producing ranked CCI predictions (Interaction_inference.py)) — https://github.com/JiangBioLab/DeepCCI
- **CellChat** (One of the seven comparison methods run on the same preprocessed dataset with default parameters) — https://github.com/sqjin/CellChat
- **R** (Environment for running R-based comparison/visualization packages during the benchmark)
- **Python 3.7.4** (Runtime for DeepCCI pipeline (conda env deepcciEnv) used to preprocess, cluster, and infer interactions for the target method)

## Examples

```
cd Interaction_model; python Feature.py --label_mode True --species Human; python Interaction_inference.py --device cuda
```

## Evaluation signals

- DeepCCI attains the highest AUC and precision among all eight methods on the testicular dataset
- DeepCCI's top predictions correspond to the 47 CCIs from STs to SSCs reported in Fig. 4a-b and Supplementary Data S6
- Over 89% of DeepCCI's predicted CCIs (42/47) are literature-supported as involved in spermatogenesis
- All eight methods were run on the same preprocessed dataset, ensuring a fair shared-input comparison
- A cci_benchmark_results.csv file exists with one AUC/precision row per method

## Limitations

- Comparison methods were run with default parameters, which may not represent their fully tuned performance
- Ground truth is literature-curated for a specific cell-type pair (STs to SSCs in spermatogenesis), so metrics are dataset- and question-specific
- DeepCCI's interaction inference and clustering require a CUDA-capable GPU (--device cuda); preprocessing of large scRNA-seq files takes ~10-20 minutes

## Evidence

- [results] On the human testicular scRNA-seq dataset, DeepCCI identified 47 CCIs from STs to SSCs, over 89% of which (42/47) are literature-supported as involved in spermatogenesis, and DeepCCI achieved the highest AUC and precision compared with the seven other methods.: [evidence span withheld — anchor status: not_found]
- [full_text] Run the seven comparison methods (SingleCellSignalR, iTALK, CellPhoneDB, CellChat, CellCall, CytoTalk, NATMI) on the same preprocessed dataset, each with default parameters.: [evidence span withheld — anchor status: not_found]
- [readme] python Feature.py --label_mode True --species Human: "The example test file can be download from  http://jianglab.org.cn/deepcci_download/."
- [readme] python Interaction_inference.py --device cuda: "The predicted interaction outfile will be provided. The predicted process will take about 1-2 minutes."
