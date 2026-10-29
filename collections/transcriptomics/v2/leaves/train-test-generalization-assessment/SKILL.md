---
name: train-test-generalization-assessment
description: Use when a trained FAVA VAE model must be checked for overfitting to
  the input scRNA-seq cells, i.e. you need evidence that latent-space performance
  generalizes to unseen cells.
when_to_use_negative:
- Input data is not single-cell expression data (e.g. bulk RNA-seq or proteomics tables),
  since the split is performed over cells
- You only have a single trained model and no held-out cell population — the check
  requires two disjoint cell splits evaluated with the same model
- The goal is to assess prediction accuracy against a benchmark, not generalization
  — use the standard FAVA evaluation workflow instead
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2269
tools:
- name: FAVA (favapy)
  role: Trains the VAE on the training split and computes latent-space Pearson correlations
    and ranked pairs on both splits
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: 'Input format: GSE75748 cells are loaded into AnnData via favapy load_data
    before splitting'
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Host language for running favapy and implementing the 67/33 cell partition
    and KEGG evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:0e15a658c4d1a1653737029b95f94e71800b356b8abca044b8caa40fda26adf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# train-test-generalization-assessment

## Summary

A generalization (overfitting) check for the FAVA variational autoencoder: train on a subset of cells, benchmark the model separately on held-out cells, and confirm that performance is comparable between splits.

## When to use

Use when a trained FAVA VAE model must be checked for overfitting to the input scRNA-seq cells, i.e. you need evidence that latent-space performance generalizes to unseen cells.

## When NOT to use

- Input data is not single-cell expression data (e.g. bulk RNA-seq or proteomics tables), since the split is performed over cells
- You only have a single trained model and no held-out cell population — the check requires two disjoint cell splits evaluated with the same model
- The goal is to assess prediction accuracy against a benchmark, not generalization — use the standard FAVA evaluation workflow instead

## Inputs

- GSE75748 scRNA-seq dataset from GEO in .tsv format
- AnnData object loaded via favapy load_data (input type 'tsv')
- KEGG pathway co-membership gold standard for evaluation

## Outputs

- Ranked protein-pair list for the training split (up to 100000 interactions, both directions)
- Ranked protein-pair list for the test split (up to 100000 interactions, both directions)
- Comparison of KEGG benchmark performance between training and test splits (cf. Supplementary Fig. S5)

## How to apply

Download the GSE75748 scRNA-seq dataset from GEO and load it with favapy's load_data into AnnData format using the 'tsv' input type. Randomly partition the cells into a 67% training split and a 33% test split, then train the FAVA VAE on the training split only with the documented default settings (50 epochs, batch size 32, hidden and latent dimensions sized automatically from the input). Run the trained model on both splits to compute Pearson correlations in the latent space, rank protein pairs with pairs_after_cutoff (default 100000 interactions, covering both directions proteinA-proteinB and proteinB-proteinA), and evaluate both ranked lists against a KEGG pathway co-membership gold standard. The check passes when benchmark performance on the test split is similar to the training split, reproducing Supplementary Fig. S5; a large gap between splits would indicate overfitting.

## Related tools

- **FAVA (favapy)** (Trains the VAE on the training split and computes latent-space Pearson correlations and ranked pairs on both splits) — https://github.com/mikelkou/fava
- **AnnData** (Input format: GSE75748 cells are loaded into AnnData via favapy load_data before splitting) — https://github.com/mikelkou/fava
- **Python** (Host language for running favapy and implementing the 67/33 cell partition and KEGG evaluation)

## Examples

```
favapy <path-to-gse75748-data.tsv> <path-to-save-output>
```

## Evaluation signals

- Test-split KEGG benchmark performance is similar to training-split performance (reproduces Supplementary Fig. S5)
- Both splits yield ranked lists of up to 100000 protein-pair interactions from pairs_after_cutoff, covering both proteinA-proteinB and proteinB-proteinA directions
- Pearson correlations are computed in the low-dimensional latent space for both training and test cells with the same trained model
- The training split contains 67% of cells and the test split 33%, with no cell overlap between splits

## Limitations

- A random split of single cells may share batch or cell-type structure between train and test sets, so 'similar performance' does not fully rule out leakage across correlated cells
- The check was performed with the GSE75748 dataset and default FAVA settings (50 epochs, batch size 32); other datasets or hyperparameters may behave differently
- Evaluation depends on the KEGG pathway co-membership gold standard, which is itself literature-biased

## Evidence

- [supplementary] Similar performance was obtained for both training and test sets (Supplementary Fig. S5), suggesting that our models do not overfit the data: "Similar performance was obtained for both training and test sets (Supplementary Fig. S5), suggesting that our models do not overfit the data"
- [methods] load_data: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
