---
name: tf-target-database-annotation-filtering
description: Use when preprocessing scRNA-seq data for TFvelo and each target gene
  needs a curated candidate TF set drawn from ENCODE/ChEA regulatory pairs before
  fitting TF-based gene dynamics.
when_to_use_negative:
- The dataset already contains spliced/unspliced counts and you intend to run standard
  splicing-based scVelo dynamical modeling instead of TF-based modeling
- No ENCODE/ChEA TF-target database has been downloaded/decompressed — the candidate-TF
  restriction step cannot be performed and the pipeline should not be run until 'unzip
  ENCODE.zip' completes
- The goal is gene-level normalization only (e.g. standalone scanpy QC) rather than
  annotating target genes with TF candidates for regulatory modeling
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: TFvelo
  role: Hosts the preprocessing pipeline that restricts candidate TFs per target gene
    via the --TF_databases parameter (ENCODE and ChEA by default) and saves the annotated
    AnnData
  repo: https://github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Provides pp.filter_and_normalize (top-2000 HVGs), pp.neighbors (n_neighbors=30),
    and pp.moments on total mRNA counts that precede the TF-pair filtering step
  repo: https://github.com/theislab/scvelo
- name: anndata
  role: Loads the raw count .h5ad and stores the preprocessed AnnData with neighbor
    graph and per-gene TF annotations
- name: scanpy
  role: Part of the pinned environment (scanpy==1.8.2) supporting the scvelo preprocessing
    used before TF-pair restriction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:43e314b4bc269f40acacf1ed9925d74fbf2155dd67c8c0eb797c13d906f6ff31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - ScVelo
  - TFvelo
  - anndata
  - scanpy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tf-target-database-annotation-filtering

## Summary

Restrict candidate transcription factors for each target gene to curated regulatory pairs from the ENCODE and ChEA TF-target databases prior to dynamic velocity modeling in TFvelo, ensuring gene dynamics are modeled from biologically grounded TF regulation rather than splicing information.

## When to use

Use when preprocessing scRNA-seq data for TFvelo and each target gene needs a curated candidate TF set drawn from ENCODE/ChEA regulatory pairs before fitting TF-based gene dynamics.

## When NOT to use

- The dataset already contains spliced/unspliced counts and you intend to run standard splicing-based scVelo dynamical modeling instead of TF-based modeling
- No ENCODE/ChEA TF-target database has been downloaded/decompressed — the candidate-TF restriction step cannot be performed and the pipeline should not be run until 'unzip ENCODE.zip' completes
- The goal is gene-level normalization only (e.g. standalone scanpy QC) rather than annotating target genes with TF candidates for regulatory modeling

## Inputs

- Raw scRNA-seq count AnnData in .h5ad format (e.g. via ad.read_h5ad), or an auto-downloaded dataset such as pancreas
- Decompressed ENCODE TF-target database file (ENCODE.zip shipped with the TFvelo package)
- ChEA TF-target database (default together with ENCODE via --TF_databases)

## Outputs

- Preprocessed AnnData (.h5ad) containing the filtered/normalized count matrix, scvelo neighbor graph (n_neighbors=30), moments on total mRNA counts, and per-gene candidate TF annotations from ENCODE/ChEA
- Files stored in the 'TFvelo_'+dataset_name+'_demo' folder (e.g. TFvelo.h5ad) for downstream TF-model fitting

## How to apply

After loading the raw count AnnData with anndata (read_h5ad) — or auto-downloading e.g. the pancreas dataset via TFvelo_run_demo.py — decompress the ENCODE TF-target database file shipped with the package (e.g. 'unzip ENCODE.zip'). Apply scvelo pp.filter_and_normalize (retaining top-2000 highly variable genes), build the neighbor graph with scvelo pp.neighbors (n_neighbors=30), and compute scvelo pp.moments on the total (unspliced+spliced) mRNA counts over that graph. Then, for each target gene, restrict candidate TFs to regulatory pairs present in the ENCODE and ChEA TF-target databases, which is the default --TF_databases setting; --max_n_TF caps how many TFs are used per gene and --WX_thres bounds the absolute TF weights. Save the preprocessed AnnData (with per-gene TF annotations and the neighbor graph) to the 'TFvelo_'+dataset_name+'_demo' folder so downstream TF-model fitting can consume it.

## Related tools

- **TFvelo** (Hosts the preprocessing pipeline that restricts candidate TFs per target gene via the --TF_databases parameter (ENCODE and ChEA by default) and saves the annotated AnnData) — https://github.com/xiaoyeye/TFvelo
- **ScVelo** (Provides pp.filter_and_normalize (top-2000 HVGs), pp.neighbors (n_neighbors=30), and pp.moments on total mRNA counts that precede the TF-pair filtering step) — https://github.com/theislab/scvelo
- **anndata** (Loads the raw count .h5ad and stores the preprocessed AnnData with neighbor graph and per-gene TF annotations)
- **scanpy** (Part of the pinned environment (scanpy==1.8.2) supporting the scvelo preprocessing used before TF-pair restriction)

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The decompressed ENCODE database file exists after running 'unzip ENCODE.zip' before the pipeline runs
- The preprocessed .h5ad exists in 'TFvelo_'+dataset_name+'_demo' and loads successfully with anndata read_h5ad
- The saved AnnData contains the scvelo neighbor graph (n_neighbors=30) and moments computed on total mRNA counts
- Each target gene carries candidate TF annotations restricted to ENCODE/ChEA regulatory pairs, with counts not exceeding --max_n_TF
- No numeric acceptance threshold is reported, so the qualitative criterion is the structural presence of neighbor graph and per-gene TF annotations in the output AnnData

## Limitations

- TFvelo is slower than scVelo baselines, which could be a weakness on large-scale datasets; parallel CPUs (--n_jobs) reduce runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling for more genes remains a challenge
- The current linear model for estimating TF weights may be insufficient; no implementation of alternative weighting models is provided

## Evidence

- [methods] Restrict candidate TFs to ENCODE/ChEA pairs (workflow): "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] Decompress ENCODE before use (README): "Please decompress the ENCODE TF-target database file firstly after downloading this package."
- [discussion] Regulation-based modeling rationale: "TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself"
- [methods] Preprocessing on total mRNA counts (workflow): "python TFvelo_run_demo.py --dataset_name pancreas"
