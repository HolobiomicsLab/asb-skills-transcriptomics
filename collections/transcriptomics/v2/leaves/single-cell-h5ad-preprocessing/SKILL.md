---
name: single-cell-h5ad-preprocessing
description: Use when a scRNA-seq dataset is available as an .h5ad file and you want
  TFvelo to fit TF-based gene dynamics, infer pseudotime/trajectory, and rank transcription
  factors on data lacking splicing information.
when_to_use_negative:
- The input is not an .h5ad AnnData file — the loader branch reads exclusively via
  ad.read_h5ad(your_h5ad_file_path), so raw count matrices (e.g., 10x mtx or FASTQ-derived
  data) must be converted first.
- Very large-scale datasets where runtime matters — the paper notes TFvelo's time
  efficiency is lower than scVelo, which 'could be a weakness when being applied to
  the large-scale datasets'.
- The goal is only visualization without refitting — TFvelo_analysis_demo.py consumes
  rc.h5ad produced by TFvelo_run_demo.py, so skipping the fitting step breaks the
  downstream demo.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TFvelo
  role: Main pipeline whose TFvelo_run_demo.py preprocess() function is extended with
    the h5ad loader and which fits the TF-based gene dynamics model (generalized EM
    with lsq_linear weight optimization)
  repo: github.com/xiaoyeye/TFvelo
- name: anndata
  role: Reads the custom dataset into an AnnData object inside preprocess() via ad.read_h5ad
    (pinned to 0.8.0)
- name: scanpy
  role: Preprocessing/visualization backbone for the single-cell data (pinned to 1.8.2
    in the TFvelo environment)
- name: scvelo
  role: Framework on which TFvelo is built (pinned to 0.2.4); TFvelo adopts scVelo's
    code organization for velocity analysis
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:b7c3cda7db587244569cbf0f2f9d314aa251b4092e8d5f661db1126c08459749
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TFvelo
  - anndata
  - scanpy
  - scvelo
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-h5ad-preprocessing

## Summary

Adapts the TFvelo pipeline (an scVelo-framework, gene-regulation-inspired RNA velocity method) to a custom scRNA-seq dataset stored as an AnnData .h5ad file by adding a dataset loader into the preprocess() function of TFvelo_run_demo.py, then running model fitting and visualization with default hyperparameters.

## When to use

Use when a scRNA-seq dataset is available as an .h5ad file and you want TFvelo to fit TF-based gene dynamics, infer pseudotime/trajectory, and rank transcription factors on data lacking splicing information.

## When NOT to use

- The input is not an .h5ad AnnData file — the loader branch reads exclusively via ad.read_h5ad(your_h5ad_file_path), so raw count matrices (e.g., 10x mtx or FASTQ-derived data) must be converted first.
- Very large-scale datasets where runtime matters — the paper notes TFvelo's time efficiency is lower than scVelo, which 'could be a weakness when being applied to the large-scale datasets'.
- The goal is only visualization without refitting — TFvelo_analysis_demo.py consumes rc.h5ad produced by TFvelo_run_demo.py, so skipping the fitting step breaks the downstream demo.

## Inputs

- single-cell scRNA-seq expression data stored as an AnnData .h5ad file
- a user-defined dataset_name string passed via --dataset_name
- ENCODE and ChEA TF-target databases (candidate TF selection, default)

## Outputs

- rc.h5ad (recovered/fitted counts) in the 'TFvelo_<dataset_name>' folder
- TFvelo.h5ad with inferred pseudotime in the 'TFvelo_<dataset_name>_demo' folder
- figures: pseudotime and streamplot on UMAP, phase-portrait fits of best-fitted genes, saved in the 'figures' folder
- TF weight rankings over candidate TFs for identifying key regulators

## How to apply

Set up the pinned environment (conda create -n TFvelo_env python=3.8.12; pandas 1.2.3, anndata 0.8.0, scanpy 1.8.2, numpy 1.21.6, scipy 1.10.1, numba 0.57.0, matplotlib 3.3.4, scvelo 0.2.4) and decompress the ENCODE TF-target database (unzip ENCODE.zip). Define a personalized dataset name and add a branch into the preprocess() function of TFvelo_run_demo.py: 'if args.dataset_name == your_dataset_name: adata = ad.read_h5ad(your_h5ad_file_path)'. Run 'python TFvelo_run_demo.py --dataset_name your_dataset_name' with default hyperparameters (init_weight_method=correlation, WX_method=lsq_linear, TF_databases=ENCODE+ChEA, max_n_TF, n_time_points for the E-step time assignment of the generalized EM algorithm); the fitted result is stored as rc.h5ad in the 'TFvelo_<name>' folder. Then run 'python TFvelo_analysis_demo.py --dataset_name your_dataset_name' to produce TFvelo.h5ad with pseudotime in 'TFvelo_<name>_demo' and save pseudotime/streamplot-on-UMAP and phase-portrait figures of best-fitted genes in the 'figures' folder.

## Related tools

- **TFvelo** (Main pipeline whose TFvelo_run_demo.py preprocess() function is extended with the h5ad loader and which fits the TF-based gene dynamics model (generalized EM with lsq_linear weight optimization)) — github.com/xiaoyeye/TFvelo
- **anndata** (Reads the custom dataset into an AnnData object inside preprocess() via ad.read_h5ad (pinned to 0.8.0))
- **scanpy** (Preprocessing/visualization backbone for the single-cell data (pinned to 1.8.2 in the TFvelo environment))
- **scvelo** (Framework on which TFvelo is built (pinned to 0.2.4); TFvelo adopts scVelo's code organization for velocity analysis) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name your_dataset_name && python TFvelo_analysis_demo.py --dataset_name your_dataset_name
```

## Evaluation signals

- TFvelo_run_demo.py runs to convergence (within --max_iter of the generalized EM) and rc.h5ad appears in the 'TFvelo_<dataset_name>' output folder.
- All generated h5ad files land in the folder named 'TFvelo_'+your_data_name+'_demo', and figures are saved in the folder 'figures', matching the README's stated output convention.
- TFvelo_analysis_demo.py produces TFvelo.h5ad containing pseudotime, and UMAP streamplots plus phase portraits of best-fitted genes render without error.
- Phase portraits of best-fitted genes show the expected clockwise curve between the TF representation and the target gene, as described in the paper.
- Top-weight TF ranking on a biologically grounded dataset recovers expected regulators (e.g., on gastrulation erythroid data: GATA1, GATA2, LMO2).

## Limitations

- TFvelo is slower than scVelo in the same framework; running with more CPUs in parallel (--n_jobs) can reduce running time, but large-scale datasets remain a weakness.
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14 in the paper); improving modeling for more genes remains a challenge.
- The linear model used to estimate TF weights may be insufficient; the paper notes more complex models could replace it, but no alternative implementation is provided.

## Evidence

- [readme] you can define a personalized name for the dataset, and simply add the following codes into the preprocess() function in TFvelo_run_demo.py: "you can define a personalized name for the dataset, and simply add the following codes into the preprocess() function in TFvelo_run_demo.py"
- [readme] As a result, all generated h5ad files will be puted in the folder named: "TFvelo_"+your_data_name+"_demo". And figures will be saved in the folder "figures".: "As a result, all generated h5ad files will be puted in the folder named: "TFvelo_"+your_data_name+"_demo". And figures will be saved in the folder "figures"."
- [methods] This will automatically download, preprocess and run TF model on pancrease dataset.: "This will automatically download, preprocess and run TF model on pancrease dataset."
- [methods] conda create -n TFvelo_env python=3.8.12: "conda create -n TFvelo_env python=3.8.12"
