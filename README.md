

> 来源：晓飞的算法工程笔记 公众号，转载请注明出处


**论文: CNN Mixture\-of\-Depths**


![](https://developer.qcloudimg.com/http-save/6496381/1415ee485689ba84af6972892f15844e.png)


* **论文地址：[https://arxiv.org/abs/2409\.17016](https://github.com)**


# 创新点




---


* 提出新的卷积轻量化结构`MoD`，在卷积块（`Conv-Blocks`）内通过动态选择特征图中的关键通道进行集中处理，提高效率。
* `CNN MoD`保留了静态计算图，这提高了训练和推理的时间效率，而且不需要定制的`CUDA`内核、额外的损失函数或微调。
* 通过将`MoD`与标准卷积交替使用，能够实现相等性能下的推理加速或相等推理速度下的性能提高。


# CNN Mixture\-of\-Depths




---


![](https://developer.qcloudimg.com/http-save/6496381/cd8928c21cae8398e105d828ca120e6e.png)


`MoD`由三个主要组件组成：


1. 通道选择器：根据输入特征图与当前预测的相关性选择前 k 个最重要的通道。
2. 卷积快：从现有架构（如`ResNets`或`ConvNext`）中进行改编，旨在增强选定通道的特征。
3. 融合算子：将处理后的通道加到特征图的前 k 个通道上。


## 通道选择器


![](https://developer.qcloudimg.com/http-save/6496381/5758ff9b6677ee613fe1ad4220af2cc9.png)


通道选择器主要分为两个阶段：


1. 自适应通道重要性计算：通过自适应平均池化压缩输入特征图，随后通过一个具有瓶颈设计的两层全连接网络进行处理，设定 r\=16 ，最后通过`sigmoid`激活函数生成一个分数向量 s∈RC ，量化了相应通道的重要性。
2. `Top-k`通道选择与路由：利用重要性分数 s 选择前 k 个通道输入卷积块处理，原始特征图 X 则直接传递融合算子。


这个选择过程使得通道选择器能够高效地管理计算资源，同时保持固定的计算图，从而实现动态选择要处理的通道。


## 动态通道处理


每个卷积块中处理的通道数量 k 由公式 k\=⌊Cc⌋ 决定，其中 C 表示该块的总输入通道数， c 是一个超参数，用于确定通道减少的程度。例如在一个标准的`ResNet`瓶颈块中，通常处理`1024`个通道，设置 c\=64 会将处理减少到仅`16`个通道（ k\=16 ）。


通过实验发现，超参数 c 应设置为第一卷积块中输入通道的最大数量，并在整个`CNN`中的每个`MoD`块中保持相同。例如，`ResNet`的 c\=64 `MobileNetV2`的 c\=16 。


卷积块的最后一步涉及将处理后的通道与从自适应通道重要性计算中获得的重要性评分相乘，确保在训练过程中梯度能够有效地传递回通道选择器，这是优化选择机制所必需的。


## 融合机制


将处理后的特征添加到 X 的前 k 个通道中，保留其余未处理的通道。融合后的特征图 ˉX 具有与原始输入 X 相同的通道数 C ，从而保留了后续层所需的维度。


论文在实验中测试了多种将处理后的通道重新集成到特征图 X 中的策略，包括将处理后的通道添加回其原始位置，但结果并未显示任何改进。实验表明，始终在特征图中使用相同位置来处理信息似乎是有益的，将处理后的通道添加到后 k 个通道中得到了与添加到前 k 个通道时相当的结果。


## 集成到`CNN`结构


`MoD`可以集成到各种`CNN`架构中，例如`ResNets`、`ConvNext`、`VGG`和`MobileNetV2，`这些架构被组织成包含多个相同类型（即输出通道数相同）的卷积块（`Conv-Blocks`）的模块。


实验表明，交替使用`MoD`块和标准卷积块在每个模块中是一种最有效的集成方法。需要注意的是，`MoD`块替换每第二个卷积块，从而保持原始架构的深度（例如，`ResNet50`中的`50`层）。每个模块以一个标准块开始，例如`BasicBlock`，然后是一个`MoD`块。


这种交替模式表明，网络能够处理显著的容量减少，只要定期进行全容量卷积。此外，该方法确保`MoD`块不会干扰通常发生在每个模块的第一个块中的空间维度缩减卷积。


# 主要实验




---


![](https://developer.qcloudimg.com/http-save/6496381/96bb9a8df0c8fb76489f2661c130aa41.png)


![](https://developer.qcloudimg.com/http-save/6496381/6c254503f48cb2ee5659ad19b1122ac4.png)


 
 
 



> 如果本文对你有帮助，麻烦点个赞或在看呗～
> 更多内容请关注 微信公众号【晓飞的算法工程笔记】


![work-life balance.](https://upload-images.jianshu.io/upload_images/20428708-7156c0e4a2f49bd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 本博客参考[楚门加速器p](https://tianchuang88.com)。转载请注明出处！
