# Reading `.npz` Density Matrix Files

Each `.npz` file contains the following keys:

- `elements` — list of atomic symbols (strings)
- `atom_coordinates` — array of shape `(N, 3)` in Angstroms
- `charge` — integer
- `spin` — integer
- `density_matrix` — NumPy array (shape depends on molecule)

---

## Example

```python
import numpy as np

# Load one file
data = np.load("dm_results_npz/BHPERI/BHPERI_0_0.npz.npz")

# List the keys stored in the file
print(data.files)
# Output:
# ['elements', 'atom_coordinates', 'charge', 'spin', 'density_matrix']
