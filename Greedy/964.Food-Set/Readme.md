### 964. Food Set

#### Description
```
给定一个长度为n个二元组数组lunch表示午餐食物集合，
其中lunch[i][0]表示第i种食物的热量，lunch[i][1]表示第i种食物的美味程度
再给定一个长度为m个二元组数组dinner表示晚餐食物集合，其中dinner[i][0]表示第i种食物的热量，dinner[i][1]表示第i种食物的美味程度
请你在午餐集合和晚餐集合各自选取至多一个食物，在满足总的美味程度不小于T的前提下，返回摄入最少的热量和。
```

#### 分析：
```
贪心法。将lunch和dinner分别按照美味程度从小到大排列。
对于lunch[0]而言，想要美味达标，能搭配的dinner必须是美味程度很大的，也就是从dinner的最后几个元素里挑。
我们将dinner从后往前遍历，直至不满足lunch[0]+lunch[j]>=T。那么我们从j往后的dinner，都可以和lunch[0]搭配满足美味的要求；
显然，我们必然会从中挑一个“热量最小的”。这用一个heap就能实现（priority_queue, set等等）

对于lunch[1]而言，想要美味达标，能够搭配的dinner的选择会更多一点，j可以往前移动一些。
同理我们会将j移动到不能满足lunch[1]+lunch[j]>=T为止。那么我们从j往后的dinner，我们依然只需要选择“热量最小的”。

依次类推，对于每一个Lunch[i]，我们都可以确定一个j，使得lunch[i]+dinner[j]在美味上>=T。这个j的移动是单调往左的。
对于这些dinner，我们将它们的热量值逐个放入heap，自动弹出的最小值就是和lunch[i]的最佳搭配。

特别注意：本题允许只选择lunch或者dinner。需要再单独对它们扫一遍。
```
