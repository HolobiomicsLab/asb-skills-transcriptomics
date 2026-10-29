---
name: gene-expression-correlation-analysis
description: Use when a bulk expression matrix (e.g. TCGA AML) and a subclone-associated
  gene list from scRNA-seq (e.g. GATA2R361C subclone) must be linked to test whether
  a marker gene like VIM defines a co-expressed regulon.
when_to_use_negative:
- The correlation anchor gene has near-zero variance across samples, making Pearson
  coefficients unstable
- The comparison gene list comes from bulk rather than single-cell subclone assignment
  — no specific subclone association to test
- No genome-wide background gene universe is available for the hypergeometric test
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Runs the correlation computation, Benjamini-Hochberg correction, and scipy.stats
    hypergeometric overlap test
  repo: https://github.com/genome/scrna_mutations
- name: Docker (sridnona/python3:180925.v1)
  role: Container image providing the python3 environment used for the analysis scripts
- name: pysam
  role: Python BAM-reading library used in the broader pipeline that generates the
    subclone gene lists fed into this overlap test
  repo: https://github.com/genome/scrna_mutations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:afb2eea0181b0454c512140693b298b702c835a2d2052c0ac14ed259cc7ec1b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genome/scrna_mutations
  tools:
  - Docker
  - Docker (sridnona/python3:180925.v1)
  - pysam
  - python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gene-expression-correlation-analysis

## Summary

Computes gene–gene Pearson correlations across bulk expression samples (here, TCGA AML), applies Benjamini-Hochberg correction to retain a high-confidence correlated gene set, and tests its overlap with a single-cell-derived subclone gene list by hypergeometric testing to infer a regulon.

## When to use

Use when a bulk expression matrix (e.g. TCGA AML) and a subclone-associated gene list from scRNA-seq (e.g. GATA2R361C subclone) must be linked to test whether a marker gene like VIM defines a co-expressed regulon.

## When NOT to use

- The correlation anchor gene has near-zero variance across samples, making Pearson coefficients unstable
- The comparison gene list comes from bulk rather than single-cell subclone assignment — no specific subclone association to test
- No genome-wide background gene universe is available for the hypergeometric test

## Inputs

- TCGA AML bulk gene expression matrix
- GATA2R361C subclone-associated gene list (output of upstream single-cell subclone expression-signature task)
- VIM gene identifier used as the regulon anchor

## Outputs

- VIM-correlated gene set (2191 genes at q < 0.001 after Benjamini-Hochberg correction)
- Hypergeometric overlap p-value (expected ~3.5e-95) and intersection of ~198 genes
- ToppFun GO enrichment terms for immune response, cytoskeletal organization, and focal adhesion
- vim_gata2_overlap_results.tsv (intersection gene list, hypergeometric p-value, GO terms)

## How to apply

Load the TCGA AML expression matrix and the subclone-associated gene list produced upstream (from the GATA2R361C subclone task). Compute the Pearson correlation coefficient between every gene and VIM across all TCGA AML samples and assign a p-value to each correlation. Apply Benjamini-Hochberg multiple-testing correction and retain genes with q < 0.001 (2191 genes) as the VIM-correlated set. Run a scipy.stats hypergeometric test via a python3 script to assess whether the VIM-correlated set overlaps the GATA2R361C subclone gene list more than expected by chance (~198 intersecting genes, p = 3.5e-95). Submit the intersection to ToppFun for GO enrichment covering immune response, cytoskeletal organization, and focal adhesion, and write the gene list, p-value, and GO terms to vim_gata2_overlap_results.tsv.

## Related tools

- **python3** (Runs the correlation computation, Benjamini-Hochberg correction, and scipy.stats hypergeometric overlap test) — https://github.com/genome/scrna_mutations
- **Docker (sridnona/python3:180925.v1)** (Container image providing the python3 environment used for the analysis scripts)
- **pysam** (Python BAM-reading library used in the broader pipeline that generates the subclone gene lists fed into this overlap test) — https://github.com/genome/scrna_mutations

## Examples

```
python3 -c "from scipy.stats import hypergeom; M=len(all_genes); n=len(vim_set); K=len(gata2_set); k=len(set(vim_set)&set(gata2_set)); print(hypergeom.sf(k-1, M, n, K))"
```

## Evaluation signals

- The Benjamini-Hochberg filtered VIM-correlated set contains ~2191 genes at q < 0.001
- The hypergeometric test reproduces p = 3.5e-95 with an intersection of ~198 genes between the VIM-correlated set and the GATA2R361C subclone gene list
- ToppFun recovers GO terms for immune response, cytoskeletal organization, and focal adhesion
- vim_gata2_overlap_results.tsv is produced containing the gene list, p-value, and GO terms
- Overlap p-value is far more significant than random permutation expectations (gene-set overlap is not by chance)

## Limitations

- Dropout (transcript and allelic dropout) in the upstream scRNA-seq mutation detection reduces sensitivity by about a factor of two, so the subclone gene lists feeding this test may miss weakly expressing cells
- Coverage drops non-linearly across transcript length on end-biased platforms such as the Chromium platform, so some subclone marker genes are more easily detected than others
- It is impossible to determine whether a scRNA-seq cell is truly wild-type for a given mutation, so subclone assignment is inherently probabilistic
- The utility of the upstream approach depends on the specific mutational composition of the sample and performs better for tumor types with higher mutation burdens than AML

## Evidence

- [abstract] identify expression signatures associated with subclonal mutations: "identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones"
- [methods] Docker image sridnona/python3:180925.v1: "Docker image
----------
* sridnona/python3:180925.v1"
- [discussion] Dropout limits sensitivity: "Dropout (including transcript dropout and allelic dropout) occurs with most scRNA-seq platforms."
- [readme] cb_sniffer README (pipeline context): "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
