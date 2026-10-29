---
name: learning-rate-sensitivity-analysis-of-embeddings
description: Use when reproducing or extending the SnapCCESS hyperparameter sensitivity
  diagnostics — specifically when testing how the initial learning rate of the variational-autoencoder
  embedding step affects clustering performance on a multimodal single-cell omics
  dataset such as Swanson TEA-seq (GSE158013).
when_to_use_negative:
- The tutorial/package explicitly does not recommend best parameters for new datasets
  — do not treat the 0.02-vs-0.2 comparison as a parameter recommendation for a different
  dataset without consulting the paper or tuning empirically.
- The goal is standard SnapCCESS embedding generation and consensus clustering at
  fixed settings, not a sensitivity analysis of hyperparameters.
- The dataset is unimodal or lacks paired RNA/ADT modalities measured across the same
  cells, so the multimodal variational-autoencoder embedding step cannot be run as
  specified.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: snapccess
  role: Python package used to run snapshot variational-autoencoder embedding generation
    on the TEA-seq RNA and ADT modalities at each learning-rate setting
  repo: https://github.com/PYangLab/SnapCCESS
- name: Python
  role: Language in which the tutorial notebook, embedding runs, k-means clustering,
    and ARI computation are executed
- name: SnapCCESS
  role: Unsupervised ensemble deep learning framework whose learning-rate sensitivity
    is being diagnosed; coupled with k-means for consensus clustering
  repo: https://github.com/PYangLab/SnapCCESS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:c906cf58fe8a2b44c2233545b9104318d7dbee9856122ae19aeedbc134fbb38d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - SnapCCESS
  - snapccess
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# learning-rate-sensitivity-analysis-of-embeddings

## Summary

A diagnostic procedure for quantifying how sensitive SnapCCESS snapshot variational-autoencoder embedding quality and downstream consensus clustering are to the initial learning rate, by rerunning embedding generation at multiple learning rates and comparing adjusted Rand index (ARI) against reference cell-type labels, as reported in Supplementary Fig. S2a of the SnapCCESS paper.

## When to use

Use when reproducing or extending the SnapCCESS hyperparameter sensitivity diagnostics — specifically when testing how the initial learning rate of the variational-autoencoder embedding step affects clustering performance on a multimodal single-cell omics dataset such as Swanson TEA-seq (GSE158013).

## When NOT to use

- The tutorial/package explicitly does not recommend best parameters for new datasets — do not treat the 0.02-vs-0.2 comparison as a parameter recommendation for a different dataset without consulting the paper or tuning empirically.
- The goal is standard SnapCCESS embedding generation and consensus clustering at fixed settings, not a sensitivity analysis of hyperparameters.
- The dataset is unimodal or lacks paired RNA/ADT modalities measured across the same cells, so the multimodal variational-autoencoder embedding step cannot be run as specified.

## Inputs

- Multimodal single-cell omics dataset: Swanson TEA-seq (GEO accession GSE158013) RNA and protein/ADT modalities over the same cells
- Reference cell-type labels for the TEA-seq cells
- SnapCCESS tutorial data and notebook (tutorials/src/an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb)

## Outputs

- ari_comparison_s2a.csv — table of adjusted Rand index values for initial learning rates 0.02 and 0.2
- Comparison rendered in the format of Supplementary Fig. S2a
- Two sets of snapshot variational-autoencoder embeddings of the TEA-seq data

## How to apply

Install the snapccess Python package via pip and download the TEA-seq multimodal dataset (GEO accession GSE158013) covering its RNA and protein (ADT) modalities, arranging each modality as features-by-cells as the SnapCCESS input expects. Following the SnapCCESS Python tutorial notebook (an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb), run snapshot variational-autoencoder embedding generation twice on the same data — once with initial learning rate 0.02 (the default that gave high performance) and once with 0.2 — keeping all other settings fixed. Cluster each set of embeddings with k-means to obtain the consensus clustering, then compute the adjusted Rand index (ARI) of each consensus clustering against the reference cell-type labels. Tabulate the two ARI values and render the comparison in the format of Supplementary Fig. S2a; the correct outcome reproduces the reported trend that learning rate 0.2 causes a significant reduction in clustering performance relative to 0.02. Note the README explicitly does not recommend best parameters for arbitrary datasets — for published-dataset parameters refer to the paper itself.

## Related tools

- **snapccess** (Python package used to run snapshot variational-autoencoder embedding generation on the TEA-seq RNA and ADT modalities at each learning-rate setting) — https://github.com/PYangLab/SnapCCESS
- **Python** (Language in which the tutorial notebook, embedding runs, k-means clustering, and ARI computation are executed)
- **SnapCCESS** (Unsupervised ensemble deep learning framework whose learning-rate sensitivity is being diagnosed; coupled with k-means for consensus clustering) — https://github.com/PYangLab/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple  # then run the tutorial notebook twice with initial learning rates 0.02 and 0.2 and compare ARI
```

## Evaluation signals

- ARI at initial learning rate 0.02 is substantially higher than ARI at 0.2 on the TEA-seq data, reproducing the sensitivity trend in Supplementary Fig. S2a.
- All settings other than the initial learning rate are held fixed between the two runs, so any ARI difference is attributable to the learning rate.
- The comparison table is saved as ari_comparison_s2a.csv with one ARI value per learning rate.
- The rendered figure matches the format of Supplementary Fig. S2a of the SnapCCESS paper.
- Both embedding runs use the same TEA-seq RNA and ADT modalities with features arranged by cells before preprocessing, per the SnapCCESS input convention.

## Limitations

- The finding is specific to the Swanson TEA-seq dataset; sensitivity of other datasets to learning rate may differ and the package does not recommend best parameters.
- 'This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets.' Best hyperparameters can vary by dataset; for the publication datasets, refer to the paper.
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation, so runtimes may vary across environments.

## Evidence

- [full_text] Default lr 0.02 high vs 0.2 reduced performance (task card finding): [evidence span withheld — anchor status: not_found]
- [full_text] Workflow for the S2a reproduction (task card workflow): [evidence span withheld — anchor status: not_found]
- [full_text] ARI computed against reference labels (task card workflow): [evidence span withheld — anchor status: not_found]
- [methods] snapccess install command (methods): "pip install snapccess  --index-url https://pypi.org/simple"
- [abstract] SnapCCESS creates VAE snapshots coupled with clustering (abstract): "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells"
- [readme] Tutorial parameter caveat (readme): "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets. For the datasets used in the published paper associated with this package, the"
