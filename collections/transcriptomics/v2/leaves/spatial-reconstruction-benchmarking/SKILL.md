---
name: spatial-reconstruction-benchmarking
description: Use when you have SC data plus an ST reference with known coordinates
  (e.g., 10X Visium h5ad spots) and must quantify how well cell spatial relationships
  are reconstructed versus competing methods.
when_to_use_negative:
- There is a large gap between query SC and the ST reference — forcing SC assignment
  to resemble the ST introduces bias and degrades global reconstruction for all methods
- The tissue has a high level of repeating structural elements, which challenges spatial
  reconstruction of SC cells
- You need genome-wide spatial deconvolution of spots rather than reconstructing SC-to-SC
  spatial relationships
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellContrast
  role: Trains the encoder-projector contrastive model (infoNCE loss) on ST reference
    data, runs the encoder on query cells, and evaluates neighbor hit, JSD, and Spearman
    metrics
  repo: https://github.com/HKU-BAL/CellContrast
- name: SageNet
  role: Graph-neural-network baseline for benchmarking local spatial reconstruction
    (mean JSD 0.369 on the human breast S06->S08 task)
- name: NovoSpaRc
  role: Mapping baseline assigning SC cells to predefined locations; benchmarked comparison
    point (mean JSD 0.393)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:9d7763b6023ff9542cfaa18a06a62c12d975ecb13da9667bfc79b7b79801b4ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellContrast
  - NovoSpaRc
  - SageNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# spatial-reconstruction-benchmarking

## Summary

Benchmark spatial reconstruction of single-cell RNA-seq data using CellContrast's contrastive encoder-projector framework trained on a spatial transcriptomics reference, evaluating neighbor hits, Jensen-Shannon distance, and Spearman correlation against baselines like SageNet and NovoSpaRc.

## When to use

Use when you have SC data plus an ST reference with known coordinates (e.g., 10X Visium h5ad spots) and must quantify how well cell spatial relationships are reconstructed versus competing methods.

## When NOT to use

- There is a large gap between query SC and the ST reference — forcing SC assignment to resemble the ST introduces bias and degrades global reconstruction for all methods
- The tissue has a high level of repeating structural elements, which challenges spatial reconstruction of SC cells
- You need genome-wide spatial deconvolution of spots rather than reconstructing SC-to-SC spatial relationships

## Inputs

- ST reference AnnData h5ad with truth coordinates in .obs[['x','y']] (e.g., 10X Visium S06, 3,037 spots)
- Query SC or ST AnnData h5ad with held-out truth locations (e.g., S08)
- Trained CellContrast model folder (cellContrast_models/epoch_3000.pt)
- Parameter JSON for spot-based ST (parameters/parameters_spot.)

## Outputs

- results.csv with neighbor hit (k=20..200), Jensen-Shannon distance, and Spearman's rank correlation per testing sample
- Spatially related SC representations from the encoder (cosine-similarity-based cell-neighbor lists)
- Optionally, spatially reconstructed SC h5ad with .uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]

## How to apply

Log-normalize the ST reference and query (e.g., 10X Visium human breast samples S06 train / S08 test from a cellxgene deposit) with scran, keeping all 36,503 genes as input in AnnData format with truth locations in .obs[['x','y']]. Train the CellContrast encoder-projector model on the reference with infoNCE loss using sequencing-based/spot default parameters (Visium neighbor size k=20 for positive-pair selection, batch size 64, temperature 0.05, SGD momentum 0.9, weight_decay 5e-4, initial LR 0.1 with cosine decay, 3,000 epochs), saving to cellContrast_models/epoch_3000.pt. At inference, discard the projector and use encoder cosine similarity — higher cosine similarity indicates shorter spatial distance — to identify potential cell neighbors. Evaluate with the eval module computing average nearest-neighbor hits for k=20..200, Jensen-Shannon distance of neighborhood cell-type distributions at k=20, and Spearman's rank correlation of global distances, writing results to results.csv; compare against baselines such as SageNet (JSD 0.369) and NovoSpaRc (JSD 0.393).

## Related tools

- **CellContrast** (Trains the encoder-projector contrastive model (infoNCE loss) on ST reference data, runs the encoder on query cells, and evaluates neighbor hit, JSD, and Spearman metrics) — https://github.com/HKU-BAL/CellContrast
- **SageNet** (Graph-neural-network baseline for benchmarking local spatial reconstruction (mean JSD 0.369 on the human breast S06->S08 task))
- **NovoSpaRc** (Mapping baseline assigning SC cells to predefined locations; benchmarked comparison point (mean JSD 0.393))

## Examples

```
python cellContrast.py eval --ref_data_path ref_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models/ --save_path results.csv
```

## Evaluation signals

- Neighbor hit at k=200 on the human breast S06->S08 task reaches approximately 58.556 true neighbors (~1.8x the next-best method)
- Mean average JSD of neighborhood cell-type distributions at k=20 is approximately 0.306, beating SageNet (0.369) and NovoSpaRc (0.393)
- results.csv contains all three metrics (neighbor hit, JSD, Spearman's rank correlation) for each testing sample
- Training run terminates with the expected artifact cellContrast_models/epoch_3000.pt under the specified 3,000-epoch cosine-decay schedule
- Reconstructed h5ad carries new .uns fields: 'cosine sim of rep', 'representation', 'referenced x', 'referenced y', 'de novo x', 'de novo y'

## Limitations

- SC-ST mapping can cause a 'cell clumping' issue where multiple neighbors project into the same reference location, losing predicted SC-SC distances
- Large gaps between ST and SC samples cause a performance drop in global reconstruction across all methods, and forcing SC to resemble ST can introduce bias
- Using predicted neighbors based on SC-SC distances may lose similarity to the reference spatial distributions
- MDS-based (de novo) mapping is more robust to SC-ST gaps but loses the pre-knowledge of the reference's cell distribution
- Tissues with a high level of repeating elements pose a challenge for spatial reconstruction

## Evidence

- [discussion] During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [discussion] Contrastive pairs derived from ST data with truth locations: "constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss"
- [readme] Evaluation metrics included in the eval module: "three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [readme] ST training data format requirement: "CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']])`."
- [discussion] Cosine-similarity-based neighbor identification: "The resulting cell-cell distances are estimated through cosine similarities of learnt representations and therefore identifying the potential cell neighbors."
