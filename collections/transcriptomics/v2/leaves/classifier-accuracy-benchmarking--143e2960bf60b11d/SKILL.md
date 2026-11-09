---
name: classifier-accuracy-benchmarking
description: Use when you have candidate subnetworks (e.g. Steiner-tree or r2z-weighted
  variants from SEMtree) plus a binary patient label vector and need to compare subnetwork
  detection methods by out-of-sample classification accuracy and predictor-gene parsimony.
when_to_use_negative:
- When the goal is subnetwork extraction or perturbation scoring itself (SEMrun node/edge
  perturbation, SEMace) rather than downstream classification benchmarking.
- 'When the input is an unlabeled or multi-class dataset with no binary case/control
  group vector — PenalizedLDA here requires a binary {1: case, 0: control} label.'
- When comparing classifier algorithms (e.g. DNN vs ML) rather than benchmarking subnetwork
  detection methods with the paper's fixed PenalizedLDA setup.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SEMgraph
  role: Provides transformData for the nonparanormal (shrunk ECDF) transformation
    and hosts the SEMtree tree-based structure learning workflow whose subnetworks
    are benchmarked
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Supplies the COVID-19 RNA-seq dataset GSE172114 (46 critical and 23 non-critical
    cases) used for the classification benchmark
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: PenalizedLDA
  role: Penalized Fisher's discriminant classifier fitted on each subnetwork's gene
    subset to predict critical vs non-critical status
- name: SEMtrees/SEMtrees
  role: Extracts the Steiner-tree backbone (type='ST', eweight='pvalue') and r2z-weighted
    subnetworks that serve as the gene sets being benchmarked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:0d0752782466340e3e2061fb35f076f18038cd3ae01ed5ddee95ab1ecfef4ad4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - PenalizedLDA
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - SEMtrees/SEMtrees
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classifier-accuracy-benchmarking

## Summary

Benchmarks how well the genes in each extracted subnetwork discriminate case vs control patients using a PenalizedLDA penalized Fisher's discriminant classifier, mirroring the Table 3 classifier block of the SEMtree paper where most active subnetwork detection methods exceed 90% accuracy on the COVID-19 critical vs non-critical task.

## When to use

Use when you have candidate subnetworks (e.g. Steiner-tree or r2z-weighted variants from SEMtree) plus a binary patient label vector and need to compare subnetwork detection methods by out-of-sample classification accuracy and predictor-gene parsimony.

## When NOT to use

- When the goal is subnetwork extraction or perturbation scoring itself (SEMrun node/edge perturbation, SEMace) rather than downstream classification benchmarking.
- When the input is an unlabeled or multi-class dataset with no binary case/control group vector — PenalizedLDA here requires a binary {1: case, 0: control} label.
- When comparing classifier algorithms (e.g. DNN vs ML) rather than benchmarking subnetwork detection methods with the paper's fixed PenalizedLDA setup.

## Inputs

- COVID-19 RNA-seq expression matrix (GEO: GSE172114, 46 critical / 23 non-critical cases)
- Binary critical vs non-critical patient label vector ({1: case, 0: control} convention)
- Extracted COVID-19 active subnetworks from the upstream SEMtree sub-task (Steiner-tree backbone ST with eweight='pvalue' and r2z-weighted ST/STr2z variants from five active subnetwork detection methods)
- Nonparanormal-transformed expression data via SEMgraph::transformData

## Outputs

- Per-subnetwork out-of-sample classification accuracy values
- Per-subnetwork count of predictor genes selected by each PenalizedLDA classifier
- classifier_accuracy.csv table mirroring the Table 3 classifier block of the SEMtree paper

## How to apply

Load the COVID-19 RNA-seq dataset (GEO: GSE172114; 46 critical and 23 non-critical cases) with a binary label vector using the {1: case, 0: control} convention, and apply SEMgraph::transformData nonparanormal (shrunk ECDF) transformation to relax the normality assumption. For each extracted subnetwork (e.g. the Steiner-tree backbone SEMtree type='ST', eweight='pvalue', and its r2z-weighted ST/STr2z variants from the five active subnetwork detection methods), subset the transformed expression matrix to that subnetwork's genes and fit a PenalizedLDA penalized Fisher's discriminant classifier to predict critical vs non-critical status. Estimate out-of-sample classification accuracy for each method and record the number of predictor genes selected by each classifier, then assemble the accuracy and parsimony table (classifier_accuracy.csv) mirroring the Table 3 classifier block. The validation expectation is that most subnetwork methods exceed 90% accuracy, only COSINE and WGM_FC fall below 90% (around 80%), and the ST/STr2z tree-based subnetworks yield the most parsimonious (smallest) predictor gene sets.

## Related tools

- **SEMgraph** (Provides transformData for the nonparanormal (shrunk ECDF) transformation and hosts the SEMtree tree-based structure learning workflow whose subnetworks are benchmarked) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Supplies the COVID-19 RNA-seq dataset GSE172114 (46 critical and 23 non-critical cases) used for the classification benchmark) — https://github.com/fernandoPalluzzi/SEMdata
- **PenalizedLDA** (Penalized Fisher's discriminant classifier fitted on each subnetwork's gene subset to predict critical vs non-critical status)
- **SEMtrees/SEMtrees** (Extracts the Steiner-tree backbone (type='ST', eweight='pvalue') and r2z-weighted subnetworks that serve as the gene sets being benchmarked)

## Examples

```
data.npn <- SEMgraph::transformData(exprs)$data; fit <- PenalizedLDA::PenalizedLDA(x=t(data.npn[genes,]), y=group, lambda=1e-4)
```

## Evaluation signals

- Most subnetwork detection methods reproduce accuracy above 90% for classifying patients as case or non-case.
- COSINE and WGM_FC fall below 90%, around 80%, matching the Table 3 pattern.
- ST/STr2z tree-based subnetworks yield the smallest (most parsimonious) predictor gene sets.
- classifier_accuracy.csv contains an accuracy plus predictor-gene-count row for each of the five active subnetwork detection methods.
- Expression matrix used for classifier fitting is the nonparanormal (transformData) version, not raw counts.

## Limitations

- Only binary case/control discrimination (critical vs non-critical) is benchmarked; no multi-class evaluation is described.
- Performance figures are dataset-specific to GSE172114 COVID-19 RNA-seq and may not transfer to other cohorts.
- Accuracy alone is insufficient — parsimony (number of predictor genes) must be reported alongside to distinguish methods with similar accuracy.

## Evidence

- [results] In the benchmark pFDA classification, most subnetwork detection methods achieve accuracy above 90% in classifying patients as case or non-case: [evidence span withheld — anchor status: not_found]
- [abstract] Classifier performance claim from abstract: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
- [methods] Nonparanormal transformation step: "we apply a nonparanormal transform to relax the normality assumption."
- [readme] COVID-19 dataset description: [evidence span withheld — anchor status: external_doc]
- [abstract] Five active subnetwork methods generate perturbed modules: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
