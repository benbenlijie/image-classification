## Image classification with CIFAR-10 (Udacity task)  

模型使用Tensorflow来进行搭建，使用了CNN，max_pool，avg_pool，fully_connect以及dropout等层。目前训练之后模型在测试集上的accuracy是63%。进行寻找突破改进模型。

### 模型说明
输入是大小为32x32的彩色图片，channel为3。将图片输入到模型之前，先进行normalize，以及将label进行one-hot编码处理。    
模型组成：

| 输入32x32x3 |
| ----- |
| 3x3 conv. 96 ReLU |
| 2x2 max-pooling stride 2|
| 3x3 conv. 192 ReLU|
| 2x2 avg-pooling stride 1|
| 3x3 conv. 192 ReLU|
| 2x2 avg-pooling stride 1|
| flatten |
| 64 fully-connect ReLU |
| dropout |
| 10-way softmax|

#### 心得

1. 在把full-connect层的激活函数从sigmoid更换成ReLU之后，模型进行训练出现停滞，loss一直没有减少。原因是ReLU激活函数可能会使得节点die。这里就需要小心地设置权重的初始值，以及设置较小的learning-rate。
2. ReLU激活函数为模型引入了大量的稀疏性，加速了复杂特征解离，非饱和的宽广映射空间，加速了特征学习。
3. 在设置权重的初始值时，采用高斯分布，mean设置在0，随着层数添加，慢慢增大stddev，最低层的conv采用的0.001，逐层加到0.01，这样效果比较好。
4. pool层采用了max+avg+avg，avg采用重叠采样，即将stride设为1。

----

### 2017-06-08提升结果

根据AlexNet的原理对模型结构进行了调整，使得test accuracy提升到了73.6%，提升效果还是很显著的。
修改后的结构是：

| 输入32x32x3 |
| ----- |
| 5x5 conv. 64 stride 1 ReLU |
| 3x3 max-pooling stride 2|
| 5x5 conv. 64 stride 1 ReLU|
| 3x3 max-pooling stride 2|
| flatten |
| 384 fully-connect ReLU |
| 192 fully-connect ReLU |
| dropout |
| 10-way softmax|

### 对ReLU的进一步认识

这几天在查阅了ReLU的各种资料之后，对ReLU有了更进一步的了解。ReLU的优势是在模型的前向与后向的计算中，不存在像sigmoid/tanh这类饱和函数，存在梯度传递问题，但是它又缺少了对数据界限的控制。
相比较的特点如下：
1. sigmoid在压缩数据幅度方面更强，可保证在深度网络中，数据幅度可控
2. sigmoid存在梯度消失问题，训练速度比ReLU慢
3. ReLU没有对数据进行压缩，随着网络层级的加深，数据幅度会越来越大，最终影响模型的表现
为了发挥ReLU的优势，同时使得经过计算后的数据幅度不扩散（即输出数据的方差==输入数据的方差），有以下三种方法：
1. 数据预处理，将数据归一化
2. 权重初始化，xavier初始化
    定义参数所在层的输入维度为n，输出维度为m，则权重将以均匀分布的方式在[-sqrt(6 / (m+n)), sqrt(6 / (m+n))]范围之内进行初始化
3. batch normalization
