---
name: hierarchical-classifier-configuration
description: Use when annotating human scRNA-seq cells against the Cell Ontology with
  a pre-trained, ready-to-run classifier, or when benchmarking CellO against scMatch/SingleR
  by projecting flat labels onto ontology term IDs.
when_to_use_negative:
- Non-human data (e.g., mouse) — CellO is pre-trained only on human primary cell bulk
  RNA-seq from the SRA and has no curated mouse training set
- When the user requires marker-gene-driven or custom-hierarchy annotation (e.g.,
  Garnett-style pre-specified hierarchies) rather than Cell Ontology-grounded hierarchical
  classification
- Input samples needing a new trained model when genes are incompatible, if the goal
  is out-of-the-box comparison against ready-to-run baselines rather than de novo
  training
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: 'Primary hierarchical classifier: annotates cells against Cell Ontology term
    IDs using isotonic regression and outputs probability/binary/most-specific tables'
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring the gene-wise linear-model coefficients of CellO's
    per-cell-type classifiers across the ontology
  repo: https://uwgraphics.github.io/CellOViewer/
- name: scMatch
  role: Ready-to-run flat annotation baseline run with the FANTOM5 reference, with
    labels projected onto Cell Ontology IDs for comparison
- name: SingleR
  role: Ready-to-run flat annotation baseline run with Blueprint+ENCODE and HPCA references,
    with labels projected onto Cell Ontology IDs for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:5458d256bad1534347d1936ce8cc35f49f450c827eca034814c4266ca35b7260
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deweylab/CellO
  repo_url: https://github.com/deweylab/CellO
  tools:
  - CellO
  - CellO Viewer
  - SingleR
  - scMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hierarchical-classifier-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run CellO, a supervised hierarchical classifier that annotates human scRNA-seq cells against the graph-structured Cell Ontology using isotonic regression (algo='IR'), producing per-cell probability, binary-decision, and most-specific cell-type tables that can be benchmarked against flat classifiers such as scMatch and SingleR.

## When to use

Use when annotating human scRNA-seq cells against the Cell Ontology with a pre-trained, ready-to-run classifier, or when benchmarking CellO against scMatch/SingleR by projecting flat labels onto ontology term IDs.

## When NOT to use

- Non-human data (e.g., mouse) — CellO is pre-trained only on human primary cell bulk RNA-seq from the SRA and has no curated mouse training set
- When the user requires marker-gene-driven or custom-hierarchy annotation (e.g., Garnett-style pre-specified hierarchies) rather than Cell Ontology-grounded hierarchical classification
- Input samples needing a new trained model when genes are incompatible, if the goal is out-of-the-box comparison against ready-to-run baselines rather than de novo training

## Inputs

- Human scRNA-seq gene expression matrix in TSV, CSV, HDF5, or 10x format (matrix.mtx, genes.tsv, barcodes.tsv)
- Pre-trained CellO model file (.model.dill), optional when retraining with -t
- Optional pre-clustering TSV (cell ID to integer cluster) for the -p option
- Benchmark test set from the Zenodo deposit 10.5281/zenodo.4289064 (non-droplet single-cell data)
- Optional Uberon tissue term IDs for the -l anatomical filter

## Outputs

- <output_prefix>.probability.tsv: N cells x M cell types classification probability table
- <output_prefix>.binary.tsv: N x M binary-decision matrix over Cell Ontology terms
- <output_prefix>.most_specific.tsv: mapping of each cell to its most-specific predicted cell type
- Per-cell-type F1-score and precision TSV for CellO, scMatch, and SingleR
- Figure 5B-style F1 and precision distribution plots

## How to apply

Install CellO via `pip install cello-classify` and supply a gene expression matrix in TSV, CSV, HDF5, or 10x format whose genes match the pre-trained classifier trained on human primary bulk RNA-seq from the SRA. Specify the units (-u, e.g. 'COUNTS' for 10x or 'LOG1_TPM' for Kallisto output) and assay type (-s '3_PRIME' or 'FULL_LENGTH'), and keep the default hierarchical algorithm -a IR (isotonic regression) or switch to 'CLR' cascaded logistic regression. CellO writes a .probability.tsv (N cells x M cell types), a .binary.tsv decision matrix, and a .most_specific.tsv mapping each cell to its most-specific predicted Cell Ontology term. For the CellO-vs-scMatch/SingleR benchmark, download the non-droplet test set from Zenodo (10.5281/zenodo.4289064), classify with CellO (IR), scMatch (FANTOM5 reference), and SingleR (Blueprint+ENCODE and HPCA), then project the flat scMatch/SingleR labels onto Cell Ontology term IDs. Restrict evaluation per method to cell types present in that method's own training set and compute per-cell-type F1-score and precision, saving metrics as a TSV and rendering Figure 5B-style distributions.

## Related tools

- **CellO** (Primary hierarchical classifier: annotates cells against Cell Ontology term IDs using isotonic regression and outputs probability/binary/most-specific tables) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring the gene-wise linear-model coefficients of CellO's per-cell-type classifiers across the ontology) — https://uwgraphics.github.io/CellOViewer/
- **scMatch** (Ready-to-run flat annotation baseline run with the FANTOM5 reference, with labels projected onto Cell Ontology IDs for comparison)
- **SingleR** (Ready-to-run flat annotation baseline run with Blueprint+ENCODE and HPCA references, with labels projected onto Cell Ontology IDs for comparison)

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- The .probability.tsv table is populated for all cells with probabilities, and the .binary.tsv and .most_specific.tsv outputs are consistent with it
- Per-cell-type F1 and precision metrics are populated for all three methods (CellO, scMatch, SingleR) after projecting flat labels onto Cell Ontology term IDs
- F1-score and precision distributions qualitatively match Figure 5B: CellO competitive with or superior to scMatch and SingleR when evaluated on each method's own training-set cell types
- Genes in the input matrix match the genes expected by the pre-trained classifier (otherwise genes_absent_from_training_set.tsv lists missing genes and a -t retrain is needed)
- Expression units and assay flags (-u, -s) match the data source, e.g. COUNTS for 10x data and LOG1_TPM for Kallisto output

## Limitations

- CellO's calibration gap: performance measured with F1-score lags performance measured with average precision, so binary yes-no decisions are less reliable
- Determining optimal clustering in scRNA-seq is an open problem that affects cluster-based annotation accuracy
- Independent per-cell-type classifiers can be inconsistent when cell types share a parent via the 'is a' relationship but have divergent evolutionary origins
- Only cell types present in each method's training set are comparable in the benchmark; classifiers are pre-trained only on human, healthy, untreated primary SRA samples

## Evidence

- [results] benchmark comparison finding: [evidence span withheld — anchor status: not_found]
