---
name: gtex-eqtl-finemapped-pair-intersection
description: 'Use when validating enhancer-gene interaction predictions against orthogonal
  genetic evidence: intersect method-ranked enhancer-gene lists with fine-mapped GTEx
  eQTL credible sets to compute and compare eQTL-pair recall per method and cell type.'
when_to_use_negative:
- When the goal is TF-affinity summarisation or gene expression prediction from TF
  affinities (STARE's GAZE workflow), not enhancer-gene assignment benchmarking
- When no fine-mapped eQTL credible sets (CAVIAR, CaVEMaN, DAP-G) are available for
  the tissue or cell types of interest, since recall cannot be computed
- When comparing methods without a matched scoring scale or ranking depth (e.g. fewer
  than the top-300,000 interactions retained), which invalidates paired recall comparison
tools:
- name: STARE / stare-abc
  role: Scores enhancer-gene interactions with the Activity-By-Contact model (STARE_ABCpp
    separate scorer) in both regular and generalized adapted-activity, all-TSS configurations
    to produce the interaction lists being benchmarked
  repo: https://github.com/schulzlab/STARE
- name: Cicero
  role: Provides co-accessibility-based enhancer-gene interactions as a baseline method
    for the same cell types
- name: bedtools
  role: Genome arithmetic for intersecting enhancer region coordinates with eQTL credible-set
    variant regions
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:29574589a04f91c695a8a53391a8d46a493203705635ba2a42ac110506c04d50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Cicero
  - STARE
  - STARE / stare-abc
  - bedtools
  - stare-abc
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gtex-eqtl-finemapped-pair-intersection

## Summary

Benchmarks enhancer-gene prediction methods (regular ABC, generalized gABC, Cicero co-accessibility) by intersecting their top-scored interactions with high-confidence GTEx eQTL-gene pairs fine-mapped with CAVIAR, CaVEMaN, and DAP-G, and testing recall differences with a Wilcoxon signed-rank test.

## When to use

Use when validating enhancer-gene interaction predictions against orthogonal genetic evidence: intersect method-ranked enhancer-gene lists with fine-mapped GTEx eQTL credible sets to compute and compare eQTL-pair recall per method and cell type.

## When NOT to use

- When the goal is TF-affinity summarisation or gene expression prediction from TF affinities (STARE's GAZE workflow), not enhancer-gene assignment benchmarking
- When no fine-mapped eQTL credible sets (CAVIAR, CaVEMaN, DAP-G) are available for the tissue or cell types of interest, since recall cannot be computed
- When comparing methods without a matched scoring scale or ranking depth (e.g. fewer than the top-300,000 interactions retained), which invalidates paired recall comparison

## Inputs

- STARE-scored enhancer-gene interaction lists per cell type (regular ABC and generalized gABC with -q True, -i all_tss), top 300,000 by score
- High-confidence GTEx eQTL-gene pairs fine-mapped with CAVIAR, CaVEMaN, and DAP-G
- Cicero co-accessibility enhancer-gene interactions for the four heart cell types and K562 (baseline)
- Enhancer region coordinates and gene annotations for matching enhancer regions to credible-set variants

## Outputs

- Per-method, per-cell-type recall table of eQTL-supported pairs (eqtl_recall_comparison.tsv)
- Wilcoxon signed-rank p-value comparing ABC vs gABC eQTL-pair recall (expected P ≈ 0.0005)

## How to apply

Score enhancer-gene interactions per cell type with STARE's separate ABC scorer (STARE_ABCpp), setting the score cutoff -t to 0 so all interactions within the default 5 MB window around the 5' TSS are written out; repeat with the adapted-activity, all-TSS generalized configuration (-q True, -i all_tss). Rank each method's interactions by score and retain the top 300,000 per cell type, alongside Cicero co-accessibility interactions as a baseline for the four heart cell types plus K562. Load high-confidence GTEx eQTL-gene pairs fine-mapped with CAVIAR, CaVEMaN, and DAP-G, and intersect each method's top-300,000 set with eQTL-supported pairs by matching the enhancer region to the credible-set variant and gene identity. Compute eQTL-pair recall per method and cell type, then compare ABC vs gABC recall across cell types with a two-sided Wilcoxon signed-rank test and record the p-value.

## Related tools

- **STARE / stare-abc** (Scores enhancer-gene interactions with the Activity-By-Contact model (STARE_ABCpp separate scorer) in both regular and generalized adapted-activity, all-TSS configurations to produce the interaction lists being benchmarked) — https://github.com/schulzlab/STARE
- **Cicero** (Provides co-accessibility-based enhancer-gene interactions as a baseline method for the same cell types)
- **bedtools** (Genome arithmetic for intersecting enhancer region coordinates with eQTL credible-set variant regions) — https://github.com/arq5x/bedtools2

## Examples

```
conda install stare-abc
```

## Evaluation signals

- gABC's eQTL-pair recall is significantly higher than regular ABC's, with a two-sided Wilcoxon signed-rank P-value around 0.0005
- The recall advantage of gABC holds across all three fine-mapping methods (CAVIAR, CaVEMaN, DAP-G) and across eQTL datasets
- The per-cell-type recall table is written to eqtl_recall_comparison.tsv with one recall value per method per cell type (four heart cell types plus K562)
- Recall is computed consistently on matched top-300,000 interaction sets for every method, with enhancer regions matched to credible-set variants and gene identity
- STARE installation is validated by running the runTestCases.sh script before scoring

## Limitations

- Recall depends on the arbitrary top-300,000 interaction cutoff per cell type; different ranking depths will change absolute recall values
- The Wilcoxon signed-rank test is computed across only five cell types (four heart cell types plus K562), so paired-sample size is small
- Intersecting enhancer regions with credible-set variants requires matching enhancer region to variant and gene identity, which can lose pairs due to coordinate or gene-annotation mismatches (e.g. hg19 vs hg38 reference builds)

## Evidence

- [results] The gABC interactions recovered significantly more eQTL-gene pairs than regular ABC interactions across all fine-mapping methods (CAVIAR, CaVEMaN, DAP-G) and eQTL datasets, at P-value ≈ 0.0005 (Wilcoxon signed-rank test).: [evidence span withheld — anchor status: not_found]
- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has to its candidate target genes: [evidence span withheld — anchor status: not_found]
- [readme] STARE is a framework that combines two tasks, which can also work independently: "STARE is a framework that combines two tasks, which can also work independently"
- [supplementary] The GTEx consortium atlas of genetic regulatory effects across human tissues: "The GTEx consortium atlas of genetic regulatory effects across human tissues"
