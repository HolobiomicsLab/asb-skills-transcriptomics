---
name: multiome-dataset-preprocessing
description: Use when applying TFvelo to a dataset not shipped with the package —
  e.g. a 10x multi-omics AnnData (.h5ad) file where you want RNA-only velocity without
  ATAC-seq — and the default pancreas demo branch cannot load it.
when_to_use_negative:
- Running the built-in pancreas demo, which already has a preprocess() branch and
  requires no customization
- Input is raw count matrices not yet converted to AnnData (.h5ad) — the load branch
  expects ad.read_h5ad on an existing file
- Large-scale datasets where TFvelo's runtime (slower than scVelo) is prohibitive,
  unless parallelizing over more CPUs via --n_jobs
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Main package whose TFvelo_run_demo.py preprocess() function is extended with
    a dataset-load branch, then run to fit RNA-only gene dynamics and write rc.h5ad
  repo: github.com/xiaoyeye/TFvelo
- name: scVelo
  role: Framework on which TFvelo is built; its AnnData/velocity conventions underpin
    the h5ad inputs and outputs
  repo: github.com/theislab/scvelo
- name: MultiVelo
  role: Baseline run on the same 10x multi-omics dataset from the package's baselines
    folder to compare pseudotime, stream direction, and gene phase portraits
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:df672aaab0d5269d0c2b75813b16de611249920ec2baeddc324819a7189b78b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiVelo
  - TFvelo
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multiome-dataset-preprocessing

## Summary

Adapts the TFvelo package to ingest a new (e.g. 10x multi-omics embryonic mouse brain) dataset by adding a dataset-name load branch to the preprocess() function in TFvelo_run_demo.py, so TFvelo can be run on RNA-only counts even when ATAC-seq data exist but are not used.

## When to use

Use when applying TFvelo to a dataset not shipped with the package — e.g. a 10x multi-omics AnnData (.h5ad) file where you want RNA-only velocity without ATAC-seq — and the default pancreas demo branch cannot load it.

## When NOT to use

- Running the built-in pancreas demo, which already has a preprocess() branch and requires no customization
- Input is raw count matrices not yet converted to AnnData (.h5ad) — the load branch expects ad.read_h5ad on an existing file
- Large-scale datasets where TFvelo's runtime (slower than scVelo) is prohibitive, unless parallelizing over more CPUs via --n_jobs

## Inputs

- AnnData (.h5ad) file of the new dataset, e.g. the 10x multi-omics embryonic mouse brain data, restricted to RNA-only counts
- personalized dataset name string passed via --dataset_name
- TF-target databases: decompressed ENCODE.zip plus ChEA (packaged defaults)

## Outputs

- rc.h5ad — TF model fit stored in folder 'TFvelo_' + dataset_name
- TFvelo.h5ad — annotated result with inferred pseudotime, stored in 'TFvelo_' + dataset_name + '_demo'
- figures of pseudotime and streamplot on UMAP and phase portrait fits of target genes, saved in folder 'figures'

## How to apply

Edit TFvelo_run_demo.py and, inside the preprocess() function, add an if-branch keyed on the personalized --dataset_name that reads the local AnnData file (adata = ad.read_h5ad(your_h5ad_file_path)). Run TFvelo with default parameters: TF_databases ENCODE and ChEA, WX_method lsq_linear, init_weight_method correlation, which yields the fitted result rc.h5ad in the 'TFvelo_' + dataset_name folder. For a multi-omics dataset, supply only the scRNA-seq (spliced/unspliced-independent RNA abundance) counts to TFvelo; ATAC-seq layers are not required because TFvelo models gene dynamics from transcription factor expression rather than MultiVelo's chromatin-informed approach. Then run TFvelo_analysis_demo.py with the same dataset name to compute and store the annotated result as TFvelo.h5ad and render pseudotime/streamplot on UMAP plus phase portraits of fitted target genes (e.g. AHI1, NTRK2, GRIN2B) into the 'figures' folder. Rationale: the README's customization hook is the supported route for out-of-package datasets, and parallel MultiVelo baselines live in the package's 'baselines' folder for head-to-head comparison.

## Related tools

- **TFvelo** (Main package whose TFvelo_run_demo.py preprocess() function is extended with a dataset-load branch, then run to fit RNA-only gene dynamics and write rc.h5ad) — github.com/xiaoyeye/TFvelo
- **scVelo** (Framework on which TFvelo is built; its AnnData/velocity conventions underpin the h5ad inputs and outputs) — github.com/theislab/scvelo
- **MultiVelo** (Baseline run on the same 10x multi-omics dataset from the package's baselines folder to compare pseudotime, stream direction, and gene phase portraits)

## Examples

```
python TFvelo_run_demo.py --dataset_name your_dataset_name; python TFvelo_analysis_demo.py --dataset_name your_dataset_name
```

## Evaluation signals

- rc.h5ad and TFvelo.h5ad are produced in the expected 'TFvelo_' + dataset_name folders after the two demo scripts run without error
- Inferred pseudotime ordering and streamplot direction on UMAP match the differentiation directions reported in Fig. 5a-c of the manuscript
- Phase portraits of target genes (AHI1, NTRK2, GRIN2B) show the expected clockwise curve between TF representation and target
- On AHI1, TFvelo captures the correct dynamic where MultiVelo mistakenly constructs a cyclic dynamic — a discriminating check of the RNA-only fit
- Default hyperparameters are in effect: TF_databases = ENCODE and ChEA, WX_method = lsq_linear, init_weight_method = correlation

## Limitations

- TFvelo is slower than baselines in the same framework (e.g. scVelo), which is a weakness on large-scale datasets; running with more CPUs in parallel via --n_jobs can reduce time
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling for more genes remains a challenge
- The 10x multi-omics dataset is referenced without accession identifiers in the paper, so obtaining the exact input h5ad requires external sourcing

## Evidence

- [methods] simply add the following codes into the preprocess() function in TFvelo_run_demo.py: "simply add the following codes into the preprocess() function in TFvelo_run_demo.py"
- [readme] customize preprocess branch: "you can define a personalized name for the dataset, and simply add the following codes into the preprocess() function in TFvelo_run_demo.py:"
- [readme] h5ad load line: "adata = ad.read_h5ad(your_h5ad_file_path)"
- [discussion] multi-omics result claim: "Results on various scRNA-seq datasets and a 10× multi-omics dataset further demonstrate that TFvelo can fit the gene dynamics well"
- [methods] hyperparameter defaults: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
