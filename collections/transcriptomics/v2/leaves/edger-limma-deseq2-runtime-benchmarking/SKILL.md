---
name: edger-limma-deseq2-runtime-benchmarking
description: Use when choosing between edgeR, limma and DESeq2 for RNA-seq differential
  analysis and computational run time matters, or when reproducing the ARMT Fig. 1B
  speed comparison on a shared counts matrix.
when_to_use_negative:
- The goal is differential expression results (DEG lists, p-values, logFC) rather
  than tool speed — this skill measures run time only and does not compare DEG output
  quality
- The input is already normalized (e.g. TPM or log2(TPM + 1) matrix used for GSVA
  in ARMT) rather than raw counts — edgeR/DESeq2 differential analysis requires the
  counts matrix
- Benchmarking other ARMT modules (GSVA, survival analysis, Cox regression, mutant
  mapping) instead of the three differential analysis tools
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: edgeR
  role: Runs differential analysis on the shared LUAD counts matrix with default TMM
    normalization; the tool ARMT itself employs 'for high efficiency'; timed as one
    of the three competitors
  repo: https://github.com/Dulab2020/ARMT
- name: limma
  role: Runs differential analysis on the identical counts matrix under identical
    settings; timed competitor expected to be faster than DESeq2
  repo: https://github.com/Dulab2020/ARMT
- name: DESeq2
  role: Runs differential analysis on the identical counts matrix; expected to have
    the largest wall-clock run time (Fig. 1B)
  repo: https://github.com/Dulab2020/ARMT
- name: R
  role: Environment in which all three packages are run and wall-clock run times are
    recorded
- name: ARMT
  role: Provides the TCGA-derived LUAD counts matrix and the four normalization options
    (TMM, TMMwsp, RLE, upperquartile) used to standardize the benchmark input
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:821a8133753e727f7f69017c29d8de1878768c7370ea66fa428107dbbdd67298
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# edger-limma-deseq2-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Wall-clock runtime benchmark of the three differential analysis tools compared in ARMT (edgeR, limma, DESeq2), run on one identical TCGA LUAD raw counts matrix to show DESeq2 is significantly slower (Fig. 1B).

## When to use

Use when choosing between edgeR, limma and DESeq2 for RNA-seq differential analysis and computational run time matters, or when reproducing the ARMT Fig. 1B speed comparison on a shared counts matrix.

## When NOT to use

- The goal is differential expression results (DEG lists, p-values, logFC) rather than tool speed — this skill measures run time only and does not compare DEG output quality
- The input is already normalized (e.g. TPM or log2(TPM + 1) matrix used for GSVA in ARMT) rather than raw counts — edgeR/DESeq2 differential analysis requires the counts matrix
- Benchmarking other ARMT modules (GSVA, survival analysis, Cox regression, mutant mapping) instead of the three differential analysis tools

## Inputs

- TCGA LUAD raw counts matrix (.csv; gene Ensembl IDs as rows, sample IDs as columns)
- Sample grouping labels defining experiment vs control groups for the differential analysis

## Outputs

- runtime_benchmark.csv with exactly three rows (edgeR, limma, DESeq2), a tool column and a positive numeric runtime column
- Per-tool wall-clock run times on the identical counts matrix

## How to apply

Load the TCGA lung adenocarcinoma (LUAD) raw counts matrix — a .csv with gene Ensembl IDs as row names and sample IDs as column names, the format ARMT requires for differential analysis — into R. Run edgeR, limma and DESeq2 on this identical counts matrix with the same sample grouping and identical settings; for edgeR use the default TMM normalization (one of the four options ARMT provides: TMM, TMMwsp, RLE, upperquartile), and record each tool's wall-clock run time (e.g. R's system.time). The rationale is that ARMT itself employs edgeR 'for high efficiency', and because different statistical principles can lead to different DEG results, all three tools must receive byte-identical input for a fair speed comparison. Assemble the three measured run times into runtime_benchmark.csv with one row per tool and a tool/runtime column structure. Expect DESeq2's run time to be the largest of the three, reproducing the Fig. 1B ordering that DESeq2 is significantly slower than edgeR and limma.

## Related tools

- **edgeR** (Runs differential analysis on the shared LUAD counts matrix with default TMM normalization; the tool ARMT itself employs 'for high efficiency'; timed as one of the three competitors) — https://github.com/Dulab2020/ARMT
- **limma** (Runs differential analysis on the identical counts matrix under identical settings; timed competitor expected to be faster than DESeq2) — https://github.com/Dulab2020/ARMT
- **DESeq2** (Runs differential analysis on the identical counts matrix; expected to have the largest wall-clock run time (Fig. 1B)) — https://github.com/Dulab2020/ARMT
- **R** (Environment in which all three packages are run and wall-clock run times are recorded)
- **ARMT** (Provides the TCGA-derived LUAD counts matrix and the four normalization options (TMM, TMMwsp, RLE, upperquartile) used to standardize the benchmark input) — https://github.com/Dulab2020/ARMT

## Examples

```
Rscript -e 'counts <- read.csv("LUAD_counts.csv", row.names=1); print(system.time({y <- edgeR::DGEList(counts); y <- edgeR::calcNormFactors(y, method="TMM")}))'
```

## Evaluation signals

- runtime_benchmark.csv exists and parses with exactly three rows labeled edgeR, limma and DESeq2
- All three runtime values are positive and numeric
- The DESeq2 runtime is the largest of the three, reproducing the Fig. 1B ordering that DESeq2 is significantly slower than edgeR and limma
- All three tools were run on the identical LUAD counts matrix with the same sample grouping, making the timings comparable
- Runtimes are wall-clock measurements taken in the same R session/hardware environment

## Limitations

- Runtime ordering is hardware- and session-dependent; single-machine wall-clock times do not generalize across environments
- Based on different statistical principles, different tools may lead to different DEG results — the runtime ordering says nothing about biological agreement between edgeR, limma and DESeq2
- No software version numbers for the tools are stated (only R >= 4.0.2), so version effects on run time are unaccounted for

## Evidence

- [results] DESeq2's computation speed is significantly slower than that of edgeR and limma, as shown in the runtime comparison (Fig. 1B).: [evidence span withheld — anchor status: not_found]
