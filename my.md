
least square solution to a linear matrix equation
a: coefficient matrix

>>> import numpy as np
>>> x=np.array((0,1,2,3))
>>> y=np.array((-1,0.2,0.9,2.1))
>>> A=np.vstack((x,np.ones(len(x)))).T
>>> A
array([[ 0.,  1.],
       [ 1.,  1.],
       [ 2.,  1.],
       [ 3.,  1.]])
>>> m,c=np.linalg.lstsq(A,y)[0]
>>> import matplotlib.pyplot as plt
>>> plt.plot(x,m*x+c,'r',label='Fitted line')
[<matplotlib.lines.Line2D object at 0x10ec5ddd0>]
>>> plt.plot(x,y,'o',label='Original data',markersize=10)
[<matplotlib.lines.Line2D object at 0x10ec60090>]
>>> plt.show()
