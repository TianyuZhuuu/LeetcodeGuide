# 索引
- [动态规划](#动态规划)
  - [最长递增子序列](#最长递增子序列)
- [图算法](#图算法)
  - [最短路径](#最短路径)
    - [Dijkstra算法](#Dijkstra算法)

<br/>

# 动态规划
## 最长递增子序列
- <b>最长递增子序列（longest increasing subsequence）</b>问题是指，在一个给定的数值序列中，找到一个子序列，使得这个子序列元素的数值依次递增，并且这个子序列的长度尽可能地大。最长递增子序列中的元素在原序列中不一定是连续的。
- O(n^2)的算法：比较直观，`dp[i]`存储以`nums[i]`结尾的最长递增子序列长度，递推即可。
```java
    public int lengthOfLIS(int[] nums) {
        if (nums.length==0) return 0;
        int[] dp = new int[nums.length];
        int res = 0;
        dp[0] = 1;
        for (int i=1; i<nums.length; i++) {
            dp[i] = 1;
            // 遍历i之前的数
            for (int j=0; j<i; j++) { 
                // 如果nums[j]<nums[i]，更新dp[i]
                if (nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[j]+1, dp[i]);
                }
            }
            // res为以任意nums[i]结尾的递增子序列长度最大值
            res = Math.max(res, dp[i]);
        }
        return res;
    }
```

- O(n*logn)的算法：
  - 观察：对于数组`{1,2,8}`，它的最长上升子序列长度为2（序列为`{1,2}`和`{1,8}`）。这两个序列中，显然`{1,2}`更优，因为比起`{1,8}`它更有希望拓展成更长的子序列（例如新加一个元素3，可拓展为`{1,2,3}`，而`{1,8}`无法拓展）。显然，对于若干个长度相同的子序列，最优子序列应为结尾元素最小的那个序列（严格地说是字典序最小的序列）。
  - 考虑数组`tails[i]`记录长度为`i+1`的最长上升子序列<b>最小</b>的结尾元素。外层循环遍历`nums[i]`(O(n))，首先考虑`nums[i]`能否拓展当前最长子序列，能则把`nums[i]`添加到`tails`末尾(对应最长子序列长度加+1，并记录结尾元素为`nums[i]`)，否则二分查找(logn)找到`tails`中比`nums[i]`大的<b>最小</b>元素，将其更新为`nums[i]`（例如`{1,4,2}`处理到2时，`tails=[1,4]`更新为`tails=[1,2]`）。
```java
    public int lengthOfLIS(int[] nums) {
        List<Integer> tails = new ArrayList<>();
        tails.add(nums[0]);
        for (int i=1; i<nums.length; i++) {
            int num = nums[i];
            if (num > tails.get(tails.size()-1)) {
                tails.add(num);
            } else {
                int insertPos = Collections.binarySearch(tails, num);
                if (insertPos < 0) insertPos = -(insertPos+1);
                tails.set(insertPos, num);
            }
        }
        return tails.size();
    }
```

- （拓展）如何找到真正的最长上升子序列
  - 有时候题目要求我们输出真正的最长上升子序列，而不仅仅是其长度。这里给出上述两种算法找对应最长子序列的实现，注意我们找到的最长上升子序列是字典序最小的，也即前面所说的最优子序列。主要是增加`prev`数组去追溯`nums[i]`前驱元素在原数组的位置，注释已经足够清晰，这里就不加赘述了。
  - 没有找到对应的题目进行测试，随机生成了长度10000的数组，检查返回的LIS长度是否与算法1/2一致，是否严格递增，测试通过。（最小字典序没有进行测试，也不清楚该怎么测试，我自己认为应该没有问题）。
  - <del>(2020/9/16 美团算法一面被考到，40分钟没写出来，理所当然挂了。怪自己平时不刨根问底，关键时刻掉链子，nlogn的方法都半天没写出来)</del>
  - 补充：在牛客网上找到了[原题](https://www.nowcoder.com/questionTerminal/30fb9b3cab9742ecae9acda1c75bf927)。方法一因超时仅能通过4.76%（也是我面试时写出的答案），方法二能通过全部测试样例。
```java
    public static int[] LIS1(int[] nums) {
        if (nums.length == 0) return new int[0];
        int[] dp = new int[nums.length];
        // 记录以nums[i]结尾的最优子序列，上一个元素（nums[i]上一个元素）的位置。
        // 若最优子序列长度为1，prev[i]设为-1
        int[] prev = new int[nums.length];
        int maxlen = 1;
        dp[0] = 1;
        prev[0] = -1;
        for (int i = 1; i < nums.length; i++) {
            dp[i] = 1;
            prev[i] = -1;
            // 遍历i之前的数
            for (int j = 0; j < i; j++) {
                // 如果nums[j]<nums[i]，更新dp[i]
                if (nums[i] > nums[j]) {
                    if (dp[j] + 1 >= dp[i]) {
                        dp[i] = dp[j] + 1;
                        prev[i] = j;
                    } else if (dp[j] + 1 == dp[i]) {
                        if (prev[i] == -1 || nums[j] <= nums[prev[i]]) {
                            prev[i] = j;
                        }
                    }
                }
            }
            // maxlen为以任意nums[i]结尾的递增子序列长度最大值
            maxlen = Math.max(maxlen, dp[i]);
        }
        int[] res = new int[maxlen];
        // （全局）最优子序列结尾位置
        int ptr = -1;
        int minval = Integer.MAX_VALUE;
        for (int i = 0; i < nums.length; i++) {
            if (dp[i] == maxlen && (ptr == -1 || nums[i] < minval)) {
                ptr = i;
                minval = nums[i];
            }
        }
        for (int i = maxlen - 1; i >= 0; i--) {
            res[i] = nums[ptr];
            ptr = prev[ptr];
        }
        return res;
    }
```

```java
    public static int[] LIS2(int[] nums) {
        // 记录以nums[i]结尾的最优子序列，上一个元素（nums[i]上一个元素）的位置。
        // 若最优子序列长度为1，prev[i]设为-1
        int[] prev = new int[nums.length];
        List<Integer> tails = new ArrayList<>();
        // 记录tail数组中每个元素在原数组中的位置
        List<Integer> tailPositions = new ArrayList<>();
        tails.add(nums[0]);
        tailPositions.add(0);
        prev[0] = -1;
        for (int i = 1; i < nums.length; i++) {
            int num = nums[i];
            if (num > tails.get(tails.size() - 1)) {
                tails.add(num);
                // nums[i]>tails[-1]，prev[i]即为（更新前）tails末尾元素在原数组的位置（tailPositions[-1])
                prev[i] = tailPositions.get(tailPositions.size() - 1);
                // tails新增加了nums[i]，把i添加到tailPositions末尾
                tailPositions.add(i);
            } else {
                int insertPos = Collections.binarySearch(tails, num);
                if (insertPos < 0) insertPos = -(insertPos + 1);
                tails.set(insertPos, num);
                if (insertPos == 0) {
                    // nums[i]为（当前见过的）最小元素，没有上一个元素，prev[i]=-1
                    prev[i] = -1;
                } else {
                    // 将tails[insertPos]更新为nums[i]，nums[i]前一个元素为tails[insertPos-1]，
                    // 其位置为tailPositions[insertPos-1]
                    prev[i] = tailPositions.get(insertPos - 1);
                }
                // tails[insertPos]已更新为nums[i]，相应地更新tailPositions[insertPos]=i
                tailPositions.set(insertPos, i);
            }
        }
        int[] res = new int[tails.size()];
        int ptr = -1;
        for (int i = nums.length - 1; i >= 0; i--) {
            if (nums[i]==tails.get(tails.size()-1)) {
                ptr = i;
                break;
            }
        }
        for (int i = tails.size() - 1; i >= 0; i--) {
            res[i] = nums[ptr];
            ptr = prev[ptr];
        }
        return res;
    }
```
# 图算法
## 最短路径
最短路径问题是图论研究中的一个经典算法问题，旨在寻找图（由结点和路径组成的）中两结点之间的最短路径。算法具体的形式包括：
- 确定起点的最短路径问题 - 即已知起始结点，求最短路径的问题。适合使用[Dijkstra算法](#Dijkstra算法)。
- 确定终点的最短路径问题 - 与确定起点的问题相反，该问题是已知终结结点，求最短路径的问题。在无向图中该问题与确定起点的问题完全等同，在有向图中该问题等同于把所有路径方向反转的确定起点的问题。
- 确定起点终点的最短路径问题 - 即已知起点和终点，求两结点之间的最短路径。
- 全局最短路径问题 - 求图中所有的最短路径。适合使用Floyd-Warshall算法。
### Dijkstra算法
令`dist[u]`表示从源点到结点`u`的暂定最短距离。Dijkstra算法为`dist`数组分配一些初始值，并尝试逐步改善它们。
1. 将所有结点标记为未访问。创建包含所有未访问结点的集合，称为unvisited set。
2. 为每个结点分配一个暂定距离值：对于初始节点，将其设置为零；对于所有其他节点，将其设置为无穷大。将初始节点设置为当前节点。
3. 对于当前结点，考虑其所有未访问的邻居结点`u`，并计算它们经过当前结点的暂定距离（即源点经过当前结点到u的距离）。将新计算的暂定距离与当前分配的值`dist[u]`进行比较，然后分配给`dist[u]`较小的值。 例如，如果当前节点A标记为6，并且将其与相邻节点B的边的长度为2，则通过A到B的距离将为6 + 2 = 8。 如果`dist[B]`距离大于8，则将其更改为8。否则，将保留当前值。
4. 当考虑完当前结点的所有未访问邻居结点后，将当前结点标记为已访问并将其从unvisited set中删除。访问过的节点将不再被检查。
5. 如果目标节点被标记为已访问（计算两点间最短路径时），或者unvisited set中的结点中的最小暂定距离是无穷大（unvisited set中剩余点从源点不可达时发生），算法终止。
6. 否则，选择unvisited set中暂定距离最小的结点，将其设置为新的当前节点，返回步骤3。

- 给定包含`n`个结点，`m`条边的有向图`G`，图以邻接矩阵的形式存储。`adj`是`n*n`的矩阵，`adj[u][v]`存储从结点`u`到结点`v`的`cost`，若不存在从`u`到`v`的边则`adj[u][v]=INF`。实现时防止溢出，令`INF=Integer.MAX_VALUE/2`。若`source`到`target`可达返回最小距离，否则返回-1。在[牛客网](https://www.nowcoder.com/questionTerminal/b39dacd574ba49c7a4d10974b1bb325f?answerType=1&f=discussion)上已通过测试。
  
```java
    public static int dijkstra(int[][] adj, int source, int target) {
        int n = adj.length;
        int INF = Integer.MAX_VALUE / 2;
        boolean[] visited = new boolean[n];
        int[] dist = new int[n];
        Arrays.fill(dist, INF);
        dist[source] = 0;
        while (!visited[target]) {
            int min_dist = INF;
            int current = -1;
            for (int i = 0; i < n; i++) {
                if (visited[i]) continue;
                if (dist[i] < min_dist) {
                    current = i;
                    min_dist = dist[i];
                }
            }
            if (current == -1) return -1;
            visited[current] = true;
            for (int i = 0; i < n; i++) {
                dist[i] = Math.min(dist[i], dist[current] + adj[current][i]);
            }
        }
        return dist[target] == INF ? -1 : dist[target];
    }
```
