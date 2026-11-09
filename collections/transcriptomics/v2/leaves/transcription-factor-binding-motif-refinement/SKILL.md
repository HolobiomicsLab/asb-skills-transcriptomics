---
name: transcription-factor-binding-motif-refinement
description: Use when an initial MI-based GRN (e.g., from RNA-Seq time series) should
  be filtered or upgraded with motif evidence from upstream promoter sequences, or
  when reproducing Augusta's 'with refinement' benchmark.
when_to_use_negative:
- The organism's genome is not available in annotated form (e.g., no GenBank file),
  since refining steps cannot be performed without additional knowledge of a genome
- A quick, low-cost network estimate is the goal and the initial unrefined MI-based
  GRN already has adequate sensitivity — refinement is computationally very demanding
- The target is Boolean logic rule assignment rather than GRN edge refinement (use
  the Cell Collective database search / GRN_to_BN path instead)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Augusta
  role: Python package that performs de novo TFBM-based GRN refinement via the refineGRN
    function
  repo: https://github.com/JanaMus/Augusta
- name: MEME Suite
  role: De novo transcription factor binding motif discovery on upstream promoter
    sequences of regulons
- name: OmniPath
  role: Manually curated database searched in the second refinement step to incorporate
    verified interactions
- name: Signor
  role: Manually curated interaction database used for refinement
- name: SignaLink
  role: Manually curated interaction database used for refinement
- name: TRRUST
  role: Manually curated interaction database used for refinement
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:1336be7886a39959bc756005740ed12d3de787ad6af4dd601af1952f424cbe8d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - GENIE3
  - KBoost
  - MEME Suite
  - OmniPath
  - Python
  - SignaLink
  - Signor
  - TDAracne
  - TRRUST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# transcription-factor-binding-motif-refinement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines an initially inferred gene regulatory network by de novo transcription factor binding motif (TFBM) discovery in the promoters of regulated genes, as implemented in the Augusta pipeline to boost network accuracy beyond mutual-information-only inference.

## When to use

Use when an initial MI-based GRN (e.g., from RNA-Seq time series) should be filtered or upgraded with motif evidence from upstream promoter sequences, or when reproducing Augusta's 'with refinement' benchmark.

## When NOT to use

- The organism's genome is not available in annotated form (e.g., no GenBank file), since refining steps cannot be performed without additional knowledge of a genome
- A quick, low-cost network estimate is the goal and the initial unrefined MI-based GRN already has adequate sensitivity — refinement is computationally very demanding
- The target is Boolean logic rule assignment rather than GRN edge refinement (use the Cell Collective database search / GRN_to_BN path instead)

## Inputs

- Initial GRN edge list (CSV) inferred via mutual information from the expression matrix
- Annotated genome in GenBank file format (e.g., AL009126.3 for B. subtilis 168) to extract 1000 bp upstream promoter sequences
- RPKM-normalized RNA-Seq count table (CSV), e.g., the B. subtilis 168 time series with 3997 genes in 7 samples
- Promoter length parameter (e.g., 1000 bp) and motifs_max_time limit for the MEME-based de novo motif search

## Outputs

- Refined GRN (CSV edge list) with edges supported or pruned by de novo TFBM evidence
- Optionally a further database-refined GRN after search against OmniPath, Signor, SignaLink, and TRRUST
- Benchmark metrics (sensitivity, specificity, accuracy, execution time) comparable to the unrefined baseline, e.g., bsubtilis_benchmark_results.csv

## How to apply

First infer the initial GRN by mutual information (Augusta sets the number of bins as D = min(n/5, 10) and orients/edges signs edges via the most significant difference among adjacent time-point pairs). Then run the refinement step: perform de novo TFBM discovery on regulons of at least 5 genes using 1000 bp upstream promoter sequences extracted from the annotated GenBank genome (Augusta uses the MEME Suite for motif discovery). In Augusta this is exposed as Augusta.refineGRN(GRN_input, genbank_file_input, count_table_input, promoter_length, motifs_max_time), where promoter_length controls upstream sequence length and motifs_max_time bounds motif-search runtime. Optionally follow TFBM discovery with a curated database search (OmniPath, Signor, SignaLink, TRRUST) as the second refinement step; the rationale is that motif and database evidence prune or confirm MI-inferred edges, raising accuracy and specificity. Finally, evaluate the refined network against a gold standard (e.g., 3x3 confusion matrix over positive/negative/absent edges) and compare against the unrefined run to confirm refinement improved accuracy.

## Related tools

- **Augusta** (Python package that performs de novo TFBM-based GRN refinement via the refineGRN function) — https://github.com/JanaMus/Augusta
- **MEME Suite** (De novo transcription factor binding motif discovery on upstream promoter sequences of regulons)
- **OmniPath** (Manually curated database searched in the second refinement step to incorporate verified interactions)
- **Signor** (Manually curated interaction database used for refinement)
- **SignaLink** (Manually curated interaction database used for refinement)
- **TRRUST** (Manually curated interaction database used for refinement)

## Examples

```
import Augusta; Augusta.refineGRN(GRN_input='MyGRN_file.csv', genbank_file_input='MyGB_file.gb', count_table_input='MyCT_file.csv', promoter_length=1000, motifs_max_time=3600)
```

## Evaluation signals

- Refined network accuracy exceeds the MI-only run: on DREAM 4, Augusta with refinement scored sensitivity 0.06, specificity 0.91, accuracy 0.89 vs. 0.38/0.54/0.54 without refinement
- On the B. subtilis whole-genome benchmark, accuracy with refinement should be even higher than the unrefined Augusta run that already beat GENIE3 and KBoost
- Promoter extraction lengths match the configured promoter_length (1000 bp upstream) and only regulons of at least 5 genes undergo de novo TFBM discovery
- The motif search terminates within motifs_max_time seconds, and per-step execution times are recorded
- Metrics and runtimes written to the results CSV (e.g., bsubtilis_benchmark_results.csv) match the paper's benchmark table values

## Limitations

- The refinement of an initial GRN may be computationally very demanding
- Refining steps cannot be performed without additional knowledge of a genome (annotated GenBank sequence is required)
- Inferred networks are approximations, not perfectly accurate representations of biological systems' regulations
- Parallelization to reduce computational time of refinement is not yet implemented

## Evidence

- [readme] RefineGRN invocation: [evidence span withheld — anchor status: external_doc]
