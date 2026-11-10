# Crystal Lab open data

Crystal structures proposed by an AI model, screened with a machine-learned potential, and the most promising
ones re-checked with density functional theory (DFT). Everything is published, including the failures.

AI generators now propose crystals much faster than anyone checks them with physics. This dataset is a small,
open record of what happens when you do check: most candidates that look stable to a machine-learned model turn
out to be known compounds, or less stable than their real competitors, once the competitors are taken seriously.

Built 2026-11-10. Rebuilt nightly while the pipeline runs. Mirrors: [Hugging Face](https://huggingface.co/datasets/zer0contextlost/crystal-lab-data);
raw Quantum ESPRESSO calculations parsed and searchable on [NOMAD](https://nomad-lab.eu/prod/v1/gui/search/entries?upload_id=kNO0ljjeTyK0l_lX9522Ww) (snapshot of 2026-11-01), plus the #1856 SrCaCdP2 phonon runs ([upload of 2026-11-06](https://nomad-lab.eu/prod/v1/gui/search/entries?upload_id=VJuk3Rs9RjKGtpU7TdSF3g))
and all #2571 BaCdGePd2 runs ([upload of 2026-11-10](https://nomad-lab.eu/prod/v1/gui/search/entries?upload_id=7-krAnoRQlyS_2i0FVfjIw)).

**Cite:** the three NOMAD uploads (102 calculations) form a frozen snapshot with DOI
[10.17172/nomad.5gmq-24hf](https://doi.org/10.17172/nomad.5gmq-24hf). This repository keeps changing; the DOI does not.

## 2026-11-10: first candidate through every check
**#2571 BaCdGePd2** (P4/nmm, generated 2026-10-03 from a `surprise` prompt, where the model picks the formula itself) is the first candidate to
pass every step we run:
- DFT (PBE): 0.003 eV/atom above the hull built from 1,281 Alexandria entries in Ba-Cd-Ge-Pd; the nearest
  competing mix is BaGePd + CdPd. That is inside normal PBE error, so read it as "on the hull, within noise",
  not "stable".
- Phonons: no imaginary modes (2x2x1 supercell, 40 atoms, 10 displacements, 78 h).
- Prior art: no report of the composition in 27 databases and paper indexes that answered on 2026-11-10
  (AFLOW, CMR, MPDD, MPOD and Matterverse did not respond).

What this does not show: that BaCdGePd2 can be made, that this is its lowest-energy structure (only the
generated structure was tested), or anything about behaviour above 0 K. It is a prediction worth a synthesis
attempt, nothing more. #1856 SrCaCdP2 also passed DFT and phonons, but OQMD, GNoME and NOMAD already list it,
so it is an independent check of an existing prediction.

## 2026-11-03: stricter novelty rule
A candidate now counts as new only if its **composition** has not been reported anywhere we check, in any
structure: Materials Project, Alexandria, OQMD, COD, MPDS, Materials Cloud, GNoME, NOMAD and about 20 more
OPTIMADE databases, plus paper titles and abstracts (OpenAlex, arXiv). Before, novelty meant "no matching
structure in Materials Project or Alexandria". The stricter check found that our first DFT pass, #1856 SrCaCdP2,
was already predicted with the same structure in OQMD (entry 5879242), and that all five DFT jobs then in the
queue were known compositions; those five were taken out of the queue. Every catalogued record was re-checked the same way and
the already-reported ones re-labelled `known composition` (their old catalogue number and name are kept in the
novelty record). Known formulas are now also skipped before the generator spends any compute on them. Details in
METHODS.md §3 and §7, and in the `prior_reports` column of `dft_results.csv`.

## What's in it

| | count |
|---|---|
| Structures generated | 5837 |
| Kept as "discoveries" (stable by ML, and the composition is not reported in any database or paper we check) | 895 |
| ML hull check finished | 644 |
| ML hull check at or below 0 eV/atom | 1 |
| DFT jobs (passes / fails / still running or queued) | 12 (3 / 2 / 7) |

### How the prompts were chosen, and how each way does
On 2026-11-01 the autopilot switched from random formulas to mostly substitution-based prompts (see METHODS.md).
This table compares the sources; "within 0.05" counts ML hull results at most 0.05 eV/atom above the hull.
"Skipped as already known" (from 2026-11-03) counts formulas dropped before any ML work because the composition
was already reported; `control` rows are known compounds requested on purpose as a calibration check.
New rows need a few nights of hull checks before their numbers mean much.

| prompt mode | skipped as already known | structures | new | ML hull checked | median e_hull (eV/atom) | within 0.05 | DFT passes |
|---|---|---|---|---|---|---|---|
| random formula (before 11-01) | 0 | 3713 | 696 | 494 | 0.389 | 1 | 0 |
| surprise (before 11-01) | 0 | 1681 | 101 | 71 | 0.115 | 13 | 3 |
| substitution | 199 | 265 | 69 | 61 | 0.211 | 2 | 0 |
| random | 93 | 78 | 21 | 14 | 0.407 | 0 | 0 |
| surprise | 148 | 63 | 8 | 4 | 0.158 | 0 | 0 |
| control | 0 | 27 | 0 | 0 | - | 0 | 0 |
| human | 0 | 10 | 0 | 0 | - | 0 | 0 |

ML verdicts: unstable 2108, rock solid 1463, plausible 1307, borderline 776, falls apart 183.
Novelty at generation time: skipped 3067, known 977, new composition 895, known composition 893, rediscovered 5.

### Files
- `structures.jsonl.gz`: one JSON object per structure: formula, space group, CIF, how the prompt was chosen
  (`prompt_mode`, from 2026-11-01), ML energies and verdict,
  novelty check (with the matching database IDs when it was already known), ML hull result, DFT summary.
- `structures.csv`: the same without CIFs, for spreadsheets.
- `dft_results.csv`: one row per DFT job.
- `dft/<id>/`: every DFT job: `candidate.cif`, competitor structures (`comp/`), Quantum ESPRESSO inputs and
  outputs for the candidate and each competitor (`runs/`), `result.json`, and for passes the properties
  (`props/`: band gap, DOS, elastic tensor) and phonon runs (`phonon/`). `.out` files are gzipped.
- `stats.json`: the counts above. `build.log`: files left out of this build and why.
- `METHODS.md`: how every number was produced, and what it does and doesn't mean.

## How to read the results
- An ML "rock solid" verdict means a low formation energy with MACE. It says nothing about competing phases.
- The ML hull compares against Alexandria near-hull phases computed with the same ML model. `pending` rows carry
  only `previous_e_hull_eV_atom`, a superseded value from before a stricter re-check; don't use it as a result.
- A DFT **pass** means: with PBE, the candidate is at most 0.025 eV/atom above the convex hull built from its
  Alexandria competitors. It does **not** mean the material has been made, or that it can be made. PBE errors
  of a few tens of meV/atom are normal, and the hull only contains the competitors we checked.
- Phonons are a screen on a small supercell (shortest side ~7 Å) for imaginary modes, not a full study.

## Licence and credit
Our data (generated structures, ML and DFT results): **CC BY 4.0**, credit "Crystal Lab open data, zer0contextlost".
Competitor structures come from the **Alexandria** database (Schmidt et al., CC BY 4.0); Alexandria and
Materials Project IDs are given wherever a structure matched them. Pseudopotentials are SSSP 1.1.2 efficiency
(Prandini et al.) and are not redistributed here. See `ATTRIBUTION.md`.

Structures from licensed databases (e.g. CCDC/ICSD) used in any check are never included; such checks are cited
by deposition number only.

Corrections and questions: open an issue.
