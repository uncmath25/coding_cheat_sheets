# Numpy

*Array operations are 1-2 orders of magnitude faster than equivalent list operations*

### Array Basics
``` python
import numpy as np
data = np.random.randn(35).reshape(7, 5)
np.array([[1, 2], [3, 4]], dtype=np.float64)
np.zeros((2, 3))
np.ones((2, 3))
data.ndim
data.shape
data.dtype
np.arange(start, stop, step_size) # non inclusive
np.linspace(start, stop, number) # inclusive
np.arange(0, 1+0.1, 0.1) == np.linspace(0, 1, 10+1)
```

### Array Slices
``` python
np.arange(1, 7)[3:5]
x = np.arange(1, 7); x[3:5] = 6
x[3:5].copy()
data = np.array([[1, 2], [3, 4]]); data[0][1] == data[0, 1] == 2
data[1:, 2] # each "non-colon" index drops a shape dimension
data = np.random.randn(2, 3); data[:, [0, 2]]
data[data.sum(axis=1)>0]
data[~np.array([False for _ in range(data.shape[0])])]
```

### Array Functions
``` python
data = np.random.randn(2, 3); data.mean(axis=1) # mean by row
data = np.where(data>0, data, 0) # array ternary operation
data = np.random.randn(2, 3); data[data>0].any(); data[data>0].all()
data = np.random.randn(2, 3); data.sort(1) # sort in-place row-wise
x = np.random.randn(1000); x.sort(); x[int(0.9 * len(x))] # 90th percentile
np.unique(np.array(['A', 'B', 'A']))
```

### Plotting
* Meshgrid
``` python
x_mesh, y_mesh = np.meshgrid(np.arange(-5, 5, 0.01), np.arange(-5, 5, 0.01))
z_mesh = np.sqrt(x_mesh**2 + y_mesh**2)
import matplotlib.pyplot as plt
plt.imshow(z_mesh, cmap=plt.cm.gray)
plt.colorbar()
plt.show()
```

### Linear Algebra
* Matrix multiplication
``` python
A = np.arange(6).reshape(3, 2)
np.dot(A.transpose(), x=A)
A.transpose().dot(A)
```
* Matrix inverse, eigen decomposition, qr decomposition and singular value decomposition
``` python
from numpy.linalg import inv, eig, qr, svd
A = np.random.randn(5, 5)
inv(A)
eg_values, eg_vectors = eig(A)
A.dot(eg_vectors) - eg_values*eg_vectors < 0.001
q, r = qr(A)
u, d, v = svd(A)
```

### Misc
* Set random seed
``` python
np.random.seed(1)
```
* Build 3-dim identity matrix
``` python
np.eye(3)
```
* Cast array copy as specified type
``` python
data.astype(np.int32)
```
