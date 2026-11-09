---
name: zinb-to-nb-loss-replacement-in-autoencoder
description: Use when you need to run or reproduce a ZINB-vs-NB loss ablation in a
  scRNA-seq deep clustering model, i.e. when the analysis goal is to determine whether
  explicit zero-inflation modeling changes clustering performance (NMI/ARI).
when_to_use_negative:
- When the goal is a full benchmark of scBGEDA against state-of-the-art methods rather
  than isolating the effect of the reconstruction loss distribution
- When input data are not raw scRNA-seq counts (e.g. already normalized or feature-reduced
  tables), since the ZINB/NB count modeling assumption no longer applies
- When a single deterministic run suffices — this ablation is defined over 10 random
  seeds (1111–10000) to ensure fairness
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: loss.py
  role: File edited to swap the ZINB reconstruction loss of the master decoder for
    a negative binomial loss while keeping the denoising reconstruction loss unchanged
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Trains the single-cell dual denoising autoencoder for each loss variant with
    10 seeds and 2000 highly variable genes
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: First scBGEDA module; preprocessing applied before training each variant
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: Main function of the third module; produces bipartite graph ensemble consensus
    clusterings for each variant
- name: computeNMI.m
  role: Computes NMI values of the consensus clusterings against ground-truth labels
- name: rand_index.m
  role: Computes ARI values of the consensus clusterings against ground-truth labels
- name: MATLAB
  role: Runtime used to execute BC.m, BGEC.m and the NMI/ARI evaluation scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:d1d841bc00d94097fe5d2c878ed95a8340ce5efea544d8515126af5b84d983f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - MATLAB
  - Python
  - SDDA.py
  - computeNMI.m
  - loss.py
  - preprocess.py
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5238095238095238
n_claims: 21
---

# zinb-to-nb-loss-replacement-in-autoencoder

## Summary

An ablation skill for scBGEDA that swaps the zero-inflated negative binomial (ZINB) reconstruction loss of the master decoder in the dual denoising autoencoder for a plain negative binomial (NB) loss, to test whether zero-inflation modeling is necessary for clustering scRNA-seq data.

## When to use

Use when you need to run or reproduce a ZINB-vs-NB loss ablation in a scRNA-seq deep clustering model, i.e. when the analysis goal is to determine whether explicit zero-inflation modeling changes clustering performance (NMI/ARI).

## When NOT to use

- When the goal is a full benchmark of scBGEDA against state-of-the-art methods rather than isolating the effect of the reconstruction loss distribution
- When input data are not raw scRNA-seq counts (e.g. already normalized or feature-reduced tables), since the ZINB/NB count modeling assumption no longer applies
- When a single deterministic run suffices — this ablation is defined over 10 random seeds (1111–10000) to ensure fairness

## Inputs

- scRNA-seq count matrix (one of the 20 real datasets, e.g. Adam, Klein, Muraro, Plasschaert, Pollen, QS_Heart, QS_Limb_Muscle, Qx_Limb_Muscle, Tosches_turtle, Wang_Lung, Young) with ground-truth cell-type labels
- loss.py from the scBGEDA repository (ZINB reconstruction loss source to modify)
- Ground-truth label vector per dataset for NMI/ARI computation

## Outputs

- Ablation comparison table (e.g. ablation_nmi_ari_table.csv) with per-dataset and seed-averaged NMI/ARI for the ZINB and NB variants
- Bipartite graph ensemble consensus clusterings for both loss variants (from BC.m/BGEC.m)
- Median ARI and NMI clustering performance reports per run

## How to apply

Edit loss.py (the loss functions of the scBGEDA network) so the ZINB reconstruction loss of the master decoder is replaced by a negative binomial loss, leaving the denoising reconstruction loss of the second decoder unchanged. Run preprocess.py followed by SDDA.py for each variant over the 20 real datasets with 10 random seeds (1111–10000) and 2000 highly variable genes (e.g. 'python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, ..., 10000"'). Then run the third module via 'matlab -r "BC '<dataset>' 100 2 40 [seeds]"' to produce bipartite graph ensemble consensus clusterings for both variants. Evaluate each consensus clustering with computeNMI.m and rand_index.m against ground-truth labels, and aggregate per-dataset and seed-averaged NMI/ARI into a comparison table (e.g. ablation_nmi_ari_table.csv). The decision point is the performance gap: the article reports the NB variant achieves only slightly higher average NMI and ARI than ZINB, so if NB performs comparably, zero-inflation modeling in the autoencoder loss is not critical for this pipeline.

## Related tools

- **loss.py** (File edited to swap the ZINB reconstruction loss of the master decoder for a negative binomial loss while keeping the denoising reconstruction loss unchanged) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Trains the single-cell dual denoising autoencoder for each loss variant with 10 seeds and 2000 highly variable genes) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (First scBGEDA module; preprocessing applied before training each variant) — https://github.com/wangyh082/scBGEDA
- **BC.m** (Main function of the third module; produces bipartite graph ensemble consensus clusterings for each variant)
- **computeNMI.m** (Computes NMI values of the consensus clusterings against ground-truth labels)
- **rand_index.m** (Computes ARI values of the consensus clusterings against ground-truth labels)
- **MATLAB** (Runtime used to execute BC.m, BGEC.m and the NMI/ARI evaluation scripts)

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000"
```

## Evaluation signals

- ablation_nmi_ari_table.csv contains NMI/ARI entries for BOTH loss variants on all 20 datasets
- Seed-averaged NMI and ARI of the NB variant are only slightly higher than the ZINB variant, matching Supplementary Fig. S3 / Results 3.8
- NB outperforms ZINB on the specific datasets Adam, Klein, Muraro, Plasschaert, Pollen, QS_Heart, QS_Limb_Muscle, Qx_Limb_Muscle, Tosches_turtle, Wang_Lung and Young
- All runs use the same 10 seeds (1111–10000) and 2000 highly variable genes so the only variable is the loss function
- Final output reports the median values of ARI and NMI as described in the README output explanation

## Limitations

- The ablation is specific to the scBGEDA dual denoising autoencoder; the denoising decoder loss must remain unchanged, otherwise the comparison is confounded
- Only 20 real datasets were tested; the conclusion that NB performs comparably to (slightly better than) ZINB may not transfer to other architectures or very sparse datasets
- Variation across the 10 random seeds means single-seed results could flip the marginal average difference; the reported pattern is based on seed-averaged metrics

## Evidence

- [supplementary] Replacing ZINB with NB in scBGEDA yields comparable clustering: the average NMI and ARI of the NB variant are just slightly higher: [evidence span withheld — anchor status: not_found]
- [supplementary] NB performs better on the 11 datasets: [evidence span withheld — anchor status: not_found]
- [supplementary] loss.py | the loss functions of the network: "loss.py      | the loss functions of the network"
- [abstract] ZINB reconstruction loss and denoising reconstruction loss: "learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss"
- [readme] The final output reports the clustering performance and the median values of ARI and NMI is provided: "The final output reports the clustering performance and the median values of ARI and NMI is provided, respectively."
