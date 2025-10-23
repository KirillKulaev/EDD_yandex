# Reading `.npz` Density Matrix Files

Each `.npz` file contains the following keys:

- `elements` — list of atomic symbols (strings)
- `atom_coordinates` — array of shape `(N, 3)` in Angstroms
- `charge` — integer
- `spin` — integer
- `density_matrix` — NumPy array (shape depends on molecule)

---

## Data example

```python
import numpy as np

# Load one file
data = np.load("dm_results_npz/BHPERI/BHPERI_0_0.npz.npz")

# List the keys stored in the file
print(data.files)
# Output:
# ['elements', 'atom_coordinates', 'charge', 'spin', 'density_matrix']
```
---

## Use PySCF from data

You can use data to PySCF calculations. Next code calculate energy of chemical system based on electron density, electron density on spatial grid 

```python
import numpy as np
from pyscf import gto, dft

# 1. Load UKS density matrix from .npz
data = np.load("dm_results_npz/BHPERI/BHPERI_0_0.npz")
elements = data["elements"]
coords = data["atom_coordinates"]
charge = int(data["charge"])
spin = int(data["spin"])
dm_uks = data["density_matrix"]  # shape [2, n_basis, n_basis]
data.close()

# 2. Sum alpha + beta to get initial density (ok if spin == 0, because in our data we have spin-polarized electron density)
dm = dm_uks[0] + dm_uks[1] 

# 3. Create molecule
mol = gto.M(
    atom=list(zip(elements, coords)),
    charge=charge,
    spin=spin,
    basis='def2-qzvp'
)

# 4. DFT calculation
mf = dft.RKS(mol)
mf.xc = 'B3LYP'

# Use DM as initial guess (ok if spin == 0)
energy = mf.kernel(dm)

# 5. Compute electron density on a spatial grid
grid_coords = mf.grids.coords
ao_values = dft.numint.eval_ao(mol, grid_coords, deriv=0)
rho = dft.numint.eval_rho(mol, ao_values, mf.make_rdm1(), xctype='LDA')

