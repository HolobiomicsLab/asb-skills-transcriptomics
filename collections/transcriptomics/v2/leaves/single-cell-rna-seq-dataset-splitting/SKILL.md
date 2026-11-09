---
name: single-cell-rna-seq-dataset-splitting
description: Use when training a FAVA VAE (or similar latent-space model) on single-cell
  RNA-seq data and you need to demonstrate the model generalizes to unseen cells rather
  than memorizing the training set.
when_to_use_negative:
- When the goal is producing the final FAVA functional association network from all
  available cells — the full dataset should be used, not a split
- When input data are bulk omics, proteomics-only, or already aggregated condition-level
  data rather than single-cell scRNA-seq
- When generalization can be tested on an external independent dataset; a random within-dataset
  split only checks memorization of the same cell population
tools:
- name: FAVA (favapy)
  role: Trains the VAE on the training split and computes latent-space correlations;
    pairs_after_cutoff ranks protein pairs from both splits
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data container in which the loaded 'tsv' scRNA-seq data are held and
    split into train/test partitions
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- name: Python
  role: Language in which the favapy module (VAE, load_data, pairs_after_cutoff) is
    invoked for the split-and-evaluate workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:cc99a4bf772a3e96bd68c2e8c8309620e919e3f7e2ff87abb7eeb9212866a44e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA (favapy)
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-rna-seq-dataset-splitting

## Summary

A train/test generalization (overfitting) check for FAVA: partition single-cell scRNA-seq data (e.g. GSE75748) into a 67% training and 33% test split, train the VAE on training cells only, and verify that latent-space correlations and KEGG-based network benchmark performance are similar across both splits.

## When to use

Use when training a FAVA VAE (or similar latent-space model) on single-cell RNA-seq data and you need to demonstrate the model generalizes to unseen cells rather than memorizing the training set.

## When NOT to use

- When the goal is producing the final FAVA functional association network from all available cells — the full dataset should be used, not a split
- When input data are bulk omics, proteomics-only, or already aggregated condition-level data rather than single-cell scRNA-seq
- When generalization can be tested on an external independent dataset; a random within-dataset split only checks memorization of the same cell population

## Inputs

- GSE75748 scRNA-seq dataset downloaded from GEO (tsv format)
- AnnData object produced by favapy's load_data
- KEGG pathway co-membership gold standard for benchmarking ranked protein pairs

## Outputs

- Trained FAVA VAE model fitted on the 67% training split only
- Two ranked protein-pair lists (one from training cells, one from test cells), each with default 100000 interactions in both directions
- Pearson correlation distributions in the latent space for both splits
- KEGG benchmark comparison analogous to Supplementary Fig. S5

## How to apply

Download the GSE75748 scRNA-seq dataset from GEO and load it with favapy's load_data into an AnnData object ('tsv' input type). Randomly partition cells into a 67% training split and 33% test split. Train the FAVA VAE only on the training split using documented defaults (50 epochs, batch size 32, hidden/latent dimensions sized automatically from the input), then run the trained model on both splits to compute Pearson correlations in the latent space. Rank protein pairs with pairs_after_cutoff using the default 100000 interactions in both directions (proteinA-proteinB and proteinB-proteinA). Evaluate both ranked lists against the KEGG pathway co-membership gold standard and confirm benchmark performance is similar between the training and test splits, reproducing Supplementary Fig. S5.

## Related tools

- **FAVA (favapy)** (Trains the VAE on the training split and computes latent-space correlations; pairs_after_cutoff ranks protein pairs from both splits) — https://github.com/mikelkou/fava
- **AnnData** (Input data container in which the loaded 'tsv' scRNA-seq data are held and split into train/test partitions) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- **Python** (Language in which the favapy module (VAE, load_data, pairs_after_cutoff) is invoked for the split-and-evaluate workflow)

## Examples

```
from favapy import load_data, VAE, pairs_after_cutoff; adata = load_data('GSE75748.tsv', input_type='tsv'); model = VAE(); model.train(train_split, epochs=50, batch_size=32)
```

## Evaluation signals

- Pearson correlations in the latent space are of similar magnitude for training and test cells
- KEGG pathway co-membership benchmark performance is similar for the training-split and test-split ranked pair lists
- Training used documented defaults: 50 epochs, batch size 32, and a 67%/33% cell partition
- pairs_after_cutoff produced 100000 interactions in both directions (proteinA-proteinB and proteinB-proteinA) per split
- Result reproduces the pattern of Supplementary Fig. S5

## Limitations

- The random split tests generalization within the same cell population; it does not demonstrate transfer to independent datasets or different tissues
- Overfitting conclusions rely on benchmark similarity, so a weak or mismatched gold standard (e.g. KEGG co-membership) could mask over- or under-fitting
- The paper's overfitting evidence is reported in a supplementary figure (Supplementary Fig. S5), so the split protocol must be re-derived from the described defaults

## Evidence

- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [intro] latent space correlations improve networks: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [supplementary] train/test similarity finding: "Similar performance was obtained for both training and test sets (Supplementary Fig. S5), suggesting that our models do not overfit the data"
- [methods] CLI invocation: [evidence span withheld — anchor status: not_found]
