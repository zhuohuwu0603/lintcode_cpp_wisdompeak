#### Description
```
有2n颗棋子，n个红的，编号1到n，n个黑的，编号1到n。
你可以交换相邻两个棋子，
给出2n个棋子的颜色和编号，请问至少需要交换多少次，使得红色和黑色的棋子的编号各自升序排列

1≤2n≤2000
```
#### 分析：
```
我们先思考一个基本问题：对于一个1-n的一个乱序，最少多少次交换能够使得它变得有序？
 3 2 5 4 1 
对于1而言，它最终排在1位，它需要往前移动超越所有元素，即移动4次
对于2而言，它最终排在2位。虽然它目前就在第2位，但是因为1会移动到第1的位置，把3和2往后顶。所以它仍然需要往前移动超越3，才能到达位置2，故移动1次
对于3而言，它其实不需要移动，只需要坐等被超越即可。
对于4而言，它最终排在4位。等到前3个元素落位之后，发现它只要前移1位超越5即可。
对于5而言，它也不需要移动。前4个排好了，它自然就是最后一个，所以只需要坐等被超越即可。

从上面，我们可以发现一个规律：数字i需要交换的次数，就是看多少个比i大的数在它前面。
我们称之为逆序数组：比如对应上面那个例子，逆序数字就是 [0,1,0,1,4]
我们总共需要移动的次数就是0+1+0+1+4 = 6

现在来考虑对于本题而言，有两个不相关的序列混在一起，要求使得这两个子序列分别有序：
3 b c 2 a 1 
我们现在从基本问题可以知道的是：使得数字序列有序的移动次数（数字超越数字），使得字母序列有序的移动次数（字母超越字母）。
但是其中并没有包括“数字超越字母”，“或者字母超越数字”所用的移动。最终答案还需要补上这些。
我们需要注意，“数字超越字母”这种操作，其实无论对数字序列而言、还是对字母序列而言，都不会影响各自的顺序。

从另外一个角度我们可以想到，要使得两个交叠的子序列各自有序，方案并不是唯一的。比如 abc123... 和 a12b3c... 都是合法的最终序列的前缀。
他们虽然都合法，但对应的操作总数不同的原因在于：数字序列和字母序列的位置不同，会导致“数字超越字母”或者“字母超越数字”这些操作的次数不同。
举个例子：原字符串是 a12b3c，
你可以什么都不操作，本身就是一个合法的结果
你也可以操作bc，使得变成abc123，这也是一个合法的结果。

这说明什么？The interleaved postion of two subsequence matters!
那么我们如何来标记两种子序列的交叠状态呢？从LeetCode 97. Interleaving String 的DP解法我们可以收到启发：
令dp[i][j]表示前i+j个字符中有i个顺序的数字序列，j个顺序的字母序列，最少需要多少操作。

这两个下标可以很好地描述两个子序列的交叠情况：
根据DP的经验，对应dp[i][j]，我们需要入手的就是第i+j个元素。它其实只有两种选择：第i个数字序列的元素，或者第j个字母序列的元素。
两者中的任意一种，都满足DP的定义：前i+j个元素中字符中有长度为i个的顺序数字序列，有长度为j的顺序字母序列。

1. 如果第i+j个元素是第i个数字元素：

根据之前的分析：i需要超越若干个排在它前面、且比它大的数字。这个“数字超越数字”的操作数目是可以用o(N^2)提前计算好的。
我们可以记做 LargerNumbersBeforeNumber[i]

此外，它还需要超越若干个排在它前面的字母，这个数目有多少呢？
举个例子：
a1b2c XXXX 3
我们现在计算dp[3][3]，并且希望第6个位置是数字。那么意味着希望将3移动到第6个位置。这中间会超越XXXX。
我们已经知道，XXXX中比3大的数字都是需要超越的，这个可以提前可以计算好；并且XXXX中比3小的数字实际上已经不存在了（都被提前处理到前缀中去了），也都不需要被超越。

我们现在比较关心的是3需要超越多少字母？
不难分析：3需要超越的字母的数目 = 原始字符串中排在3前面的字母的数目 - 不需要超越的字母的数目。
那么“不需要超越的字母的数目”是什么呢？因为此时我们在计算dp[i][j]，所以有j个字母已经被提前安排好放入前缀了（也就是abc）。如果abc在原始字符串中是位于3之前的，那么它即使在3的前面，也不需要被超越。
所以我们定义这样一个东西：
LettersBeforeNumber[i]: 在原始字符串中，排在数字i前面的字母有多少个。
LettersBeforeNumberAndSmallerThan[i][j]: 在原始字符串中，排在数字i前面、但是ASCII小于等于j的字母有多少个。

于是总结一下：
对于前i+j个位置，如果我们打算填i个数字，j个字母，并且第i+j个位置是数字，那么把i搬运过来需要的操作是：
LargerNumbersBeforeNumber[i] + LettersBeforeNumber[i] - LettersBeforeNumberAndSmallerThan[i][j]

那么就可以计算dp[i][j]：它的前驱状态是什么？当然是i-1个数字和j个字母啦。
dp[i][j] = dp[i-1][j] + LargerNumbersBeforeNumber[i] + LettersBeforeNumber[i] - LettersBeforeNumberAndSmallerThan[i][j]

2. 如果第i+j个元素是第j个字母元素：
具体分析同上。它的前驱状态是 i个数字和j-1个字母：
dp[i][j] = dp[i][j-1] + LargerLettersBeforeLetter[j] + NumbersBeforeLetter[j] - NumbersBerforeLetterAndSmallerThan[j][i]

当然，dp[i][j]会取两个candidate较小的一个。

最终的答案是什么？自然是这两种序列都安排上。就是dp[n][n].

我们再来分析一下这些预处理需要花多少时间？
LargerNumbersBeforeNumber[i]：固定i，扫一遍在i前面的东西即可。o(N^2)
LettersBeforeNumber[i]: 固定i，扫一遍在i前面的东西即可。o(N^2)
LettersBeforeNumberAndSmallerThan[i][j]: 这本质是个前缀数组，没法直接用o(N^2)求得。但是我们固定i，扫一遍在i前面的东西，可以先得到一个：
	BoolLetterBeforeNumber[i][j] 表示字母j是否位于数字i前面。
然后我们做前缀和的累加，就可以知道小于等于j的字母有多少个位于数字i的前面。这个又是o(N^2).

分析一下程序主体需要花多少时间？两层循环，o(N^2)
for (int total=1; total<=2*n; total++)
     for (int i=0; i<=min(n,total); i++)
        {
            int j = total - i;
            dp[i][j] = min (dp[i-1][j]+...,  dp[i][j-1]+...)            
        }
return dp[n][n];

所以总体的时间复杂度是o(N^2)
```

