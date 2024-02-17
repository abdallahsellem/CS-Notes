### Common Models 
#### SeNet Architecture 
this architecture extends **inception** and **resnet** models and boosts their performance 
SeNet add a small neural network called SE block which analysis the output of the unit that attached to focusing on the depth dimension and learns **which features are usually most active together and it uses this information to re-calibrate feature maps** ![[Pasted image 20240206111323.png]]


#### Xception 
a model that merges the idea of GoogleNet and ResNet , it replace the inception module with a module called depth wise separable convolution layer 
while a regular CNN try to capture spatial patterns using filters(kernels) like mouth , nose , etc . depth wise separable module assume that spatial patterns and cross channel patterns can be module separately 
and it consist of 2 parts 
- first part for spatial patterns 
- second part for cross channel patterns 
(**note**): don't use depth wise separable module after layers that has few channels like input layers 
(**note**): separable convolution layers are less memory and fewer computation and do better so we should consider using them by default
(**note**): it take each channel do some operations on the separably then concatenate them 
![[Pasted image 20240206113002.png]]
GlobalAvarage2d is a module that take the average of all pixels of a feature map and convert it to a single value , so if we applied it to 252 feature maps it will produce 252 values **( we can use it instead of *flatten* in some cases)**