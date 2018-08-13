# sort翻译

这篇论文介绍了一个用tracking by detection模式解决多物体tracking(MOT)的良好的实现方法，在该模式中每一帧中detect的物体都用一个bounding box来表示。和很多batch based tracking（一个视频中的所有帧同时给出，可以进行批处理）方法[1][2][3]比较，本文介绍的方法主要针对在线tracking（比如tracking摄像头的数据，直播等），只有前面帧和当前帧被提供给tracker。

多物体追踪(MOT)问题也可以被看成数据关联问题，目标是关联一个视频序列当中的多个帧当中detections。为了实现数据关联，各种trackers用了很多方法给场景中的物体[5][3]和运动[1][4]来建模。本文采用方法的灵感来自最近的视觉MOT benchmark[6]。首先就包括了一些成熟的数据关联技术如MHT[7,3]和JPDA[2]，在MOT benchmark上都有其地位。其次，唯一一个没有用detector的tracker ACF[8]也在rank的前列这表示detector的质量拖了其他的一些tracker的后腿。此外在精度和速度之间的权衡也很明显，有些精度很高的Tracker根本就不满足实时应用的速度需求。通过和benchmark上的trackers的比较，显示了MOT问题的复杂性和性能水平。

和Occam’s Razor保持一致，在tracking中我们忽略了detection组件的不同表现，只用了bounding box的位置和大小来做运动估计和数据关联。物体的长期遮挡和短期遮挡也被我们忽略掉了，因为这种情况很少出现，而且处理这种情况给我们的tracking 框架引入了不必要的复杂性。加入这些重识别的处理会加重tracking框架的负载，可能会限制它在实时系统当中的应用。

这种设计理念和很多tracker不同，它们[9,10,11,12]添加了很多组件来处理各种边际情况和detection错误。我们专注于高效的处理帧与帧之间的关联，依赖于使用更高级的detector来避免detection错误而不是对detection的错误保持健壮性。通过比较ACF的行人detector[8]和最近的一个基于CNN的detector[13]可以证明这一点。此外我们用了两个经典有效的方法，卡尔曼滤波器[14]和Hungarian method[15]，来处理运动预测和数据关联的问题。这种简约的方法使得我们的方法在精度和效率上都有不错的表现。本文中的方法本来只用于行人的tracking，然而由于基于cnn的detector[13]的灵活性，也可以用来追踪其他的物体。

这篇论文的主要贡献是：<br>
1.我们在MOT领域中利用了基于CNN的detection的力量。<br>
2.提出了一种基于卡尔曼滤波器和Hungarian 算法的tracking方法，评估了最近的MOT的benchmark。<br>
3.代码将是开源的<br>

方法是由核心的detection组件将物体的状态传播到未来的帧，将当前的detections和现有的物体关联起来，并且管理被tracking的物体的生命周期。

3.1.detection<br>
为了利用基于CNN的detection的优点，我们引入了FrRCNN detection框架[13]。FrRCNN是端对端的框架，它包含了两个步骤。第一步是提取特征和目标区域，第二步是对目标区域中的对象进行分类。这样的优点是两个步骤的参数可以共享提高了detection的效率。另外网络结构可以随意改变，来提升detection的性能。

这里我们比较了FrRCNN支持的两种网络结构，分别是FrRCNN(ZF)和更深的FrRCNN(VGG16)。在这项工作中我们使用了PASCAL VOC挑战赛的FrRCNN的默认的训练好的参数。因为我们只关注行人，我们忽略了其他的物体种类，并且将概率在50%以上的detection结果传递给tracking框架。

![image][https://github.com/yunyuncc/paper_translation/blob/master/sort/img/img1.png]
