---
name: rna-seq-expression-classification
description: Use when you need to evaluate whether genes from extracted active subnetworks
  discriminate between two patient groups (case/control) and to compare classifier
  performance (accuracy, predictor gene set size) across subnetwork detection methods.
when_to_use_negative:
- Input data are raw counts without the nonparanormal transform applied — normality
  assumptions needed for SEM/LDA-based classification are violated; run SEMgraph::transformData
  first.
- The task is subnetwork extraction itself rather than classification benchmarking
  — use the upstream SEMtree/Steiner-tree skill instead.
- Only one sample group is available (no binary case/control label vector) — the penalized
  discriminant classifier requires the group vector to train.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: SEMgraph
  role: Provides transformData for nonparanormal (shrunk ECDF) transformation of the
    RNA-seq matrix prior to classification; supplies the SEMtree functions that generated
    the subnetworks being benchmarked
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: PenalizedLDA
  role: Fits the penalized Fisher's discriminant classifier on subnetwork genes to
    predict critical vs non-critical status
- name: SEMdata
  role: Provides the COVID-19 RNA-seq dataset (GSE172114) used as classification input
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Gene ID conversion (e.g., Entrez to gene symbols) so subnetwork genes can
    be matched to expression matrix columns
- name: igraph
  role: Subnetworks are handled as igraph objects when subsetting genes per subnetwork
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:bd7d41a62dcc53b838f961eff4f34b15aef674b1a45ebcb36b9275428390f82f
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
  - igraph
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rna-seq-expression-classification

## Summary

Classifies patients (e.g., critical vs non-critical COVID-19 cases from GSE172114) by fitting a PenalizedLDA penalized Fisher's discriminant classifier on genes extracted from SEMtree-derived active subnetworks, benchmarking classifier accuracy and parsimony per subnetwork detection method (mirroring Table 3 of the SEMtree paper).

## When to use

Use when you need to evaluate whether genes from extracted active subnetworks discriminate between two patient groups (case/control) and to compare classifier performance (accuracy, predictor gene set size) across subnetwork detection methods.

## When NOT to use

- Input data are raw counts without the nonparanormal transform applied — normality assumptions needed for SEM/LDA-based classification are violated; run SEMgraph::transformData first.
- The task is subnetwork extraction itself rather than classification benchmarking — use the upstream SEMtree/Steiner-tree skill instead.
- Only one sample group is available (no binary case/control label vector) — the penalized discriminant classifier requires the group vector to train.

## Inputs

- COVID-19 RNA-seq expression matrix (GEO: GSE172114; 46 critical and 23 non-critical cases)
- Binary group vector with {1: case, 0: control} convention
- Extracted active subnetworks from upstream SEMtree sub-task (Steiner-tree backbone ST, STr2z variants, and five active subnetwork detection methods)
- Nonparanormal-transformed expression data (SEMgraph::transformData)

## Outputs

- Per-subnetwork out-of-sample classification accuracy
- Number of predictor genes selected by each PenalizedLDA classifier
- classifier_accuracy.csv mirroring the Table 3 classifier block (accuracy + parsimony per method)

## How to apply

Load the RNA-seq dataset with a binary group vector (1 = case, 0 = control) and apply SEMgraph::transformData nonparanormal (shrunk ECDF) transformation to relax the normality assumption. For each active subnetwork produced by the upstream detection methods (e.g., the Steiner-tree backbone from SEMtree type = 'ST', eweight = 'pvalue', and its r2z-weighted ST/STr2z variants), subset the transformed expression matrix to that subnetwork's genes and fit a PenalizedLDA penalized Fisher's discriminant classifier. Estimate out-of-sample classification accuracy for each method and record the number of predictor genes selected by each classifier. Assemble the accuracy and parsimony table (classifier_accuracy.csv) mirroring the Table 3 classifier block; correct reproduction shows most methods exceed 90% accuracy with COSINE and WGM_FC around 80%, and tree-based ST/STr2z subnetworks giving the smallest predictor gene sets.

## Related tools

- **SEMgraph** (Provides transformData for nonparanormal (shrunk ECDF) transformation of the RNA-seq matrix prior to classification; supplies the SEMtree functions that generated the subnetworks being benchmarked) — https://github.com/fernandoPalluzzi/SEMgraph
- **PenalizedLDA** (Fits the penalized Fisher's discriminant classifier on subnetwork genes to predict critical vs non-critical status)
- **SEMdata** (Provides the COVID-19 RNA-seq dataset (GSE172114) used as classification input) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Gene ID conversion (e.g., Entrez to gene symbols) so subnetwork genes can be matched to expression matrix columns)
- **igraph** (Subnetworks are handled as igraph objects when subsetting genes per subnetwork)

## Examples

```
data.npn <- transformData(exprs)$data; fit <- PenalizedLDA(x = t(exprs.npn[, subnetwork_genes]), y = group, K = 1)
```

## Evaluation signals

- Most subnetwork detection methods achieve classification accuracy above 90%; COSINE and WGM_FC fall below 90% (around 80%)
- ST/STr2z tree-based subnetworks yield the most parsimonious (smallest) predictor gene sets among methods
- classifier_accuracy.csv reproduces the Table 3 classifier block pattern of the SEMtree paper
- Expression matrix was subset to each subnetwork's gene set before classifier fitting (verify gene-set sizes per method)
- Nonparanormal transform (transformData) was applied before classification — verify the input matrix is the transformed data.npn-style object

## Limitations

- Benchmark accuracy depends on the upstream subnetwork detection quality; methods like COSINE and WGM_FC plateau near 80% accuracy on this dataset.
- Binary group labeling must follow the {1: case, 0: control} convention; other encodings may break classifier training.
- Discussion section text is absent from the source document, so caveats about classifier generalizability to other cohorts are not available from this source.
- The dataset is young-patient COVID-19 (46 critical vs 23 non-critical); performance numbers may not transfer to other populations or diseases.

## Evidence

- [results] Finding: classifier performance vs method: [evidence span withheld — anchor status: not_found]
- [methods] Method: PenalizedLDA on subnetwork genes: [evidence span withheld — anchor status: not_found]
- [methods] Nonparanormal transformation: "we apply a nonparanormal transform to relax the normality assumption."
- [abstract] Abstract: classifier performance claim: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
- [readme] README: COVID-19 dataset description: [evidence span withheld — anchor status: external_doc]
