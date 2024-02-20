**weight decay :** involves adding a penalty term to the loss function during training. The penalty is proportional to the sum of the squared values of the weights in the network. The goal is to encourage the model to use smaller weights, which can help prevent the model from fitting the training data too closely and overemphasizing small details that may be noise.

Mathematically, the loss function with weight decay is modified as follows:
$$\text{Loss} = \text{Original Loss} + \lambda \sum_{i} w_i^2$$
**Batch normalization (BatchNorm)** : is a technique used in neural networks to improve the training stability and speed up convergence. It normalizes the inputs of a layer by subtracting the mean and dividing by the standard deviation, and then scales and shifts the normalized values using learnable parameters. This helps to mitigate issues like vanishing or exploding gradients and can improve the training of deep networks.
$$\text{BatchNorm}(x) = \gamma \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta$$
