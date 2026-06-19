# Example Roadmap

This folder is intentionally lightweight. Do not commit full LAMMPS trajectory
files here. Instead, copy official LAMMPS examples into your own local working
folder, run them, and keep only input modifications or notes in GitHub.

## Where To Get The Examples

Official LAMMPS examples come with the LAMMPS source and many binary
installations:

```bash
git clone -b release https://github.com/lammps/lammps.git
ls lammps/examples
```

If LAMMPS is already installed, find the examples directory from your package
manager, Homebrew prefix, Conda environment, or source clone.

## Recommended Sequence

### 1. `melt`

Purpose: first successful run.

Commands:

```bash
cp -r /path/to/lammps/examples/melt ~/lammps-work/
cd ~/lammps-work/melt
lmp -in in.melt
```

Add this if the script does not already write a trajectory:

```lammps
dump            ovito all custom 100 traj.lammpstrj id type x y z
dump_modify     ovito sort id
```

Open `traj.lammpstrj` in OVITO.

### 2. `indent`

Purpose: see deformation and defect nucleation.

Add per-atom quantities:

```lammps
compute         peatom all pe/atom
compute         csym all centro/atom fcc
dump            ovito all custom 100 traj_indent.lammpstrj id type x y z c_peatom c_csym
dump_modify     ovito sort id
```

In OVITO, color by `c_csym` and inspect the region under the indenter.

### 3. `shear`

Purpose: understand shear loading before studying stacking faults or
grain-boundary motion.

In OVITO, compare atom positions at early, middle, and late frames. Use
centrosymmetry or CNA/PTM to find non-FCC regions.

### 4. `ELASTIC`

Purpose: validate a potential before using it for production science.

See the worked guide in `elastic_constants/README.md`.

Record:

- LAMMPS version.
- Potential file.
- Crystal structure and lattice constant.
- Elastic constants from the log/output.

### 5. `ELASTIC_T`

Purpose: understand why finite-temperature elastic constants need time
averaging and careful statistics.

Start here only after finishing the zero-temperature `ELASTIC` example. For
finite-temperature work, compare the `BORN_MATRIX` and `DEFORMATION` examples
inside the official `ELASTIC_T` directory.

### 6. `voronoi` and `steinhardt`

Purpose: learn local structure descriptors useful for grain boundaries,
surfaces, and disordered regions.

Compare these outputs with OVITO's built-in structure analysis tools.

## Project Bridge Notes

When moving from official examples to our scripts, check:

- `units metal`
- `atom_style atomic`
- `pair_style` and `pair_coeff`
- atom type to element mapping
- boundary conditions
- minimization settings
- NPT/NVT/NVE ensemble
- dump frequency and columns
- restart file names

Use `run 0` and visualize the initial state before running a long simulation.
