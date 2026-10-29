---
name: drug-response-gene-signature-analysis
description: Use when comparing which genes a bulk-trained drug response model versus
  its single-cell transferred counterpart identify as IG-contributing, to test consistency
  of learned signatures across bulk and scRNA-seq levels.
when_to_use_negative:
- The goal is drug response label prediction accuracy rather than gene-level interpretation
  — run scmodel.py/bulkmodel.py with --printgene "F" instead
- The single-cell data is not from the benchmark drug-treated scRNA-seq datasets and
  no matching bulk checkpoint exists, so IG values are not comparable across levels
- Differential expression-based signature identification is desired rather than model-attribution
  (integrated gradient) signatures
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Framework whose DaNN-based bulk and single-cell models produce integrated-gradient
    critical gene lists
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend on which the scDEAL bulkmodel.py/scmodel.py training and integrated-gradient
    attribution are executed
- name: Scanpy
  role: Handles the AnnData (.h5ad) single-cell objects that carry the prediction
    and IG outputs
- name: AnnData
  role: Expected output object format (.h5ad) for scDEAL predictions from which gene
    attributions are read
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:774489f982ccb9a6269589ab4dfe086318b7e1f866f203d1b3a6537d891ef485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# drug-response-gene-signature-analysis

## Summary

Reproduce the scDEAL bulk-vs-single-cell integrated-gradient (IG) gene overlap analysis: extract critical genes driving drug response label prediction at bulk and single-cell levels and quantify the fraction of shared genes for sensitivity (~46%) and resistance (~53%) gene sets (Supplementary Table S6).

## When to use

Use when comparing which genes a bulk-trained drug response model versus its single-cell transferred counterpart identify as IG-contributing, to test consistency of learned signatures across bulk and scRNA-seq levels.

## When NOT to use

- The goal is drug response label prediction accuracy rather than gene-level interpretation — run scmodel.py/bulkmodel.py with --printgene "F" instead
- The single-cell data is not from the benchmark drug-treated scRNA-seq datasets and no matching bulk checkpoint exists, so IG values are not comparable across levels
- Differential expression-based signature identification is desired rather than model-attribution (integrated gradient) signatures

## Inputs

- data/ directory contents extracted from scDEAL.zip (bulk expression CSVs and GSE110894/GSE112274/GSE117872/GSE140440/GSE149383 scRNA-seq data)
- Pre-trained checkpoint files in save/models, e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl
- casestudy/analysis_criticalgenes.ipynb notebook template

## Outputs

- Per-dataset ranked lists of IG-contributing genes for sensitivity and resistance at bulk and single-cell levels
- critical_genes_overlap.tsv with one overlap-fraction row per dataset per polarity
- Overlap fractions approximating 46% (sensitivity) and 53% (resistance), as reported in Supplementary Table S6

## How to apply

Load the scDEAL.zip datasets into data/ and locate the pre-trained checkpoints in save/models (naming pattern bulk/sc dataset + drug + bottleneck + encoder/predictor dims + dropout + lr + sampling). Run bulkmodel.py with --printgene "T" (e.g. drug I.BET.762, --dimreduce DAE, --encoder_h_dims 256,128, --predictor_h_dims 128,64, --bottleneck 512, --dropout 0.1, --lr 0.5, --sampling upsampling) to obtain the bulk-level critical gene list via integrated gradients, then run scmodel.py with --printgene "T" and the matching --checkpoint for each benchmark dataset (GSE110894, GSE112274, GSE117872, GSE140440, GSE149383). Use the casestudy/analysis_criticalgenes.ipynb template to extract the IG matrices and rank top contributing genes separately for sensitivity and resistance polarity at both levels. For each dataset and polarity, compute the fraction of shared genes between the bulk-level and single-cell-level critical gene lists; keep encoder/predictor dimensions identical across the bulk and single-cell runs (--encoder_h_dims/--bulk_h_dims 256,128, --predictor_h_dims 128,64).

## Related tools

- **scDEAL** (Framework whose DaNN-based bulk and single-cell models produce integrated-gradient critical gene lists) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend on which the scDEAL bulkmodel.py/scmodel.py training and integrated-gradient attribution are executed)
- **Scanpy** (Handles the AnnData (.h5ad) single-cell objects that carry the prediction and IG outputs)
- **AnnData** (Expected output object format (.h5ad) for scDEAL predictions from which gene attributions are read)

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "T"
```

## Evaluation signals

- Computed bulk-vs-single-cell overlap fractions for drug-sensitivity gene sets approximate the reported 46% average
- Computed overlap fractions for drug-resistance gene sets approximate the reported 53% average, matching Supplementary Table S6
- critical_genes_overlap.tsv contains one overlap row per dataset per polarity (sensitivity/resistance)
- Encoder and predictor dimensions match between bulkmodel.py and scmodel.py runs (--encoder_h_dims/--bulk_h_dims 256,128, --predictor_h_dims 128,64, matching bottleneck)
- Checkpoint filenames conform to the documented naming pattern ending in _DaNN.pkl for the single-cell model

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; scDEAL training data will be updated by integrating additional bulk-level databases
- A recent study pointed out a potential sample swap between Data 1 and 2 (HN120 and HN137, both from GSE117872), affecting dataset-level comparisons
- Critical gene analysis is only meaningful with matching checkpoints; model versions and hyperparameters (e.g. dropout 0.1 vs 0.3, lr 0.5 vs 0.01, sampling upsampling/SMOTE/no) differ per dataset and must be kept consistent between bulk and single-cell runs

## Evidence

- [abstract] Integrated gradient feature interpretation to infer signature genes of drug resistance mechanisms: "Another feature of scDEAL is the integrated gradient feature interpretation to infer the signature genes of drug resistance mechanisms."
- [readme] checkpoint naming and printgene usage: "The naming rules of the checkpoint are as follows: bulk dataset+"_data_"+scRNA-Seq dataset+"_drug_"+drug+"_bottle_"+bottle+"_edim_"+encoder dimensions"
- [readme] identical dimensions requirement: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [methods] output format AnnData: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
- [methods] benchmark dataset list: "Data 1&2  |     Sharma,   et al.     |     Cisplatin    |     GSE117872"
