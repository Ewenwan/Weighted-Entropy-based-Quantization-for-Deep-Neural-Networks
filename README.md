# 基于加权熵对深度神经网络进行量化的方法

[原文链接](http://openaccess.thecvf.com/content_cvpr_2017/papers/Park_Weighted-Entropy-Based_Quantization_for_CVPR_2017_paper.pdf)

作者：

* Eunhyeok Park\(canusglow@gmail.com\)
* Junwhan Ahn § \(junwhan@snu.ac.kr\)
* Sungjoo Yoo \(sungjoo.yoo@gmail.com\)
* 首尔国立大学：计算和内存架构实验室 和 §自动化设计实验室

## 摘要

当使用嵌入式或移动设备进行开发时，由于硬件资源的限制，将原始神经网络进行量化\(优化前向预测的开销\)成为一种高效的优化手段。量化时需要对其进度损失进行严格的控制。本文旨在提供一种新颖的量化权重和激活输出的方式\(基于加权熵\)。与二值神经网络不同，我们的方法是进行多位\(bit\)量化，也就是将权重和激活输出可以被量化到对应位数，从而达到对应的精度。这有利于更灵活地利用不同级别的量化，达到精度和性能的折衷。此外，我们的方案提供了一种基于传统训练算法的自动化量化流程，从而节省了为量化网络所花费的时间。根据对实际神经网络模型的评估：图像分类\(_AlexNet_, _GoogLeNet_ 和 _ResNet-50/101_\)，物体检测\(基于_ResNet-50_的_R-FCN_\)和语言建模\(一种 _LSTM_ 网络\)，我们的方法在损失很小精度的前提下，对模型大小和计算量都有大幅度的消减。同样，与目前已知的量化方式相比，我们的方式在工作量更低和相同的资源约束的前提下，具有更高的精度。

- gitbook 在线阅读：https://www.gitbook.com/book/chenxiaowei/weighted-entropy-based-quantization