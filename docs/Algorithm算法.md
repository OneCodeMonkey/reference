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

2024-08
-------

### 2024-08-12
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

##### 翻转单链表
```java 
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode temp = head;
    while (head != null) {
        temp = head.next;
        head.next = prev;
        prev = head;
        head = temp;
    }
    return prev;
}
```

### 2024-08-16
#### LRU cache 实现
ref: [https://leetcode.com/problems/lru-cache/description/](https://leetcode.com/problems/lru-cache/description/)
Solution 1： using JAVA collection `linkedHashMap`
```java
class LRUCache {
    private HashMap<Integer, Integer> map;

    public LRUCache(int capacity) {
        map = new LinkedHashMap<Integer, Integer>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > capacity;
            }
        };
    }

    public int get(int key) {
        return map.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        map.put(key, value);
    }
}

```

Solution 2: Raw implementation. Using HashMap + Double linked-list
```java 
class LRUCache {
    HashMap<Integer, LinkedNode> cache = new HashMap<>();
    private int count, capacity;
    LinkedNode head, tail;

    class LinkedNode {
        int key;
        int value;
        LinkedNode prev;
        LinkedNode post;
    }

    /**
     * add to head
     *
     * @param node
     */
    private void addNode(LinkedNode node) {
        node.prev = head;
        node.post = head.post;
        head.post.prev = node;
        head.post = node;
    }

    /**
     * remove from double-linked list
     *
     * @param node
     */
    private void removeNode(LinkedNode node) {
        LinkedNode prevNode = node.prev;
        LinkedNode postNode = node.post;
        prevNode.post = postNode;
        postNode.prev = prevNode;
    }

    /**
     * move node to head
     *
     * @param node
     */
    private void moveToHead(LinkedNode node) {
        removeNode(node);
        addNode(node);
    }

    /**
     * move tail node
     *
     * @return
     */
    private LinkedNode popTail() {
        LinkedNode res = tail.prev;
        removeNode(res);

        return res;
    }

    public LRUCache(int capacity) {
        count = 0;
        this.capacity = capacity;

        head = new LinkedNode();
        tail = new LinkedNode();

        head.post = tail;
        tail.prev = head;
    }

    public int get(int key) {
        LinkedNode node = cache.get(key);
        if (node == null) {
            return -1;
        }
        moveToHead(node);

        return node.value;
    }

    public void put(int key, int value) {
        LinkedNode node = cache.get(key);
        if (node == null) {
            LinkedNode newNode = new LinkedNode();
            newNode.key = key;
            newNode.value = value;
            cache.put(key, newNode);
            addNode(newNode);

            count++;
            // check if full
            if (count > capacity) {
                LinkedNode tailNode = popTail();
                cache.remove(tailNode.key);
                count--;
            }
        } else {
            node.value = value;
            moveToHead(node);
        }
    }
}

```

#### 全排列
`C(n, k)` 的实现： 重点是 Backtracking() 函数的参数设置和定义
```java
class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> ret = new LinkedList<>();
        List<Integer> temp = new LinkedList<>();

        backtracking(n, k, temp, ret, 1);

        return ret;
    }

    public void backtracking(int n, int k, List<Integer> path, List<List<Integer>> out, int startIndex) {
        List<Integer> pathCopy = new LinkedList<>(path);
        if (path.size() == k) {
            out.add(pathCopy);
            return;
        }

        for (int i = startIndex; i <= n + pathCopy.size() - k + 1; i++) {
            pathCopy.add(i);
            backtracking(n, k, pathCopy, out, i + 1);
            pathCopy.remove(pathCopy.size() - 1);
        }
    }
}

```

#### 全组合
`P(n, n)` 的实现： 重点在于，permutation 生成时，逐步尝试在当前中间 list 中，遍历每一个插入位置去尝试生成新的 list，直至其长度被填满。
```java
// 求 1~n 的全排列
class Solution {
    public List<List<Integer>> permutation(int n) {
        List<List<Integer>> permutations = new ArrayList<>();
        if (n == 0) {
            return permutations;
        }

        collectPermutations(n, 1, new ArrayList<>(), permutations);

        return permutations;
    }

    private void collectPermutations(int n, int start, List<Integer> permutation, List<List<Integer>> permutations) {
        if (permutation.size() == n) {
            permutations.add(permutation);
            return;
        }

        // 遍历不同位置插入
        for (int i = 0; i <= permutation.size(); i++) {
            List<Integer> newPermutation = new ArrayList<>(permutation);
            newPermutation.add(i, start);
            collectPermutations(n, start + 1, newPermutation, permutations);
        }
    }
}

```

### 2024-08-22
#### LFU cache 实现
LFU cache: 按频次淘汰 cache 算法
实现要点: 一个 hashmap 存 key-value，一个 hashmap 存 key-count，再维护一个不同频率的 count 所对应的元素 double-linked-lists，实现 O(1) 的 put(), get() 效率
ref 1: https://zhuanlan.zhihu.com/p/660174023
ref 2: https://www.jianshu.com/p/437f53341f67

```java 
// Solution 1: Two Hashmap + N double-linked lists. Using `LinkedHashSet()` to simplify double-linked list.
// Runtime 75 ms Beats 27.03%
// Memory 142.30 MB Beats 5.38%
// Two Hashmap + N double-linked lists.
// T:O(1), S:O(1)
// 
class LFUCache {
    HashMap<Integer, Integer> countTime;
    HashMap<Integer, Integer> keyToValue;
    HashMap<Integer, LinkedHashSet<Integer>> timeToKeys;
    int maxCapacity;
    int curMinTime = -1;

    public LFUCache(int capacity) {
        countTime = new HashMap<>();
        keyToValue = new HashMap<>();
        timeToKeys = new HashMap<>();
        maxCapacity = capacity;
    }

    public int get(int key) {
        if (!keyToValue.containsKey(key)) {
            return -1;
        }
        int count = countTime.get(key);
        countTime.put(key, count + 1);
        timeToKeys.get(count).remove(key);
        if (count == curMinTime && timeToKeys.get(count).size() == 0) {
            curMinTime += 1;
        }
        if (!timeToKeys.containsKey(count + 1)) {
            timeToKeys.put(count + 1, new LinkedHashSet<>());
        }
        timeToKeys.get(count + 1).add(key);

        return keyToValue.get(key);
    }

    public void put(int key, int value) {
        if (maxCapacity <= 0) {
            return;
        }
        if (keyToValue.containsKey(key)) {
            keyToValue.put(key, value);
            get(key);
            return;
        }
        if (keyToValue.size() >= maxCapacity) {
            int removeElem = timeToKeys.get(curMinTime).iterator().next();
            timeToKeys.get(curMinTime).remove(removeElem);
            keyToValue.remove(removeElem);
        }
        keyToValue.put(key, value);
        countTime.put(key, 1);
        curMinTime = 1;
        timeToKeys.computeIfAbsent(1, k -> new LinkedHashSet<>());
        timeToKeys.get(1).add(key);
    }
}

```

#### Trie 树实现
Trie 树（字典树）的实现：
重点在于 TreeNode 的定义，需要两个指标，一个是 boolean isWord，代表是否是一个有效的 word 的终止点。一个是定义 26 个字符的子节点，代表下一个字符的顺延。
当然如果支持大小写字母同时存在，那么 children[26] 改为 childrent[52] 也可以达到目的。

```java 
class TrieTreeNode {
    // Leaf Node decide whether this node is valid word.
    boolean isWord;
    TrieTreeNode[] children;

    TrieTreeNode() {
        isWord = false;
        children = new TrieTreeNode[26];
    }
}

class Trie {
    TrieTreeNode root;

    public Trie() {
        root = new TrieTreeNode();
    }

    public void insert(String word) {
        TrieTreeNode parent = root;
        for (int i = 0; i < word.length(); i++) {
            int charNum = word.charAt(i) - 'a';
            if (parent.children[charNum] == null) {
                parent.children[charNum] = new TrieTreeNode();
            }
            parent = parent.children[charNum];
        }

        parent.isWord = true;
    }

    public boolean search(String word) {
        TrieTreeNode parent = root;
        for (int i = 0; i < word.length(); i++) {
            int charNum = word.charAt(i) - 'a';
            if (parent.children[charNum] == null) {
                return false;
            }
            parent = parent.children[charNum];
        }

        return parent.isWord;
    }

    public boolean startsWith(String prefix) {
        TrieTreeNode parent = root;
        for (int i = 0; i < prefix.length(); i++) {
            int charNum = prefix.charAt(i) - 'a';
            if (parent.children[charNum] == null) {
                return false;
            }
            parent = parent.children[charNum];
        }

        return true;
    }
}
```


#### SkipList 实现
SkipList 跳表实现：基于概率的多层链表 的搜索结构，使得查找复杂度能到 logn

```java 
// 基于概率的 多层链表 数据结构，查找复杂度 log(n)
// Runtime 18 ms Beats 26.53%
// Memory 53.35 MB Beats 15.10%
// implementation.
// T:O(logn), S:O(logn)
// 
class Skiplist {
    class Node {
        int val;
        Node next, down;

        Node(int val, Node next, Node down) {
            this.val = val;
            this.next = next;
            this.down = down;
        }
    }

    Node head = new Node(-1, null, null);
    Random rand = new Random();

    public Skiplist() {
        //
    }

    public boolean search(int target) {
        Node cur = head;
        while (cur != null) {
            while (cur.next != null && cur.next.val < target) {
                cur = cur.next;
            }
            if (cur.next != null && cur.next.val == target) {
                return true;
            }
            cur = cur.down;
        }

        return false;
    }

    public void add(int num) {
        Stack<Node> stack = new Stack<>();
        Node cur = head;
        while (cur != null) {
            while (cur.next != null && cur.next.val < num) {
                cur = cur.next;
            }
            stack.push(cur);
            cur = cur.down;
        }
        boolean insert = true;
        Node down = null;
        while (insert && !stack.isEmpty()) {
            cur = stack.pop();
            cur.next = new Node(num, cur.next, down);
            down = cur.next;
            insert = rand.nextDouble() < 0.5;
        }

        if (insert) {
            head = new Node(-1, null, head);
        }
    }

    public boolean erase(int num) {
        Node cur = head;
        boolean isFound = false;
        while (cur != null) {
            while (cur.next != null && cur.next.val < num) {
                cur = cur.next;
            }
            if (cur.next != null && cur.next.val == num) {
                isFound = true;
                cur.next = cur.next.next;
            }
            cur = cur.down;
        }

        return isFound;
    }
}

```
