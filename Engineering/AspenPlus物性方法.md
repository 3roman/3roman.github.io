在化工流程模拟过程中，所执行的最主要的热力学计算是相平衡。而其中又以汽液相平衡最为主要。我们在Aspen Plus当中也可以看到非常多的物性方法供我们进行选择，其中主要有两大类：状态方程法和活度系数法。本期推送，我们就来揭开这两类方法的区别。

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/NRTL.png)

一个平衡系统的汽液相中对于每个组分最基本的关系，是组分在气相中的逸度等于在液相当中的逸度。组分的逸度既可以用逸度系数表示，也可以用活度系数来表示。对于气相而言，由于气相的活度系数关系式尚未建立，因此逸度常用逸度系数来表示。而对于液相而言，逸度既可以用逸度系数来表示，也可以用活度系数来表示。也正是因为液相中逸度的表达方式有两种，所以也就出现两种方法：状态方程法和活度系数法。
$$
f_{i}^{v}=f_{i}^{l}
$$
当汽液相逸度均使用逸度系数来进行表示时，如下图，那么根据气液相逸度相等（等式1和2左侧相等），便可得到第三个等式。

$$
\begin{aligned}
f_i^v=\varphi_i^vy_iP\\
f_i^l=\varphi_i^lx_iP\\
\varphi_i^vy_i=\varphi_i^lx_i
\end{aligned}
$$
公式中逸度系数的计算如下所示，可以发现逸度系数的计算与温度、压力和组成相关，需要依赖状态方程和混合规则，因此被称为状态方程法。
$$
\ln \varphi_i^{\alpha}=-\frac{1}{RT}\int_{\infty}^{V\alpha} [(\frac{\partial P}{\partial n_i})_{T,v,n}-\frac{RT}{V}]\rm dV-\ln Z_m^a
$$
Aspen Plus当中常用的状态方程法有Peng-Robinson、R-K-Soave等等，具体如下表所示：

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/thermodynamics_methods.webp)

对于活度系数法，气相逸度采用逸度系数表示，而液相逸度则采用活度系数来表示，如下图。同样根据气液相逸度相等（等式1和2左侧相等），便可得到第三个等式。其中左边包含了逸度系数，前面提到，需要依赖状态方程和混合规则计算。而右边包含了活度系数，活度系数的计算，是通过活度系数模型来得到的。
$$
\begin{aligned}
f_i^v=\varphi_i^vy_iP\\
f_i^l=x_ir_i^lf_i^{*,l}\\
\varphi_i^vy_iP=x_ir_i^lf_i^{*,l}
\end{aligned}
$$
活度系数模型分为两类，一类是经典模型，以vanLaar、Margules方程为代表，对于较简单的系统能获得较理想的结果。第二类是20世纪60年代以后从局部组成概念发展起来的活度系数模型，典型代表有Wilson和NRTL方程等等。能从较少的特征参数关联或推算混合物的相平衡，特别是对于非理想性系统的气液相平衡有较为满意的结果。所以说，相比而言，第二类模型更加优秀。

根据前面的介绍我们也知道，活度系数法其实是包含了液相活度系数的计算和气相逸度系数的计算。所以物性方法的缩写也就是这两个计算方法的缩写，比如NRTL-RK所代表的物性方法其实就是液相活度系数用NRTL方程计算，气相逸度系数用RK方程计算。所以活度系数方程和逸度系数方程进行组合可以有很多种物性方法。以下是Aspen当中有的活度系数模型。

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/thermodynamics_methods2.webp)

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/thermodynamics_methods3.webp)

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/thermodynamics_methods4.webp)

状态方程法和活度系数法各有优缺点，因此适用于不同的体系和温度压力范围。下表是孙兰义老师在书中对这两种方法进行的比较，之前在亨利组分推送当中也提到了一些（点击即可跳转[什么时候要选择Henry组分，为什么要选Henry组分？](https://mp.weixin.qq.com/s%3F__biz%3DMzU5OTQxMjI5MA%3D%3D%26mid%3D2247488625%26idx%3D1%26sn%3D0a6b4eb3d63fb920fc8b71bd5a359b4d%26chksm%3Dfeb40f5cc9c3864a69e1d6c8d4bff9b88738fa081c64e0679dd2c50c1b8fe6cd02caf5518d04%26scene%3D21%23wechat_redirect)）。

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/thermodynamics_methods_comparison.webp)

两种物性方法的最关键区别，就是液相当中逸度计算方法的选择。看完本篇推送，以后在进行物性方法的选择时，至少应该能知道选择的物性方法是状态方程法还是活度系数法了吧!



原文链接：https://zhuanlan.zhihu.com/p/524133786



扩展内容：

压缩因子和逸度系数都是对实际气体偏离理想气体程度的修正，前者修正的是 EOS 中的摩尔体积，后者修正的是压力。

理想气体的假设忽略了分子的体积和分子之间的作用力；理想溶液假定溶质完全均匀分散于溶剂中，且忽略分子之间的作用力。活度系数从浓度的角度反映了实际溶液偏离理想溶液的程度。

一般来说，热力学模型包括状态方程和活度系数模型是针对纯组分建立的，因此它们有跟组分相关的参数。但是在研究混合物的时候，相互作用和纯组分是不一样的，也就是说相似分子和相异分子，这个时候就要利用二元交互参数来校正它们之间的相互作用。两种分子越相似，二元相互交互系数越小。当然，这个值与热力学模型相关。

混合物的相行为使用纯组分的性质无法很好地描述。当使用对称或非对称的方法时，常需要使用一些实验数据来帮助“半理论方程”来描述实际情况。如果选择的热力学方法是状态方程，实验参数通常被称为“kij”，它常被用于校正立方型状态方程的二次混合项，可大致表示混合物组分之间相互作用的能量。如果选择的热力学方法是活度系数模型，实验参数通常被称为“aij”和“aji”。

 不同的热力学模型有不同的模型参数，对不同模型参数有各自独立的表达式。 热力学模型在处理混合物时，会使用可调参数（即交互作用参数）对模型参数进行修正。 交互作用参数能够一定程度地反应分子间的相互作用，特别是当物系为极性分子时，缺少交互作用参数， 模拟结果的准确性是没有丝毫保证的。

