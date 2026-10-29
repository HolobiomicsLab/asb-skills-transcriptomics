---
name: synthetic-chimera-sequence-construction
description: Use when you need to quantify how well a transcriptome assembly filtering
  pipeline (e.g. Bellerophon's TPM, CD-HIT-EST, TransRate-C steps) removes chimeric
  versus nonchimeric contigs, by planting known synthetic chimeras whose fate can
  be traced.
when_to_use_negative:
- You need to detect chimeras in an unspiked production assembly rather than benchmark
  a pipeline's removal rate with known ground truth
- Your assembly comes from a genome-guided or reference-based workflow where chimeras
  are not the dominant error mode and the spike-in benchmark would not reflect real
  contamination
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: 'Pipeline under test: its TPM filter (cut-off 1), CD-HIT-EST (95% identity),
    and TransRate filtering steps are benchmarked against the planted synthetic chimeras'
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Provides the TransRate-C filtering step whose chimera-removal contribution
    (42.9% in one experiment) is tracked
- name: CD-HIT-EST
  role: Redundancy-removal step (default 95% identity) whose per-step chimera removal
    is recorded during the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:6536aa15f011a447707ffc25b69370cbceb2185cee752ee231ef467453ae79f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# synthetic-chimera-sequence-construction

## Summary

Benchmark chimera-removal performance of transcriptome filtering pipelines by constructing 500 synthetic chimeras — concatenating random pairs of RNA-Seq sequences with a 30–70% overlap — spiking them into a de novo assembly, and tracking their fate through each filtering step.

## When to use

Use when you need to quantify how well a transcriptome assembly filtering pipeline (e.g. Bellerophon's TPM, CD-HIT-EST, TransRate-C steps) removes chimeric versus nonchimeric contigs, by planting known synthetic chimeras whose fate can be traced.

## When NOT to use

- You need to detect chimeras in an unspiked production assembly rather than benchmark a pipeline's removal rate with known ground truth
- Your assembly comes from a genome-guided or reference-based workflow where chimeras are not the dominant error mode and the spike-in benchmark would not reflect real contamination

## Inputs

- RNA-Seq dataset from which sequence pairs are sampled (e.g. H. subflexa pheromone gland Illumina reads, BioProject PRJNA527373)
- De novo transcriptome assembly (FASTA) into which the synthetic chimeras are inserted

## Outputs

- Set of 500 synthetic chimera sequences (per repeat) embedded in a spiked assembly
- Per-filtering-step removal counts/percentages for the 500 planted chimeras (TPM, CD-HIT-EST, TransRate-C)
- Mean non-chimera removal percentage per step across five repeats, with unpaired t test p values (Bonferroni-adjusted)

## How to apply

Randomly select pairs of sequences from the RNA-Seq dataset (e.g. the H. subflexa pheromone gland RNA-Seq reads) and concatenate each pair with a random 30–70% overlap to build 500 synthetic chimeras, then insert them into the de novo assembly; repeat the whole process five times. Run the pipeline under test on each chimera-spiked assembly with default settings (TPM filter cut-off of 1, CD-HIT-EST at 95% identity, TransRate filtering, in the default order TPM then CD-HIT-EST). At each filtering step, record how many of the 500 planted chimeras are removed (via BLAST against reference transcripts: a contig is chimeric when it has two non-overlapping matches to different genes). Compute the mean percentage of non-chimeric sequences removed by each corresponding step across the five repeats, then compare per-step chimera removal against non-chimera removal with an unpaired t test followed by a Bonferroni correction.

## Related tools

- **Bellerophon** (Pipeline under test: its TPM filter (cut-off 1), CD-HIT-EST (95% identity), and TransRate filtering steps are benchmarked against the planted synthetic chimeras) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Provides the TransRate-C filtering step whose chimera-removal contribution (42.9% in one experiment) is tracked)
- **CD-HIT-EST** (Redundancy-removal step (default 95% identity) whose per-step chimera removal is recorded during the benchmark)

## Examples

```
python Bellerophon.py --assembly spiked_assembly.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Overall chimera removal across the pipeline should reach approximately 97% (about 485 ± 3.06 of 500 chimeras) versus roughly 70% removal of other sequences (adjusted p = 5.83 × 10−6)
- The TPM step alone should discard about 96% of planted chimeras (95.72%) versus 60.77% of non-chimeric sequences
- Each planted chimera must contain a 30–70% overlap between its two concatenated source sequences
- The spike-in-and-measure procedure must be repeated five times so that per-step percentages can be compared with an unpaired t test after Bonferroni correction
- Chimera identification must follow the rule: two non-overlapping mapping-region matches to different genes via BLAST against reference transcripts

## Limitations

- Contigs representing alternative splicing of one gene may be misclassified as chimeras by TransRate when they are unevenly expressed along their sequence
- Custom ordering of filtering steps is disabled in the current Bellerophon version, restricting which step orders can be benchmarked
- In the simulated D. melanogaster RNA-Seq experiment only 46.1% of chimeras were removed, so results do not transfer unconditionally to other datasets

## Evidence

- [methods] a set of 500 chimeras was created by randomly selecting two sequences from the H. subflexa RNA-Seq dataset.: [evidence span withheld — anchor status: not_found]
- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [other] Run the Bellerophon pipeline on each chimeras-spiked assembly with default filtering: TPM filter with cut-off score of 1: [evidence span withheld — anchor status: not_found]
- [results] Bellerophon removed 485 ± 3.06 of 500 computationally created chimeras (97.04% ± 0.61): [evidence span withheld — anchor status: not_found]
- [abstract] the Bellerophon pipeline was able to remove between 40% and 91.9% of the chimeras in transcriptome assemblies: "the Bellerophon pipeline was able to remove between 40% and 91.9% of the chimeras in transcriptome assemblies"
