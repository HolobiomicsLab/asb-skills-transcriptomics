---
name: differential-expression-similarity-based-cell-filtration
description: Use when preprocessing Perturb-seq/CROP-seq data (e.g., crop_stimulated
  expression_profile.txt + perturb_information.txt) to purge cells carrying invalid
  sgRNA edits before topic-model building in MUSIC.
when_to_use_negative:
- Input has not passed Cell_qc and SAVER imputation — Cell_filtering expects the imputed
  QC-passed matrices, not raw counts
- 'Off-target analysis is the sole goal and invalid-edit filtering is irrelevant (note:
  MUSIC only provides off-target information for knockout, not CRISPRi)'
- The data are not single-cell CRISPR screening data with per-cell perturbation labels
  (e.g., plain scRNA-seq without sgRNA assignments)
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: MUSIC
  role: Provides the Cell_filtering function implementing the DE-profile similarity
    filter, the 90% filter-proportion rule, and the 30-cell minimum perturbation threshold
  repo: https://github.com/bm2-lab/MUSIC
- name: SAVER
  role: Upstream imputation (via MUSIC's Data_imputation wrapper) recovering sparse
    single-cell expression values so DE-profile similarity is computed on denoised
    data
  repo: https://github.com/mohuangx/SAVER
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:0ba76abc2e0fc21cb6b33ea6fee09bbe73cc3d97d08fc8c8053a16e50ebcddca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bm2-lab/MUSIC
  tools:
  - CRISPRseek
  - MUSIC
  - R
  - SAVER
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differential-expression-similarity-based-cell-filtration

## Summary

MUSIC's Cell_filtering step removes perturbed single cells whose differentially expressed (DE) gene profile resembles control cells more than same-perturbation cells, then drops perturbations with >=90% filtered cells or fewer than 30 perturbed cells, addressing the evidence that 20–30% of sgRNA-detected cells show a wild-type phenotype.

## When to use

Use when preprocessing Perturb-seq/CROP-seq data (e.g., crop_stimulated expression_profile.txt + perturb_information.txt) to purge cells carrying invalid sgRNA edits before topic-model building in MUSIC.

## When NOT to use

- Input has not passed Cell_qc and SAVER imputation — Cell_filtering expects the imputed QC-passed matrices, not raw counts
- Off-target analysis is the sole goal and invalid-edit filtering is irrelevant (note: MUSIC only provides off-target information for knockout, not CRISPRi)
- The data are not single-cell CRISPR screening data with per-cell perturbation labels (e.g., plain scRNA-seq without sgRNA assignments)

## Inputs

- SAVER-imputed expression profile matrix (genes x cells) from Data_imputation
- Perturbation information vector (named character, control label must be 'CTRL') from Cell_qc/Data_imputation
- Optional sgRNA_information.txt and sgRNA fasta (e.g., crop_stimulated_sgrna.fa) if off-target checks will follow

## Outputs

- Filtered expression profile and perturbation information object (crop_seq_filtered)
- Per-cell keep/filter table
- Invalid_rate plot (Invalid_rate.png)

## How to apply

Run Cell_filtering on the SAVER-imputed expression profile and perturbation information (after Cell_qc with species='Hs'), passing cell_num_threshold=20 and cpu_num=30. The function computes each perturbed cell's DE profile similarity: cells more similar to the 'CTRL' control cells than to other cells of the same perturbation are filtered, which accounts for the reported 20–30% of sgRNA-detected cells showing a wild-type phenotype. Decision points follow: a perturbation whose filtered cells reach 90% (default) of all its cells is dropped entirely, and perturbations with fewer than 30 perturbed cells (default) are excluded from downstream MUSIC analysis. Validation requires the per-cell keep/filter table and the Invalid_rate plot as outputs.

## Related tools

- **MUSIC** (Provides the Cell_filtering function implementing the DE-profile similarity filter, the 90% filter-proportion rule, and the 30-cell minimum perturbation threshold) — https://github.com/bm2-lab/MUSIC
- **SAVER** (Upstream imputation (via MUSIC's Data_imputation wrapper) recovering sparse single-cell expression values so DE-profile similarity is computed on denoised data) — https://github.com/mohuangx/SAVER

## Examples

```
crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Per-cell keep/filter table is produced listing each perturbed cell's filter decision
- Invalid_rate plot (Invalid_rate.png) is generated when plot=T
- Filtering honored the 90% rule: perturbations with >=90% of cells filtered are absent from the output
- Perturbations retained in output each have >=30 perturbed cells
- Control label is 'CTRL' in the perturbation information so the similarity comparison to control cells is well-defined

## Limitations

- Current filtering steps for data noise may need future refinement — the authors state 'future improvements are expected to refine and update such filtering steps to make it more effective'
- Cell filtering is computationally expensive: 'it may take a little long time without parallel computation' (mitigate with cpu_num=30)
- Only 70–80% of sgRNAs generate true loss-of-function of the targeted gene, so filtering cannot fully eliminate invalid-edit effects; off-target checks (CRISPRseek) are a separate, optional step

## Evidence

- [discussion] Filter perturbed cells whose DE profile is more similar to control cells than to other perturbed cells: "if the differentially expressed gene profile of a perturbed cell is more similar to the control cells than that of other same perturbed cells, this cell will be filtered"
- [discussion] 90% filter-proportion and 30-cell minimum thresholds: "if the influenced cells filtered are amount to a high proportion (default 90%) among all, such perturbation is filtered"
- [discussion] Exclude perturbations with fewer than 30 perturbed cells: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [readme] Cell_filtering invocation from README: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
- [discussion] Motivation: 20–30% wild-type phenotype among sgRNA-detected cells: "but only 70–80% of them will generate true loss-of-function of the targeted gene30,31"
