[TOC]

# 对抗性样本

## lab

### 3_BruteForcing

* 模型：手写数据识别模型
* 目标: 从下图开始，生成可以被模型识别为4的图片（针对训练集的model inversion attack是从模型给出的结果，后验概率，开始推测输入的信息）

![fake_id](image/fake_id.png)

* 实际应用：对于一个能识别CEO的虹膜识别系统，假设已经知道CEO是蓝眼睛，可以从任意蓝眼睛开始，不断微调，直到模型将其识别为CEO
* 方法：下图所示，不断重复生成noise（对图片进行微调），直到模型认为该图片为4

![lab_solution](image/lab_solution.jpeg)

* 攻击结果

下图模型将其识别为4

![result](image/result.png)



## **EXPLAINING AND HARNESSING ADVERSARIAL EXAMPLES**

解释了对抗性样本产生的原因，给出了生成对抗性样本的方法，以及抵御对抗性样本的方法



### introduction

**对抗性样本**

* 正常样本中加入轻微扰动，这种扰动人类不能识别，但会造成神经网络误分类。这种加入扰动的样本被称为对抗性样本
* 下图中，正常样本为panda，加入扰动，分类器识别为gibbon，但肉眼看上去没有区别

![exp1](image/exp1.png)



### 对抗性样本的线形解释

**样本输入x**

* x的精度有限（一般为8bit，低于1/255的信息会被丢弃）
* 模型没办法区分 $x^{'} = x+ \eta$

**现考虑生成的对抗性样本和权重向量的点积**

* $w^Tx' = w^Tx + w^T\eta$ ,可以看出，加入对抗性扰动后，激活函数的输入会增加$w^T\eta$,作者提出令$\eta = sign(w)$，从而使$w^T\eta$最大化。假设$w$是一个$n$维向量，并且均值为$m$，那么激活函数的输入将增加$w^T\eta\le \epsilon mn$,显然这个增量与n线形相关
* 那么对一个高维的向量，一个样本的微小扰动会对输出产生巨大的影响



### 生成对抗性样本

**生成扰动的方法**
$$
\eta =  \epsilon sign(\nabla_xJ(\theta, x, y))  \  其中J(\theta, x, y)是模型代价函数
$$


**结果**

![exp2_result](image/exp2_result.png)



## ADVERSARIAL EXAMPLES IN THE PHYSICAL WORLD

在上一篇论文的基础上，提出新的对抗性样本生成方法，并在物理世界进行攻击

###Basic Iterative Method

![exp3_BIM](image/exp3_BIM.png)



### Iterative Least-Like Class Method(最不可能类)

其中$y_{LL}$是$x$最不可能的分类，$y_{LL} = argmin \{p(y|x)\}$

![exp4_ILL](image/exp4_ILL.png)





## Deep Neural Networks are Easily Fooled: High Conﬁdence Predictions for Unrecognizable Images（2015‘ CVPR）

### 问题

* 很多图片分类的模型有接近人类的能力，但计算机视觉与人类视觉有什么区别？
* 对图片进行人类不能识别，但机器可以使图片被正确划入其他类

![exp5](image/exp5.png)

* 本文是对计算机视觉应用的一个思考



### 产生对抗性样本的方法

使用Evolutionary Algorithm(EA)算法生成对抗性样本，有两种编码方式

**direct encoding**: 对于MNIST的28*28像素，每个像素点对应一个灰度值；对于ImageNet的256$\times$256像素，每个像素点对应(H、S、V)，每个像素点在[0,255]内用一个uniform random noise初始化。选择变异的数量是10%，每1000次迭代下降一半。经过200次迭代

![exp6_direct_encode](image/exp6_direct_encode.png)

**indirect encoding**:采用compositional pattern-producing network(CPPN，产生复杂的，规则的，类似人造或自然的图像)

![exp7_indirect_encode](image/exp7_indirect_encode.png)



###两种对抗性干扰在ImageNet上的结果

**Figure 6**是direct encoding的结果：可以看出这种方式即使迭代20000次产生的图片，模型依然不能给出很高的置信率（模型“不认识”迭代产生的图片）

**Figure7**是indirect encoding的结果: 可以看出这种方式产生的图片，模型能以很高的置信率正确分类（但在156～286中<这些类别是猫&狗>的置信率不高，一个可能的解释是这些类别的训练集数据多，overfit较轻，模型更难被对抗性样本欺骗），如果这个解释正确，那么意味着更大的训练集能解决这个问题

![exp8_imagenet_result](image/exp8_imagenet_result.png)

CPPN结果在ImageNet上的一些说明

* 第一张图中，海星图像包含蓝色的水和橙色的海星，棒球有红色拼接在白色背景的特征，遥控器有一系列button…DNN其实大致就是根据这些特征对图像进行分类。
* EA算法只需要生成DNN分类所需要的特征（该类特有或有区别的特征）而不是全部特征即可

CPPN产生的结果有重复多次的特征，作者移除一些重复元素观察是否会对置信率产生较大影响，发现重复会提高置信率

![exp9_cppn](image/exp9_cppn.png)





