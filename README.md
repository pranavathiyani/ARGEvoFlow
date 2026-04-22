# ARGEvoFlow

**A Nextflow DSL2 pipeline for quantifying mutational tolerance of antibiotic resistance genes in ESKAPE pathogens using zero-shot protein language model pseudo-likelihoods.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Data License: CC BY 4.0](https://img.shields.io/badge/Data%20License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

---

## Overview

ARGEvoFlow takes bacterial protein sequences and produces per-residue mutational tolerance scores computed from ESM2 pseudo-likelihoods. The pipeline benchmarks PLM-derived scores against a conservation + relative solvent accessibility (RSA) baseline — the current state-of-the-art parameter-free predictor — and validates against the available corpus of β-lactamase deep mutational scanning (DMS) data.

Results for ESKAPE pathogen antibiotic resistance genes are released as **ARGEvoAtlas**, a MaveDB-compatible variant effect atlas hosted at [pranavathiyani.github.io/ARGEvoAtlas](https://pranavathiyani.github.io/ARGEvoAtlas) (forthcoming).

## What ARGEvoFlow computes

For every protein in the input set:

- **Per-residue log-likelihood ratio (LLR)** from ESM2-650M masked-marginal scoring
- **Sequence-averaged tolerance score** (mean LLR over all positions, all 19 alternatives)
- **Per-residue substitution tolerance profile** (20 × L matrix)
- **Stratified tolerance by residue category** (catalytic, interface, core, surface)
- **Species-corrected tolerance** to account for UniRef sampling bias (Ding & Steinhardt 2024)
- **Baseline comparisons**: Shannon entropy from Pfam MSA, RSA from structure, BLOSUM62 null

## What ARGEvoFlow does NOT claim to do

ARGEvoFlow does not predict clinical resistance spread, future ARG radiation, or antibiotic efficacy. Mutational tolerance is a necessary but not sufficient component of evolvability; realised diversification depends on selection pressure, horizontal gene transfer, and surveillance — factors outside the model. See `docs/analysis_plan.md` for explicit scope.

## Requirements

- Nextflow ≥ 23.04
- Python ≥ 3.10
- PyTorch 2.x + `fair-esm` (runs on Google Colab free tier with T4 GPU)
- Conda or Pixi recommended for dependency management
- R ≥ 4.2 with `lme4`, `ape`, `nlme`, `phytools` for statistical analysis

## Quick start

```bash
nextflow run pranavathiyani/ARGEvoFlow -profile test
```

For the full ESKAPE run:

```bash
nextflow run pranavathiyani/ARGEvoFlow -profile eskape
```

## Inputs

- Protein FASTA files for the five protein groups (see `docs/group_definitions.md`)
- DMS reference tables for β-lactamases (TEM-1, VIM-2, NDM-1) in `assets/dms_references/`
- Pfam MSA alignments for conservation baseline
- AlphaFold2 structure predictions for RSA calculation

## Outputs

| File | Contents |
|---|---|
| `results/tolerance_scores.tsv` | Per-protein sequence-averaged tolerance scores |
| `results/per_residue/` | Per-residue LLR matrices (JSON, one per protein) |
| `results/baselines.tsv` | Conservation + RSA and BLOSUM62 scores for comparison |
| `results/dms_validation.tsv` | Spearman ρ, AUC, MCC, NDCG@10%, Top-recall with 95% BCa CIs |
| `results/lmm_fit.rds` | Linear mixed model fit (species + gene family random effects) |
| `results/atlas/` | MaveDB-compatible JSON for ARGEvoAtlas site |

## Pipeline structure

```
FETCH_CARD ──┐
FETCH_REFSEQ ─┼→ EXTRACT_PROTEINS → CLUSTER_REDUNDANCY → ANNOTATE_FEATURES
              │                                                │
              │                                                ├→ ESM2_EMBED ──→ TOLERANCE_SCORE ──┐
              │                                                ├→ MSA_CONS_RSA ───────────────────┼→ DMS_VALIDATE
              │                                                └→ BLOSUM_BASELINE ─────────────────┘       │
              │                                                                                            ├→ LMM_ANALYSIS
              │                                                                                            ├→ PGLS_ROBUSTNESS
              │                                                                                            ├→ PERMUTATION_NULL
              │                                                                                            └→ BUILD_ATLAS
```

## Pre-registered analysis plan

The statistical analysis plan was pre-registered before any ESM2 inference was run. See [`docs/analysis_plan.md`](docs/analysis_plan.md) for the locked hypotheses, thresholds, and statistical tests. Changes to the plan after pre-registration are documented in `docs/analysis_plan_changelog.md`.

## Data and code licensing

- **Code**: MIT License — see [`LICENSE`](LICENSE)
- **Data outputs** (ARGEvoAtlas): CC BY 4.0 — compatible with Atlas of Variant Effects Alliance standards
- **Model weights**: ESM2-650M is distributed by Meta AI under MIT; we do not redistribute

## Citation

If you use ARGEvoFlow, please cite:

> Gnanasekar, P. (2026). ARGEvoFlow: protein language model-based mutational tolerance scoring for the ESKAPE resistome. https://github.com/pranavathiyani/ARGEvoFlow

Full machine-readable metadata in [`CITATION.cff`](CITATION.cff).

## Author

**Pranavathiyani Gnanasekar**
Assistant Professor (Research), Division of Bioinformatics
SASTRA Deemed University, Thanjavur, Tamil Nadu, India
[pranavathiyani@scbt.sastra.edu](mailto:pranavathiyani@scbt.sastra.edu)

## Acknowledgements

This work builds on the ESM2 protein language model (Lin et al., *Science* 2023), the ProteinGym benchmark (Notin et al., NeurIPS 2023), the Comprehensive Antibiotic Resistance Database (Alcock et al., NAR 2023), and the Atlas of Variant Effects Alliance framework (Fowler et al., Genome Biology 2023).
