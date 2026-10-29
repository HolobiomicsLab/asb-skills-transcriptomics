---
name: comparative-tool-performance-evaluation
description: Use when you must compare the computational performance (e.g. runtime)
  of differential expression tools on the same RNA-seq counts matrix, such as testing
  whether DESeq2 is slower than edgeR and limma.
when_to_use_negative:
- Input is a TPM or log2(TPM+1) matrix rather than raw counts — count-based differential
  models (edgeR, DESeq2) require the counts matrix, while TPM is used in ARMT only
  for normalization and GSVA.
- The goal is to compare DEG result sets (overlapping genes, DEG counts) rather than
  computational performance — the article notes different tools yield different results
  on statistical grounds, which is a separate analysis from runtime benchmarking.
- Only a single differential analysis run is needed and no cross-tool timing comparison
  is planned.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ARMT
  role: Automatic RNA-seq tool that supplies the TCGA clinical/counts data and the
    edgeR-based differential analysis pipeline being benchmarked
  repo: https://github.com/Dulab2020/ARMT
- name: edgeR
  role: Differential analysis tool timed on the LUAD counts matrix with default TMM
    normalization; used by ARMT for DEG detection 'for high efficiency'
- name: limma
  role: Differential analysis tool timed on the identical counts matrix under identical
    sample grouping as the runtime comparison baseline
- name: DESeq2
  role: Differential analysis tool timed on the identical counts matrix; expected
    to be the slowest of the three, reproducing Fig. 1B
- name: R
  role: Environment in which all three differential analyses are run and wall-clock
    times recorded (system.time / Sys.time)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:c203ad989f3fadc5a9ee84fae37afab80cfb0d8c2fabc82e18a8e3e0745ad19a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - DESeq2
  - R
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# comparative-tool-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock runtime of RNA-seq differential analysis tools (edgeR, limma, DESeq2) on the identical counts matrix under identical settings, to reproduce ARMT's finding that DESeq2 is significantly slower than edgeR and limma (Fig. 1B).

## When to use

Use when you must compare the computational performance (e.g. runtime) of differential expression tools on the same RNA-seq counts matrix, such as testing whether DESeq2 is slower than edgeR and limma.

## When NOT to use

- Input is a TPM or log2(TPM+1) matrix rather than raw counts — count-based differential models (edgeR, DESeq2) require the counts matrix, while TPM is used in ARMT only for normalization and GSVA.
- The goal is to compare DEG result sets (overlapping genes, DEG counts) rather than computational performance — the article notes different tools yield different results on statistical grounds, which is a separate analysis from runtime benchmarking.
- Only a single differential analysis run is needed and no cross-tool timing comparison is planned.

## Inputs

- TCGA LUAD raw counts matrix as .csv (rows: gene Ensembl IDs; columns: sample IDs)
- Sample grouping definition (experiment vs. control, or numeric factor split into high/low groups with the low part as control, as in ARMT differential analysis)

## Outputs

- runtime_benchmark.csv: one row per tool (edgeR, limma, DESeq2) with a positive numeric wall-clock runtime value
- A comparative ordering of tool runtimes (DESeq2 > edgeR and limma, per Fig. 1B of the ARMT paper)

## How to apply

Load the TCGA lung adenocarcinoma (LUAD) raw counts matrix (gene Ensembl IDs as rows, sample IDs as columns, in a .csv file as required by ARMT's input format). Run edgeR differential analysis on this exact matrix with default TMM normalization (one of the four options ARMT supports: TMM, TMMwsp, RLE, upperquartile) and record wall-clock time; repeat with limma and then DESeq2 on the same matrix with the same sample grouping. Wrap each call in a timer (e.g. R's system.time) so the timing includes the full differential analysis, then assemble the three measurements into a one-row-per-tool table. The rationale is that the tools differ in statistical principle, so a fair performance comparison must fix the input matrix and grouping and vary only the tool; ARMT itself uses edgeR 'for high efficiency', motivating an explicit runtime comparison against limma and DESeq2.

## Related tools

- **ARMT** (Automatic RNA-seq tool that supplies the TCGA clinical/counts data and the edgeR-based differential analysis pipeline being benchmarked) — https://github.com/Dulab2020/ARMT
- **edgeR** (Differential analysis tool timed on the LUAD counts matrix with default TMM normalization; used by ARMT for DEG detection 'for high efficiency')
- **limma** (Differential analysis tool timed on the identical counts matrix under identical sample grouping as the runtime comparison baseline)
- **DESeq2** (Differential analysis tool timed on the identical counts matrix; expected to be the slowest of the three, reproducing Fig. 1B)
- **R** (Environment in which all three differential analyses are run and wall-clock times recorded (system.time / Sys.time))

## Examples

```
t_edger <- system.time({y <- DGEList(counts=mat, group=grp); y <- calcNormFactors(y); fit <- glmQLFit(estimateDisp(y, design), design)}); print(t_edger)
```

## Evaluation signals

- runtime_benchmark.csv exists and contains exactly three rows: edgeR, limma, DESeq2
- Every recorded runtime is a positive numeric value
- DESeq2's runtime is the largest of the three, matching the ordering reported in Fig. 1B of the ARMT paper
- All three tools were run on the identical LUAD counts matrix with identical sample grouping (no tool saw a different input)
- edgeR run used default TMM normalization, one of the four normalization options ARMT provides (TMM, TMMwsp, RLE, upperquartile)

## Limitations

- No software version numbers for edgeR, limma, or DESeq2 are stated in the article, so exact reproducibility of absolute runtimes is not guaranteed; runtimes are hardware-dependent.
- Because different tools are based on different statistical principles and 'may lead to different results', runtime equality of setup does not imply comparable DEG outputs — this skill evaluates performance, not result concordance.
- ARMT's own differential analysis employs edgeR for DEGs, so the limma and DESeq2 timing arms must be run outside the ARMT GUI pipeline using the same input matrix.

## Evidence

- [methods] We employ ‘edgeR’ [17] packages to detect differentially expressed genes (DEGs) for high efficiency: "We employ ‘edgeR’ [17] packages to detect differentially expressed genes (DEGs) for high efficiency"
- [intro] Based on different statistical principles, different tools may lead to different results [28], [29]: "Based on different statistical principles, different tools may lead to different results [28], [29]"
- [readme] Counts matrix must be in a .csv file. The row name is gene Ensembl ID and the column name is the sample ID.: "Counts matrix must be in a .csv file. The row name is gene Ensembl ID  and the column name is the sample ID."
- [other] Run edgeR differential analysis on the identical LUAD counts matrix using the default TMM normalization: [evidence span withheld — anchor status: not_found]
