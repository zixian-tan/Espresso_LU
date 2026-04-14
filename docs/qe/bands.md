---
sidebar_position: 4
---
# Band Structure

> Calculation type: `calculation = 'bands'`

This document provides a **standard band structure calculation** template for `pw.x` in Quantum ESPRESSO.  
It includes all necessary parameters as well as optional advanced settings (commented with explanations).  
The band calculation is performed **after an SCF run**, using the self-consistent charge density.

---

## 1. Overview

### Typical Workflow

1. **Geometry optimization** — `relax` or `vc-relax`
2. **SCF calculation** — generate charge density (`prefix.save/charge-density.dat`)
3. **BANDS calculation** — compute eigenvalues along high-symmetry k-path (`K_POINTS crystal_b`)
4. **Post-processing** — `bands.x`, `plotband.x`, or custom plotting script

---

## 2. Standard Band Structure Input Template

```pw.x
&control
    calculation = 'bands',          ! non-self-consistent band calculation
    prefix      = 'MY_SYS',         ! must match the SCF prefix
    outdir      = './tmp',          ! same scratch directory used in SCF
    pseudo_dir  = './pseudo',       ! pseudopotential directory
    verbosity   = 'high',           ! detailed output for debugging

    ! restart_mode = 'restart',     ! restart from existing SCF data
    ! wf_collect = .true.,          ! collect wavefunctions on master (useful for post-processing)
    ! disk_io = 'none',             ! minimize disk I/O if sufficient memory
/

&system
    ibrav   = 0,                    ! use CELL_PARAMETERS from input
    nat     = 4,                    ! number of atoms
    ntyp    = 2,                    ! number of atomic species

    ecutwfc = 60.0,                 ! wavefunction cutoff (Ry)
    ecutrho = 480.0,                ! charge density cutoff (Ry)
    occupations = 'fixed',          ! fixed occupations (for insulators)
    ! occupations = 'smearing',     ! use smearing for metallic systems
    ! smearing    = 'mp',           ! Methfessel–Paxton smearing
    ! degauss     = 0.02,           ! smearing width (Ry)

    ! input_dft = 'PBE',            ! explicitly define the exchange-correlation functional
    ! vdw_corr  = 'dft-d3',         ! enable dispersion correction (useful for layered materials)
    ! nspin     = 2,                ! enable spin-polarization if magnetic
    ! lda_plus_u = .true.,          ! enable DFT+U correction
    ! Hubbard_U(1) = 4.0,           ! U parameter (in eV)
    ! ecutfock = 60.0,              ! Fock cutoff for hybrid functionals
/

&electrons
    conv_thr    = 1.0d-8,           ! tight convergence threshold
    diagonalization = 'david',      ! Davidson diagonalization
    mixing_beta = 0.3,              ! SCF mixing parameter (used only if SCF is triggered)
    ! electron_maxstep = 100,       ! max SCF iterations (normally skipped in bands)
/

ATOMIC_SPECIES
Si  28.0855  Si.pbe-n-kjpaw_psl.1.0.0.UPF
O   15.9994  O.pbe-n-kjpaw_psl.1.0.0.UPF

CELL_PARAMETERS angstrom
  5.431   0.000   0.000
  0.000   5.431   0.000
  0.000   0.000   5.431

ATOMIC_POSITIONS angstrom
Si  0.000000  0.000000  0.000000
Si  2.715500  2.715500  2.715500
O   1.357750  1.357750  1.357750
O   4.073250  4.073250  4.073250

K_POINTS crystal_b
4
! The following are high-symmetry points in the Brillouin zone (example: cubic Si)
! Format: kx ky kz  number_of_points_between
    0.000  0.000  0.000  20  ! Γ
    0.500  0.000  0.000  20  ! X
    0.500  0.250  0.250  20  ! W
    0.000  0.000  0.000  1   ! back to Γ
```

---

## 3. Key Notes

### Essential Points

- The **charge density** from SCF must exist in prefix.save/.
- Use **the same cutoff, pseudopotentials, and XC functional** as in SCF.
- Use `K_POINTS crystal_b` (not automatic) for band-path definitions.
- Each line in the `K_POINTS` block corresponds to a **path segment** between high-symmetry points.
- After the run, process the eigenvalues using:

```bash
pw.x < bands.in > bands.out
bands.x < pp.band.in > pp.band.out
```

### Band Plot Workflow

1. Run `pw.x` for the bands input above.
2. Run `bands.x`:

    ```band.x
    &BANDS
        prefix = 'MY_SYS',
        outdir = './tmp',
        filband = 'MY_SYS.bands.dat'
        ! lsym    = .true.
    /
    ```

3. (Optional) Plot using:

    ```optional
    &plot
        fileout = 'bands.gnu'
        efermi = 6.6416
    /
    ```

### Notes on Parameters

| Parameter | Purpose | Notes |
| ---- | ---- | ---- |
| `prefix` | Identifier | Must match SCF step |
| `outdir` | Working directory | Must contain `prefix.save` |
| `ecutwfc`, `ecutrho` | Cutoffs | Must be identical to SCF |
| `occupations` | Type of occupation | Use `'fixed'` for insulators, `'smearing'` for metals |
| `K_POINTS crystal_b` | Band-path specification | List of high-symmetry points |
| `conv_thr` | Convergence | `1e-8` for accurate eigenvalues |

---

## 4. Example: Band Structure for a Metal (with Smearing)

```eg
&control
    calculation = 'bands',
    prefix      = 'Cu',
    outdir      = './tmp',
    pseudo_dir  = './pseudo',
    verbosity   = 'high',
/

&system
    ibrav = 0,
    nat = 1,
    ntyp = 1,
    ecutwfc = 70.0,
    ecutrho = 560.0,

    occupations = 'smearing',
    smearing = 'mp',
    degauss = 0.02,
/

&electrons
    conv_thr = 1.0d-8,
    diagonalization = 'david',
/

ATOMIC_SPECIES
Cu  63.546  Cu.pbe-dn-kjpaw_psl.1.0.0.UPF

CELL_PARAMETERS angstrom
  3.615  0.000  0.000
  0.000  3.615  0.000
  0.000  0.000  3.615

ATOMIC_POSITIONS angstrom
Cu  0.000  0.000  0.000

K_POINTS crystal_b
6
! Example band path for fcc metal: Γ–X–W–L–Γ–K
  0.000  0.000  0.000  20  ! Γ
  0.500  0.000  0.000  20  ! X
  0.500  0.250  0.750  20  ! W
  0.500  0.500  0.500  20  ! L
  0.000  0.000  0.000  20  ! Γ
  0.375  0.375  0.750  20  ! K
```

---

## 5. Compact Template for Automated Workflows

```template
&control
    calculation = 'bands',
    prefix = 'PREFIX',
    outdir = './tmp',
    pseudo_dir = './pseudo',
/

&system
    ibrav = 0,
    nat = NAT,
    ntyp = NTYP,
    ecutwfc = ECUTWFC,
    ecutrho = ECUTRHO,
    occupations = 'fixed',
/

&electrons
    conv_thr = 1.0d-8,
    diagonalization = 'david',
/

ATOMIC_SPECIES
  ... (auto-generated)

CELL_PARAMETERS angstrom
  ... (auto-generated)

ATOMIC_POSITIONS angstrom
  ... (auto-generated)

K_POINTS crystal_b
  ... (k-path automatically generated)
```

---

## Summary Table

| Step | Type | Purpose | Input Keyword | Notes |
| ---- | ---- | ---- | ---- | ---- |
| scf | Self-consistent | Obtain charge density | calculation='scf' | Must precede band step |
| bands | Non-self-consistent | Compute eigenvalues along path | calculation='bands' | Reads from SCF results |
|bands.x | Post-processing | Extracts and formats bands | filband | Generates .bands data |
| plotband.x | Plotting utility | Produces band structure plot | efermi, emin, emax | Optional step |

---

## Typical Files Produced

File	Description
prefix.save/	Folder containing SCF results
prefix.bands.out	Output log of band calculation
prefix.bands	File containing eigenvalues (for bands.x)
bands.gnu	Gnuplot-ready file for band plotting

---

This template is designed to serve as a robust and well-commented starting point for any Quantum ESPRESSO band structure calculation, ensuring reproducibility and easy adaptation to both semiconductors and metals.

---
