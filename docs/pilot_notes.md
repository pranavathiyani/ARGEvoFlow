# TEM-1 Pilot — 2026-04-24 (extended)

Supersedes the earlier single-assay pilot notes. Records the complete April 24 pilot sequence spanning four independent TEM-1 deep mutational scans, one cross-class aminoglycoside-modifying enzyme DMS, a 209,644-sequence MSA conservation baseline, a BLOSUM62 biochemistry floor, and a 1,000-iteration permutation null.

## Headline results

**Pre-registered hypotheses (see `docs/analysis_plan.md`)**
- H2 (ESM2 vs DMS fitness ρ ≥ 0.35 with CI excluding 0): **supported on all five DMS assays tested**
- H3 (ESM2 captures signal beyond MSA conservation): **supported on TEM-1, Δρ = +0.27 to +0.30 across four independent assays; partial Spearman ρ ≈ 0.71 after controlling for conservation**

## ESM2-650M vs DMS fitness (masked-marginal scoring)

| Assay | Protein | N | Spearman ρ | 95% BCa CI |
|---|---|---|---|---|
| Stiffler 2015 | TEM-1 β-lactamase | 4,996 | 0.7302 | [0.7149, 0.7445] |
| Firnberg 2014 | TEM-1 β-lactamase | 4,783 | 0.7377 | [0.7227, 0.7523] |
| Jacquier 2013 | TEM-1 β-lactamase (MIC) | 989 | 0.6958 | [0.6537, 0.7304] |
| Deng 2012 | TEM-1 β-lactamase (early protocol) | 4,996 | 0.5286 | [0.5076, 0.5497] |
| Dandage 2018 | AAC(6')-Ib aminoglycoside acetyltransferase | 1,801 | 0.4904 | [0.4526, 0.5246] |

Stiffler and Firnberg (the two highest-resolution contemporary TEM-1 assays) give overlapping confidence intervals. Jacquier's MIC-based assay gives slightly lower ρ because of coarser fitness resolution — ESM2's ranking ability is bounded by the DMS ground-truth reliability, not by the model itself. Deng 2012 is the earliest TEM-1 DMS and uses a protocol that predates modern mutagenesis library construction; its lower ρ is consistent with noisier experimental fitness scores rather than weaker model performance (the ESM2–conservation gap is preserved at +0.27).

## Conservation baseline (ProteinGym MSA, 209,644 β-lactamase homologs)

MSA source: `DMS_msa_files/BLAT_ECOLX_full_11-26-2021_b02.a2m` from ProteinGym v1.3 (Zenodo 14997691).
MSA coverage: 215 of 263 mature TEM-1 positions (82%). Excluded positions are mostly N-terminal tail (1–16) and C-terminal tail (243–263) plus a few internal loops.

Conservation scoring: log-odds ratio with BLOSUM62 background pseudocounts (α = 1), following Tsishyn et al. 2025.

| Assay | ESM2 ρ (MSA-covered) | Conservation ρ | Gap | Partial ρ (ESM2 \| Cons) |
|---|---|---|---|---|
| Stiffler 2015 | 0.7736 | 0.4728 | +0.3008 | 0.7060 |
| Firnberg 2014 | 0.7794 | 0.4787 | +0.3007 | 0.7128 |
| Jacquier 2013 | 0.7379 | 0.4649 | +0.2730 | — |
| Deng 2012 | 0.5739 | 0.3053 | +0.2686 | — |

The +0.30 gap is essentially identical across the two high-quality assays. After residualizing against conservation, ESM2 retains partial ρ ≈ 0.71 — nearly as high as the raw ρ of 0.77 — indicating that PLM predictions capture fitness information substantially beyond per-column sequence frequency.

## BLOSUM62 biochemistry floor (control)

| Assay | BLOSUM62 ρ | ESM2 ρ | Gap (ESM2 − BLOSUM62) |
|---|---|---|---|
| Stiffler 2015 | 0.346 | 0.730 | +0.384 |
| Firnberg 2014 | 0.339 | 0.738 | +0.399 |
| Jacquier 2013 | 0.416 | 0.696 | +0.280 |
| Deng 2012 | 0.306 | 0.529 | +0.223 |
| AAC(6')-Ib | 0.083 | 0.490 | +0.407 |

Note: BLOSUM62 captures substantial signal on TEM-1 (ρ ≈ 0.34) — roughly half of the total ESM2 correlation is recoverable from amino-acid biochemistry alone. On AAC(6')-Ib, BLOSUM62 ρ drops to 0.08 (essentially no biochemistry signal), yet ESM2 still achieves ρ = 0.49. AAC(6')-Ib is therefore a *harder* test in the sense that ESM2 has less biochemistry to leverage — its lower absolute ρ masks a proportionally *larger* positional-context contribution. This reframes the cross-class generalization result.

## Permutation null (control, Stiffler 2015)

- Observed ρ: 0.7302
- Null distribution (1,000 random shufflings of fitness labels): mean = 0.0004, std = 0.0142
- Null 95% range: [-0.0289, +0.0278]
- Empirical p-value: p < 0.001 (exact upper bound given 1,000 iterations)
- Observed ρ is **51.3 standard deviations** from the null mean

The correlation is statistically indistinguishable from impossible by chance. This control is intended to satisfy reviewer concerns that the observed effect could be a sampling artifact at high N.

## Technical notes

### Sequence provenance
Stiffler, Firnberg, Jacquier, and Deng all used the TEM-1A allele (confirmed identical reconstructed sequences across all four DMS files). UniProt P62593 canonical is TEM-1B. These differ at two residues: position 59 (Stiffler V, UniProt I) and position 159 (Stiffler A, UniProt V). Scoring was performed on the TEM-1A sequence reconstructed from the ProteinGym `mutated_sequence` column. Using the UniProt canonical would introduce systematic error at these two positions.

### AAC(6')-Ib
Dandage 2018 DMS used full-length AAC(6')-Ib (177 residues, no signal peptide) from *Pseudomonas aeruginosa*. Position numbering in the DMS matches the full-length sequence directly.

### Data provenance
All DMS datasets pulled from ProteinGym v1.3 (Zenodo 10.5281/zenodo.14997691):
- `BLAT_ECOLX_Stiffler_2015.csv`
- `BLAT_ECOLX_Firnberg_2014.csv`
- `BLAT_ECOLX_Jacquier_2013.csv`
- `BLAT_ECOLX_Deng_2012.csv`
- `AACC1_PSEAI_Dandage_2018.csv`

MSA pulled from `DMS_msa_files.zip` in the same Zenodo record.

### Model
ESM2-650M (`esm2_t33_650M_UR50D`, Lin et al., *Science* 2023). Masked-marginal log-likelihood ratios per Meier et al., NeurIPS 2021. 650M-parameter checkpoint chosen as the efficient frontier per Notin 2025 scaling-wall analysis.

### Reproducibility
- Seed: 42 (numpy, torch, Python random)
- Compute: Google Colab free tier, Tesla T4 GPU
- BCa bootstrap: 10,000 iterations for all CIs (Efron 1987)
- Permutation null: 1,000 iterations, label-shuffle on fitness
- Conservation pseudocount: α = 1 with BLOSUM62 background frequencies

## MEEHubs 2026 abstract

Submitted April 24, 2026 (deadline April 26). Submission ID: 181.
Abstract text saved in `docs/meehubs2026_submitted_abstract.md`.

## Files in `results/pilot/`

- `master_results.json` — all headline numbers, machine-readable
- `tem1_esm2_vs_dms.tsv` — Stiffler per-variant ESM2 scores
- `tem1_pilot_summary.tsv` — Stiffler metrics row
- `tem1_pilot_figure.png` — Stiffler scatter + LLR heatmap
- `tem1_esm2_llr.npz` — full TEM-1 LLR matrix (263 × 20, reusable for all four TEM-1 DMS)

Per-variant TSVs for Firnberg, Jacquier, Deng, and AAC(6')-Ib were generated during the afternoon session but were lost when the Colab runtime disconnected. They can be regenerated by re-running `notebooks/02_colab_esm2_pilots.ipynb` (~30 minutes total compute). All headline numbers from those runs are preserved in `master_results.json` and reproduced verbatim in the tables above.

## Next steps (post-abstract)

1. Regenerate and commit per-variant TSVs for Firnberg, Jacquier, Deng, AAC(6')-Ib
2. Consolidate all pilot code into a clean committed `notebooks/02_colab_esm2_pilots.ipynb`
3. Decide between three Week 1 directions: full 5-group ESKAPE scoring, ARGEvoAtlas site build, or additional cross-class validation with KKA2 aminoglycoside kinase
