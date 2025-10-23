# Reading `.npz` Density Matrix Files

Each `.npz` file contains the following keys:

- `elements` — list of atomic symbols (strings)
- `atom_coordinates` — array of shape `(N, 3)` in Angstroms
- `charge` — integer
- `spin` — integer
- `density_matrix` — NumPy array (shape depends on molecule)

---

## Python Example
import numpy as np
data = np.load("dm_results_npz/ISO34/ISO34_0_0.npz")
print(data.files)
-> ['elements', 'atom_coordinates', 'charge', 'spin', 'density_matrix']
