---
name: de-novo-transcriptome-filtering-order-benchmarking
description: Use when a de novo transcriptome assembly must be filtered and you need
  to empirically decide which combination and order of TPM, CD-HIT-EST, TransRate-C,
  and ORF-length filters maximizes quality or chimera removal.
when_to_use_negative:
- Input already has a reference genome/transcriptome — reference-based assessment
  methods are more appropriate than this reference-free filtering benchmark
- The assembly has not been quality-scored first with TransRate/BUSCO — Bellerophon
  requires a ground quality score of the unfiltered assembly before filtering variants
  can be compared
- You need custom filter ordering out of the box — the README states custom order
  of filtering steps is disabled in this version
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: Pipeline executing the configurable filter components (TPM, CD-HIT-EST, TransRate-C,
    ORF length) in alternative orders
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Reference-free quality scoring of each filtered assembly variant; also provides
    TransRate-C filtering
- name: CD-HIT-EST
  role: Removes highly identical redundant contigs at default 95% identity
- name: BUSCO
  role: Completeness context for the ground quality score of unfiltered and filtered
    assemblies
- name: BLASTX
  role: Marks chimeric contigs against NCBI NR (E-value 1e-4, two non-overlapping
    hits to different genes) before and after each filter step
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:71fe5b60317b59c7005b3befdc916bbe1f90644693862b5022c45c5908906590
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTX
  - BUSCO
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# de-novo-transcriptome-filtering-order-benchmarking

## Summary

Systematically varies the composition and ordering of Bellerophon's filter components (TPM cut-off 1.0, CD-HIT-EST 95% identity, TransRate-C, ORF length 50/100 aa) and scores each variant with TransRate to identify the optimal de novo transcriptome filtering strategy.

## When to use

Use when a de novo transcriptome assembly must be filtered and you need to empirically decide which combination and order of TPM, CD-HIT-EST, TransRate-C, and ORF-length filters maximizes quality or chimera removal.

## When NOT to use

- Input already has a reference genome/transcriptome — reference-based assessment methods are more appropriate than this reference-free filtering benchmark
- The assembly has not been quality-scored first with TransRate/BUSCO — Bellerophon requires a ground quality score of the unfiltered assembly before filtering variants can be compared
- You need custom filter ordering out of the box — the README states custom order of filtering steps is disabled in this version

## Inputs

- De novo transcriptome assembly in FASTA format (e.g. Trinity assembly of H. subflexa pheromone gland RNA-Seq)
- Paired-end RNA-Seq reads (FASTQ, e.g. Illumina SRR8735410-5412 from BioProject PRJNA527373)
- BLASTX chimera annotations against NCBI NR (E-value 1e-4, two non-overlapping hits to different genes)
- Ortholog set for completeness context (e.g. Insecta odb9)

## Outputs

- Per-condition TransRate quality scores (collated as filter_order_scores.csv)
- Remaining contig counts per filter condition (e.g. 12,096 contigs for CD-HIT-EST-only; 16,057 for CD-HIT+TPM+ORF)
- Chimera counts remaining after each filtering step (e.g. 74 → 18 under the TransRate-C-before-RSEM order)
- A benchmark table ranking filter orders by TransRate score (Figure 2-style, default order 12)

## How to apply

First run TransRate (plus BUSCO with e.g. the Insecta odb9 ortholog set) on the unfiltered assembly and reads to establish a ground quality score. Then apply each candidate filter condition — e.g. CD-HIT-EST alone at 95% identity; the combined CD-HIT-EST (0.95), TPM (cut-off 1.0) and ORF-length (50 or 100 aa) filters; or the default Bellerophon order of TPM (1.0) first then CD-HIT-EST (0.95) — scoring every filtered assembly with TransRate and collating results into a filter_order_scores table. To test chimera-specific effects, mark chimeras beforehand by BLASTX against the NCBI NR database (E-value cut-off 1e-4), flagging contigs with two non-overlapping matches to different genes, and recount marked chimeras after each filter step. Rationale for the default order comes from the benchmark: ORF-length filtering removed too many transcripts, so the default was set to TPM filtering (cut-off 1.0) followed by CD-HIT-EST (order 12 in Figure 2). Note that reordering TransRate-C before RSEM-based TPM filtering can strongly reduce chimeric isoforms (74 → 18 in the isoform-rich H. subflexa benchmark), but custom filter order is disabled in the current Bellerophon release, so such experiments require a modified pipeline.

## Related tools

- **Bellerophon** (Pipeline executing the configurable filter components (TPM, CD-HIT-EST, TransRate-C, ORF length) in alternative orders) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Reference-free quality scoring of each filtered assembly variant; also provides TransRate-C filtering)
- **CD-HIT-EST** (Removes highly identical redundant contigs at default 95% identity)
- **BUSCO** (Completeness context for the ground quality score of unfiltered and filtered assemblies)
- **BLASTX** (Marks chimeric contigs against NCBI NR (E-value 1e-4, two non-overlapping hits to different genes) before and after each filter step)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- The CD-HIT-EST-only filter (95% identity) yields the highest TransRate score ≈0.319 (Figure 2a, experiment 3)
- The combined CD-HIT + TPM (1.0) + ORF-length (50/100 aa) filter scores ≈0.316 while leaving ≈16,057 contigs (82.9% / 77,602 contigs removed)
- The default order — TPM (cut-off 1.0) first, then CD-HIT-EST — corresponds to filtering order 12 in Figure 2
- Under the TransRate-C-before-RSEM variant, chimeric isoforms drop from 74 to 18 (56 removed) in the isoform-rich contig groups
- Every condition is scored with TransRate using the same reads, so scores across conditions are directly comparable

## Limitations

- Custom order of filtering steps is disabled in the current Bellerophon version, preventing TransRate-C-before-TPM experiments without modification
- ORF-length filtering appeared to remove too many transcripts, biasing combined filters toward lower scores
- Bellerophon may erroneously remove contigs from alternative splicing that TransRate misclassifies as chimeras when expression is uneven along the sequence
- No tool version numbers are stated for BUSCO, TransRate, CD-HIT-EST, Trinity, or Polyester; the Vagrant VM is recommended to avoid backwards-compatibility problems with TransRate
- The Vagrant VM disk size is not large enough for most full transcriptome analyses — outputs should go to /vagrant/

## Evidence

- [results] Highest TransRate score from CD-HIT-EST-only filter: [evidence span withheld — anchor status: not_found]
- [results] Default filter order and ORF-length caveat: [evidence span withheld — anchor status: not_found]
- [results] Combined filter removes 82.9% of contigs: "this filter removed 77,602 (82.9%) contigs, leaving the set with only 16,057 contigs."
- [readme] Custom ordering disabled: "Custom order of filtering steps is disabled in this version"
- [methods] Ground quality score before filtering: "(a) Busco and Transrate are used to establish a ground quality score of the unfiltered assembly."
- [discussion] Alternative-splicing contigs misclassified as chimeras: "Contigs representative of transcripts arising from alternative splicing of one gene might be considered as chimera by Transrate if they are unevenly expressed along their sequence"
