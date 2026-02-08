

今天无聊看到了ring-allreduce的gpu通信算法，本来希望在[相关网页](http://research.baidu.com/bringing-hpc-techniques-deep-learning/)上看看相关介绍的，但是在baidu research上却搜不到相关的东西，后来看了看[baidu-allreduce](https://github.com/baidu-research/baidu-allreduce)代码的注释，才明白。这是一个说明起来挺简单的算法，自己打算随便说说。

觉得英文好的可以直接看看GitHub上注释，写的很清晰：[https://github.com/baidu-research/baidu-allreduce/blob/master/collectives.cu#L156](https://github.com/baidu-research/baidu-allreduce/blob/master/collectives.cu#L156)

- _如无特殊说明，本博客的图片都来自于[知乎上的一个回答](https://www.zhihu.com/question/57799212/answer/292494636?utm_source=ZHShareTargetIDMore&utm_medium=social&utm_oi=37729630945280)，主要原因是baidu research上找不到这篇文章，所以相关的图例只在这个知乎回答里找到了_*

一般的多卡gpu训练有一个很大的缺陷，就是因为每次都需要一个gpu从其他gpu上收集训练的梯度，然后将新的模型分发到其他gpu上。如下图：

![](attachments/Pasted%20image%2020250504005120.png)

这样的模型最大的缺陷是gpu 0的通信时间是随着gpu卡数的增长而线性增长的。所以就有了ring-allreduce，如下图：

![](attachments/Pasted%20image%2020250504005139.png)
该算法的基本思想是取消Reducer，让数据在gpu形成的环内流动，整个ring-allreduce的过程分为两大步，第一步是scatter-reduce，第二步是allgather。

先说第一步：首先我们有n块gpu，那么我们把每个gpu上的数据（均等的）划分成n块，并给每个gpu指定它的左右邻居（图中0号gpu的左邻居是4号，右邻居是1号，1号gpu的左邻居是0号，右邻居是2号……），然后开始执行n-1次操作，在第i次操作时，gpu j会将自己的第(j - i)%n块数据发送给gpu j+1，并接受gpu j-1的(j - i - 1)%n块数据。并将接受来的数据进行reduce操作，示意图如下：

![](attachments/Pasted%20image%2020250504005156.png)
当n-1次操作完成后，ring-allreduce的第一大步scatter-reduce就已经完成了，此时，第i块gpu的第(i + 1) % n块数据已经收集到了所有n块gpu的第(i + 1) % n块数据，那么，再进行一次allgather就可以完成算法了。

第二步allgather做的事情很简单，就是通过n-1次传递，把第i块gpu的第(i + 1) % n块数据传递给其他gpu，同样也是在i次传递时，gpu j把自己的第(j - i - 1)%n块数据发送给右邻居，接受左邻居的第(j - i - 2) % n数据，但是接受来的数据不需要像第一步那样做reduce，而是直接用接受来的数据代替自己的数据就好了。

最后每个gpu的数据就变成了这样：

![](attachments/Pasted%20image%2020250504005208.png)
如果觉得不懂的话，我们举一个3gpu的例子：

首先是第一步，scatter-reduce：

![](attachments/Pasted%20image%2020250504005228.png)

然后是allgather的例子：

![](attachments/Pasted%20image%2020250504005244.png)

reference:

[https://github.com/baidu-research/baidu-allreduce](https://github.com/baidu-research/baidu-allreduce)

[https://www.zhihu.com/question/57799212/answer/292494636?utm_source=ZHShareTargetIDMore&utm_medium=social&utm_oi=37729630945280](https://www.zhihu.com/question/57799212/answer/292494636?utm_source=ZHShareTargetIDMore&utm_medium=social&utm_oi=37729630945280)

## 引用本文

APA：
Yu, J. (2018年5月24日). ring-allreduce 简介. Jiachen Yu. https://www.yujiachen.com/ring-allreduce/zh/

BibTeX：
```bibtex
@online{yu2018ringallreduce,
  author = {Yu, Jiachen},
  title = {ring-allreduce 简介},
  year = {2018},
  publisher = {Jiachen Yu},
  url = {https://www.yujiachen.com/ring-allreduce/zh/},
  urldate = {2026-02-08},
}
```
