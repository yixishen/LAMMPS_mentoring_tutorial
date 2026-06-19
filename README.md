# LAMMPS Mentoring Tutorial

This tutorial is for new students learning to run atomistic simulations with
LAMMPS and visualize them with OVITO. It starts with installation, then uses
official LAMMPS examples as a safe learning path before connecting those ideas
to our metal/alloy work: EAM potentials, lattice constants, elastic response,
defects, grain boundaries, stacking-fault-style displacements, and
energy/structure analysis.

Last checked: 2026-06-19. Software installers and package names change, so use
the official documentation links in the reference section when something looks
different on your machine.

## 1. What You Should Learn

By the end, you should be able to:

- Install LAMMPS on Windows, macOS, or Linux.
- Build LAMMPS from source with CMake when pre-built binaries are not enough.
- Run an official LAMMPS example from the command line.
- Add trajectory output that OVITO can read.
- Use OVITO to inspect atom types, energy, centrosymmetry, CNA/PTM structure,
  and deformation.
- Recognize which official examples are most relevant to our Ni/Fe/Cr alloy and
  grain-boundary workflows.

## 2. Quick Vocabulary

- LAMMPS: molecular dynamics engine. It reads a text input script and writes
  logs, restart files, dumps, and analysis files.
- Input script: usually named `in.something`; this is the recipe for the
  simulation.
- Potential file: parameter file for the interatomic potential, for example
  EAM/alloy or EAM/fs files for metals.
- Dump file: atom snapshots written during a simulation. OVITO reads these for
  visualization.
- Restart file: binary file that lets LAMMPS continue a previous simulation.
- Thermo output: scalar values printed to screen and `log.lammps`, such as
  temperature, energy, pressure, and box size.

## 3. Install LAMMPS

### Recommended Path

Use a pre-built LAMMPS first. Compile from source only when you need a package,
custom fix, custom compute, accelerator, or exact cluster configuration that the
binary does not provide.

After any installation, test it:

```bash
lmp -help
```

If your executable has another name, try:

```bash
lammps_serial -help
lammps_mpi -help
lmp_serial -help
lmp_mpi -help
```

LAMMPS input scripts should normally be run with `-in`:

```bash
lmp -in in.file
```

For MPI runs:

```bash
mpirun -np 4 lmp -in in.file
```

### Windows

Simplest route:

1. Download the Windows installer from the official LAMMPS package site:
   <https://packages.lammps.org/windows.html>
2. Run the installer and allow it to add LAMMPS to your `PATH`.
3. Open PowerShell or Command Prompt and test:

```powershell
lmp -help
```

Run an input:

```powershell
lmp -in in.melt
```

Parallel runs on Windows need the MPI runtime described on the LAMMPS installer
site. For most beginner exercises, serial LAMMPS is enough.

Good alternative for students who are comfortable with Linux tools:

1. Install WSL2 with Ubuntu.
2. Follow the Linux or Conda instructions below inside Ubuntu.
3. Keep simulation paths simple, for example under `~/lammps-work`.

### macOS

Homebrew route:

```bash
brew install lammps
brew test lammps -v
```

Homebrew currently installs executables such as `lammps_serial` and
`lammps_mpi`. Test whichever one was installed:

```bash
lammps_serial -help
lammps_mpi -help
```

Conda route:

```bash
conda config --add channels conda-forge
conda create -n lammps-tutorial
conda activate lammps-tutorial
conda install lammps
lmp -help
```

Conda is convenient when students do not want to modify the whole system.

### Linux

Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install lammps
lmp -help
```

Fedora:

```bash
sudo dnf install lammps-openmpi
module load mpi/openmpi-x86_64
mpirun -np 2 lmp -in in.lj
```

OpenSUSE:

```bash
sudo zypper install lammps
lmp -help
```

Conda route for Linux:

```bash
conda config --add channels conda-forge
conda create -n lammps-tutorial
conda activate lammps-tutorial
conda install lammps
lmp -help
```

Static Linux binaries are also available from the official LAMMPS download site
and are useful on systems where you do not have administrator access.

## 4. Build LAMMPS From Source With CMake

Compile from source when:

- You need optional packages not included in your binary.
- You need a lab-custom command, for example a custom grain-boundary analysis
  `fix`.
- You want to run on a cluster with a specific MPI/compiler stack.
- You want GPU/Kokkos/OpenMP acceleration configured in a particular way.

LAMMPS currently requires a C++17-capable compiler and CMake 3.20 or newer.
CMake is the preferred build system.

### Linux or macOS Source Build

Install build tools first.

Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install build-essential cmake git ninja-build openmpi-bin libopenmpi-dev
```

macOS with Homebrew:

```bash
brew install cmake git ninja open-mpi
```

Clone the release branch and build:

```bash
git clone -b release https://github.com/lammps/lammps.git
cd lammps
cmake -S cmake -B build \
  -D CMAKE_BUILD_TYPE=Release \
  -D CMAKE_INSTALL_PREFIX=$HOME/lammps-install \
  -D BUILD_MPI=on \
  -D PKG_MANYBODY=on \
  -D PKG_EXTRA-COMPUTE=on \
  -D PKG_EXTRA-FIX=on \
  -D PKG_VORONOI=on
cmake --build build -j 4
cmake --install build
$HOME/lammps-install/bin/lmp -help
```

For EAM metal simulations, `PKG_MANYBODY=on` is important. If a script fails
with `Unknown pair style`, `Unknown fix`, or `Unknown compute`, run:

```bash
lmp -help
```

Check whether the required style/package is listed. If not, rebuild with the
needed package enabled.

### Windows Source Build

For most students, use the Windows installer or WSL2. Native Windows builds are
possible with Visual Studio, CMake, and a compatible MPI stack, but they require
more setup. If you need a native Windows source build:

1. Install Visual Studio with C++ development tools.
2. Install CMake and Git.
3. Clone LAMMPS:

```powershell
git clone -b release https://github.com/lammps/lammps.git
cd lammps
```

4. Configure with CMake GUI or CMake command line using the `cmake` source
   folder and a separate build folder.
5. Enable required packages such as `MANYBODY`.
6. Build the generated Visual Studio solution.

If the goal is to learn simulations rather than compiler toolchains, use WSL2
and the Linux instructions.

## 5. Run Your First Official Example

Do not edit the original installed example. Copy it into a working folder:

```bash
mkdir -p ~/lammps-work
cp -r /path/to/lammps/examples/melt ~/lammps-work/
cd ~/lammps-work/melt
lmp -in in.melt
```

The examples directory location depends on how LAMMPS was installed. Common
places include:

- Source clone: `lammps/examples`
- Linux package: `/usr/share/lammps/examples` or a nearby documentation path
- Homebrew: under the Homebrew prefix, usually shown by `brew --prefix lammps`
- Conda: under the active environment, often inside `share/lammps/examples`

Useful checks after a run:

```bash
ls
tail log.lammps
```

The log should end normally. If it stops with `ERROR`, read the first error
message carefully. Later messages are often side effects.

## 6. Add OVITO-Friendly Dump Output

Many official examples already contain dump commands, sometimes commented out.
For OVITO, a reliable beginner dump is:

```lammps
dump            ovito all custom 100 traj.lammpstrj id type x y z
dump_modify     ovito sort id
```

For defect and grain-boundary work, add per-atom quantities:

```lammps
compute         peatom all pe/atom
compute         csym all centro/atom fcc
dump            ovito all custom 100 traj_defects.lammpstrj id type x y z c_peatom c_csym
dump_modify     ovito sort id
```

Notes:

- `id type x y z` is enough for positions and atom types.
- `c_peatom` helps find high-energy defect regions.
- `c_csym` helps highlight local disorder in FCC systems.
- `dump_modify sort id` gives stable atom ordering across frames.
- For multi-element systems, keep a clear type map in your notebook, for
  example type 1 = Cr, type 2 = Ni.

## 7. Official Examples Most Relevant To Our Work

Start with these official LAMMPS example directories:

| Example | Why it matters for us | What to look for |
| --- | --- | --- |
| `melt` | First sanity check; teaches run command, thermo output, and trajectories. | Temperature, potential energy, atom motion. |
| `indent` | Local deformation and defect nucleation. | High-energy atoms near the indenter; centrosymmetry changes. |
| `shear` | Deformation under shear, useful before stacking-fault or GB migration scripts. | Slip, void response, stress evolution. |
| `crack` | Defect/fracture response in a solid. | Crack-tip disorder and stress concentration. |
| `ELASTIC` | Computes zero-temperature elastic constants. See `examples/elastic_constants/README.md` in this repo for a worked guide. | How potential choice affects stiffness. |
| `ELASTIC_T` | Elastic constants at finite temperature. | Thermal fluctuations and averaging. |
| `DIFFUSE` | Diffusion coefficient workflows. | Mean-squared displacement and temperature dependence. |
| `prd` or `tad` | Vacancy diffusion and accelerated dynamics examples. | Rare-event thinking; not a first-week exercise. |
| `voronoi` | Per-atom volume and local geometry. | GB excess volume, free volume near defects. |
| `steinhardt` | Local order parameters. | Disorder and phase/structure classification. |
| `UNITS` | Same simulation in different unit systems. | Why our metal scripts use `units metal`. |

Recommended student sequence:

1. Run `melt`.
2. Add a dump command and open it in OVITO.
3. Run `indent`.
4. Add `pe/atom` and `centro/atom`, then color by those values in OVITO.
5. Run `ELASTIC` and compare output to literature or potential documentation.
6. Inspect `voronoi` or `steinhardt` for local-structure analysis.
7. Only then move to our Ni/Cr/NiFe/FeCrNi scripts.

## 8. Bridge From Official Examples To Our Project Scripts

Our project scripts use many patterns that appear in the official examples:

- `units metal`: time in ps, distance in Angstrom, energy in eV, pressure in bar.
- `atom_style atomic`: appropriate for simple metallic atoms.
- `pair_style eam/alloy`, `eam/fs`, or `hybrid/overlay`: metallic potentials.
- `fix npt`: equilibrate temperature and pressure.
- `fix box/relax`: relax box shape/size during minimization.
- `compute pe/atom`: local energy around defects and grain boundaries.
- `compute centro/atom fcc`: identify local FCC disorder.
- `dump custom`: write OVITO-readable trajectories.
- `write_restart` and `read_restart`: split long workflows into stages.

Typical project progression:

1. Validate the potential with a lattice constant script.
2. Validate elastic constants or cohesive behavior.
3. Create or read a defect/grain-boundary structure.
4. Minimize energy.
5. Equilibrate at target temperature.
6. Run the production step.
7. Visualize and analyze high-energy/disordered atoms in OVITO.

Before running a project input, check these lines:

```lammps
pair_style      eam/alloy
pair_coeff      * * path/to/potential.eam.alloy Ni
```

or:

```lammps
pair_style      hybrid/overlay eam/alloy eam/fs
pair_coeff      * * eam/alloy path/to/FeCrNi_d.eam.alloy Cr Ni
pair_coeff      * * eam/fs    path/to/FeCrNi_s.eam.fs    Cr Ni
```

The element order at the end of `pair_coeff` must match LAMMPS atom types. A
wrong order can produce a simulation that runs but is physically meaningless.

### Worked Example: Elastic Constants

For a guided exercise, use the local note:

```text
examples/elastic_constants/README.md
```

This example starts from the official LAMMPS `examples/ELASTIC` directory,
explains the zero-temperature deformation/stress method, shows what students
should record from the output, and gives a short checklist for adapting the
script to a Ni or Ni-Cr EAM potential. This is a good bridge between "LAMMPS
runs" and "the potential is trustworthy enough for defect or grain-boundary
simulations."

## 9. Install OVITO

Download OVITO Basic or OVITO Pro from:

<https://www.ovito.org/>

As of OVITO 3.15.4, desktop requirements include:

- Windows 10/11 on x86_64.
- Linux distributions with sufficiently recent system libraries and OpenGL.
- macOS 14+ on Apple Silicon.

Install summary:

- Windows: run the `.exe` installer.
- macOS: open the `.dmg` and drag OVITO to Applications.
- Linux: extract the `.tar.xz`, enter the extracted folder, and run
  `./bin/ovito`.
- Conda: OVITO Basic is also available from conda-forge.

OVITO needs working OpenGL graphics. On clusters, the easiest route is usually
to copy dump files to your laptop/desktop and open them locally. X11 forwarding
over SSH often fails or gives a blank window because OVITO needs direct graphics
support.

## 10. Visualize A LAMMPS Dump In OVITO

1. Open OVITO.
2. Select `File` -> `Load File`.
3. Choose `traj.lammpstrj`, `dump.indent`, or another LAMMPS dump.
4. If OVITO asks for a file type, choose LAMMPS dump.
5. Press play or drag the time slider to inspect the trajectory.
6. In the pipeline, add modifiers as needed.

Useful modifiers:

- `Wrap at periodic boundaries`: put atoms back in the periodic cell.
- `Color coding`: color by `Particle Type`, `c_peatom`, `c_csym`, or other
  properties.
- `Common Neighbor Analysis`: classify FCC/HCP/BCC/other local environments.
- `Polyhedral Template Matching`: robust crystal-structure classification.
- `Expression selection`: select atoms using conditions such as
  `c_csym > 5`, using the property name shown in OVITO.
- `Delete selected`: temporarily hide matrix atoms and see defect cores.
- `Construct surface mesh`: inspect free surfaces or voids.
- `Dislocation analysis`: useful for slip/dislocation structures when available.

For a clean figure:

1. Color atoms by a meaningful property.
2. Hide unimportant atoms with a selection modifier if needed.
3. Set the camera view.
4. Use `Render active viewport` for an image or `Render animation` for a movie.
5. Record the exact dump file, frame number, and modifiers used.

## 11. OVITO Workflow For Defects And Grain Boundaries

For our metal/alloy work, a useful OVITO pipeline is:

1. Load `traj_defects.lammpstrj`.
2. Add `Wrap at periodic boundaries`.
3. Add `Polyhedral Template Matching` or `Common Neighbor Analysis`.
4. Add `Color coding` by structure type.
5. Add another `Color coding` by `c_peatom` or `c_csym` when studying defect
   energy/disorder.
6. Use `Expression selection` to select high-energy or high-centrosymmetry atoms.
7. Hide the perfect FCC matrix if you want to focus on the grain boundary,
   stacking fault, crack tip, or dislocation core.

Example selection ideas:

```text
c_csym > 5
```

```text
c_peatom > -4.0
```

Use the property names shown in OVITO's data inspector. For example, a LAMMPS
column named `c_csym` may appear as `c_csym` or be mapped to a more descriptive
property name, depending on the file and OVITO version. Adjust thresholds for
each potential and temperature. Thermal noise increases centrosymmetry and
energy fluctuations.

## 12. Common Problems

`lmp: command not found`

LAMMPS is not on your `PATH`, or your executable has another name. Try
`lammps_serial`, `lammps_mpi`, `lmp_serial`, or give the full executable path.

`ERROR: Unknown pair style eam/alloy`

Your LAMMPS binary was built without the needed package. For EAM potentials,
rebuild with `PKG_MANYBODY=on` or install a fuller LAMMPS binary.

`ERROR: Cannot open potential file`

The path in `pair_coeff` is wrong. Use a relative path inside your project
folder or an absolute path that exists on the machine running LAMMPS.

Simulation runs but results look wrong

Check atom type to element mapping, units, timestep, boundary conditions, and
potential file. A script can run without being physically valid.

`Lost atoms`

Common causes include too large a timestep, bad initial geometry, excessive
temperature, overlapping atoms, or an unstable potential/structure combination.
Minimize first, lower timestep, and visualize the initial structure.

OVITO opens the file but atoms look like a cloud

Check whether coordinates are wrapped/unwrapped/scaled. Try `Wrap at periodic
boundaries`. Confirm that the dump columns are named correctly.

OVITO cannot run on an HPC login node

Use local OVITO and copy the dump files, or ask about a supported remote
visualization setup. Do not rely on plain SSH X11 forwarding.

## 13. Suggested Mentoring Assignments

### Assignment 1: Install And Run

Goal: prove LAMMPS works.

1. Install LAMMPS.
2. Run `lmp -help`.
3. Run the official `melt` example.
4. Submit `log.lammps` and a one-paragraph summary of thermo output.

### Assignment 2: First OVITO Movie

Goal: connect LAMMPS output to visualization.

1. Add a `dump custom` line to `melt`.
2. Open the dump in OVITO.
3. Render one image and one short animation.
4. Explain what changes during the trajectory.

### Assignment 3: Defect Visualization

Goal: learn local structural analysis.

1. Run `indent` or `shear`.
2. Add `compute pe/atom` and `compute centro/atom fcc`.
3. Color by centrosymmetry and potential energy in OVITO.
4. Identify where disorder starts and how it evolves.

### Assignment 4: Potential Validation

Goal: understand why we validate potentials before using them.

1. Follow `examples/elastic_constants/README.md`.
2. Run the official `ELASTIC` example.
3. Record elastic constants, units, LAMMPS version, and potential file.
4. Compare results from two potentials or two LAMMPS builds if available.
5. Explain whether the potential is reasonable for the target material.

### Assignment 5: Project Bridge

Goal: prepare for our Ni/Cr or Fe/Ni/Cr workflow.

1. Read a project lattice-constant or grain-boundary input script.
2. Mark the initialization, structure creation, potential, minimization,
   equilibration, dump, and restart sections.
3. Explain the atom type to element mapping.
4. Run only a small test system first.

## 14. Good Habits

- Copy examples into your own folder before editing.
- Keep input scripts, potential files, and run notes together.
- Save the exact LAMMPS version from `lmp -help`.
- Use small systems for debugging.
- Run `run 0` and inspect the initial structure before a long simulation.
- Add dump output early, even if the dump interval is large.
- Never trust a simulation only because it finished.
- Keep large dump/movie files out of GitHub; commit scripts and notes instead.

## References

- LAMMPS install overview: <https://docs.lammps.org/Install.html>
- LAMMPS Linux packages: <https://docs.lammps.org/Install_linux.html>
- LAMMPS macOS/Homebrew: <https://docs.lammps.org/Install_mac.html>
- LAMMPS Windows installer: <https://docs.lammps.org/Install_windows.html>
- LAMMPS Conda install: <https://docs.lammps.org/Install_conda.html>
- LAMMPS Git source: <https://docs.lammps.org/Install_git.html>
- LAMMPS build overview: <https://docs.lammps.org/Build.html>
- LAMMPS CMake build: <https://docs.lammps.org/Build_cmake.html>
- LAMMPS build packages: <https://docs.lammps.org/Build_package.html>
- LAMMPS run basics: <https://docs.lammps.org/Run_basics.html>
- LAMMPS official examples: <https://docs.lammps.org/Examples.html>
- LAMMPS elastic constants howto: <https://docs.lammps.org/Howto_elastic.html>
- OVITO installation: <https://www.ovito.org/docs/current/installation.html>
- OVITO data import: <https://www.ovito.org/docs/current/usage/import.html>
- OVITO LAMMPS dump reader:
  <https://www.ovito.org/docs/current/reference/file_formats/input/lammps_dump.html>
