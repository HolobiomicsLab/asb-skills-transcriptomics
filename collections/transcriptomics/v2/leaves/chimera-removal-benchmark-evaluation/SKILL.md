---
name: chimera-removal-benchmark-evaluation
description: Use when a de novo transcriptome assembly pipeline's chimera-removal
  performance must be quantified against a reference where chimeras and correct contigs
  are already classifiable.
when_to_use_negative:
- You need to filter a real (non-simulated) transcriptome where no reference exists
  to classify contigs as chimeric or correct — this skill is only the benchmarking/evaluation
  protocol, not the filtering itself
- The assembly is genome-guided rather than de novo; the paper's chimera problem and
  Bellerophon's scope target de novo assembled transcriptomes
- You require a chimera-removal method with per-step custom filter ordering; the README
  states custom order of filtering steps is disabled in this version
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: Pipeline under evaluation; performs TPM filtering (cut-off 1), CD-HIT-EST
    (95% identity), and TransRate-based filtering of the Trinity assembly
  repo: github.com/JesseKerkvliet/Bellerophon
- name: Polyester
  role: Simulates paired-end RNA-Seq reads from the 3,000-transcript D. melanogaster
    reference using 'fpkm_to_counts', 'create_read_numbers', and 'simulate_experiment_countmat'
- name: Trinity
  role: De novo assembles the simulated reads (default parameters) to produce the
    contigs that are benchmarked
- name: BLASTn
  role: Classifies contigs as chimeric (>1 non-overlapping match to different reference
    genes) or correct (exactly 1 match) with 90% identity and E-value 10 cut-offs
- name: TransRate
  role: Quality assessment and final TransRate-C filtering step within Bellerophon;
    removed 792 (42.9%) chimeras in the benchmark
- name: CD-HIT-EST
  role: Redundant-contig filtering at 95% identity within Bellerophon; removed 600
    (24.5%) of chimeras still present at that stage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:175bbd3ae441b412f3bb5c33d3c2b8bf44626e9469cf802dd587c72cabf80321
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Bellerophon
  - CD-HIT-EST
  - Polyester
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# chimera-removal-benchmark-evaluation

## Summary

Benchmark how many chimeric versus correctly assembled contigs the Bellerophon pipeline removes, using a simulated D. melanogaster RNA-Seq experiment with a known reference transcriptome as ground truth.

## When to use

Use when a de novo transcriptome assembly pipeline's chimera-removal performance must be quantified against a reference where chimeras and correct contigs are already classifiable.

## When NOT to use

- You need to filter a real (non-simulated) transcriptome where no reference exists to classify contigs as chimeric or correct — this skill is only the benchmarking/evaluation protocol, not the filtering itself
- The assembly is genome-guided rather than de novo; the paper's chimera problem and Bellerophon's scope target de novo assembled transcriptomes
- You require a chimera-removal method with per-step custom filter ordering; the README states custom order of filtering steps is disabled in this version

## Inputs

- Reference transcriptome of 3,000 randomly selected D. melanogaster transcripts from NCBI RefSeq GCF_000001215.4
- Expression profile of the H. subflexa female pheromone gland assembly (FPKM values) for Polyester count simulation
- Three simulated paired-end FASTQ read sets (20,370,192; 19,995,866; 20,045,180 reads)
- Trinity de novo assembly of the simulated reads (contigs with >= 5 read support)
- BLASTn classification of contigs as chimeric (matches >1 reference transcript) or correct (matches exactly 1)

## Outputs

- Bellerophon-filtered transcriptome after TPM (cut-off 1), CD-HIT-EST (95% identity), and TransRate filtering
- Counts of chimeras and correct sequences removed at each filtering step (expected: 136/295 chimeras, 46.1%; 575/3,283 correct, 17.5%)
- Output table of chimera-removal and correct-sequence-removal rates for the benchmark

## How to apply

Simulate reads with Polyester from 3,000 transcripts randomly selected from the D. melanogaster NCBI RefSeq GCF_000001215.4 annotation, using the H. subflexa female pheromone gland expression profile as input ('fpkm_to_counts', 'create_read_numbers', 'simulate_experiment_countmat') to generate three paired-end read sets (20,370,192; 19,995,866; 20,045,180 reads). Assemble the reads de novo with Trinity using default parameters and discard contigs supported by fewer than 5 reads. Classify contigs as chimeric or non-chimeric by BLASTn against the reference D. melanogaster transcriptome with a 90% percent-identity cut-off and E-value cut-off of 10: contigs matching more than one reference transcript are chimeras, contigs matching exactly one are correct. Run Bellerophon with default settings (TPM filter cut-off 1, CD-HIT-EST 95% identity, TransRate filtering step) and recount which BLASTn-classified chimeras and correct sequences survive each filtering step (TPM, CD-HIT-EST, TransRate-C). The expected benchmark outcome is removal of 136 of 295 chimeras (46.1%) versus 575 of 3,283 correct sequences (17.5%), demonstrating preferential removal of chimeras over non-chimeric contigs.

## Related tools

- **Bellerophon** (Pipeline under evaluation; performs TPM filtering (cut-off 1), CD-HIT-EST (95% identity), and TransRate-based filtering of the Trinity assembly) — github.com/JesseKerkvliet/Bellerophon
- **Polyester** (Simulates paired-end RNA-Seq reads from the 3,000-transcript D. melanogaster reference using 'fpkm_to_counts', 'create_read_numbers', and 'simulate_experiment_countmat')
- **Trinity** (De novo assembles the simulated reads (default parameters) to produce the contigs that are benchmarked)
- **BLASTn** (Classifies contigs as chimeric (>1 non-overlapping match to different reference genes) or correct (exactly 1 match) with 90% identity and E-value 10 cut-offs)
- **TransRate** (Quality assessment and final TransRate-C filtering step within Bellerophon; removed 792 (42.9%) chimeras in the benchmark)
- **CD-HIT-EST** (Redundant-contig filtering at 95% identity within Bellerophon; removed 600 (24.5%) of chimeras still present at that stage)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- Reproduce the headline benchmark: Bellerophon removes 136 of 295 chimeras (46.1%) and 575 of 3,283 correct sequences (17.5%)
- Per-step invariant: TPM filtering first removed 227 (8.5%) chimeras, CD-HIT-EST removed 600 (24.5%) of remaining chimeras, TransRate-C removed 792 (42.9%) chimeras
- Chimera-removal proportion must exceed correct-sequence-removal proportion (46.1% > 17.5%), confirming preferential chimera removal
- BLASTn classification sanity check: every contig either matches more than one reference transcript (chimera) or exactly one (correct), at 90% identity / E-value 10
- Assembly input check: contigs supported by fewer than 5 reads must be discarded before classification and filtering

## Limitations

- Contigs representative of alternatively spliced transcripts of one gene might be misclassified as chimeras by TransRate if unevenly expressed along their sequence
- No tool version numbers stated for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester; README warns of backwards-compatibility problems with TransRate and recommends the Vagrant VM
- Custom order of filtering steps is disabled in the current version, so the per-step attribution cannot be reordered experimentally

## Evidence

- [results] Benchmark counts: TPM step: "the TPM filtering step first removed 227 (8.5%) of the chimeras"
- [methods] TPM default threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmark result in discussion: "46.1% of the chimeras in the simulated D. melanogaster RNA‐Seq experiment"
- [readme] Pipeline purpose from README: "Bellerophon is a pipeline created to remove falsely assembled chimeric transcripts in de novo transcriptome assemblies."
- [methods] Ground-truth classification rationale: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
