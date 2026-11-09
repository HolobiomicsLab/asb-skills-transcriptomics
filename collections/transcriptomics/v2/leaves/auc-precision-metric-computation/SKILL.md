---
name: auc-precision-metric-computation
description: Use when multiple CCI prediction methods (e.g. DeepCCI, SingleCellSignalR,
  iTALK, CellPhoneDB, CellChat, CellCall, CytoTalk, NATMI) have produced predicted
  CCI lists on the same dataset and you must quantify accuracy via AUC and precision.
when_to_use_negative:
- Input is raw scRNA-seq data (CSV/h5) that still needs DeepCCI preprocessing (preprocess.py
  / Feature.py) rather than finished CCI prediction lists
- There is no literature-curated ground-truth interaction set for the cell-type pair
  of interest, so AUC/precision cannot be scored
- The evaluation target is cell clustering quality (Cluster.py output) rather than
  ranked cell-cell interaction predictions
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: DeepCCI
  role: Produces the ranked interaction outfile whose top-ranked CCIs (47 STs-to-SSCs
    predictions) are scored for AUC and precision against the ground-truth set
  repo: https://github.com/JiangBioLab/DeepCCI
- name: CellChat
  role: One of the seven comparison methods whose predicted CCI lists are collected
    and scored with the same AUC/precision protocol
  repo: https://github.com/sqjin/CellChat
- name: Python
  role: Environment for running DeepCCI (python Interaction_inference.py) and computing
    the per-method AUC/precision comparison table
- name: R
  role: Used for data processing and result visualization of the benchmark outputs
    (Seurat, igraph, circlize, ComplexHeatmap)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:02a46cbcad88513ad2139ee6669dd02132f48122845b22190d1327919a9ccced
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiangBioLab/DeepCCI
  tools:
  - CellChat
  - DeepCCI
  - Python
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# auc-precision-metric-computation

## Summary

Computes AUC and precision per CCI prediction method by scoring ranked predicted cell-cell interaction lists against a literature-curated ground-truth interaction set, enabling head-to-head ranking of DeepCCI against seven statistical baselines.

## When to use

Use when multiple CCI prediction methods (e.g. DeepCCI, SingleCellSignalR, iTALK, CellPhoneDB, CellChat, CellCall, CytoTalk, NATMI) have produced predicted CCI lists on the same dataset and you must quantify accuracy via AUC and precision.

## When NOT to use

- Input is raw scRNA-seq data (CSV/h5) that still needs DeepCCI preprocessing (preprocess.py / Feature.py) rather than finished CCI prediction lists
- There is no literature-curated ground-truth interaction set for the cell-type pair of interest, so AUC/precision cannot be scored
- The evaluation target is cell clustering quality (Cluster.py output) rather than ranked cell-cell interaction predictions

## Inputs

- Predicted CCI lists from eight methods: DeepCCI, SingleCellSignalR, iTALK, CellPhoneDB, CellChat, CellCall, CytoTalk, NATMI
- DeepCCI ranked interaction outfile (from python Interaction_inference.py --device cuda)
- Literature-curated ground-truth interaction set (STs-to-SSCs interactions involved in spermatogenesis, human testicular dataset)
- Preprocessed human testicular scRNA-seq dataset (Wang et al. 2018, public GEO accession)

## Outputs

- Per-method AUC and precision values with method ranking
- cci_benchmark_results.csv comparison table
- Identified top-ranked predictions (47 DeepCCI CCIs from STs to SSCs)

## How to apply

Collect the predicted CCI lists from all eight methods run on the same preprocessed scRNA-seq dataset, including DeepCCI's ranked interaction outfile produced by `python Interaction_inference.py --device cuda`. Score each predicted interaction against the literature-curated ground-truth set (for the human testicular dataset, STs-to-SSCs interactions involved in spermatogenesis). Compute AUC (treats each method's ranked list as a binary classifier over true vs false interactions) and precision (fraction of top-ranked predictions that are ground-truth-supported) per method, then rank the methods. In the reference study this confirmed DeepCCI identified 47 CCIs from STs to SSCs of which 42/47 (over 89%) were literature-supported, giving DeepCCI the highest AUC and precision. Write the per-method comparison table to cci_benchmark_results.csv and cross-check DeepCCI's top predictions against Fig. 4a-b and Supplementary Data S6.

## Related tools

- **DeepCCI** (Produces the ranked interaction outfile whose top-ranked CCIs (47 STs-to-SSCs predictions) are scored for AUC and precision against the ground-truth set) — https://github.com/JiangBioLab/DeepCCI
- **CellChat** (One of the seven comparison methods whose predicted CCI lists are collected and scored with the same AUC/precision protocol) — https://github.com/sqjin/CellChat
- **Python** (Environment for running DeepCCI (python Interaction_inference.py) and computing the per-method AUC/precision comparison table)
- **R** (Used for data processing and result visualization of the benchmark outputs (Seurat, igraph, circlize, ComplexHeatmap))

## Evaluation signals

- DeepCCI attains the highest AUC and precision among all eight methods on the scored comparison
- DeepCCI's top predictions correspond to the 47 CCIs reported in Fig. 4a-b and Supplementary Data S6
- Over 89% of DeepCCI's 47 top-ranked STs-to-SSCs CCIs (42/47) are literature-supported as involved in spermatogenesis
- cci_benchmark_results.csv contains an AUC and precision value for every one of the eight methods
- All methods were run on the same preprocessed dataset with default parameters before scoring

## Limitations

- The ground-truth set is literature-curated, so precision/AUC are biased toward interactions already documented (e.g. spermatogenesis-related STs-SSCs interactions) and may undervalue novel predictions
- DeepCCI's clustering and interaction inference steps expect a CUDA device; without GPU access the pretrained model path (--pretain False) and runtimes (~25 min clustering, 1-2 min inference) may not be reproducible
- Baseline methods are run with default parameters, so the comparison reflects out-of-the-box performance rather than per-method tuning

## Evidence

- [abstract] one-stop solution claim: "DeepCCI can provide the one-stop solution to discover meaningful intercellular interactions and build CCI networks from scRNA-seq data"
- [abstract] effective prediction claim: [evidence span withheld — anchor status: not_found]
- [methods] interaction inference command: "python Interaction_inference.py --device cuda"
- [readme] ranked interaction output: "The predicted interaction outfile will be provided. The predicted process will take about 1-2 minutes."
