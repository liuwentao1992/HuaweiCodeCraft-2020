2020华为软件精英挑战赛
=======
* 初赛代码：`primary_test.cpp`

初赛篇
======
* 武长赛区团队`您吃了么`，初赛成绩0.39s，虽排名仅18，但是我们的代码“短小精悍”，仅400行，赛后线下优化速度再提升一倍，未测试，估计0.2x，学习了前排大佬代码后，在算法思路上基本都一致，难点在于细节的处理，例如cache命中率，避免stl，多线程负载均衡等各种优化小手段，这也许是和大佬的差距所在吧。
* 本仓仅能提供一些常规思路，代码经过整理，相对一两千行的代码，400行更容易理解吧。

优化思路
=========
* [mmap读取](!mmap读取) 
* [id映射为char类型](#id映射为char类型)
* [拓扑排序剪枝](拓扑排序剪枝 ) 
* [3邻域剪枝](3邻域剪枝)
* [逆向搜索存图](逆向搜索存图)
* [并行](并行)

mmap读取 
========
* 在进行大规模数据处理时，读文件很有可能成为速度瓶颈。  
* 用fread/fwrite方式访问硬盘，用户须向内核指定要读多少，内核再把得到的内容从内核缓冲池cache拷向内存中。  
* 而mmap就是通过把文件的某一块内容直接映射到内存上，用户可以直接向内核缓冲池读写这一块内容，减少了内核与内存的来回拷贝。  

id映射为char类型
==============
* 因为题目要求最小的结点在每个环的第一位，所以对所有id排序，使用`unordered_map`关联容器，查找和插入都是O(1)复杂度，将id重映射为[0,n)这样在搜索时，直接对每个结点的出边从小到大遍历，得到的答案正好是排好序的。
* ps：当映射的id数量足够多，stl必定会拖慢程序的性能。
* 对结点id排好序后，直接将`int`型的id转换为`char*` 类型储存，方便再后续找环过程中，直接将环映射为`char*` 的类型输入到结果容器中，避免了写入前类型转换的耗时。
* `int`转`char`能手工就手工，少用库函数

拓扑排序剪枝 
===========
* 第一把利剑，对于稠密图几乎没有作用，但对于稀疏图提升非常明显，提前把不可能成环的所有结点标记或删除。
* 原理：找到所有入度为0的结点，因为一定不能成环，标记或删除，并将其出边结点的入度也-1，检查是否降为0，为0就删。

3邻域剪枝
========
* 第二把利剑，大大提升速度，因为题目要求最长为7环，既第1个结点到其他任一结点必定在3步以内，正向走或反向走。
* 所以再遍历每个结点的时候，提前正向深度3的遍历，和反向深度3的遍历，将其3步范围内的所有大于`head`的结点做好标记。在进行搜索递归时排除掉未标记的点即可。
* ps:线下速度提升了好几倍

逆向搜索存图
===========
* 算法的核心，这里用的4+3，在反向3领域剪枝的同时，顺便进行存图，存2步到达`head`的点和3步能到达`head`的点，数据结构采用的`unordered_map<int,vector>`和`unordered_set<int>`。
* 存2步涉及了排序问题，`v2=>v1.1=>head`;`v2=>v1.2=>head`那么容器key是`v2`,值是`[v1.1,v1.2]`，为了确保`v1.1`必须小于`v1.2`，所以需提前将入边结点从小到大排序。
* 插曲：之前在存3步也想像存2步一样，用了更为复杂的数据结构记录路径中的所有结点，在递归时直接查找再遍历输出，但多存一个数字加上又是反字典顺序的遍历，就涉及顺序问题无法提前排好，导致再递归时，当查找到3步到达`head`的结点时还需再做一次排序才能正确输出，线下手写了快排，测试速度大大提升（仅单线程速度就超过了原4线程3倍）但线上精度有问题，卡在了18%。
* 之后存3步只是用到了无序容器，将所有三步到达`head`的结点存入。之后递归时，只需递归到第4层，检查其子节点是否在无序容器中，是则进入第5层，通过2步的存图直接找到结果。

并行
====
* 线上测试服务器是4U16G，多线程只是采用简单按照节点分块处理任务，线上测试改为6线程或8线程的提升几乎可以忽略，所以最终还是使用的4线程
* 由于越到后面的节点id，其环路个数越少，所以需要负载均衡，使得它们四个得搜索用时进来一样。
* 提供一份初赛线上数据集的负载分配分析https://zhuanlan.zhihu.com/p/136785097  
* 假设图中每个节点连接到剩余的任意一个节点的概率都是一样的，并且每个节点的出度和入读平均度数为10，计算出大致分配`n1=0.055N,n2=0.13N,n3=0.25N`

各位大佬分析
=========
* [1] [武长赛区hust_1037](https://github.com/trybesthbk/-Huawei-Code-Craft-)  
* [2] [武长赛区wdyggg](https://github.com/trybesthbk/-Huawei-Code-Craft-)  
* [3] [记2020年华为软件精英挑战赛（初赛)](https://zhuanlan.zhihu.com/p/136785097)  
* [4] [渝赛区初赛赛后方案分享](https://blog.csdn.net/qq_34914551/article/details/105788200)  
* [5] [华为软件精英挑战赛-2020-初赛复赛-题目分析/算法Baseline](https://zhuanlan.zhihu.com/p/125764650)  

可能有用的数据
==========
* [1][民间测试数据集](https://github.com/liusen1006/2020HuaweiCodecraft-TestData)  
* [2][ddd大神的分析](https://github.com/justarandomstring/2020-Huawei-Code-Craft)  
* [3][数据生成](https://github.com/byl0561/HWcode2020-TestData)  
* [4]  
