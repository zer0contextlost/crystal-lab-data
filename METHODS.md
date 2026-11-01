# Methods

## 1. Generation
Structures come from **CrystaLLM** (a language model trained on CIF files), prompted with a composition. Most
prompts are picked automatically ("autopilot"); a few were typed by a person. `origin` and `prompt` record which.

## 2. ML screen
Each structure is relaxed with **MACE** (float64). The verdict comes from the formation energy relative to the
elements and from how much the cell changed in relaxation:
`rock solid` / `plausible` / `borderline` / `unstable` / `falls apart`.

## 3. Novelty
Composition and structure are matched (pymatgen StructureMatcher) against a local index of known structures and,
since 2026-10-28, live against **Materials Project** and **Alexandria** via OPTIMADE. `known` and `rediscovered`
records include the matching ID. Before 2026-10-28 only the local index was used, which missed some known
compounds; affected records were re-audited.

## 4. ML convex hull
Nightly, catalogued structures get an energy above the hull: MACE energies of the candidate vs MACE-relaxed
Alexandria phases near the hull in the same chemical system. `decomposes_to` lists the hull products and
fractions. Values below about -0.15 eV/atom are flagged `suspect` (almost always an ML artefact).

## 5. DFT re-check
Candidates with ML e_hull ≤ 0.025 eV/atom and ≤ 30 atoms go to DFT, together with their decomposition products
on **Alexandria's PBE hull** (≤ 64 atoms each).

- Code: Quantum ESPRESSO 6.7, PBE, SSSP 1.1.2 efficiency pseudopotentials; cutoffs = max over the elements of the
  SSSP-recommended wavefunction/charge cutoffs.
- Smearing: Marzari-Vanderbilt, 0.01 Ry. Spin: non-magnetic, except systems containing Fe/Co/Ni/Mn/Cr, which
  are run spin-polarised from a 2 μB start on those atoms.
- Each structure: `vc-relax` on a coarse k-grid (spacing 0.4 Å⁻¹; forces 1e-3 Ry/bohr, pressure 0.5 kbar),
  then an `scf` on a finer grid (0.25 Å⁻¹, conv_thr 1e-7). Energies come from the final scf only.
- Candidate and competitors are computed with identical settings, so the hull is internally consistent.
- **Pass**: e_hull ≤ 0.025 eV/atom. That margin is about the size of PBE's typical error and of entropy at room
  temperature; a pass means "worth a closer look", nothing more.
- Some jobs carry a `note` (e.g. a relaxation stopped by hand when it was clearly not going to pass, or an
  extra competitor checked afterwards). Read it before using the number.

## 6. Properties (passes only)
PBE band gap and density of states at the Fermi level; elastic tensor from ±1% strains in the six Voigt directions (ions relaxed at
fixed cell), with Born stability check. PBE underestimates band gaps, often by a factor of ~2.

## 7. Phonons (passes only)
phonopy finite displacements on a supercell with every side ≥ 7 Å, after a tight fixed-cell relaxation;
frequencies on a 10×10×10 mesh. Any mode softer than -0.1 THz away from the Γ acoustic modes = unstable. The
supercell is small, so long-wavelength instabilities can be missed.

## Known limits
- Hulls only include competitors that exist in Alexandria (and, for the ML hull, phases near its hull). A missing
  competitor can make a candidate look more stable than it is.
- PBE alone; no van der Waals, no Hubbard U, no finite-temperature free energies.
- Pass ≠ synthesisable. No candidate here has been made in a lab.
