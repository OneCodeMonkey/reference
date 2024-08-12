Algorithm算法
===

算法相关整理：

- 1.Greedy
- 2.DP
- 3.Prefix sum
- 4.分而治之
- 5.图算法
- 6.递归
- 7.回溯
- 8.数据结构
  - 8.1 二叉树
  - 8.2 trie 树
  - 8.3 skipList
  - 8.4 LRU,LFU 淘汰算法
- 9.排序

面试问题
--------

入门
--------

2024
-------

### 2024-08
##### leetcode 3249. Count the Number of Good Nodes
要点：无向图，DFS搜索
```java
// AC: Runtime 141 ms Beats 100.00%
// Memory 131.06 MB Beats 50.00%
// Graph: Un-directed graph, DFS search.
// T:O(V + E), S:O(V + E)
// 
class Solution {
    private List<List<Integer>> graph;
    private int[] subTreeSize;

    private int dfs(int cur, int parent) {
        int size = 1;
        for (int nextPoint : graph.get(cur)) {
            if (nextPoint != parent) {
                size += dfs(nextPoint, cur);
            }
        }

        subTreeSize[cur] = size;

        return size;
    }

    public int countGoodNodes(int[][] edges) {
        int n = edges.length + 1;
        graph = new ArrayList<>(n);
        subTreeSize = new int[n];

        for (int i = 0; i < n; i++) {
            graph.add(new ArrayList<>());
        }
        for (int[] edge : edges) {
            graph.get(edge[0]).add(edge[1]);
            graph.get(edge[1]).add(edge[0]);
        }
        dfs(0, -1);

        int ret = 0;
        for (int i = 0; i < n; i++) {
            boolean flag = true;
            int prevNodeSize = -1;
            for (int nextPoint : graph.get(i)) {
                // not its subtree.
                if (subTreeSize[nextPoint] >= subTreeSize[i]) {
                    continue;
                }
                if (prevNodeSize == -1) {
                    prevNodeSize = subTreeSize[nextPoint];
                } else if (prevNodeSize != subTreeSize[nextPoint]) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                ret++;
            }
        }

        return ret;
    }
}

```