---
name: cell-annotation-method-benchmarking
description: Use when evaluating or comparing cell-type annotation methods on multi-timepoint
  scRNA-seq data, especially when later-timepoint cell types may shift in composition,
  disappear, or emerge as novel types.
when_to_use_negative:
- The task is annotating a single cross-sectional timepoint with no paired t0 training
  data — CASi's information-borrowing design and the simulation scenarios target multi-timepoint
  designs
- The t0 and t1 datasets differ dramatically (e.g., different tissues or platforms),
  since the neural network would fail to borrow useful information and could not be
  well trained
- Only unsupervised clustering on one timepoint is needed without any method comparison
  against competing supervised annotators
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: Cross-timepoint annotator being benchmarked; CreateCASiList builds the t0/t1/labels
    input and CrossTimeAnnotation trains the Keras/TensorFlow neural network
  repo: github.com/yizhuo-wang/CASi
- name: keras
  role: Required dependency providing the TensorFlow backend for CASi's neural network
    classifier
- name: scmap
  role: Competing supervised annotation method run on identical t0/t1 splits
- name: CHETAH
  role: Competing supervised annotation method run on identical t0/t1 splits
- name: scPred
  role: Competing supervised annotation method run on identical t0/t1 splits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:00296061dca365af6a8942c9ea94616f52cfab1e844f20d80771fe25996e13b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - CHETAH
  - NeuCA
  - R
  - SC3
  - Seurat
  - keras
  - scPred
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- cell-annotation-benchmark-evaluation
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-annotation-method-benchmarking

## Summary

Benchmark supervised cross-timepoint single-cell RNA-seq annotation methods by simulating paired t0/t1 PBMC data under defined composition-change scenarios and comparing per-cell annotation accuracy and adjusted Rand index (ARI) across methods over Monte Carlo repetitions.

## When to use

Use when evaluating or comparing cell-type annotation methods on multi-timepoint scRNA-seq data, especially when later-timepoint cell types may shift in composition, disappear, or emerge as novel types.

## When NOT to use

- The task is annotating a single cross-sectional timepoint with no paired t0 training data — CASi's information-borrowing design and the simulation scenarios target multi-timepoint designs
- The t0 and t1 datasets differ dramatically (e.g., different tissues or platforms), since the neural network would fail to borrow useful information and could not be well trained
- Only unsupervised clustering on one timepoint is needed without any method comparison against competing supervised annotators

## Inputs

- Gene expression matrix of the initial timepoint (t0) cell counts
- Gene expression matrix of the following timepoint (t1) cell counts
- Cell labels of the initial timepoint (t0), optional
- Publicly available PBMC dataset (more than 60,000 sorted cells from eight immune cell types) or pbmc_demo data lazy-loaded with the CASi package

## Outputs

- Per-cell annotation labels for t1 cells from each method (CASi, scmap, CHETAH, scPred)
- Per-cell annotation accuracy per method per simulation scenario, aggregated over 200 Monte Carlo repetitions
- Adjusted Rand index (ARI) comparisons across the three scenarios (as reported in supplementary Figure 11)
- Final aggregated benchmark table written to annotation_benchmark_results.csv

## How to apply

Load the public sorted PBMC dataset (more than 60,000 cells from eight immune cell types) or the lazy-loaded pbmc_demo data, and randomly draw cells from five cell types as the simulation pool. Generate paired t0/t1 count matrices under three scenarios: same cell types with shifted compositions, a disappearing cell type, and a novel emerging cell type. For each of 200 Monte Carlo repetitions, build the CASi list with CreateCASiList (t0 counts, t1 counts, t0 labels), train the Keras/TensorFlow neural network via CrossTimeAnnotation to label t1 cells, and run scmap, CHETAH, and scPred on the identical t0/t1 splits. Record per-cell annotation accuracy for each method per scenario, and compute adjusted Rand index (ARI) comparisons for unsupervised clustering under the same scenario settings. Aggregate accuracies and ARIs across replicates and scenarios and write the final table to annotation_benchmark_results.csv.

## Related tools

- **CASi** (Cross-timepoint annotator being benchmarked; CreateCASiList builds the t0/t1/labels input and CrossTimeAnnotation trains the Keras/TensorFlow neural network) — github.com/yizhuo-wang/CASi
- **keras** (Required dependency providing the TensorFlow backend for CASi's neural network classifier)
- **scmap** (Competing supervised annotation method run on identical t0/t1 splits)
- **CHETAH** (Competing supervised annotation method run on identical t0/t1 splits)
- **scPred** (Competing supervised annotation method run on identical t0/t1 splits)

## Examples

```
require(keras); require(CASi); CASiList <- CreateCASiList(t0_counts, t1_counts, t0_labels); res <- CrossTimeAnnotation(CASiList)
```

## Evaluation signals

- In scenarios 1 (shifted compositions) and 2 (disappearing cell type), CASi should show the highest accuracy and ARI among the compared methods
- In scenario 3 (novel emerging cell type), adding the DetectNovelCells pipeline should substantially raise CASi's accuracy and ARI versus the baseline annotation alone
- Every method is evaluated on identical t0/t1 splits across 200 Monte Carlo repetitions, so variance across replicates should be reported rather than a single split
- The aggregated output table is written to annotation_benchmark_results.csv with per-method, per-scenario accuracy and ARI entries

## Limitations

- If the initial timepoint's data is dramatically different from later timepoint's data, the neural network would fail to borrow useful information and could not be very well trained
- The novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types
- The current cell type annotation step pools the data from all the samples together for training, which may be inappropriate when cell populations differ across conditions
- ARI comparison results for the three scenarios are only available in supplementary Figure 11, not in the main text

## Evidence

- [intro] Neural network cross-timepoint annotation is the skill's core mechanism: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [abstract] Benchmark demonstrates favorable performance versus existing methods: "we demonstrate the robust and favorable performance of the proposal versus existing methods serving similar purposes"
- [methods] PBMC source dataset scale for the simulation pool: "We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types."
- [methods] CrossTimeAnnotation is the key CASi function for the annotation benchmark: "Below are the input parameters of the CrossTimeAnnotation function"
- [discussion] ARI comparison results live in supplementary Figure 11: "The results of three scenarios are included in the supplementary file Figure 11."
- [readme] CASi package installation and keras/TensorFlow dependency from the README: [evidence span withheld — anchor status: external_doc]
