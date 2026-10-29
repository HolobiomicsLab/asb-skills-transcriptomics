---
name: precision-recall-curve-analysis
description: Use when comparing variant-calling or classification tools against a
  known ground truth, and you need cutoff-independent metrics (AUPRC/AUROC) plus PR/ROC
  curves across simulation settings.
when_to_use_negative:
- No ground-truth labels exist for the called variants — PR/ROC against unknown truth
  is undefined; use clonal reconstruction validation instead.
- The tool only outputs binary calls with no tunable score cutoff, so a PR/ROC curve
  cannot be traced.
- Evaluating clustering or clone-assignment quality (e.g., vireoSNP assignments) rather
  than per-variant detection performance.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: MQuad
  role: Primary tool under evaluation; its ΔBIC cutoff is swept to trace its PR/ROC
    curve
  repo: https://github.com/single-cell-genetics/MQuad
- name: mgatk
  role: Comparator; log10(VMR) is varied (strand information unused) to obtain its
    PR/ROC curve
- name: Monovar
  role: Comparator; its MPR statistic is varied to trace its PR/ROC curve
- name: NEAT-genReads
  role: Synthesizes paired-end mtDNA reads (hg19, 152 bp, ~1000x coverage) to create
    simulation ground truth
- name: SomatoSim
  role: Adds background noise and clonal mutations with known parameters to define
    ground-truth variants
- name: cellsnp-lite
  role: Piles up reads into the SNP-by-cell AD/DP matrix consumed by MQuad in the
    benchmark
  repo: https://github.com/single-cell-genetics/cellsnp-lite
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:1996cc3bf8fd9e177e4a98dbabe67409c03acf450d6722ce384f18348d75ea28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/MQuad
  tools:
  - MQuad
  - Monovar
  - NEAT-genReads
  - SomatoSim
  - cellsnp-lite
  - mgatk
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# precision-recall-curve-analysis

## Summary

Benchmark variant-calling performance by sweeping a tool's score cutoff to trace precision-recall (PR) and ROC curves, then comparing AUPRC/AUROC against ground-truth clonal mtDNA variants — as done to show MQuad (AUPRC 0.976) outperforms mgatk and Monovar on simulated Smart-seq2-like data.

## When to use

Use when comparing variant-calling or classification tools against a known ground truth, and you need cutoff-independent metrics (AUPRC/AUROC) plus PR/ROC curves across simulation settings.

## When NOT to use

- No ground-truth labels exist for the called variants — PR/ROC against unknown truth is undefined; use clonal reconstruction validation instead.
- The tool only outputs binary calls with no tunable score cutoff, so a PR/ROC curve cannot be traced.
- Evaluating clustering or clone-assignment quality (e.g., vireoSNP assignments) rather than per-variant detection performance.

## Inputs

- Ground-truth labeled variant sets (clonal variants injected by SomatoSim into simulated 90-cell mtDNA read datasets)
- Per-variant confidence scores per tool: MQuad ΔBIC values, mgatk log10(VMR), Monovar MPR statistics
- Simulated paired-end mtDNA BAMs/pileups (cellsnp-lite cell-by-SNP VAF matrix) across settings M ∈ {5,10,25,50}, μ ∈ {0.01,0.05,0.1}

## Outputs

- Precision-recall and ROC curves per tool per simulation setting (PNG figures for default setting)
- AUPRC and AUROC values per tool per setting (benchmark_auprc_auroc.csv)
- Tool ranking against ground truth (e.g., MQuad AUPRC 0.976 / AUROC 1.00 vs mgatk 0.800 / 0.999 vs Monovar 0.147 / 0.968)

## How to apply

Generate ground-truth data by synthesizing mitochondrial reads in silico with NEAT-genReads v3.0 (hg19, 152 bp reads, ~1000x coverage) and SomatoSim v1.0.0, varying clonal variant counts M ∈ {5,10,25,50}, allele frequencies Lognormal(μ, 0.005) with μ ∈ {0.01,0.05,0.1}, clone ratios, and lineage trees. For each tool, obtain a continuous confidence score: sweep MQuad's ΔBIC cutoff (Kneedle knee point as default), mgatk v0.6.2's log10(VMR), and Monovar's MPR statistic. At every cutoff, compare each tool's calls against ground-truth clonal variants to compute precision, recall, and true/false positive rates. Record per-tool per-setting AUPRC and AUROC (e.g., to benchmark_auprc_auroc.csv) and save PR/ROC curves for the default setting (M=10, μ=0.1, r=1:1:1, t=branched). Validate by checking relative ordering: MQuad should exceed mgatk (AUPRC 0.800) and Monovar (AUPRC 0.147) at defaults.

## Related tools

- **MQuad** (Primary tool under evaluation; its ΔBIC cutoff is swept to trace its PR/ROC curve) — https://github.com/single-cell-genetics/MQuad
- **mgatk** (Comparator; log10(VMR) is varied (strand information unused) to obtain its PR/ROC curve)
- **Monovar** (Comparator; its MPR statistic is varied to trace its PR/ROC curve)
- **NEAT-genReads** (Synthesizes paired-end mtDNA reads (hg19, 152 bp, ~1000x coverage) to create simulation ground truth)
- **SomatoSim** (Adds background noise and clonal mutations with known parameters to define ground-truth variants)
- **cellsnp-lite** (Piles up reads into the SNP-by-cell AD/DP matrix consumed by MQuad in the benchmark) — https://github.com/single-cell-genetics/cellsnp-lite

## Examples

```
mquad run --cellsnp out/cellsnp/ -o out/mquad/ && python plot_pr_roc.py --calls out/mquad/filtered_cells_labels.tsv --truth truth_clonal_variants.tsv --out benchmark_auprc_auroc.csv
```

## Evaluation signals

- At default settings (M=10 variants per clone), MQuad reaches AUPRC ≈ 0.976 and AUROC ≈ 1.00, exceeding mgatk (AUPRC 0.800, AUROC 0.999) and Monovar (AUPRC 0.147, AUROC 0.968).
- benchmark_auprc_auroc.csv contains one AUPRC/AUROC row per tool per simulation setting (M, μ, clone ratio, lineage tree).
- Each traced PR/ROC curve is monotone and derived from a full sweep of the tool's score cutoff (ΔBIC, log10(VMR), or MPR), not a single operating point.
- Calls are compared position-wise against the SomatoSim-injected ground-truth clonal variant list at every cutoff.
- The default-setting PR and ROC curves are saved as PNG figures matching Fig. 1b-style reporting.

## Limitations

- Results depend on simulation assumptions (NEAT-genReads + SomatoSim parameters); relative rankings may differ on real Smart-seq2 or other assay data.
- mgatk was designed primarily for scATAC-seq, so its poor AUPRC here (0.147 for Monovar, 0.800 for mgatk) reflects a diploid-context/assay mismatch for scRNA-seq.
- The default MQuad threshold is the Kneedle knee-point ΔBIC; other cutoff choices change the operating point though the full curve is cutoff-independent.
- Frequent false positives in error-prone rCRS regions 302–315, 513–525, 3105–3109 should be removed before scoring.

## Evidence

- [results] MQuad best default performance: [evidence span withheld — anchor status: not_found]
- [discussion] Simulation design: "Mitochondria reads were synthesized in silico with NEAT-genReads v3.038 and SomatoSim (v1.0.0)39"
- [discussion] ΔBIC cutoff selection: "a ΔBIC cutoff for selection of SNPs is determined using the Kneedle algorithm36"
- [discussion] mgatk cutoffs: "we used the suggested cutoffs to call variants: log10(VMR) ≥ −2 and strand correlation coefficient ≥0.65"
- [abstract] MQuad purpose: "MQuad can identify mitochondrial variants with both high sensitivity and specificity, outperforming existing methods by a large extent"
- [readme] Pipeline role of cellsnp-lite: "use cellsnp-lite (a faster version of cellSNP, still at testing stage so might be unstable) to pileup mtDNA variants from raw .bam file(s)"
