---
name: annadata-h5ad-prediction-export
description: Use when a scDEAL single-cell prediction run (e.g. scmodel.py on GSE110894)
  has finished and the per-cell drug response predictions must be persisted and shared
  as an AnnData object for evaluation or visualization.
when_to_use_negative:
- The goal is bulk-level model training only (bulkmodel.py produces model checkpoints,
  not AnnData prediction exports)
- The input is raw scRNA-seq count data that has not yet been passed through the scDEAL
  DaNN transfer prediction — there are no sens_label predictions to export yet
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Produces the single-cell drug response predictions (sens_label) that are written
    into the AnnData object
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Package used to read and manipulate the exported AnnData object for downstream
    evaluation
- name: AnnData
  role: Data container format (.h5ad) in which the prediction results are exported
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:8971466dedada61d4532e0b74c384e365322994651aa0398b0cd2991ed3c43ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - Scanpy
  - conda-pack
  - scDEAL
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# annadata-h5ad-prediction-export

## Summary

Export scDEAL single-cell drug response predictions as a Scanpy/AnnData .h5ad object, storing the predicted sensitivity label in adata.obs['sens_label'] (0 = resistance, 1 = sensitivity) so results can be reloaded and evaluated downstream.

## When to use

Use when a scDEAL single-cell prediction run (e.g. scmodel.py on GSE110894) has finished and the per-cell drug response predictions must be persisted and shared as an AnnData object for evaluation or visualization.

## When NOT to use

- The goal is bulk-level model training only (bulkmodel.py produces model checkpoints, not AnnData prediction exports)
- The input is raw scRNA-seq count data that has not yet been passed through the scDEAL DaNN transfer prediction — there are no sens_label predictions to export yet

## Inputs

- Trained scDEAL single-cell model output (predictions for the target scRNA-seq dataset, e.g. GSE110894)
- scRNA-seq AnnData/h5ad input (e.g. GSE110894 from scDEAL.zip data directory)
- Ground-truth drug-screening label table (e.g. ALL_label_binary_wf.csv) for validation context

## Outputs

- AnnData .h5ad file named [dataset][timestamp].h5ad (e.g. GSE110894[timestamp].h5ad) with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity)
- Stored under save/adata in the scDEAL directory tree

## How to apply

After running scmodel.py (e.g. on GSE110894 with drug I.BET.762, DAE encoder, bottleneck 512, bulk_h_dims 256,128, predictor_h_dims 128,64, dropout 0.1, lr 0.5, upsampling sampling, mod new), have the pipeline write the resulting AnnData object to a timestamped .h5ad file such as GSE110894[timestamp].h5ad. Store the predicted per-cell response in adata.obs['sens_label'] using the binary convention 0 = resistance and 1 = sensitivity so downstream label-match evaluation can compare directly against ground-truth groups. Place outputs under the save/adata directory of the scDEAL package, which is designated for 'results from AnnData outputs'. Then reload the .h5ad with Scanpy to compute label-match percentages within resistant and sensitive ground-truth groups, correlate predicted DEG scores against ground-truth differential expression scores, and run the 1000-permutation null test.

## Related tools

- **scDEAL** (Produces the single-cell drug response predictions (sens_label) that are written into the AnnData object) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Package used to read and manipulate the exported AnnData object for downstream evaluation)
- **AnnData** (Data container format (.h5ad) in which the prediction results are exported)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "F" -mod "new" --checkpoint "save/sc_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
```

## Evaluation signals

- The exported .h5ad loads with Scanpy.read_h5ad without error and contains adata.obs['sens_label']
- sens_label values are binary and follow the convention 0 = resistance, 1 = sensitivity
- Reloading the export and comparing against ground truth reproduces ~97.1% label match in predicted resistant cells and ~95.8% in predicted sensitive cells for the GSE110894/I-BET-762 case
- DEG-score correlations computed from the exported object reproduce R2 = 0.90 (sensitive) / 0.77 (resistant) with p < 0.001 at n = 1000 permutations
- File is placed in save/adata following the scDEAL directory conventions

## Limitations

- Prediction accuracy depends on the collection of bulk gene expression of cell lines; scDEAL training data will be updated by integrating additional bulk-level databases
- For the mouse Data 6 case, only homologous genes were kept and the model was still trained with human cancer cell lines; trans-species reliability could not be systematically evaluated
- No software version numbers are stated for tools or the environment, so reproducibility of the export relies on the provided conda-pack environment (scdeal.tar.gz)

## Evidence

- [methods] Export of predictions as AnnData: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad)"
- [readme] save/adata directory purpose: "- save/adata: results from AnnData outputs."
- [readme] Demo run producing GSE110894 predictions: "python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --printgene "F" -mod "new" --lr 0.5"
- [results] Ground-truth evaluation context: [evidence span withheld — anchor status: not_found]
