---
name: benchmark-sensitivity-precision-f1-computation
description: Use when comparing fcircRNA or fusion-junction detection tools against
  a simulated ground truth and you need sensitivity, precision, and F1 per tool to
  demonstrate ity (e.g., INTEGRATE-Circ 87.3% sensitivity vs Fcirc 57.1%).
when_to_use_negative:
- The data are real (non-simulated) tumors or cell lines with no ground-truth fcircRNA
  set — sensitivity and precision cannot be computed without known backsplices; use
  orthogonal RT-PCR validation instead
- The goal is detecting fcircRNAs in a new dataset rather than comparing tools' accuracy
  — run INTEGRATE-Circ directly rather than building a benchmark table
- Evaluating linear gene-fusion (non-circular) detection only, where fcircRNA-specific
  backsplice matching to ground truth does not apply
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0085
tools:
- name: INTEGRATE-Circ
  role: Primary tool under benchmark; run with default parameters (-cfn 3, -minW 2.0,
    -bacc 1) on each simulated iteration to produce fcirc.txt calls
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: Fcirc
  role: Comparator tool; run on the identical simulated dataset using its published
    fusion call list of 773 fusions
- name: Acfs
  role: Secondary comparator; run alongside CircFusion's published 305-fusion list
    (reported no fcircRNAs on simulated data)
- name: cmake
  role: Build dependency (>= 2.8) for compiling INTEGRATE-Circ before benchmarking
- name: g++
  role: Compiler required to build INTEGRATE-Circ (version >12 recommended)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:81968905146ce71c8b9b0a22948a9657b0af1a977141c46d834f43ceb46493f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Acfs
  - Fcirc
  - INTEGRATE-Circ
  - cmake
  - g++
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# benchmark-sensitivity-precision-f1-computation

## Summary

Benchmark fcircRNA detection tools on simulated RNA-Seq data by matching each tool's backsplice calls to ground-truth fcircRNAs and computing sensitivity, precision, and F1 per tool across all iterations, as done for INTEGRATE-Circ vs Fcirc/Acfs.

## When to use

Use when comparing fcircRNA or fusion-junction detection tools against a simulated ground truth and you need sensitivity, precision, and F1 per tool to demonstrate superiority (e.g., INTEGRATE-Circ 87.3% sensitivity vs Fcirc 57.1%).

## When NOT to use

- The data are real (non-simulated) tumors or cell lines with no ground-truth fcircRNA set — sensitivity and precision cannot be computed without known backsplices; use orthogonal RT-PCR validation instead
- The goal is detecting fcircRNAs in a new dataset rather than comparing tools' accuracy — run INTEGRATE-Circ directly rather than building a benchmark table
- Evaluating linear gene-fusion (non-circular) detection only, where fcircRNA-specific backsplice matching to ground truth does not apply

## Inputs

- 100 iterations of simulated RNA-Seq data containing 30 fusion transcripts with randomly generated backsplice junctions (ground-truth fcircRNA calls)
- Mapped and unmapped RNA-Seq BAM files per simulated iteration
- Reference genome FASTA (e.g., hg19.chr21.fa) and Ensembl annotation file
- Fcirc published fusion call list (773 fusions); CircFusion published 305-fusion list for Acfs comparator

## Outputs

- Per-tool fcirc.txt / summary.tsv calls for each of the 100 simulated iterations
- Sensitivity, precision, and F1 per tool reported as mean ± standard deviation across all 100 iterations (e.g., INTEGRATE-Circ F1 91.5% ± 4% vs Fcirc 64.5% ± 2%)

## How to apply

Generate simulated RNA-Seq iterations (here, 100 iterations of data containing 30 fusion transcripts with randomly generated backsplice junctions) so each fcircRNA has a known ground-truth backsplice. Run each tool with its published parameters on the identical simulated dataset — for INTEGRATE-Circ keep defaults (-cfn 3, -minW 2.0, -bacc 1) and collect fcirc.txt per iteration; run Fcirc using its published fusion call list of 773 fusions and Acfs alongside CircFusion's published 305-fusion list as comparators. For each tool, match its called backsplices to the simulated ground-truth fcircRNAs across all 100 iterations, then compute sensitivity (true positives / ground truth), precision (true positives / calls), and F1 (harmonic mean) per tool. Report per-metric mean and standard deviation across iterations so the reproduced table can be compared to the paper's reported values (INTEGRATE-Circ: sensitivity 87.3% ± 4%, precision 96.1% ± 3%, F1 91.5% ± 4%; Fcirc: 57.1% ± 2%, 74.2% ± 4%, 64.5% ± 2%). Validation succeeds when INTEGRATE-Circ is shown to be more sensitive, precise, and accurate than the comparator tools on the simulated RNA-Seq data, as reported in the paper.

## Related tools

- **INTEGRATE-Circ** (Primary tool under benchmark; run with default parameters (-cfn 3, -minW 2.0, -bacc 1) on each simulated iteration to produce fcirc.txt calls) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **Fcirc** (Comparator tool; run on the identical simulated dataset using its published fusion call list of 773 fusions)
- **Acfs** (Secondary comparator; run alongside CircFusion's published 305-fusion list (reported no fcircRNAs on simulated data))
- **cmake** (Build dependency (>= 2.8) for compiling INTEGRATE-Circ before benchmarking)
- **g++** (Compiler required to build INTEGRATE-Circ (version >12 recommended))

## Examples

```
Integrate-Circ fusion -r reference.fa -a annotation.txt -dir bwts -mb mapped.bam -ub unmapped.bam
```

## Evaluation signals

- Reproduced sensitivity/precision/F1 table shows INTEGRATE-Circ more sensitive, precise, and accurate than comparator tools on the simulated RNA-Seq data, as reported in the paper
- Per-tool metrics fall within the reported ranges: INTEGRATE-Circ sensitivity 87.3% ± 4%, precision 96.1% ± 3%, F1 91.5% ± 4%; Fcirc 57.1% ± 2%, 74.2% ± 4%, 64.5% ± 2%
- Acfs reports no fcircRNAs on the simulated data (0 sensitivity), matching the paper's finding
- Each tool's backsplice calls are matched to the same simulated ground-truth fcircRNAs across all 100 iterations before computing metrics
- INTEGRATE-Circ runs completed for all 100 iterations with default parameters (-cfn 3, -minW 2.0, -bacc 1) and produced fcirc.txt and summary.tsv outputs

## Limitations

- Acfs reported no fcircRNAs on the simulated data, so its sensitivity/precision/F1 cannot be meaningfully compared beyond zero
- Benchmark validity depends on the simulation assumptions (30 fusion transcripts with randomly generated backsplice junctions); results may not transfer to real tumors
- No version number is stated for INTEGRATE-Circ, and the tool cannot be compiled by some older compilers (g++ >12 recommended), which may block reproduction

## Evidence

- [results] On the simulated data, INTEGRATE-Circ outperformed Fcirc in sensitivity (87.3% ± 4% vs 57.1% ± 2%), precision (96.1% ± 3% vs 74.2% ± 4%), and F1 accuracy (91.5% ± 4% vs 64.5% ± 2%): [evidence span withheld — anchor status: not_found]
- [abstract] INTEGRATE-Circ was more sensitive, precise and accurate than other tools based on our analysis of simulated RNA-Seq data: "We found that INTEGRATE-Circ was more sensitive, precise and accurate than other tools based on our analysis of simulated RNA-Seq data"
- [methods] keeping default parameters (-cfn 3, -minW 2.0, -bacc 1) and collecting fcirc.txt and summary.tsv: "-cfn      integer : Cutoff of spanning RNA-Seq reads for fusions with non-canonical exonic boundaries"
- [readme] INTEGRATE-Circ must perform a BWT (Burrows-Wheeler transform) of the reference genome: "Prior to analyzing sequencing data, INTEGRATE-Circ must perform a BWT (Burrows-Wheeler transform) of the reference genome. This operation only needs to be performed once"
