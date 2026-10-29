---
name: enhancer-gene-pair-recall-evaluation
description: Use when comparing candidate enhancer–gene interaction rankings (e.g.
  ABC vs gABC vs co-accessibility) against an independent gold standard such as fine-mapped
  eQTL credible-set pairs, to measure which method recovers more supported pairs.
when_to_use_negative:
- No fine-mapped eQTL credible sets (CAVIAR/CaVEMaN/DAP-G) are available for the tissue
  or cell type — the recall gold standard cannot be built
- Evaluating a method whose output is a TF–gene regulatory score (e.g. STARE/GAZE
  TF affinity summaries) rather than ranked enhancer–gene interaction predictions
- Only one method is being assessed without a paired comparison method — the Wilcoxon
  signed-rank test design requires paired per-cell-type recalls
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: STARE
  role: Framework providing the separate ABC scorer (STARE_ABCpp) used to score enhancer–gene
    interactions in regular and generalised (gABC) configurations before eQTL recall
    evaluation
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package used to install the ABC scoring component of STARE
  repo: https://github.com/schulzlab/STARE
- name: Cicero
  role: Baseline method supplying co-accessibility-based enhancer–gene interactions
    against which ABC/gABC recall is compared
- name: bedtools
  role: Genome arithmetic toolkit for intersecting enhancer regions with eQTL credible-set
    variants
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:a7576a585b3b8b241e144ad3d0a17c4e137d8c72472cf51f17b214c4781e47c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Cicero
  - STARE
  - bedtools
  - stare-abc
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# enhancer-gene-pair-recall-evaluation

## Summary

Benchmark strategy for enhancer–gene assignment methods that computes the recall of fine-mapped GTEx eQTL-supported enhancer–gene pairs recovered by top-ranked ABC, gABC (adapted-activity, all-TSS) and Cicero co-accessibility interactions, and tests recall differences between paired methods with a two-sided Wilcoxon signed-rank test.

## When to use

Use when comparing candidate enhancer–gene interaction rankings (e.g. ABC vs gABC vs co-accessibility) against an independent gold standard such as fine-mapped eQTL credible-set pairs, to measure which method recovers more supported pairs.

## When NOT to use

- No fine-mapped eQTL credible sets (CAVIAR/CaVEMaN/DAP-G) are available for the tissue or cell type — the recall gold standard cannot be built
- Evaluating a method whose output is a TF–gene regulatory score (e.g. STARE/GAZE TF affinity summaries) rather than ranked enhancer–gene interaction predictions
- Only one method is being assessed without a paired comparison method — the Wilcoxon signed-rank test design requires paired per-cell-type recalls

## Inputs

- STARE_ABCpp interaction outputs for regular ABC scoring (cutoff -t 0, all interactions within 5 MB of the 5' TSS)
- STARE_ABCpp interaction outputs for generalised gABC scoring (-q True, -i all_tss)
- Cicero co-accessibility enhancer–gene interactions for the same cell types
- High-confidence GTEx eQTL–gene pairs fine-mapped with CAVIAR, CaVEMaN and DAP-G
- Gene annotation (e.g. Gencode GTF) for TSS definitions and gene identity matching

## Outputs

- Per-method, per-cell-type recall table of eQTL-supported enhancer–gene pairs (eqtl_recall_comparison.tsv)
- Wilcoxon signed-rank test p-value comparing ABC vs gABC recall
- Per-cell-type intersection counts of top-ranked interactions with eQTL credible-set pairs

## How to apply

Score enhancer–gene interactions per cell type with STARE's separate ABC scorer (STARE_ABCpp), setting the score cutoff -t to 0 so all interactions within the default 5 MB window around the 5' TSS are written to output; repeat with the generalised gABC configuration (-q True, -i all_tss) that adapts enhancer activity by contact count and considers all annotated TSSs. Rank each method's interactions by score and retain the top 300,000 per cell type, alongside Cicero co-accessibility interactions as a baseline. Load high-confidence GTEx eQTL–gene pairs fine-mapped with CAVIAR, CaVEMaN and DAP-G, and intersect each method's top-300,000 set with the eQTL-supported pairs, matching the enhancer region to the credible-set variant and gene identity. Compute recall of eQTL-supported pairs per method and per cell type (four heart cell types plus K562), then compare recall between ABC and gABC across cell types with a two-sided Wilcoxon signed-rank test. The paired, per-cell-type design keeps the number of comparisons equal between methods.

## Related tools

- **STARE** (Framework providing the separate ABC scorer (STARE_ABCpp) used to score enhancer–gene interactions in regular and generalised (gABC) configurations before eQTL recall evaluation) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package used to install the ABC scoring component of STARE) — https://github.com/schulzlab/STARE
- **Cicero** (Baseline method supplying co-accessibility-based enhancer–gene interactions against which ABC/gABC recall is compared)
- **bedtools** (Genome arithmetic toolkit for intersecting enhancer regions with eQTL credible-set variants) — https://github.com/arq5x/bedtools2

## Examples

```
conda install stare-abc; STARE_ABCpp -q True -i all_tss -t 0  # regular ABC run omits -q/-i; then intersect top-300k interactions with GTEx eQTL pairs
```

## Evaluation signals

- gABC's eQTL-pair recall is significantly higher than regular ABC's, with a two-sided Wilcoxon signed-rank P ≈ 0.0005
- The per-cell-type recall table is written to eqtl_recall_comparison.tsv
- Each intersection match requires both enhancer-region overlap with the credible-set variant and gene identity agreement
- Recall values fall within [0, 1] and are computed consistently per method per cell type (four heart cell types plus K562)
- Exactly 300,000 top-ranked interactions are retained per method and cell type before intersection

## Limitations

- Recall is bounded by the top-300,000 score-ranking cutoff per cell type; results are sensitive to this retained-set size
- Evaluation relies on GTEx eQTL fine-mapping as the only gold standard, so recall reflects agreement with eQTL-supported pairs rather than true regulatory accuracy
- The comparison covered four heart cell types plus K562; generalisation to other tissues depends on data availability

## Evidence

- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has to its candidate target genes: [evidence span withheld — anchor status: not_found]
- [methods] If you're only interested in the ABC-scored interactions, you can also call that part independently.: "If you're only interested in the ABC-scored interactions, you can also call that part independently."
- [supplementary] The GTEx consortium atlas of genetic regulatory effects across human tissues: "The GTEx consortium atlas of genetic regulatory effects across human tissues"
- [readme] STARE is a framework that combines two tasks, which can also work independently:

 - score enhancer-gene interactions with the Activity-By-Contact (ABC): "STARE is a framework that combines two tasks, which can also work independently:"
