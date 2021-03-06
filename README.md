# 索引
- [二分查找](#二分查找)
- [快速排序](#快速排序)
- [动态规划](#动态规划)
  - [最长递增子序列](#最长递增子序列)
- [图算法](#图算法)
  - [最短路径](#最短路径)
    - [Dijkstra算法](#Dijkstra算法)
    - [Bellman-Ford算法](#Bellman-Ford算法)
- [常用操作](#常用操作)
- [模板](#模板)

<br/>

# 二分查找
```java
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] >= target) right = mid;
        else left = mid + 1;
    }
    return left < nums.length && nums[left] == target ? left : -1;
}
```

# 快速排序
```java
public void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}

public void quickSort(int[] nums, int left, int right) {
        if (right - left <= 1) return;
        int pivot = partition(nums, left, right);
        quickSort(nums, left, pivot);
        quickSort(nums, pivot + 1, right);
    }
    
public int partition(int[] nums, int left, int right) {
    int pivot = nums[right - 1];
    int i = left;
    for (int j = left; j < right - 1; j++) {
        if (nums[j] < pivot) {
            swap(nums, i, j);
            i++;
        }
    }
    swap(nums, i, right - 1);
    return i;
}
```

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
Dijkstra算法通过保留目前为止所找到的每个顶点`v \in V`从`s`到`v`的最短路径来工作的。初始时，原点s的路径权重被赋为0（即原点的实际最短路径=0）。同时把所有其他顶点的路径长度设为无穷大，即表示我们不知道任何通向这些顶点的路径。当算法结束时，`d[v]`中存储的便是从`s`到`v`的最短路径，或者如果路径不存在的话是无穷大。

松弛操作是Dijkstra算法的基础操作：如果存在一条从`u`到`v`的边，那么从`s`到`v`的一条新路径是将边`w(u,v) \in E`添加到从`s`到`u`的路径尾部来拓展一条从`s`到`v`的路径。这条路径的长度是`d[u]+w(u,v)`。如果这个值比目前已知的`d[v]`的值要小，那么可以用这个值来替代当前`d[v]`中的值。松弛边的操作一直运行到所有的`d[v]`都代表从`s`到`v`的最短路径的长度值。

算法维护两个顶点集合`S`和`Q`。集合`S`保留所有已知实际最短路径值的顶点，而集合`Q`则保留其他所有顶点。集合`S`初始状态为空，而后每一步都有一个顶点从`Q`移动到`S`。这个被选择的顶点是`Q`中拥有最小的`d[u]`值的顶点。当一个顶点`u`从`Q`中转移到了`S`中，算法对`u`的每条外接边`w(u,v)`进行松弛。

给定包含`n`个结点，`m`条边的有向图`G`，图以邻接矩阵的形式存储。`adj`是`n*n`的矩阵，`adj[u][v]`存储从结点`u`到结点`v`的`cost`，若不存在从`u`到`v`的边则`adj[u][v]=INF`。实现时防止溢出，令`INF=Integer.MAX_VALUE/2`。若`source`到`target`可达返回最小距离，否则返回-1。在[牛客网](https://www.nowcoder.com/questionTerminal/b39dacd574ba49c7a4d10974b1bb325f?answerType=1&f=discussion)上已通过测试。
  
```java
    public static int dijkstra(int[][] adj, int source, int target) {
        int n = adj.length;
        int INF = Integer.MAX_VALUE / 2;
        HashSet<Integer> S = new HashSet<>();
        HashSet<Integer> Q = new HashSet<>();
        for (int i = 0; i < n; i++) {
            Q.add(i);
        }
        int[] d = new int[n];
        Arrays.fill(d, INF);
        d[source] = 0;
        while (S.size() < n) {
            int min_dist = INF;
            int current = -1;
            for (int i : Q) {
                if (d[i] < min_dist) {
                    current = i;
                    min_dist = d[i];
                }
            }
            if (current == -1) break;
            S.add(current);
            Q.remove(current);
            for (int i = 0; i < n; i++) {
                d[i] = Math.min(d[i], d[current] + adj[current][i]);
            }
        }
        return d[target] == INF ? -1 : d[target];
    }
```
### Bellman-Ford算法
Bellman-Ford算法是一种计算从一个源顶点到加权有向图中所有其他顶点的最短路径的算法。对于同样的问题，它比Dijkstra算法要慢，但是更通用，因为它能够处理一些边<b>权值为负数</b>的图。

Bellman-Ford算法与Dijkstra算法类似，都以松弛操作为基础，即估计的最短路径值渐渐地被更加准确的值替代，直至得到最优解。在两个算法中，计算时每个边之间的估计距离值都比真实值大，并且被新找到路径的最小长度替代。 然而，Dijkstra算法以贪心法选取未被处理的具有最小权值的节点，然后对其的出边进行松弛操作；而Bellman-Ford算法简单地对所有边进行松弛操作，共`|V|-1`次，其中`|V|`是图的结点的数量。在重复地计算中，已计算得到正确的距离的边的数量不断增加，直到所有边都计算得到了正确的路径。这样的策略使得Bellman-Ford算法比Dijkstra算法适用于更多种类的输入。

由于不存在循环的最长路径可以包含`|V|-1`条边，因此必须扫描所有边`|V|-1`次，以确保找到所有结点的最短路径。对所有的边进行最终扫描，如果更新了任何距离，则找到一条长度为`|V|`条边的路径，该路径只能在图中存在至少一个负环时出现。

实现中返回`int[]`，第一位`0/1`存储图是否包含负权环，第二位存储从source到target的距离（如存在负权环则为1）。实现在[牛客网](https://www.nowcoder.com/questionTerminal/5ed9a1ad23b64044ad546f55df7a7f6d?f=discussion)上已通过测试。

```java
    public static int[] bellmanFord(int n, int[][] edges, int source, int target) {
        int INF = Integer.MAX_VALUE / 2;
        // Initialize distance
        int[] dist = new int[n];
        Arrays.fill(dist, INF);
        dist[source] = 0;
        // |V|-1 relaxation
        for (int t = 0; t < n - 1; t++) {
            for (int[] e : edges) {
                int u = e[0];
                int v = e[1];
                int w = e[2];
                if (dist[u] != INF && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                }
            }
        }
        // Check for circle
        for (int[] e : edges) {
            int u = e[0];
            int v = e[1];
            int w = e[2];
            if (dist[u] != INF && dist[u] + w < dist[v]) {
                return new int[]{1, -1};
            }
        }
        return new int[]{0, dist[target]};
    }
```

# 常用操作
## 二维数组初始化
```java
int[][] arr = {{1, 2, 3}, {4, 5, 6}};
```

## Collection Literals (集合字面值)
java 9中加入了工厂方法，简化集合的创建：
```java
// 最多10个元素（Map中10个键值对）
List<Character> vowelList = List.of('a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U');
Set<Character> vowelSet = Set.of('a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U');
Map<String, String> map1 = Map.of("key1", "value1", "key2", "value2");
// 以下方法可创建包含任意数量元素的Map
Map<String, String> map2 = Map.ofEntries(
        Map.entry("key1", "value1"),
        Map.entry("key2", "value2")
);
```
更老版本：
```java
String[] products = new String[]{"mobile","mouse","moneypot","monitor","mousepad"};
List<String> candidates = Arrays.asList(products);
```

## 使用array初始化list
```java
int[] arr = new int[]{1, 2, 3};
List<Integer> list = Arrays.asList(arr); 
```

## 重复字符串若干次
重复字符串`abcd`3次，java 11以上：
```java
String repeated = "abcd".repeat(3);
```

java 10及更老版本需要使用正则替换：
```java
String repeated = new String(new char[3]).replace("\0", "abcd");
```

## List转换为Array
```java
String[] strings = strlist.toArray(new String[0]);
int[] ints = intlist.stream().mapToInt(i->i).toArray();
```

## Character类常用操作
```java
Character.isLetterOrDigit(ch) // 是否为字母或数字
Character.isLetter(ch)        // 是否为字母
Character.isUpperCase(ch)     // 是否为大写字母
Character.isLowerCase(ch)     // 是否为小写字母
Character.isDigit(ch)         // 是否为数字

Character.toUpperCase(ch)     // 转化为大写字母
Character.toLowerCase(ch)     // 转化为小写字母
```

## 基本类型数组作为HashSet/HashMap的Key
基本类型的数组不能直接用作HashSet/HashMap的Key（hashcode不同，两个数组会被认为是不同的对象）。可以考虑使用`Arrays.toString(..)`方法把数组转化成字符串，再当做Key使用。
```java
int[] arr1 = {1, 2, 3};
int[] arr2 = {1, 2, 3};

HashSet<int[]> set1 = new HashSet<>();
set1.add(arr1);
System.out.println(set1.contains(arr2)); //false

HashSet<String> set2 = new HashSet<>();
set2.add(Arrays.toString(arr1));
System.out.println(set2.contains(Arrays.toString(arr2))); //true
```

# 模板
## 双指针
当需要把数组/字符串划分成符合条件的连续分组时使用。

例题[Positions of Large Groups](https://leetcode.com/problems/positions-of-large-groups/)需要我们找到由一些连续的相同字符所构成的分组，记录较大分组(长度大于等于3)的开始结束位置。例如，在字符串 S = "abbxxxxzyy" 中，就含有 "a", "bb", "xxxx", "z" 和 "yy" 这样的一些分组。

其他题目如[Most Common Word](https://leetcode.com/problems/most-common-word/)可以使用双指针对字符串进行切分得到单词，再进行后续处理。

```java
class Solution {
    public List<List<Integer>> largeGroupPositions(String s) {
        List<List<Integer>> res = new ArrayList<>();
        for (int i=0, j=0; i<s.length(); i=j) {
            // s.charAt(j)==s.charAt(i)视情况改为相应的条件
            while (j<s.length()&&s.charAt(j)==s.charAt(i)) j++;
            if (j-i>=3) res.add(Arrays.asList(i,j-1));
        }
        return res;
    }
}
```
