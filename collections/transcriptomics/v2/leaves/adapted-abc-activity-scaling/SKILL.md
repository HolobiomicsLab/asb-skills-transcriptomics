---
name: adapted-abc-activity-scaling
description: Use when scoring enhancer–gene interactions from activity assays (DNase-seq,
  H3K27ac/ATAC-seq) plus contact data and you want higher AUPRC than the regular ABC
  score, or when validating whether a single activity assay suffices.
when_to_use_negative:
- When the goal is de novo enhancer identification from ATAC/ChIP peaks rather than
  scoring interactions among already-defined candidate enhancers
- When you cannot supply gene annotation or contact data and the distance-based default
  c = max{d, 5000}^-1 is unacceptable for your biological question
- When working with single-cell regulatory-network outputs from GAZE rather than bulk
  CRISPRi-validated enhancer–gene assignment
tools:
- name: STARE
  role: Framework that executes the adapted-ABC enhancer–gene scoring (installed as
    stare-abc); the -q True / -i all_tss flags invoke the generalized model
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package providing STARE_ABCpp for ABC/gABC scoring with configurable
    activity column, contact data, and cutoffs
  repo: http://bioconda.github.io/recipes/stare-abc/README.html
- name: pROC
  role: R package used to compute precision-recall AUPRC of gABC vs. regular ABC predictions
    against CRISPRi gold-standard labels
- name: Juicer
  role: Used via Juicebox_normalization.sh to produce normalized chromatin contact
    files from .hic inputs for STARE_ABCpp -f
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:9a7a02d5a84cf56bdb07b0175d7a56a323ca0feb8b39bdd65335e7eb1b3d8c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Juicer
  - STARE
  - pROC
  - stare-abc
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# adapted-abc-activity-scaling

## Summary

Score enhancer–gene interactions with the generalized (adapted) ABC model — gene-specific enhancer activity averaged across all annotated TSSs and adapted to each enhancer's contact count — which improved AUPRC by ~0.107 over the regular ABC score on three K562 CRISPRi validation datasets.

## When to use

Use when scoring enhancer–gene interactions from activity assays (DNase-seq, H3K27ac/ATAC-seq) plus contact data and you want higher AUPRC than the regular ABC score, or when validating whether a single activity assay suffices.

## When NOT to use

- When the goal is de novo enhancer identification from ATAC/ChIP peaks rather than scoring interactions among already-defined candidate enhancers
- When you cannot supply gene annotation or contact data and the distance-based default c = max{d, 5000}^-1 is unacceptable for your biological question
- When working with single-cell regulatory-network outputs from GAZE rather than bulk CRISPRi-validated enhancer–gene assignment

## Inputs

- Candidate-enhancer bed file with separate columns for DNase-seq, H3K27ac ChIP-seq, geometric-mean, and ATAC-seq read counts (all activity values ≥ 0), selected via -n <activity_column>
- Gene annotation in gtf format (e.g. Gencode)
- Normalized chromatin contact files (e.g. Fulco et al. 2019 average hg19 Hi-C matrix, or Gschwind et al. 2023 hg38 ENCODE matrix ENCFF134PUN); .hic files pre-normalized via Juicer's Juicebox_normalization.sh
- CRISPRi perturbation-effect gold-standard labels (Gasperini 2019, Schraivogel 2020, Fulco 2019)

## Outputs

- STARE_ABCpp-scored enhancer–gene interaction table with adapted ABC scores (gABC, -q True -i all_tss)
- Matched regular ABC-score table (-q False) for baseline comparison
- AUPRC comparison table (auprc_comparison.csv) per dataset/assay/contact combination

## How to apply

Run STARE_ABCpp on the candidate-enhancer bed file with a Gencode gtf annotation (-a), a 5MB window centred at the 5' TSS (-w 5000000), and score cutoff -t 0.02. Enable the generalized model with -q True (adapted activity, scaled by the number of contacts the enhancer has to each candidate target gene within a window around the candidate regions) and -i all_tss (activity averaged across all annotated TSSs of a gene), with contact pseudocount -d True. Supply contact data via -f <contact_data_dir> with -k <bin_size> or use -f false for the distance-based default c = max{d, 5000}^-1. Build a matched baseline run with -q False (original activity, no adaptation) so the two scores are compared on identical inputs. Label predicted pairs against CRISPRi perturbation gold standards (Gasperini 2019, Schraivogel 2020, Fulco 2019) and compute precision-recall AUPRC with the pROC package in R.

## Related tools

- **STARE** (Framework that executes the adapted-ABC enhancer–gene scoring (installed as stare-abc); the -q True / -i all_tss flags invoke the generalized model) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package providing STARE_ABCpp for ABC/gABC scoring with configurable activity column, contact data, and cutoffs) — http://bioconda.github.io/recipes/stare-abc/README.html
- **pROC** (R package used to compute precision-recall AUPRC of gABC vs. regular ABC predictions against CRISPRi gold-standard labels)
- **Juicer** (Used via Juicebox_normalization.sh to produce normalized chromatin contact files from .hic inputs for STARE_ABCpp -f) — https://github.com/aidenlab/juicer

## Examples

```
STARE_ABCpp -b candidates.bed -n activity_column -a gencode.gtf -o out/ -w 5000000 -f contact_data_dir -k bin_size -t 0.02 -q True -i all_tss -d True
```

## Evaluation signals

- gABC (-q True -i all_tss) must outperform the regular ABC (-q False) baseline on every CRISPRi validation dataset (Gasperini, Schraivogel, Fulco)
- Average AUPRC gain of gABC over regular ABC should be ≈ 0.107 across the three datasets
- On the Gasperini dataset, ATAC-seq alone with the adapted score should surpass the combined DNase-seq + H3K27ac ChIP-seq baseline AUPRC
- At least one single-assay setup yields an AUPRC comparable to the combined two-assay baseline
- All activity values in the candidate-enhancer bed columns are ≥ 0 and the score cutoff -t 0.02 and 5MB TSS window are applied consistently across conditions

## Limitations

- The reported 0.107 AUPRC gain is specific to K562 CRISPRi validation datasets (Gasperini, Schraivogel, Fulco); accuracy on other cell types must be re-validated
- For the adapted-activity adjustment, genes are only considered within a window around each candidate region, so window choice affects results
- The comparability of single-assay activity was demonstrated with specific assays (DNase-seq, H3K27ac ChIP-seq, ATAC-seq, and their geometric mean); other assays were not benchmarked here

## Evidence

- [abstract] We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes: "We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes"
- [methods] Size of the window around your candidate regions in which genes are considered for the adapted activity adjustment: "Size of the window around your candidate regions in which genes are considered for the adapted activity adjustment"
- [readme] score enhancer-gene interactions with the Activity-By-Contact (ABC) model: "STARE is a framework that combines two tasks, which can also work independently:  - score enhancer-gene interactions with the Activity-By-Contact (ABC)"
