# Approved Chemotherapeutic Dataset

## Overview

This repository contains a computational dataset of **approved small-molecule chemotherapeutic drugs** collected for the investigation of their photophysical and photochemical properties and for the identification of potential candidates for **photodynamic therapy (PDT)**.

The initial collection consisted of **349 DrugBank identifiers** associated with chemotherapeutic agents retrieved from CancerDrugs_DB, an open-access database of anticancer compounds compiled from multiple regulatory and pharmacological sources.

The dataset was filtered to retain only **approved small-molecule drugs**, excluding antibodies, antibody-drug conjugates, organometallic compounds, and inorganic species. Canonical SMILES strings were obtained from DrugBank, resulting in a final dataset of **235 compounds**.

Each compound is stored in a directory named according to its **DrugBank identifier** (e.g. `DB00188`).

---

## Dataset File

The repository contains the file:

```text
anticancer_approved_small_molecules.csv
```

This file provides the complete list of approved chemotherapeutic compounds included in the dataset.

The CSV file contains three columns:

| Column | Description |
|---|---|
| `drugbank_id` | DrugBank identifier of the compound |
| `name` | Drug name |
| `smiles` | Canonical SMILES representation |

Example:

```text
drugbank_id;name;smiles
DB00014;Goserelin;...
DB00106;Abarelix;...
DB00188;Bortezomib;...
```

---

## Directory Structure

Each DrugBank directory contains input files, optimized molecular geometries, and output files generated during the computational workflow.

A typical directory has the following structure:

```text
DBXXXXX/
├── xtbopt.xyz
├── xtb.log
│
├── opt_s0_dft.inp
├── opt_s0_dft.xyz
│
├── orca_tddft_notda.inp
├── orca_tddft_notda.log
│
├── orca_tddft_notda_dft.inp
├── orca_tddft_notda_dft.log
│
├── opt_t1.inp
├── opt_t1.xyz
├── opt_t1.log
│
├── opt_cat.inp
├── opt_cat.xyz
├── opt_cat.log
│
├── opt_an.inp
├── opt_an.xyz
└── opt_an.log
```

Not every file is necessarily present for every compound. More computationally demanding calculations were performed only for molecules retained after the corresponding screening steps.

---

## File Description

| File | Description |
|---|---|
| `xtbopt.xyz` | Geometry of the lowest-energy CREST conformer after re-optimization at the GFN2-xTB level |
| `xtb.log` | Output of the GFN2-xTB optimization performed on the best conformer obtained from CREST |
| `opt_s0_dft.inp` | ORCA input file for DFT optimization of the electronic ground state (`S0`), starting from `xtbopt.xyz` |
| `opt_s0_dft.xyz` | Ground-state geometry optimized at the DFT level |
| `orca_tddft_notda.inp` | ORCA TD-DFT input for the first photophysical screening, using the xTB-optimized geometry (`xtbopt.xyz`) |
| `orca_tddft_notda.log` | ORCA TD-DFT output for the first screening. The calculation was performed without the Tamm-Dancoff approximation (TDA) |
| `orca_tddft_notda_dft.inp` | ORCA TD-DFT input using the DFT-optimized ground-state geometry (`opt_s0_dft.xyz`). This calculation was performed for compounds retained after the first screening |
| `orca_tddft_notda_dft.log` | ORCA TD-DFT output for the second screening, based on the DFT-refined ground-state geometry and calculated without TDA |
| `opt_t1.inp` | ORCA input for optimization of the lowest triplet state (`T1`) |
| `opt_t1.xyz` | Optimized geometry of the lowest triplet state |
| `opt_t1.log` | ORCA output for the `T1` geometry optimization |
| `opt_cat.inp` | ORCA input for optimization of the radical cation |
| `opt_cat.xyz` | Optimized geometry of the radical cation |
| `opt_cat.log` | ORCA output for the radical-cation optimization |
| `opt_an.inp` | ORCA input for optimization of the radical anion |
| `opt_an.xyz` | Optimized geometry of the radical anion |
| `opt_an.log` | ORCA output for the radical-anion optimization |

---

# Computational Protocol

## 1. Dataset Construction

A total of **349 DrugBank identifiers** associated with chemotherapeutic agents were retrieved from CancerDrugs_DB, an open-access database of anticancer compounds compiled from multiple sources, including the National Cancer Institute, the U.S. Food and Drug Administration, the European Medicines Agency, and other regulatory and pharmacological repositories.

The initial collection was filtered to retain only approved small-molecule drugs. Antibodies, antibody-drug conjugates, organometallic compounds, and inorganic species were excluded.

Canonical Simplified Molecular Input Line Entry System (**SMILES**) strings were obtained from DrugBank, yielding a final dataset of **238 compounds**.

---

## 2. Three-Dimensional Structure Generation and Conformational Sampling

Three-dimensional molecular structures were generated using **RDKit**, and the relevant protonation states were assigned under physiological conditions.

Each protomer was subjected to conformational sampling using **CREST** at the **GFN2-xTB** level of theory.

The conformational searches were performed at **300 K** using a **5 fs integration time step**, with water represented through the **GB implicit-solvent model**.

For each compound, the lowest-energy conformer identified by CREST was subsequently re-optimized at the GFN2-xTB level before entering the quantum-chemical screening workflow.

The resulting geometry is stored as:

```text
xtbopt.xyz
```

---

## 3. Initial TD-DFT Screening

All electronic-structure calculations were performed with **ORCA 6.0.1**.

The xTB-optimized geometries were initially analyzed using time-dependent density functional theory (**TD-DFT**) at the:

```text
B3LYP/G / def2-TZVP
```

level of theory, where `B3LYP/G` denotes the Gaussian-compatible B3LYP implementation available in ORCA.

The **Tamm-Dancoff approximation (TDA) was not employed**.

Solvent effects were included using the **conductor-like polarizable continuum model (CPCM)** with water as the dielectric medium.

The first screening therefore used:

```text
TD-B3LYP/G/def2-TZVP
CPCM(Water)
No TDA
```

Vertical excitation energies and oscillator strengths were calculated to simulate the UV-Vis absorption properties of the compounds.

The corresponding files are:

```text
orca_tddft_notda.inp
orca_tddft_notda.log
```

The B3LYP functional was selected to provide a computationally feasible and internally consistent approach for the high-throughput screening. The dominant electronic transitions of the investigated drugs were expected to be predominantly localized valence excitations, for which global hybrid density functionals provide a practical compromise between computational cost and accuracy.

The `def2-TZVP` basis set provides triple-zeta valence flexibility together with polarization functions and was selected as a suitable compromise for the medium- and large-sized molecules investigated in the dataset.

---

## 4. Selection of Visible-Light-Absorbing Compounds

The calculated excitation energies and oscillator strengths were used to identify molecules displaying significant absorption in the visible spectral region.

This first screening reduced the original dataset to a subset of **23 candidate compounds**.

These candidates were subjected to a more detailed quantum-chemical characterization.

---

## 5. DFT Ground-State Geometry Optimization

For compounds passing the initial screening, the electronic ground-state geometry was fully optimized at the:

```text
B3LYP/G/def2-TZVP
CPCM(Water)
```

level of theory.

The GFN2-xTB optimized geometry was used as the starting structure.

The corresponding files are:

```text
opt_s0_dft.inp
opt_s0_dft.xyz
```

---

## 6. Refined TD-DFT Calculations

The UV-Vis properties of the selected compounds were subsequently recalculated using their DFT-optimized ground-state geometries.

The calculations were performed at the same:

```text
TD-B3LYP/G/def2-TZVP
CPCM(Water)
No TDA
```

level of theory.

The corresponding files are:

```text
orca_tddft_notda_dft.inp
orca_tddft_notda_dft.log
```

These calculations provided refined excitation energies, oscillator strengths, low-lying singlet and triplet excited-state information, and spin-orbit coupling (**SOC**) matrix elements required for the subsequent photodynamic screening.

---

## 7. Intersystem Crossing Assessment

The potential efficiency of **intersystem crossing (ISC)** was evaluated using the calculated spin-orbit couplings and singlet-triplet energy separations.

A qualitative, semiempirical descriptor was used in which ISC propensity increases with increasing spin-orbit coupling and decreasing singlet-triplet energy separation.

This descriptor was employed as a **relative screening criterion** and should not be interpreted as a complete kinetic rate constant. A full Fermi-golden-rule treatment was not considered suitable for the high-throughput screening performed here.

The SOC threshold used in the final screening represents an **internal, protocol-specific reference value** rather than a universal criterion for efficient intersystem crossing.

Temoporfin, an experimentally approved photosensitizer independently recovered by the screening workflow, was used as an internal reference for the SOC criterion.

---

## 8. Triplet-State Optimization

The lowest triplet state (`T1`) of the selected compounds was optimized using unrestricted DFT:

```text
UB3LYP/G/def2-TZVP
CPCM(Water)
```

The associated files are:

```text
opt_t1.inp
opt_t1.xyz
opt_t1.log
```

The optimized triplet geometries were used to evaluate the thermodynamic accessibility of photodynamic processes.

---

## 9. Type I Photodynamic Pathway

Type I photodynamic activity was investigated by considering electron-transfer processes involving the photosensitizer radical anion and radical cation.

Starting from the optimized structures, the corresponding charged radical species were optimized at the same computational level:

```text
UB3LYP/G/def2-TZVP
CPCM(Water)
```

Radical-anion calculations are stored as:

```text
opt_an.inp
opt_an.xyz
opt_an.log
```

Radical-cation calculations are stored as:

```text
opt_cat.inp
opt_cat.xyz
opt_cat.log
```

The resulting electronic energies were used to evaluate the thermodynamic feasibility of the relevant electron-transfer reactions involved in Type I photodynamic mechanisms.

---

## 10. Type II Photodynamic Pathway

Type II photodynamic activity was evaluated by considering energy transfer from the lowest triplet state of the photosensitizer to ground-state molecular oxygen.

The pathway was considered thermodynamically accessible when the adiabatic triplet energy of the photosensitizer was sufficient to populate singlet oxygen.

Reference oxygen species were calculated using the same level of theory and solvent model employed for the screened compounds.

The triplet ground state of molecular oxygen and the superoxide radical anion were independently optimized.

The energy of singlet oxygen was evaluated using a **broken-symmetry treatment followed by the Yamaguchi approximate spin-projection correction**.

---

## 11. Final Screening Strategy

The final selection combined photophysical and thermodynamic criteria.

Compounds were evaluated according to:

| Property | Role in the screening |
|---|---|
| Visible-light absorption | Identification of compounds excitable in the desired spectral region |
| Oscillator strength | Assessment of the intensity of the relevant electronic transition |
| Spin-orbit coupling | Assessment of the potential for singlet-to-triplet population transfer |
| Singlet-triplet energy separation | Additional descriptor influencing intersystem crossing |
| Triplet-state energetics | Evaluation of the Type II photodynamic pathway |
| Radical-anion energetics | Evaluation of Type I electron-transfer pathways |
| Radical-cation energetics | Evaluation of Type I electron-transfer pathways |

The oscillator-strength cutoff was selected as a conservative lower limit informed by visible transitions reported for established photosensitizer systems.

The SOC cutoff was defined using the value obtained for **temoporfin** with the same computational protocol. Because temoporfin is an experimentally established photosensitizer and was independently recovered during the screening, it provided an internal reference for the computational workflow.

In the final selection step, compounds were retained when the calculated photophysical properties and thermodynamic criteria supported at least one plausible photodynamic pathway.

---

## Computational Workflow

The overall workflow can be summarized as:

```text
Approved chemotherapeutic drugs
          │
          ▼
3D structure generation
        RDKit
          │
          ▼
Conformational sampling
 CREST / GFN2-xTB / GB water
          │
          ▼
Lowest-energy conformer
          │
          ▼
GFN2-xTB re-optimization
       xtbopt.xyz
          │
          ▼
Initial TD-DFT screening
B3LYP/G/def2-TZVP
CPCM(Water), no TDA
          │
          ▼
Visible-light absorption screening
          │
          ▼
       23 candidates
          │
          ▼
DFT optimization of S0
B3LYP/G/def2-TZVP
CPCM(Water)
          │
          ▼
Refined TD-DFT calculation
+ excitation energies
+ oscillator strengths
+ SOCs
          │
          ▼
Triplet-state optimization
          │
          ├──────────────┐
          ▼              ▼
 Radical anion       Radical cation
 optimization         optimization
          │              │
          └──────┬───────┘
                 ▼
   Type I / Type II assessment
                 │
                 ▼
      Final PDT candidates
```

---

## Software

| Software | Purpose |
|---|---|
| **RDKit** | Molecular structure generation and molecular processing |
| **CREST** | Conformational sampling |
| **GFN2-xTB / xTB** | Semiempirical geometry optimization and conformational workflow |
| **ORCA 6.0.1** | DFT, TD-DFT, triplet-state, radical-ion, and SOC calculations |

---

## Notes

The screening criteria used in this repository were designed for **relative comparison within this computational workflow**.

In particular, the selected oscillator-strength and spin-orbit-coupling thresholds should not be interpreted as universal criteria for photodynamic activity.

Likewise, the ISC descriptor used during screening provides a computational ranking criterion rather than a complete kinetic description of intersystem crossing.

---

## References

The detailed methodological rationale and literature references associated with the computational protocol are provided in the accompanying scientific work.

Key methods and resources include **CancerDrugs_DB, DrugBank, RDKit, CREST, GFN2-xTB, ORCA, B3LYP, CPCM, and the Yamaguchi approximate spin-projection approach**.
