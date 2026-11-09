---
name: random-exonic-snv-simulation
description: Use when you need to quantify how much of the scRNA-seq SNV recovery
  rate is due to read depth vs. caller bias, by simulating known variants in aligned
  reads.
when_to_use_negative:
- You need realistic somatic mutation spectra rather than uniform random exonic variants
  — random placement does not model mutation hotspots or allele-specific coverage
- Your data are bulk DNA-seq or whole-exome with deep, near-complete coverage; the
  depth-stratified TPR benchmark targets sparse single-cell transcriptome data
- You only need eeSNV ranking/feature selection — that is handled downstream by SSrGE's
  LASSO model, not by the simulation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3569
tools:
- name: SSrGE
  role: Provides the SNV calling docker pipeline (opoirion/ssrge) whose process_snv
    step seeds BAMs with random exonic SNVs and calls variants at each depth
  repo: https://github.com/lanagarmire/SSrGE.git
- name: GATK
  role: SNV caller whose true/false positive rates against the simulated ground truth
    are being benchmarked
- name: STAR
  role: Aligns single-cell reads to the hg19 index (read_length 100) to produce the
    BAMs that receive simulated SNVs
- name: Python 2.7
  role: Runs the SSrGE pipeline scripts (download_sra, extract_sra, process_star,
    process_snv) and the TPR/FPR depth-bin computation
  repo: https://github.com/lanagarmire/SSrGE.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:d26d42315d8bba18d6f466d89fd84f779fb5da036aa3dfd48a9dd8f62a8929db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - GATK
  - Python
  - Python 2.7
  - SSrGE
  - STAR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# random-exonic-snv-simulation

## Summary

Benchmark a GATK-based SNV calling pipeline for scRNA-seq by seeding each single-cell BAM with 50,000 randomly placed exonic SNVs as ground-truth variants, then computing true/false positive rates as a function of read depth.

## When to use

Use when you need to quantify how much of the scRNA-seq SNV recovery rate is due to read depth vs. caller bias, by simulating known variants in aligned reads.

## When NOT to use

- You need realistic somatic mutation spectra rather than uniform random exonic variants — random placement does not model mutation hotspots or allele-specific coverage
- Your data are bulk DNA-seq or whole-exome with deep, near-complete coverage; the depth-stratified TPR benchmark targets sparse single-cell transcriptome data
- You only need eeSNV ranking/feature selection — that is handled downstream by SSrGE's LASSO model, not by the simulation

## Inputs

- Aligned single-cell BAM files (STAR alignment to hg19, e.g. 20 single cells from the Kim dataset)
- hg19 reference genome with STAR index (read_length 100, cell_type HUMAN)
- Exonic region definitions from the hg19 GTF annotation (for restricting random SNV placement)

## Outputs

- Ground-truth variant sets: 50,000 randomly placed exonic SNVs seeded per single-cell BAM
- Per-depth-bin true positive rate and false positive rate tables for the SNV caller
- Validation report comparing measured TPR/FPR against the reference values (TPR >50% at depth 4, ~68% TPR and median FPR <0.05 at depth >6)

## How to apply

Align single-cell reads (e.g., the Kim dataset, 20 cells, hg19, read_length 100) with STAR, then use the SSrGE process_snv step (cell_type HUMAN) to seed each BAM with 50,000 SNVs placed randomly in exonic regions; these become the ground truth. Call SNVs at each read depth and bin recovery by depth, computing per-bin true positive rate and false positive rate. Rationale: scRNA-seq coverage is sparse and incomplete, so TPR must be stratified by depth rather than reported globally. Accept the simulation as reproducing the paper's benchmark when the measured TPR exceeds 50% at read depth 4, reaches ~68% at depth >6, and median FPR stays below 0.05 at depth >6 (Fig. 1a, b).

## Related tools

- **SSrGE** (Provides the SNV calling docker pipeline (opoirion/ssrge) whose process_snv step seeds BAMs with random exonic SNVs and calls variants at each depth) — https://github.com/lanagarmire/SSrGE.git
- **GATK** (SNV caller whose true/false positive rates against the simulated ground truth are being benchmarked)
- **STAR** (Aligns single-cell reads to the hg19 index (read_length 100) to produce the BAMs that receive simulated SNVs)
- **Python 2.7** (Runs the SSrGE pipeline scripts (download_sra, extract_sra, process_star, process_snv) and the TPR/FPR depth-bin computation) — https://github.com/lanagarmire/SSrGE.git

## Examples

```
docker run --rm -v $PATHDATA:/data/results/:Z opoirion/ssrge process_snv -project_name kim_20cells -cell_type HUMAN
```

## Evaluation signals

- Measured TPR exceeds 50% at read depth 4, matching the reported value
- Measured TPR reaches ~68% aggregated at depth >6 (Fig. 1b)
- FPR stays consistently <0.1, with median below 0.05 at depth >6 (Fig. 1a)
- Exactly 50,000 ground-truth SNVs were seeded per BAM and all are exonic
- Per-depth recovery is stratified by depth bin rather than reported as a single global rate

## Limitations

- SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads
- Randomly placed SNVs do not capture real mutational patterns, hotspots, or correlated coverage along transcripts
- Results depend on read length (100 bp) and the hg19/human configuration; other genomes or read lengths need the STAR index rebuilt

## Evidence

- [results] Seeding each BAM with 50,000 randomly placed exonic SNVs as ground-truth variants: [evidence span withheld — anchor status: not_found]
- [results] TPR thresholds by depth: [evidence span withheld — anchor status: not_found]
- [results] FPR bound: [evidence span withheld — anchor status: not_found]
- [discussion] Incomplete coverage caveat: "SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads"
- [readme] Pipeline module: "SNV_calling](https://github.com/lanagarmire/SSrGE/blob/master/README_snv_calling.md): align reads/infer SNVs and infer gene expression matrices from .fastq files"
