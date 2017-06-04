##Image classification with CIFAR-10 (Udacity task)
模型使用Tensorflow来进行搭建，使用了CNN，max_pool，avg_pool，fully_connect以及dropout等层。目前训练之后模型在测试集上的accuracy是63%。进行寻找突破改进模型。
###模型说明
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

####心得
1. 在把full-connect层的激活函数从sigmoid更换成ReLU之后，模型进行训练出现停滞，loss一直没有减少。原因是ReLU激活函数可能会使得节点die。这里就需要小心地设置权重的初始值，以及设置较小的learning-rate。
2. ReLU激活函数为模型引入了大量的稀疏性，加速了复杂特征解离，非饱和的宽广映射空间，加速了特征学习。
3. 在设置权重的初始值时，采用高斯分布，mean设置在0，随着层数添加，慢慢增大stddev，最低层的conv采用的0.001，逐层加到0.01，这样效果比较好。
4. pool层采用了max+avg+avg，avg采用重叠采样，即将stride设为1。

