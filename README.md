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

## Using a UKS Density Matrix for RKS SCF in PySCF

You can use data to PySCF caluclations

```python
import numpy as np
from pyscf import gto, dft

# 1. Load UKS density matrix from .npz
data = np.load("dm_results_npz/BHPERI/BHPERI_0_0.npz")
elements = data["elements"]
coords = data["atom_coordinates"]
charge = int(data["charge"])
spin = int(data["spin"])
dm_uks = data["density_matrix"]  # shape [2, nbas, nbas]
data.close()

# 2. Sum alpha + beta to get RKS initial density (ok if spin == 0)
dm_rks = dm_uks[0] + dm_uks[1]

# 3. Create molecule
mol = gto.M(
    atom=list(zip(elements, coords)),
    charge=charge,
    spin=spin,
    basis='def2-qzvp'
)

# 4. RKS SCF with B3LYP
mf = dft.RKS(mol)
mf.xc = 'B3LYP'
mf.max_cycle = 50

# Use summed DM as initial guess (ok if spin == 0)
mf.kernel(dm0=dm_rks)

# 5. Compute electron density on a spatial grid
mf.grids.level = 2
mf.grids.build()
grid_coords = mf.grids.coords
ao_values = dft.numint.eval_ao(mol, grid_coords, deriv=0)
rho = dft.numint.eval_rho(mol, ao_values, mf.make_rdm1(), xctype='LDA')

