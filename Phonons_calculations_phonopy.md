## Calculating Phonons with Phonopy and VASP

### Step 1: Preparation of Supercells with Phonopy

To generate displaced supercells for phonon calculations, use:

```bash
phonopy -d --dim="3 3 3" --pa=auto
```

- `--dim` specifies the supercell dimensions (here, a 3×3×3 supercell).
- `--pa=auto` automatically selects primitive axes.

This generates displacement directories named `disp-xxxx`.

### Step 2: VASP Calculations

Perform VASP calculations in each `disp-xxxx` directory.

Use the following recommended INCAR settings for accurate force calculations:

```ini
ENCUT = 900
SIGMA = 0.05
EDIFF = 1.00E-08
ALGO = Normal
GGA = PE
PREC = Accurate
SYSTEM = Phonon Calculation
IBRION = -1
ICHARG = 2
ISMEAR = 0
ISTART = 0
NELM = 200
NSW = 0
NCORE = 4
LCHARG = .FALSE.
LWAVE = .FALSE.
LREAL = .FALSE.
```

Run VASP for each displaced structure.

### Step 3: Collect Forces from VASP Calculations

After finishing all VASP runs, collect forces:

```bash
phonopy -f disp-*/vasprun.xml
```

Phonopy will output the force constants needed for phonon calculations.

### Step 4: Dielectric and Born Effective Charges (for LO-TO splitting)

To calculate phonons with LO-TO splitting:

1. Prepare a separate dielectric calculation with VASP using the following INCAR:

```ini
ENCUT = 550
SIGMA = 0.05
EDIFF = 1.00E-08
ALGO = Normal
GGA = PE
PREC = Accurate
SYSTEM = Dielectric and Born Charge Calculation
IBRION = -1
ISMEAR = 0
LCHARG = .FALSE.
LWAVE = .FALSE.
LREAL = .FALSE.
LEPSILON = .TRUE.
```

2. After completing the calculation, extract Born charges and dielectric constants:

```bash
phonopy-vasp-born dielectric-calc/vasprun.xml
```

If symmetry constraints are required:

```bash
phonopy-vasp-born --st dielectric-calc/vasprun.xml
```

### Step 5: Calculating and Plotting Phonon Band Structure

Calculate phonon bands along specific high-symmetry points in the Brillouin zone:

```bash
phonopy-load --band="0.5 0.5 0.5  0 0 0  0.5 0.5 0  0 0.5 0" -p -s
```

- `--band` defines the q-point path.
- `-p` plots the band structure.
- `-s` saves plot data for further use.

### Notes:

- Ensure consistent settings across displaced and dielectric calculations.
- Use sufficient supercell sizes (3×3×3 or larger) for accurate phonon dispersion.

This comprehensive workflow ensures accurate phonon calculations using Phonopy and VASP.
