### 已读列表：
##### 1. [NMS by Representative Region : Towards Crowded Pedestrian Detection by Proposal Pairing](https://arxiv.org/pdf/2003.12729.pdf)
- 使用pedestian的可见区域（visible region）q去做NMS，加了一个检测可见区域的分支，需要训练
##### 2. [Double Anchor R-CNN for Human Detection in a Crowd](https://arxiv.org/pdf/1909.09998.pdf)
- 与【1】思路类似，用的不是visable region而是head的bbox。by the way, 1和2都还没能发表，可能是一篇18年的文章已经用过这个思路了，[Bi-box Regression for Pedestrian Detection and
Occlusion Estimation](http://openaccess.thecvf.com/content_ECCV_2018/papers/CHUNLUAN_ZHOU_Bi-box_Regression_for_ECCV_2018_paper.pdf)，这篇文章同时预测了full body 和visible part，粗略浏览了一下没有细看
##### 3. [Towards Accurate One-Stage Object Detection with AP-Loss](http://openaccess.thecvf.com/content_CVPR_2019/papers/Chen_Towards_Accurate_One-Stage_Object_Detection_With_AP-Loss_CVPR_2019_paper.pdf)
- 基于prediction的排序提出的loss，直接以score的排序（决定AP值）为优化目标。展现了利用sample之间相比较的互信息的有效性
##### 4. [FCOS: Fully Convolutional One-Stage Object Detection](https://arxiv.org/pdf/1904.01355.pdf)
- anchor-free的算法，逐像素预测bbox的想法和用cetreness对偏远的bbox降低confidence的想法很有启发性
##### 5.[Pixel Consensus Voting for Panoptic Segmentation](https://arxiv.org/pdf/2004.01849.pdf)
- 这篇文章真的不是很好理解，看了很多遍。
  首先，大的框架是语意分割和实例分割分开做，有两个分支。一个分支只管预测每个pixel的classification，一个分支只管将每个pixel分配给一个instance，不区分instance的类别；都完成了以后，直接取每个instance中占多数的类别为instance分类。
  然后，这篇文章主要的工作还是在instance分支上，其中有几个最重要的步骤：先通过卷积，获得图片上每个pixel所属instance的中心分别位于周围233个bins中的概率，训练过程到这一步就结束了。具体做法是，233个周围的bins被划分为233个输出通道，用cross entropy loss训练。每个输出通道具体对应周围哪个bins，是用所谓的voting filter记录的。
  后面的过程就没有需要学习的参数了。上一步得到每个pixel所属instance的center point在周围233个bins中概率后，再用1-hot的反卷积，得到每个bins确实是某个instance的中心的概率和，即所有相应pixel在这个bins上的概率的相加，得到了一张heatmap。概率和大于threshhold=4的bins就会被认为是某个instance的中心，会得到一个一个的centre region；然后再将每个centre region与当时用来计算概率和的每个pixel做比对，确定每个pixel应该从属于哪个centre region，实例分割也就完成了。具体过程，他是用voting filter在空间上倒过来，获得的一个map关系，将每个bins又对应回了每个pixel；
  文章还有很多细节，需要进一步看代码来确定，目前github找不到官方代码。但是整体这个思想确实是很新颖，借鉴了hough变换的投票机制，很有启发，但是怎么借鉴到检测任务，怎么能帮助去除NMS，需要消化一下，进一步思考。至少它这个获得heatmap的方式，其实是一种attention机制，attention机制用到NMS之前的步骤的工作有一些，用到NMS上的似乎还没见过？
  好评，很solid工作，传统CV艺能与CNN结合的好想法。目测当时这参也不好调吧。这个投票机制既然能选实例中心，也可以用来选其它feature，这是一个借鉴的路径。
##### 6.[Mask-Guided Attention Network for Occluded Pedestrian Detection](http://openaccess.thecvf.com/content_ICCV_2019/papers/Pang_Mask-Guided_Attention_Network_for_Occluded_Pedestrian_Detection_ICCV_2019_paper.pdf)
- 想法比较简单，将整个visible region与其它部分二值化，作为监督去训练一个可见区域的分割，然后拿得到的分割概率图，作为attention，在分类前直接点乘了原来的feature map，去抑制非人形物体的特征，降低FPR。
##### 7.[Bounding Box Regression with Uncertainty for Accurate Object Detection](http://openaccess.thecvf.com/content_CVPR_2019/papers/He_Bounding_Box_Regression_With_Uncertainty_for_Accurate_Object_Detection_CVPR_2019_paper.pdf)
- 文章的related work部分提到了很多做NMS和BBOX修正的文章，是个不错的review列表。这篇文章的理论上的fomulation很好看，其实是把预测看成了一个概率事件，比如你预测的bbox的左上角的x坐标值是150，那请问你能不能告诉我，你估计你预测的这个150就是GT值的概率是多少？它旁边的149是GT的概率是多少，151是GT的概率是多少？这就需要你给出一个概率分布来。那怎么得到预测的概率分布呢？作者是这样做的：假设预测的四个位置参数是独立分布的高斯函数，把groudtruth作为狄拉克分布（常用办法），用KL距离去训练，minimize这两个分布，获得一个对variance的预测。这样每次前向过程不仅能输出预测的location，也能输出这个location的概率。然后再定义一个置信函数，利用预测的概率（实际只用variance即可）结合周围的bbox远近，用所有周围bbox的一个加权，去修正某一个bbox。这个过程放在了NMS里。这个想法不错，先记着，看看能不能在去掉NMS之后还能用到类似的想法。
##### 8.[Learning Deep Features for Discriminative Localization](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Zhou_Learning_Deep_Features_CVPR_2016_paper.pdf)
- 四年前的老paper了，跟resnet是一届的，没什么太多可以借鉴的地方，看起来像attention的的图也只不过是特征图upsamling了一下，按照最后一次特征图相加时的权重去加权求和，然后在原图上做了visualization
##### 9.[Non-local Neural Networks](http://openaccess.thecvf.com/content_cvpr_2018/papers/Wang_Non-Local_Neural_Networks_CVPR_2018_paper.pdf)
- 这篇文章真的让我有一种万剑归宗的感觉。特征是什么呢？一个东西走起来像鸭子，叫起来像鸭子，那它就是一只鸭子。特征，就是观察者认为是同一类的事物所应该共同拥有的东西。那么，假设现在有一堆varible，x1,x2,x3,...,xn,他们的共同的feature从信息论的角度讲，是他们的mutual information。independent viriable之间的mutual information是0，只要这个值不是0，那么这两个变量之间必然有概率上的联系。我们要找的就是这个信息。假设你现在有了一个特征提取函数f(x),最佳的特征就是使得f(x1),f(x2),....f(x3)这些变量之间都有最大的互信息。而deep learning，就是学习这个f(x)的过程。kaiming这篇non local nerual networks干的就是这件事情，只不过用神经网络实现了它。这里每个variable就是固定时空域的一个样本数据，比如某一帧画面上的一个像素，non local的操作$y_i=(1/C(x))*(sum_j(f(xi,xj))*xi)$其实就是在提取这样的互信息。
##### 10.[Self-supervised Equivariant Attention Mechanismfor Weakly Supervised Semantic Segmentation](https://arxiv.org/pdf/2004.04581.pdf)
- 亮点，1加了一个affinity的监督，2对CAM加了一个让correlation最大化的self-attion机制(对【9】的一个针对弱监督场景的改造)

### 未读列表
##### 11.[Bridging the Gap Between Anchor-based and Anchor-free Detection viaAdaptive Training Sample Selection](https://arxiv.org/pdf/1912.02424.pdf)
##### 12.[Distance-IoU Loss: Faster and Better Learning for Bounding Box Regression](https://arxiv.org/pdf/1911.08287.pdf)
##### 13.[DR Loss: Improving Object Detection by Distributional Ranking](https://arxiv.org/pdf/1907.10156.pdf)


