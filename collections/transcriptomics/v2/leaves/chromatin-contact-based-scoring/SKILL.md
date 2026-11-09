---
name: chromatin-contact-based-scoring
description: Use when candidate enhancer regions need enhancer-gene interaction scores
  that incorporate chromatin contact data (e.g. Hi-C maps), such as ranking CRISPRi-validated
  enhancer-gene pairs on the three validation datasets or comparing against model-based
  scores like Enformer in-silico mutagenesis.
when_to_use_negative:
- When no chromatin contact data (Hi-C matrix or normalized contact files) is available
  for the cell type or genome build — the adapted ABC model depends on the contact
  term
- When the task is TF affinity or TF-gene scoring rather than enhancer-gene interaction
  scoring — STARE's TF-affinity part or TEPIC is the appropriate component instead
- When interactions beyond ~5 MB from the TSS must be considered, since STARE uses
  a default 5 MB window around the TSS
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0602
tools:
- name: STARE
  role: Computes adapted ABC (gABC) enhancer-gene interaction scores from regions,
    activity, gene annotation, and chromatin contacts
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package used to install STARE's ABC-scoring component (conda install
    stare-abc)
  repo: http://bioconda.github.io/recipes/stare-abc/README.html
- name: Juicer
  role: Data extraction and normalization of .hic files into normalized genomic float
    contact files for the ABC contact term
  repo: https://github.com/aidenlab/juicer
- name: bedtools
  role: Genome arithmetic on candidate-region intervals during region-gene mapping
  repo: https://github.com/arq5x/bedtools2
- name: ABC-Enhancer-Gene-Prediction
  role: Reference original ABC implementation against which the adapted (gABC) score
    is compared
  repo: https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction
- name: pROC
  role: Computes precision-recall curves and ROC AUCs for scored interactions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:9dd643d66adc00a7945f1637c9f6b5a325858152a449898aff1907b1c8197c6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - ABC-Enhancer-Gene-Prediction
  - Juicer
  - STARE
  - bedtools
  - pROC
  - stare-abc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chromatin-contact-based-scoring

## Summary

Score enhancer-gene interactions with the adapted Activity-By-Contact (ABC) model, which adjusts enhancer activity by the number of chromatin contacts between the enhancer and each candidate target gene and considers all annotated TSSs of a gene. This yields gABC-scored enhancer-gene interactions that can be ranked and validated against perturbation data.

## When to use

Use when candidate enhancer regions need enhancer-gene interaction scores that incorporate chromatin contact data (e.g. Hi-C maps), such as ranking CRISPRi-validated enhancer-gene pairs on the three validation datasets or comparing against model-based scores like Enformer in-silico mutagenesis.

## When NOT to use

- When no chromatin contact data (Hi-C matrix or normalized contact files) is available for the cell type or genome build — the adapted ABC model depends on the contact term
- When the task is TF affinity or TF-gene scoring rather than enhancer-gene interaction scoring — STARE's TF-affinity part or TEPIC is the appropriate component instead
- When interactions beyond ~5 MB from the TSS must be considered, since STARE uses a default 5 MB window around the TSS

## Inputs

- Candidate-region bed file with enhancer activity column (e.g. ATAC-seq/H3K27ac signal)
- Gencode gene annotation in gtf format (-a)
- Normalized chromatin contact files prepared from .hic (e.g. KR-normalized, via Juicer) placed in a contact folder (-f)
- Optionally the Fulco et al. 2019 average hg19 Hi-C matrix (average_hic.v2.191020) or the Gschwind et al. 2023 averaged hg38 matrix (ENCFF134PUN)
- CRISPR perturbation validation measurements for labeling true enhancer-gene interactions

## Outputs

- gABC-scored enhancer-gene interaction list (interactions passing the 0.02 ABC-score cutoff, within the 5 MB TSS window)
- Per-dataset precision-recall curves and ROC AUC values (e.g. assembled into gabc_enformer_pr_roc_comparison.csv)

## How to apply

Run STARE/STARE_ABCpp on each dataset with the candidate-region bed file, the enhancer-activity column (-n, e.g. ATAC-seq or H3K27ac signal), a Gencode gtf gene annotation (-a), and a chromatin contact folder (-f) holding normalized Hi-C contact files; keep the default 5 MB window around the TSS and the 0.02 ABC-score cutoff. Contacts must be prepared as normalized genomic floats from .hic files via the Juicer normalization script (e.g. Juicebox_normalization.sh with KR normalization). The adapted model then scales enhancer activity by the number of contacts the enhancer has to its candidate target genes and, unlike the original ABC, considers all annotated transcription start sites of a gene, which the authors show increases accuracy. Rank the surviving interactions (contact-weighted ABC score above the 0.02 cutoff) and label positives from CRISPR perturbation measurements; compute precision-recall curves and ROC AUCs (e.g. with pROC) to judge whether the contact-based scoring ranks true interactions above competitors.

## Related tools

- **STARE** (Computes adapted ABC (gABC) enhancer-gene interaction scores from regions, activity, gene annotation, and chromatin contacts) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package used to install STARE's ABC-scoring component (conda install stare-abc)) — http://bioconda.github.io/recipes/stare-abc/README.html
- **Juicer** (Data extraction and normalization of .hic files into normalized genomic float contact files for the ABC contact term) — https://github.com/aidenlab/juicer
- **bedtools** (Genome arithmetic on candidate-region intervals during region-gene mapping) — https://github.com/arq5x/bedtools2
- **ABC-Enhancer-Gene-Prediction** (Reference original ABC implementation against which the adapted (gABC) score is compared) — https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction
- **pROC** (Computes precision-recall curves and ROC AUCs for scored interactions)

## Examples

```
STARE_ABCpp -b candidates.bed -n activity.bedgraph -a gencode.gtf -f contacts_kr/ ; then rank interactions (ABC cutoff 0.02) and compute ROC AUCs in R with pROC
```

## Evaluation signals

- gABC attains higher accuracy than the Enformer in-silico-mutagenesis baseline on all three CRISPRi validation datasets, with significantly higher ROC AUCs
- All retained interactions lie within the default 5 MB window around the TSS and pass the 0.02 ABC-score cutoff
- Contact input files are KR-normalized genomic floats produced by the Juicer normalization script from .hic files
- Activity adjustment reflects the number of contacts each enhancer has to its candidate target genes (the core adapted-ABC invariant)
- Installation and pipeline validity confirmed by running STARE's runTestCases.sh script

## Limitations

- Accuracy depends on contact data availability and quality; with only one assay to measure enhancer activity the model is comparably accurate, so adding a second activity assay yields limited gain
- The 5 MB window around the TSS bounds the range of assignability; distant interactions are not scored
- Contact matrices are genome-build specific (hg19 vs hg38; Fulco 2019 average_hic.v2.191020 vs Gschwind 2023 ENCFF134PUN) and must match the region coordinates
- .hic files must be pre-processed with Juicer normalization before use; raw .hic files cannot be fed directly

## Evidence

- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has to its candidate target genes and by considering all annotated transcription start sites of a gene: "We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes and also by"
- [intro] STARE combines ABC scoring and TF-affinity derivation, which can run independently: "STARE is a framework that combines two tasks, which can also work independently"
- [full_text] STARE_ABCpp invocation parameters: candidate-region bed file, activity column (-n), Gencode gtf (-a), chromatin contact folder (-f), 5MB window, 0.02 cutoff: [evidence span withheld — anchor status: not_found]
- [methods] Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [methods] Normalized contact preparation from .hic via Juicer: "./Code/Juicebox_normalization.sh -j <path_juicer_jar_file> -h <hic_path> -n KR -d <out_path> -c <chromosomes> -b <bin_size>"
- [readme] STARE's stated purpose includes scoring enhancer-gene interactions with the ABC model: "score enhancer-gene interactions with the Activity-By-Contact (ABC)"
