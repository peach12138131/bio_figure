3 data作图

输出2个(1,4)含4个子图的大图（10度，和18度），共享y轴，x轴各自独立，
y轴
D1
D6
D7
L43
L1091
L1253
L2182


子图1x轴
D1
D6
D7
L43
L1091
L1253
L2182


子图2x轴
D1-D7
D6-D7
D1-L43
D1-L1091
D1-L1253
D1-L2182
D6-L43
D6-L1091
D6-L1253
D6-L2182
D7-L43
D7-L1091
D7-L1253
D7-L2182

子图3x轴
D1-D6-D7
D1-D6-L43
D1-D6-L1091
D1-D6-L1253
D1-D6-L2182
D1-D7-L43
D1-D7-L1091
D1-D7-L1253
D1-D7-L2182
D6-D7-L43
D6-D7-L1091
D6-D7-L1253
D6-D7-L2182

子图4x轴
D1-D6-D7-L43
D1-D6-D7-L1091
D1-D6-D7-L1253
D1-D6-D7-L2182

作图前请先进行数据处理

数据解释，分为两个大组，（10度和18度）
每个大组组按菌株组合数量分为4组，Single species，Dual,Three,Four
举例说明：
Single species，行名为D1，D6,D7...，后续只有一个数据块，对应该菌株数据
Dual，如D1-D6,，后续带有两个数据块，对应在这种组合下D1,D6，分别的数据
Three,D1-D6-D7,后续带有3个数据块
Four,D1-D6-D7-L2182,后续带有4个数据块
每个数据块都有3个实验值（有些已经计算了mean和SD,有些没有），都需要先计算平均和SD

绘图规则
1.先进行数据处理，处理成适合作图的格式
2.对应规则，对于子图1，点（D1，D1）对应其在Single species中D1的值，（D1，D7）为空，以此类推
3.对于子图2，3，4，（D1-D7，）需要画两个点，如（D1-D7，D1）和（D1-D7，D7），并且这里的数据，对于的是在Dual组合下D1,D7的数据，其余点为空。
4.每个点带有两个属性，圆圈大小代表Biofilm Cell Number,即实验值均值；颜色代表 delta Biofilm Value (Multispecies - Monospecies),即多菌组合的值减去单菌的值，比如（D1-D7，D1）的颜色对应Dual中D1的值减去Single中D1的值（一般为负数）
5.如果Biofilm Value (Multispecies - Monospecies)为正，则需要进行Independent t-test显著性分析，颜色用 *表示p<0.05 *,p<0.01 **
