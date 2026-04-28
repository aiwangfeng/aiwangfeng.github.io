# [numpy.ndarray.flatten](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.flatten.html)

## ndarray.flatten(order='C')
     Return a copy of the array collapsed into one dimension.

```
>>> import numpy as np
>>> a = np.array([[1,2], [3,4]])
>>> a.flatten()
array([1, 2, 3, 4])
>>> a.flatten('F')
array([1, 3, 2, 4])
```
# [numpy.ravel](https://numpy.org/doc/stable/reference/generated/numpy.ravel.html)

## numpy.ravel(a, order='C')
     Return a contiguous flattened array.

```
>>> import numpy as np
>>> x = np.array([[1, 2, 3], [4, 5, 6]])
>>> np.ravel(x)
array([1, 2, 3, 4, 5, 6])

>>> x.reshape(-1)
array([1, 2, 3, 4, 5, 6])

>>> np.ravel(x, order='F')
array([1, 4, 2, 5, 3, 6])
```

# [numpy.stack](https://numpy.org/doc/stable/reference/generated/numpy.stack.html)

## numpy.stack(arrays, axis=0, out=None, *, dtype=None, casting='same_kind')
    Join a sequence of arrays along a new axis.

```
>>> a = np.array([1, 2, 3])
>>> b = np.array([4, 5, 6])
>>> np.stack((a, b))
array([[1, 2, 3],
       [4, 5, 6]])
```

# [numpy.split](https://numpy.org/doc/stable/reference/generated/numpy.split.html)

## numpy.split(ary, indices_or_sections, axis=0)
     Split an array into multiple sub-arrays as views into ary.

```
>>> import numpy as np
>>> x = np.arange(9.0)
>>> np.split(x, 3)
[array([0.,  1.,  2.]), array([3.,  4.,  5.]), array([6.,  7.,  8.])]
```

```
>>> x = np.arange(8.0)
>>> np.split(x, [3, 5, 6, 10])
[array([0.,  1.,  2.]),
 array([3.,  4.]),
 array([5.]),
 array([6.,  7.]),
 array([], dtype=float64)]
```