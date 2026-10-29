---
name: scanpy-pbmc3k-dataset-loading
description: Use when a workflow needs the canonical PBMC 3K raw count matrix as a
  starting object — e.g. to estimate Splatter generative parameters, run the scanpy
  standard preprocessing pipeline, or benchmark cluster-significance methods like
  Cytocipher.
when_to_use_negative:
- The analysis already starts from a user-supplied count matrix or AnnData object
  — loading pbmc3k would overwrite/ignore it rather than serve as the reference dataset.
- Downstream steps require normalised (log-cpm) or pre-clustered input — this skill
  supplies only the raw starting matrix before the scanpy standard workflow.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: scanpy
  role: Provides the pbmc3k tutorial dataset loader (scanpy.datasets.pbmc3k) used
    to fetch the raw PBMC 3K count matrix
- name: AnnData
  role: Container format for the loaded PBMC 3K counts; Cytocipher expects the processed
    object as AnnData
- name: Splatter
  role: 'Downstream consumer: splatEstimate fits generative count-model parameters
    from the PBMC 3K matrix for simulation'
- name: Cytocipher
  role: Benchmarking tool whose expected input is an AnnData processed like the scanpy
    pbmc3k standard workflow
  repo: https://github.com/BradBalderson/Cytocipher
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:a981b093b0e89dc60287a9e05e4ac33cec80eed4991a0c641539ef73c41d721d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - Splatter
  - scanpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# scanpy-pbmc3k-dataset-loading

## Summary

Loads the raw PBMC 3K human peripheral blood mononuclear cell scRNA-seq count matrix via scanpy's pbmc3k tutorial dataset, serving as the reference real-world input for the Cytocipher benchmarking workflow (e.g. estimating Splatter count-model parameters for simulation).

## When to use

Use when a workflow needs the canonical PBMC 3K raw count matrix as a starting object — e.g. to estimate Splatter generative parameters, run the scanpy standard preprocessing pipeline, or benchmark cluster-significance methods like Cytocipher.

## When NOT to use

- The analysis already starts from a user-supplied count matrix or AnnData object — loading pbmc3k would overwrite/ignore it rather than serve as the reference dataset.
- Downstream steps require normalised (log-cpm) or pre-clustered input — this skill supplies only the raw starting matrix before the scanpy standard workflow.

## Inputs

- No file input required — the dataset is fetched remotely via scanpy.datasets.pbmc3k
- Internet connection to the scanpy tutorial data host

## Outputs

- AnnData object containing the raw PBMC 3K scRNA-seq count matrix (human peripheral blood mononuclear cells)

## How to apply

Call scanpy.datasets.pbmc3k to fetch the raw PBMC 3K count matrix as an AnnData object, matching the reference workflow cited in the Cytocipher methods (the scanpy pbmc3k tutorial at https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html). In the Cytocipher benchmarking, this raw count matrix is then passed to Splatter's splatEstimate (Splatter v1.20) to fit count-model parameters, and a synthetic 10000 genes x 3000 cells matrix is generated with splatSimulate. Cytocipher itself expects downstream objects processed 'similarly to the scanpy standard workflow' into log-cpm normalised data with tentative Leiden cluster labels. Keep the loaded object at raw counts before normalisation so parameter estimation and quality control operate on the untransformed data.

## Related tools

- **scanpy** (Provides the pbmc3k tutorial dataset loader (scanpy.datasets.pbmc3k) used to fetch the raw PBMC 3K count matrix)
- **AnnData** (Container format for the loaded PBMC 3K counts; Cytocipher expects the processed object as AnnData)
- **Splatter** (Downstream consumer: splatEstimate fits generative count-model parameters from the PBMC 3K matrix for simulation)
- **Cytocipher** (Benchmarking tool whose expected input is an AnnData processed like the scanpy pbmc3k standard workflow) — https://github.com/BradBalderson/Cytocipher

## Examples

```
import scanpy as sc; adata = sc.datasets.pbmc3k()
```

## Evaluation signals

- Returned object is an AnnData instance with a sparse count matrix of non-negative integers (raw counts, not log-normalised).
- Dataset corresponds to the PBMC 3K tutorial data referenced by https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html — cell/gene counts should match that tutorial's known dimensions.
- Raw counts are usable as input to Splatter v1.20 splatEstimate without additional transformation.
- Downstream Cytocipher call accepts the processed derivative object without input-format errors (it requires log-cpm normalised data with Leiden labels after further processing).

## Limitations

- Requires network access to download the tutorial dataset; no local fallback is described in the context.
- The pbmc3k object is raw and unprocessed — Cytocipher explicitly requires log-cpm normalisation and tentative Leiden cluster labels before its functions can be applied, so additional preprocessing is mandatory.
- The README provides no changelog or discussion of dataset versioning; reproducibility of the exact downloaded matrix depends on scanpy's hosted data.

## Evidence

- [methods] PBMC 3K as scanpy reference workflow: "processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)"
- [readme] README expected input: "An AnnData object, *data*, that has been processed similarly to the 
[scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)
to produce log-cpm normalised data"
- [readme] Splatter simulation scale: "Cytocipher merges the non-significantly different clusters."
