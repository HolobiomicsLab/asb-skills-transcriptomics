---
name: alternative-splicing-event-identification-from-junction-reads
description: Use when starting from single-cell SMART-seq BAM or junction (SJ.out.tab)
  files and you need de novo alternative-splicing events and a per-cell AS probability
  matrix without relying on transcriptome annotation.
when_to_use_negative:
- You already have an AS probability / feature matrix — instead use the demo intermediate
  files path (data/junction) or downstream imputation and clustering stages directly
- Input is bulk RNA-seq or requires annotation-dependent PSI estimation (e.g. rMATS-style
  exon inclusion) — SCASL targets single-cell SMART-seq junction reads and de novo,
  annotation-free event definition
- Cell labels are known and the goal is differential splicing between labeled groups
  — tools like BRIE, Expedition, rMATS, or MARVEL address differential analysis rather
  than unsupervised AS-based clustering
edam_operation: http://edamontology.org/operation_0262
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: SCASL
  role: 'Main pipeline: integrates junction reads into the matrix, performs QC and
    AS grouping, and computes the AS probability matrix'
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Leafcutter-style de novo identification of AS events from junction reads;
    installable via conda r-leafcutter or cloned source added to PATH
  repo: https://github.com/davidaknowles/leafcutter
- name: STAR
  role: Optional alternative source of junction reads via automatically generated
    SJ.out.tab files passed as the junction input
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:1714c80dd453c62f8e450d7aceaaf7b55fff316848ea4b358efed561121f7128
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  - STAR
  edam_operation: http://edamontology.org/operation_0262
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# alternative-splicing-event-identification-from-junction-reads

## Summary

SCASL's first pipeline stage: Leafcutter-style de novo grouping of splicing schemes sharing a common 5' or 3' splice site into AS events from single-cell SMART-seq junction reads, followed by QC and conversion of split-read counts into a cell-by-site AS probability matrix (normalized_matrix).

## When to use

Use when starting from single-cell SMART-seq BAM or junction (SJ.out.tab) files and you need de novo alternative-splicing events and a per-cell AS probability matrix without relying on transcriptome annotation.

## When NOT to use

- You already have an AS probability / feature matrix — instead use the demo intermediate files path (data/junction) or downstream imputation and clustering stages directly
- Input is bulk RNA-seq or requires annotation-dependent PSI estimation (e.g. rMATS-style exon inclusion) — SCASL targets single-cell SMART-seq junction reads and de novo, annotation-free event definition
- Cell labels are known and the goal is differential splicing between labeled groups — tools like BRIE, Expedition, rMATS, or MARVEL address differential analysis rather than unsupervised AS-based clustering

## Inputs

- Per-cell BAM files from single-cell SMART-seq data (any aligner) or junction files (STAR SJ.out.tab)
- Leafcutter installation (conda r-leafcutter or cloned source on PATH)
- configs/srr.yaml with bam, lc, junction, sites_initial, runs_initial, sites_quality, runs_quality, log_scale_histogram, output_path

## Outputs

- junc_matrix.csv / junc_mat.npy — raw junction reads matrix (rows = sites, columns = cells)
- filtered_matrix_start / filtered_matrix_end — QC-filtered junction matrices grouped by upstream/downstream splice site
- normalized_matrix — final AS probability matrix (rows = sites, columns = cells)
- site_hist.png — log-scale histograms of non-NA counts to guide QC threshold selection

## How to apply

Configure configs/srr.yaml with the bam (or junction) and leafcutter (lc) directories plus the four QC thresholds — sites_initial, runs_initial (junction-matrix QC), sites_quality, runs_quality (AS-matrix QC) — then run 'python main.py -y configs/srr.yaml'. First integrate junction reads across cells Leafcutter-style into a raw junction reads matrix (junc_matrix.csv / junc_mat.npy, rows = sites, columns = cells). Apply initial QC keeping sites expressed in at least sites_initial cells and cells expressing at least runs_initial sites, optionally emitting a log-scale histogram to guide threshold choice. Group quality-filtered junctions sharing a splice site into AS events, yielding the upstream-grouped (filtered_matrix_start) and downstream-grouped (filtered_matrix_end) matrices, then compute AS probabilities from proportions of junction reads assigned to each 5' or 3' AS event, filter by sites_quality and runs_quality, and normalize to produce the final AS probability matrix normalized_matrix. STAR SJ.out.tab files may be substituted for Leafcutter-derived junction files by pointing the junction path in configs/srr.yaml.

## Related tools

- **SCASL** (Main pipeline: integrates junction reads into the matrix, performs QC and AS grouping, and computes the AS probability matrix) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Leafcutter-style de novo identification of AS events from junction reads; installable via conda r-leafcutter or cloned source added to PATH) — https://github.com/davidaknowles/leafcutter
- **STAR** (Optional alternative source of junction reads via automatically generated SJ.out.tab files passed as the junction input)

## Examples

```
source activate scasl && python main.py -y configs/srr.yaml
```

## Evaluation signals

- normalized_matrix exists with rows corresponding to sites and columns to cells
- QC-filtered intermediates filtered_matrix_start and filtered_matrix_end are generated for upstream and downstream AS grouping
- junc_matrix.csv / junc_mat.npy contain the raw junction counts integrated across all input cells before filtering
- site_hist.png histograms of non-NA counts per site and per sample are emitted to justify the chosen sites_initial/runs_initial and sites_quality/runs_quality thresholds
- On the TNBC-2 demo intermediate files in data/junction, the full run completes in under 3 minutes writing results to process_result_demo

## Limitations

- Single-cell AS profiles suffer heavily from missing data points (dropouts) and unbalanced coverage of AS events across cells, hence the QC thresholds and downstream imputation
- Many canonical clustering methods are unsuitable for the largely polarized AS probability profiles this stage produces, requiring the imputation/concatenation-with-NA-indicator strategy downstream
- Demo test files are minimal (GitHub is unsuitable for large data); results may vary between runs due to randomness in imputation, dimensionality reduction, and unsupervised clustering

## Evidence

- [intro] SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data: "SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data"
- [intro] AS probabilities inferred from junction read proportions per 5'/3' AS event: "AS probabilities are then inferred from proportions of the junction reads assigned to the same 5’ AS or 3’ AS events."
- [readme] QC parameter definitions from README: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site."
- [readme] Intermediate file description from README: "normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [results] Finding from source card context on event grouping: [evidence span withheld — anchor status: not_found]
