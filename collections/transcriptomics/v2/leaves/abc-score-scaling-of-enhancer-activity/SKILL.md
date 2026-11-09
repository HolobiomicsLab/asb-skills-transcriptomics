---
name: abc-score-scaling-of-enhancer-activity
description: Use when scoring enhancer–gene interactions from candidate regions and
  a chromatin contact map, and you want enhancer activity adapted by enhancer–gene
  contact counts (adapted ABC), rather than plain activity-by-contact.
when_to_use_negative:
- You only need region-level TF affinity computation (TRAP affinities) without any
  enhancer–gene interaction scoring — the plain affinity part suffices.
- You have no chromatin contact data and only want the non-adapted window-based gene
  assignment; in that case the ABC scaling (activity × contact / maximum contact)
  cannot be applied.
- You need the adapted activity for scaling but chose to run non-adapted ABC-scoring
  — the pipeline explicitly cannot use adapted activity in that mode.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: STARE
  role: Implements the adapted ABC enhancer–gene scoring pipeline; the ABC-scoring
    part can be called independently
  repo: https://github.com/schulzlab/STARE
- name: Juicer
  role: Data extraction and KR normalization of .hic files to produce the contact
    inputs for ABC scoring
  repo: https://github.com/aidenlab/juicer
- name: ABC-Enhancer-Gene-Prediction
  role: Original ABC model codebase whose score STARE adapts; Fulco et al. 2019 version
    available on the NG2019 branch
  repo: https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction
- name: bedtools
  role: Genome-arithmetic operations on candidate region intervals within the STARE
    workflow
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:6ba6503f5626065a8d2d887df33205ae5e4b715f2b090b6243eb2b63b4fc54a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - ABC-Enhancer-Gene-Prediction
  - Juicer
  - STARE
  - TEPIC 2
  - TRAP
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# abc-score-scaling-of-enhancer-activity

## Summary

Scores enhancer–gene interactions with the Activity-By-Contact (ABC) model and improves accuracy by adapting enhancer activity according to the number of contacts each enhancer has to its candidate target genes, considering all annotated TSSs. STARE implements this adapted ABC scoring and can run it independently of TF-affinity summarisation.

## When to use

Use when scoring enhancer–gene interactions from candidate regions and a chromatin contact map, and you want enhancer activity adapted by enhancer–gene contact counts (adapted ABC), rather than plain activity-by-contact.

## When NOT to use

- You only need region-level TF affinity computation (TRAP affinities) without any enhancer–gene interaction scoring — the plain affinity part suffices.
- You have no chromatin contact data and only want the non-adapted window-based gene assignment; in that case the ABC scaling (activity × contact / maximum contact) cannot be applied.
- You need the adapted activity for scaling but chose to run non-adapted ABC-scoring — the pipeline explicitly cannot use adapted activity in that mode.

## Inputs

- Non-overlapping candidate region bed-file with enhancer activity column(s) (specified via -n)
- Normalized chromatin contact files derived from .hic via Juicer normalization (./Code/Juicebox_normalization.sh -j <juicer_jar> -h <hic> -n KR -d <out> -c <chromosomes> -b <bin_size>)
- Gencode GTF gene annotation (e.g. from gencodegenes.org)
- Hi-C matrix if not using the average hg19 matrix (average_hic.v2.191020 from Fulco et al. 2019) or the ENCFF134PUN hg38 matrix (Gschwind et al. 2023)

## Outputs

- Enhancer–gene interaction scores under the adapted ABC model (activity adapted per number of enhancer–gene contacts, all annotated TSSs considered)
- Region–gene mapping based on the ABC score, with TSS-proximal (≤ 2.5 kb) and distant regions treated separately for downstream affinity scaling

## How to apply

Run STARE's ABC-scoring part independently: supply a non-overlapping candidate region bed-file with an activity column specified via -n, a Gencode GTF gene annotation, and normalized chromatin contact data (prepare .hic files with Juicebox_normalization.sh, e.g. KR normalization and a chosen bin size). Compute contacts between each candidate region and all annotated TSSs of candidate genes within the contact window, then adapt each enhancer's activity according to the number of contacts it has to its candidate target genes before applying the ABC formula (activity × contact / maximum contact, using region activity and contact values). Distinguish TSS-proximal regions (≤ 2.5 kb) from more distant ones when the ABC-score is also used for region–gene mapping, since the scaling differs between the two. If you run ABC-scoring without the adapted score (e.g. only one enhancer-activity assay is available), be aware that the adapted activity cannot be used for scaling. The model remains comparably accurate with only one assay measuring enhancer activity, so missing DNF or a second epigenetic track is not disqualifying.

## Related tools

- **STARE** (Implements the adapted ABC enhancer–gene scoring pipeline; the ABC-scoring part can be called independently) — https://github.com/schulzlab/STARE
- **Juicer** (Data extraction and KR normalization of .hic files to produce the contact inputs for ABC scoring) — https://github.com/aidenlab/juicer
- **ABC-Enhancer-Gene-Prediction** (Original ABC model codebase whose score STARE adapts; Fulco et al. 2019 version available on the NG2019 branch) — https://github.com/broadinstitute/ABC-Enhancer-Gene-Prediction
- **bedtools** (Genome-arithmetic operations on candidate region intervals within the STARE workflow) — https://github.com/arq5x/bedtools2

## Examples

```
conda install stare-abc; ./Code/Juicebox_normalization.sh -j <path_juicer_jar_file> -h <hic_path> -n KR -d <out_path> -c <chromosomes> -b <bin_size>
```

## Evaluation signals

- Run STARE's runTestCases.sh and confirm outputs match the precomputed controls in /Test/Test_Controls/ in content and quantity with no ERROR messages.
- Verify that enhancer activity is adapted by the number of enhancer–gene contacts (scores must differ from plain activity × contact / max-contact when an enhancer contacts multiple candidate genes).
- Check that all annotated TSSs of a gene are considered, not only the 5' TSS.
- Confirm contact files were KR-normalized via Juicebox_normalization.sh before scoring.
- Confirm TSS-proximal regions (≤ 2.5 kb) are handled separately from distant regions when ABC-based mapping feeds downstream affinity scaling.

## Limitations

- If you chose non-adapted ABC-scoring, the adapted activity is unavailable for scaling (and for downstream affinity summarisation).
- Accuracy of the plain ABC score is improved by the adaptation, but with only one enhancer-activity assay the adapted-activity information may be limited.
- Contact maps depend on normalization choices (Juicer KR, bin size) and on the reference matrix (hg19 Fulco 2019 average vs. hg38 ENCFF134PUN); mismatched assemblies/normalizations will distort scores.

## Evidence

- [abstract] We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes and also by: "We demonstrate that the ABC score can yield a higher accuracy by adapting the enhancer activity according to the number of contacts the enhancer has to its candidate target genes"
- [abstract] Further, we show that the model is comparably accurate with only one assay to measure enhancer activity.: "Further, we show that the model is comparably accurate with only one assay to measure enhancer activity."
- [methods] If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones.: "If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones."
- [methods] If you chose to run the ABC-scoring without the adapted score, we can't use the adapted activity for scaling.: "If you chose to run the ABC-scoring without the adapted score, we can't use the adapted activity for scaling."
- [readme] STARE is a framework that combines two tasks, which can also work independently: - score enhancer-gene interactions with the Activity-By-Contact (ABC): "STARE is a framework that combines two tasks, which can also work independently:
 - score enhancer-gene interactions with the Activity-By-Contact (ABC)"
