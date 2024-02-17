# PCA Notes :

##### Definition :
- An automatic dimensionality reduction algorithm.it is one of the unsupervised machine learning methods that finds a linear projection of the original data that captures as much variance as possible.
- It is a lossy algorithm since the encoded data loses some information

##### Use cases:
- Statistical: fewer dimension → better generalization (used as preprocessing
step for other algorithms)
- Visualization: understand the structure of data
- Interpolation
- Computational: compress data → time/space efficiency
- Anomaly detection: describe normal data, detect outliers

##### Mathematical Proof:
![[Pasted image 20240102135550.png]]



##### Steps to Compute :
- convert data to vector 
- center data-points by subtract mean (u) of data 
- compute covariance to get eigenvalues and eigenvectors 
- pick K eigen values (k is hyperparameter)
- project data points to these eigenvectors 
- ![[Pasted image 20240102140154.png]]
##### Important notes :

