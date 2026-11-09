---
name: blosum62-filtered-peptide-search-parameterization
description: Use when raw RNA-seq FASTQ reads from a non-model eukaryotic species
  must be quantified at the KO level via translated search, and the Seq2Fun search
  thresholds (mismatches, matching length, BLOSUM62 score) need to be chosen or replicated.
when_to_use_negative:
- Input is already a count table or gene list — use the ExpressAnalyst/ExpressAnalystR
  statistical and enrichment modules instead of re-running the translated search.
- The species has a well-annotated reference transcriptome and standard alignment-based
  quantification (e.g., Kallisto with minimum quality score 25) is preferred — the
  KO-based translated search gives partial coverage (only 61.40% of zebrafish protein-coding
  genes carry KOs).
- Target biomarkers outside the KO system (e.g., vitellogenin) — reads cannot be mapped
  to KOs that do not exist in the ortholog database.
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0102
tools:
- name: Seq2Fun 1.0
  role: Command-line tool performing reads QC, paired-end joining, six-frame translation,
    and BLOSUM62-filtered translated search against KO databases; benchmarked at 1.49
    GB peak memory.
- name: Seq2Fun 2.0
  role: Updated Seq2Fun version applying the same search parameters with a reduced
    memory footprint (0.94 GB) despite a database >125x larger; sustains ~2 million
    reads per minute.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:fb7341cd6464eefeed7451ab6028e797bfba3accf7bd24738a5bb246ce9d678a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seq2Fun 1.0
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# blosum62-filtered-peptide-search-parameterization

## Summary

Configures the two-filter translated read search of Seq2Fun — peptide-length filtering of reading frames plus BLOSUM62-score-based seed matching — by setting the maximum number of mismatches, minimum matching length, and minimum matching BLOSUM62 score used to map RNA-seq reads to KEGG ortholog (KO) databases.

## When to use

Use when raw RNA-seq FASTQ reads from a non-model eukaryotic species must be quantified at the KO level via translated search, and the Seq2Fun search thresholds (mismatches, matching length, BLOSUM62 score) need to be chosen or replicated.

## When NOT to use

- Input is already a count table or gene list — use the ExpressAnalyst/ExpressAnalystR statistical and enrichment modules instead of re-running the translated search.
- The species has a well-annotated reference transcriptome and standard alignment-based quantification (e.g., Kallisto with minimum quality score 25) is preferred — the KO-based translated search gives partial coverage (only 61.40% of zebrafish protein-coding genes carry KOs).
- Target biomarkers outside the KO system (e.g., vitellogenin) — reads cannot be mapped to KOs that do not exist in the ortholog database.

## Inputs

- Paired-end RNA-seq FASTQ files (e.g., zebrafish SRA samples SRR13332314–SRR13332325 from study SRP299836)
- KEGG ortholog (KO) protein database for the target taxonomic sub-group (e.g., zebrafish KO database, or EcoOmicsDB sub-group such as 'Fishes')

## Outputs

- Per-sample KO-level read quantification (count tables)
- Per-run runtime and peak-memory log for throughput/memory benchmarking (e.g., ~2 million reads per minute; 1.49 GB v1.0 vs 0.94 GB v2.0)

## How to apply

Seq2Fun first performs reads quality control and joins paired-end reads, then translates each cleaned read into dozens of peptide sequences using all possible reading frames. Apply the two search-space-reducing filters: translated sequences from different reading frames are filtered by length (the 'true' translation yields the longest amino acid sequence), and the search itself is bounded by thresholds — minimum matching scores and maximum number of substitutions. The search starts with a seed of seven amino acids and extends potential matches backwards from the C terminal to the N terminal, stopping when thresholds are met. For the zebrafish benchmark in this paper, set 'maximum number of mismatches' = 2, 'minimum matching length' = 19, and 'minimum matching BLOSUM62 score' = 80, where BLOSUM62 is the widely used scoring matrix summarizing amino acid substitution frequencies in known peptides. Log wall-clock runtime and peak memory per run so throughput and memory can be compared against reported figures (e.g., 1.49 GB for Seq2Fun 1.0 vs 0.94 GB for Seq2Fun 2.0).

## Related tools

- **Seq2Fun 1.0** (Command-line tool performing reads QC, paired-end joining, six-frame translation, and BLOSUM62-filtered translated search against KO databases; benchmarked at 1.49 GB peak memory.)
- **Seq2Fun 2.0** (Updated Seq2Fun version applying the same search parameters with a reduced memory footprint (0.94 GB) despite a database >125x larger; sustains ~2 million reads per minute.)

## Evaluation signals

- All 12 samples (SRR13332314–SRR13332325) processed under both Seq2Fun versions produce time and peak-memory values in the benchmark table.
- Seq2Fun 2.0 peak memory falls near 0.94 GB versus 1.49 GB for version 1.0, matching the reported reduction.
- Measured throughput is near ~2 million reads per minute per run.
- The parameter set exactly matches the paper's values: max mismatches = 2, min matching length = 19, min BLOSUM62 score = 80.
- Functional profiles remain consistent with traditional transcriptome-based workflows, as Seq2Fun outperforms de novo assembly in accuracy, precision, computing time, and memory usage.

## Limitations

- Performance does not improve beyond 16 threads due to an I/O constraint; using a high-speed drive could relieve the bottleneck.
- Reads from the same gene may map to similar but different orthologs across species, requiring care when interpreting taxonomically diverse comparisons.
- Improper sub-group database selection (e.g., protist reads mapped to the 'Fishes' database) may cause key functional groups to be missed.

## Evidence

- [discussion] Search parameters and thresholds: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [discussion] Two-filter search-space reduction: "Seq2Fun employs two filters to greatly reduce the search space: peptide length and BLOSUM62 score"
- [discussion] Search termination thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Seed-based backwards search: "A backwards searching strategy is used to extend potential matches from the C terminal to the N terminal."
- [intro] Purpose of the translated search: "Seq2Fun for mapping RNA-seq reads from eukaryotic species to KEGG ortholog (KO) databases with a translated search"
- [discussion] Memory-footprint update motivating version choice: "We have updated Seq2Fun to version 2.0 to reduce its memory footprint (e.g., from 1.49 GB to 0.94 GB)"
