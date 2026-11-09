---
name: adapted-abc-gabc-score-configuration
description: Use when scoring enhancer-gene interactions with STARE and you want the
  adapted (generalised gABC) variant instead of the regular ABC score — e.g., comparing
  gABC vs ABC recall against eQTL-supported enhancer-gene pairs.
when_to_use_negative:
- You chose to run the ABC-scoring without the adapted score — STARE states the adapted
  activity cannot then be used for scaling
- Your goal is TF affinity computation or per-gene TF score summarisation rather than
  enhancer-gene interaction scoring (use STARE's independent affinity task instead)
- No chromatin contact (Hi-C) data are available, since the Activity-By-Contact model
  depends on contact matrices
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3173
tools:
- name: STARE
  role: Framework hosting the adapted ABC (gABC) scoring of enhancer-gene interactions
    for candidate regions
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package providing STARE_ABCpp, the separate ABC scorer run with -t
    0 (regular) and -q True -i all_tss (gABC) configurations
  repo: https://github.com/schulzlab/STARE
- name: ABC-Enhancer-Gene-Prediction
  role: Original ABC model implementation whose score the adapted (gABC) configuration
    improves upon; baseline for comparison
  repo: https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:7379a035ab85e6fe1a26592386730cdb5673b31bc21f5f7f4c5db6753f4a78cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - ABC-Enhancer-Gene-Prediction
  - Cicero
  - STARE
  - stare-abc
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# adapted-abc-gabc-score-configuration

## Summary

Configures STARE's adapted Activity-By-Contact (gABC) scoring, which adapts enhancer activity by the number of contacted candidate genes and considers all annotated TSSs of a gene, to produce enhancer-gene interaction scores that outperform the regular ABC score on GTEx eQTL-supported pairs.

## When to use

Use when scoring enhancer-gene interactions with STARE and you want the adapted (generalised gABC) variant instead of the regular ABC score — e.g., comparing gABC vs ABC recall against eQTL-supported enhancer-gene pairs.

## When NOT to use

- You chose to run the ABC-scoring without the adapted score — STARE states the adapted activity cannot then be used for scaling
- Your goal is TF affinity computation or per-gene TF score summarisation rather than enhancer-gene interaction scoring (use STARE's independent affinity task instead)
- No chromatin contact (Hi-C) data are available, since the Activity-By-Contact model depends on contact matrices

## Inputs

- Single-cell or bulk ATAC-seq candidate regions (bed_file, e.g. from STARE region-gene mapping)
- Hi-C chromatin contact files normalized from .hic via Juicer (e.g. Fulco et al. 2019 average hg19 matrix or ENCODE ENCFF134PUN averaged hg38 matrix)
- Gene annotation in GTF format (e.g. Gencode)
- GTEx eQTL-gene pairs fine-mapped with CAVIAR, CaVEMaN, DAP-G (for recall benchmarking)

## Outputs

- Enhancer-gene interaction table with regular ABC scores (all interactions in 5 MB window, -t 0)
- Enhancer-gene interaction table with adapted gABC scores (-q True, -i all_tss)
- Per-cell-type eQTL recall comparison table (eqtl_recall_comparison.tsv) with Wilcoxon signed-rank p-value

## How to apply

Install STARE via Bioconda (`conda install stare-abc`) and validate with the Code/runTestCases.sh script before real runs. Run the separate ABC scorer (STARE_ABCpp) twice per cell type: once in the default configuration with the score cutoff -t set to 0 so all interactions within the default 5 MB window around the 5' TSS are written, and once with the adapted-activity, all-TSS (generalised gABC) configuration using -q True and -i all_tss. The adapted score rescales enhancer activity according to the number of contacts the enhancer has to its candidate target genes within the window around candidate regions, and distinguishes regions close to the promoter from more distant ones when region-gene mapping used the ABC score. Rank each method's interactions by score and retain the top 300,000 per cell type for downstream benchmarking against Cicero co-accessibility interactions and GTEx eQTL-gene pairs fine-mapped with CAVIAR, CaVEMaN, and DAP-G. Judge success by two-sided Wilcoxon signed-rank recall comparison between ABC and gABC across cell types (expected gABC advantage, P ≈ 0.0005 in the published heart single-cell ATAC-seq and K562 analyses).

## Related tools

- **STARE** (Framework hosting the adapted ABC (gABC) scoring of enhancer-gene interactions for candidate regions) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package providing STARE_ABCpp, the separate ABC scorer run with -t 0 (regular) and -q True -i all_tss (gABC) configurations) — https://github.com/schulzlab/STARE
- **ABC-Enhancer-Gene-Prediction** (Original ABC model implementation whose score the adapted (gABC) configuration improves upon; baseline for comparison) — https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction

## Examples

```
conda install stare-abc; STARE_ABCpp -t 0 -q True -i all_tss --bed_file <candidate_regions.bed>
```

## Evaluation signals

- runTestCases.sh passes after installation, confirming the build/scorer works before real runs
- With -t 0, the output contains all interactions within the default 5 MB window around the 5' TSS, not a filtered top set
- The gABC run differs from the regular run only in the adapted-activity rescaling (contact-count-adapted activity) and all-TSS assignment (-i all_tss)
- Both methods' top-300,000 interaction sets per cell type intersected with GTEx eQTL pairs show gABC recall significantly higher than ABC (two-sided Wilcoxon signed-rank P ≈ 0.0005 across CAVIAR, CaVEMaN, DAP-G)
- eqtl_recall_comparison.tsv is produced with per-cell-type recall rows for the four heart cell types plus K562

## Limitations

- The adapted activity cannot be used for scaling if the ABC-scoring was run without the adapted score, so the adapted model requires ABC-based region-gene mapping
- The adapted model requires contact (Hi-C) information; accuracy depends on the contact matrix used (e.g. Fulco et al. hg19 average vs Gschwind et al. ENCFF134PUN hg38 averaged matrix)
- The model is comparably accurate with only one enhancer-activity assay, but multiple assays can further change scores

## Evidence

- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has to its candidate target genes: [evidence span withheld — anchor status: not_found]
- [methods] adapted activity adjustment window parameter: "Size of the window around your candidate regions in which genes are considered for the adapted activity adjustment"
- [methods] ABC-score-based scaling with promoter-proximal distinction: "If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones."
- [readme] STARE framework scope: "STARE is a framework that combines two tasks, which can also work independently:"
