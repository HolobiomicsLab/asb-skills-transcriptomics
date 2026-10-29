---
name: soft-thresholding-power-selection
description: Use when constructing a weighted gene co-expression network from a bulk
  RNA-seq count matrix (e.g. the 192-sample 5xFAD/C57BL/6J cortex+hippocampus or 38-sample
  3xTg-AD hippocampus datasets) and a soft-thresholding power must be chosen before
  adjacency and module detection.
when_to_use_negative:
- Input is a normalized single-gene differential expression table with no sample-level
  correlation structure to model — PyWGCNA's network construction (and thus power
  selection) is not applicable
- The goal is only functional enrichment or PPI analysis of an already-defined gene
  list — use the functional_enrichment_analysis or protein_protein_interaction tutorials
  instead of re-deriving a network
- Data are single-cell expression profiles, where the bulk RNA-seq co-expression module
  framework described here does not directly apply
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Provides pickSoftThreshold for scanning candidate powers against scale-free
    topology fit, plus adjacency/TOM computation and dynamic tree cut module detection
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime environment for PyWGCNA; version 3.9 or greater required per the article
    (README states 3.10 or greater)
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:26676a80ab085e46717f16ccdc566e34f2ccd9326947f0de0543687935dd7862
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# soft-thresholding-power-selection

## Summary

Selects the soft-thresholding power (pickSoftThreshold) that best brings gene-gene correlation adjacency matrices to approximate scale-free topology before TOM-based module detection in PyWGCNA. This step determines the weighting exponent that defines the WGCNA network and directly affects the number and size of co-expression modules recovered.

## When to use

Use when constructing a weighted gene co-expression network from a bulk RNA-seq count matrix (e.g. the 192-sample 5xFAD/C57BL/6J cortex+hippocampus or 38-sample 3xTg-AD hippocampus datasets) and a soft-thresholding power must be chosen before adjacency and module detection.

## When NOT to use

- Input is a normalized single-gene differential expression table with no sample-level correlation structure to model — PyWGCNA's network construction (and thus power selection) is not applicable
- The goal is only functional enrichment or PPI analysis of an already-defined gene list — use the functional_enrichment_analysis or protein_protein_interaction tutorials instead of re-deriving a network
- Data are single-cell expression profiles, where the bulk RNA-seq co-expression module framework described here does not directly apply

## Inputs

- Bulk RNA-seq gene x sample count matrix (e.g. 192-sample 5xFAD/C57BL/6J cortex and hippocampus matrix)
- Sample trait table (age, genotype, tissue, sex) for module-trait association
- Low-expression-filtered expression matrix from PyWGCNA Data_format preprocessing

## Outputs

- Selected soft-thresholding power with scale-free topology fit statistics (pickSoftThreshold output)
- Weighted adjacency matrix built from scaled correlations at the chosen power
- Topological overlap matrix (TOM) and TOM dissimilarity used for dynamic tree cut

## How to apply

First load the count matrix into a PyWGCNA object and preprocess per the Data_format tutorial, filtering genes with low counts to retain only well-expressed genes for network construction. Then scan candidate soft-thresholding powers (pickSoftThreshold) and select the power giving the best scale-free topology fit of the network, as done in both the 5xFAD and 3xTg-AD workflows. Compute the adjacency from the scaled correlations using the chosen power, then build the topological overlap matrix (TOM) and cut modules from the TOM dissimilarity. PyWGCNA requires Python 3.9+ (README states 3.10+) and can be installed from PyPi; it implements this faster than the R WGCNA package. The chosen power feeds downstream module eigengene computation and module-trait correlation against age, genotype, tissue, and sex traits.

## Related tools

- **PyWGCNA** (Provides pickSoftThreshold for scanning candidate powers against scale-free topology fit, plus adjacency/TOM computation and dynamic tree cut module detection) — https://github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment for PyWGCNA; version 3.9 or greater required per the article (README states 3.10 or greater))

## Examples

```
pip install PyWGCNA; pywgcna = PyWGCNA.WGCNA(name='5xFAD', inputFilePath='counts.csv'); pywgcna.pickSoftThreshold(powers=range(1, 21))
```

## Evaluation signals

- The 3xTg-AD reference run (38 female hippocampus samples, three ages) must yield exactly 17 modules, with the dark gray module containing exactly 380 genes and correlating with 3xTg-AD genotype and the 18-month time point
- The 5xFAD reference run (192 samples, four ages, both sexes) must recover 17 co-expression modules, including the coral module (1335 genes) and white module (435 genes), associated with age, genotype, tissue, and sex
- The exported module_assignments.csv must contain a gene ID column and an assigned module color column with every retained gene mapped
- The pickSoftThreshold scan should show the chosen power achieving a strong scale-free topology fit relative to other candidate powers before adjacency construction
- Module eigengenes and module membership measures must be computed so modules can be correlated to external sample traits via the module_trait_relationships workflow

## Limitations

- Requires well-expressed genes only: genes with low counts must be filtered out during Data_format pre-processing before network construction, or the correlation estimates are unreliable
- Python version constraints differ across sources (article: 3.9+; README: 3.10+) — check the installed PyWGCNA release requirements
- Soft-thresholding power selection does not guarantee identical module counts across datasets; module recovery is dataset- and parameter-dependent, as reflected by the two MODEL-AD datasets each yielding 17 modules but different module compositions

## Evidence

- [intro] PyWGCNA is a Python library designed to do weighted correlation network analysis (WGCNA). It can be used for finding clusters (modules) of highly correlated genes: "PyWGCNA is a Python library designed to do weighted correlation network analysis (WGCNA). It can be used for finding clusters (modules) of highly correlated genes"
- [other] Select the soft-thresholding power by scanning candidate powers for the best scale-free topology fit (pickSoftThreshold).: [evidence span withheld — anchor status: not_found]
- [readme] To install PyWGCNA, Python version 3.10 or greater is required.: [evidence span withheld — anchor status: external_doc]
- [methods] How to load data into PyWGCNA, find modules, and analyze them: "How to load data into PyWGCNA, find modules, and analyze them"
