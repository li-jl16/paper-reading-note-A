##### 1. [NMS by Representative Region : Towards Crowded Pedestrian Detection by Proposal Pairing](https://arxiv.org/pdf/2003.12729.pdf)
- 使用pedestian的可见区域（visiable region）q去做NMS，加了一个检测可见区域的分支，需要训练
##### 2. [Double Anchor R-CNN for Human Detection in a Crowd](https://arxiv.org/pdf/1909.09998.pdf)
- 与【1】思路类似，用的不是visable region而是head的bbox
##### 3. [Towards Accurate One-Stage Object Detection with AP-Loss](http://openaccess.thecvf.com/content_CVPR_2019/papers/Chen_Towards_Accurate_One-Stage_Object_Detection_With_AP-Loss_CVPR_2019_paper.pdf)
- 基于prediction的排序提出的loss，直接以score的排序（决定AP值）为优化目标。展现了利用sample之间相比较的互信息的有效性
##### 4. [FCOS: Fully Convolutional One-Stage Object Detection](https://arxiv.org/pdf/1904.01355.pdf)
- anchor-free的算法，逐像素预测bbox的想法和用cetreness对偏远的bbox降低confidence的想法很有启发性
