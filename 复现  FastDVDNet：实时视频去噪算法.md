## 复现 | FastDVDNet：实时视频去噪算法

张斌 [CVer](javascript:void(0);) *7月13日*

点击上方“**CVer**”，选择加"星标"或“置顶”

重磅干货，第一时间送达![img](https://mmbiz.qpic.cn/mmbiz_jpg/ow6przZuPIENb0m5iawutIf90N2Ub3dcPuP2KXHJvaR1Fv2FnicTuOy3KcHuIEJbd9lUyOibeXqW8tEhoJGL98qOw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 作者：张斌
>
> https://blog.csdn.net/weixin_30793735/article/details/95353530 



**0.GitHub**



已复现 FastDVDNet: https://github.com/z-bingo/FastDVDNet



**1. Introduction**



![img](https://mmbiz.qpic.cn/mmbiz_png/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xWrmASz79HibnKPtibXztDGHmEcefwQ1t8ukrwhTkHVodIhdKUCYjvopA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

《FastDVDnet: Towards Real-Time Video Denoising Without Explicit Motion Estimation》

arXiv：https://arxiv.org/abs/1907.01361



FastDVDNet是一种视频去噪中的STOA方法，与其他STOA方法有着相近或者更好的性能，但是有着更低的时间复杂度。



计算机视觉中，对于视频去噪的研究相对较少，大多方法还是基于传统的算法，如VBM4D等non-local的方法，还有一些方法是图像去噪方法的简单扩展。由于视频有着较强的时间相关性，那么一个好的视频去噪算法必将要充分利用这一特点。利用时间相关性主要体现为两个方面：



对于给定的patch，不仅要在同一帧的相邻区域搜索像素的patch，也要在时间相近的frame上进行搜索；



使用相邻时间的frame还可以有效减少flockering，因为每一帧之间的残余就会是相关的。



为了解决motion带来的对齐困难问题，DVDNet中使用光流进行了显式的估计，但是光流的计算是比较耗时的，即便是快速算法也是如此。对于encoder-decoder结构的U-Net，其本身具有在感受也范围内对齐的功能，因此，在FastDVDNet中采用了这种做法，也就提高了性能。



**2. Network Architecture**



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xRsibUT1jFjMViaky5WiaMErSAibrR1qGWOeV1yaoxaFC9ocBa9u8OSSlgg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Fig. 1是FastDVDNet是结构图，通常连续5帧和一个噪声的估计一起作为网络的输入，从网络的top view来看FastDVDNet是一个two-stage的结构，5帧图像分为三组作为Block1的输入，三个Block1的输出又作为Block2的输入。其中，三个Block1共享参数，Block1和Block2有着相同的结构，如Fig. 1(b)，是一个修改版的U-Net网络。值得注意的是，相较于original U-Net，此处的网络有2个下采样层（U-Net有4个），且下采样并非通过pool来实现，而是通过stride为2的Conv层实现的；此外，上采样也没有通过Bilinear插值或deconv来实现，而是通过PixelShuffle来实现。相较于DVDNet，FastDVDNet的结构就非常简单了。



**3. Loss Function**



在图像/视频去噪领域中，L1 Loss使用较多，因为L1 Loss可以保护去噪后图像的整体信息；较为不同的是，FastDVDNet使用了L2 Loss。



**4. Results and Analysis**



已经使用PyTorch复现了改论文，没有使用文中使用的DAVIS数据集，而是使用了Vimeo-90K数据集，该数据集专用于图像增强等领域，包含了接近90K组图像帧，每组数据为7帧，每帧图像分辨率为448*256。目前，网络正在训练中，后续(本月中旬)会将训练好的模型上传至github，代码现已开源至我的github，欢迎各位批评指正！



由于训练尚未结束，暂不讨论该模型在Vimeo-90K数据集上的表现能力，先讨论其在DAVIS数据集上、噪声为加性高斯噪声(AWGN)时的去噪性能。



**4.1 two-stage结构必要性**



如Fig. 1所示，FastDVDNet采用了two-stage结构，连续5帧图像首先分为三组右Block 1提取特征，进而，三个共享参数的Block 1的输出作为Block 2的输出进一步地提取特征、去噪。若该two-stage结构对于去噪任务是冗余的，那么，将two-stage结构改为single-stage后，模型性能应几乎保持不变。文中给出了相关的数据说明，假设FastDVDNet采用Fig. 3所示的single-stage结构，即，五帧图像连接在一起作为一个Block模块的输入，这无疑在很大程度上减少了学习参数，但Table 1中的数据表明，参数减少带来的结果是性能下降。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xvnrHAsAqKq0kXtcOGQcrFgYW8DKf4KysH8sETicI2FKOc0ZIibmz5fdA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xmmR3WgM1geGSMQR9hHmN4x7MEpgmxzAv6z6kU180te2XONBiaZRX0LA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**4.2 encoder-decoder结构必要性**



encoder-decoder结构是一种典型的multi-scale结构，可以在不同的scale提取图像的特征，增大图像感受野。在近年来对于图像去噪的研究中，encoder-decoder结构的模型也占了多数，但是也不乏有single-scale结构的网络，如经典的DnCNN也有着很好的性能。文中，也通过实验来证明了encoder-decoder结构的必要性，实验结果如Table 2所示。



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xyKflVA9fphpoF7wuMz3kovlxAXiaD9anjM1evAQJTZmJhsZrJv6CBpg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



4.3 结果分析



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xMSNEHWG9MGXnGQZRm4ALV7xp5G3ic0wZKofcSeKuTA8UEH4jH2J022w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xMxoxxeZhXLe0aOUolyRetNLCpvQhDJnXzgf8hmiaice6oEpr6dKgicS0A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oXVF6tCiccsl1dIoZkUVZt8xOiaYoMZGNqB3iaa5Mz3YQzaCc1a5qzLwH9CbwrDpHWtWbpyibgN4dUhZQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Fig. 4 ~ Fig. 6是几组不同算法和FastDVDNet的对比图，Fig. 4中(d)为FastDVDNet的去噪结果图，Fig. 5和Fig. 6中(h)为FastDVDNet的去噪效果图，将图片方法观察其细节，可见，DVDNet和FastDVDNet去噪后的图像细节保存较好，边缘比较平滑。



**References**

[1] FastDVDNet:ToWards Real-Time Video Denoising Without Explicit Motion Estimation

[2] DVDNet: A Fast Network For Deep Video Denoising

[3] PyTorch

[4] Vimeo-90K