---
layout:     post

title:      "图相关算法"
# subtitle:   "Java并发编程的艺术笔记"
# excerpt: "Java并发编程的艺术笔记"
author:     "谢文进"
date:       2022-05-06
description: "图的遍历、图中判断是否有环、拓扑排序等。"
# image: "/img/2022-03-28-The-Art-of-Java-Concurrency-Programming/background.jpg"
published: 2022-05-06 
tags:
     - Java
     - 算法
     - 图

categories: [ Tech ]
URL: "/2022/05/06/Algorithms-graph/"
---
## 图的遍历
```java
// 记录被遍历过的节点
boolean[] visited;
// 记录从起点到当前节点的路径
boolean[] onPath;

/* 图遍历框架 */
void traverse(Graph graph, int s) {
    if (visited[s]) return;
    // 经过节点 s，标记为已遍历
    visited[s] = true;
    // 做选择：标记节点 s 在路径上
    onPath[s] = true;
    for (int neighbor : graph.neighbors(s)) {
        traverse(graph, neighbor);
    }
    // 撤销选择：节点 s 离开路径
    onPath[s] = false;
}
```
## 图中判断是否存在环
### [207.课程表](https://leetcode-cn.com/problems/course-schedule/)
首先将课程表的依赖关系转换为图，接着如果图中存在环，就返回false，否则返回true。

深度优先搜索方法：
```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        onPath = new boolean[numCourses];
        visited = new boolean[numCourses];
        List<Integer>[] graph = buildGraph(numCourses, prerequisites);
        for (int i = 0; i < numCourses; i++) {
            traverse(graph, i);
        }
        return !hasCycle;
    }

    boolean[] onPath;   // 记录路径
    boolean[] visited;  // 记录访问的节点
    boolean hasCycle;   // 图中是否有环
    // 回溯，深度优先搜索
    private void traverse(List<Integer>[] graph, int s) {
        if (onPath[s]) {
            hasCycle = true;
        }
        if (hasCycle || visited[s]) { // 已经找到环或者已经遍历过了
            return;
        }

        // 前序代码位置
        onPath[s] = true;
        visited[s] = true;
        for (int t : graph[s]) {
            traverse(graph, t);
        }
        // 后序代码位置
        onPath[s] = false;
    } 
    // 建图
    private List<Integer>[] buildGraph(int numCourses, int[][] prerequisites) {
        List<Integer>[] graph = new List[numCourses];
        for (int i = 0; i < numCourses; i++) {
            graph[i] = new LinkedList<>();
        }
        for (int[] edge : prerequisites) {
            int from = edge[1], to = edge[0];
            // 修完 edge[1] 课程后才能修 edge[0]
            graph[from].add(to);
        }
        return graph;
    }
}
```
时间复杂度：$O(n+m)$，$n$为课程数，$m$为先修课程的要求数。

空间复杂度：$O(n+m)$。

广度优先搜索方法，首先要建立一个入度的数组，如果入度为0就加入到队列中，接着广度搜索有依赖的节点，并将这些节点的入度减一，如果最后遍历的节点个数与numCourses相同，则说明没有形成环。因为如果形成环的话，环中的节点的入度不会减少到0，肯定不会加入到队列中，也就不会遍历到。

```java
// 方法二：广度优先搜索
public boolean canFinish(int numCourses, int[][] prerequisites) {
    // 建图
    List<Integer>[] graph = buildGraph(numCourses, prerequisites);
    // 构建入度数组
    int[] indegree = new int[numCourses];
    for (int[] edge : prerequisites) {
        int from = edge[1], to = edge[0];
        // 节点 to 的入度加一
        indegree[to]++;
    }
    // 根据入度初始化队列
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) {
            // 如果入度为0，则加入队列
            q.offer(i);
        }
    }
    int count = 0; // 记录遍历的节点的个数
    while (!q.isEmpty()) {
        // 弹出节点并将它指向的节点的入度减一
        int cur = q.poll();
        count++;
        for (int next : graph[cur]) {
            indegree[next]--;
            // 如果入度为0，则加入队列中
            if (indegree[next] == 0) {
                q.offer(next);
            }
        }
    }
    // 如果所有节点都遍历过，则说明没有形成环
    return count == numCourses;
}
```

## 拓扑排序
### [210.课程表II](https://leetcode-cn.com/problems/course-schedule-ii/)
拓扑排序就是反转后序遍历的结果。注意这里的依赖关系依然是先修完edge[1]课程再修edge[0]课程，图中edge[1] -> edge[0]。
```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        onPath = new boolean[numCourses];
        visited = new boolean[numCourses];
        List<Integer>[] graph = buildGraph(numCourses, prerequisites);
        for (int i = 0; i < numCourses; i++) {
            traverse(graph, i);
        }
        if (hasCycle) { // 如果存在环
            return new int[]{};
        }
        // 逆后序遍历的结果就是拓扑排序的结果
        Collections.reverse(postorder); // 这里直接调用Collections的接口
        int[] res = new int[numCourses];
        for (int i = 0; i < numCourses; i++) {
            res[i] = postorder.get(i);
        }
        return res;
    }

    boolean[] onPath;
    boolean[] visited;
    boolean hasCycle;
    List<Integer> postorder = new ArrayList<>();

    void traverse(List<Integer>[] graph, int s) {
        if (onPath[s]) {
            hasCycle = true;
        }
        if (visited[s] || hasCycle) {
            return;
        }

        // 前序遍历位置
        visited[s] = true;
        onPath[s] = true;
        for (int t : graph[s]) {
            traverse(graph, t);
        }
        // 后序遍历位置
        postorder.add(s);
        onPath[s] = false;
    }

    // 建图函数
    private List<Integer>[] buildGraph(int numCourses, int[][] prerequisites) {
        List<Integer>[] graph = new LinkedList[numCourses];
        for (int i = 0; i < numCourses; i++) {
            graph[i] = new LinkedList<>();
        }

        for (int[] edge : prerequisites) {
            int from = edge[1], to = edge[0];
            // 先修 edge[1] 再修 edge[0]
            graph[from].add(to);
        }
        return graph;
    }
}
```

广度优先搜索方法：
```java
// 方法二：广度优先搜索
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<Integer>[] graph = buildGraph(numCourses, prerequisites);
    int[] indegree = new int[numCourses];
    for (int[] edge : prerequisites) {
        int from = edge[1], to = edge[0];
        indegree[to]++;
    }
    // 根据入度数组初始化队列
    Queue<Integer> q = new LinkedList<>();
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) {
            q.offer(i);
        }
    }
    // 广度优先搜索
    int count = 0;                    // 记录遍历的次数
    int[] res = new int[numCourses];  // 记录结果
    while (!q.isEmpty()) {
        int cur = q.poll();
        res[count] = cur;
        count++;
        for (int next : graph[cur]) {
            indegree[next]--;
            if (indegree[next] == 0) {
                q.offer(next);
            }
        }
    }
    // 如果存在环就返回空数组
    return count == numCourses ? res : new int[]{};
}
```
### [剑指 Offer II 115. 重建序列](https://leetcode-cn.com/problems/ur2n8P/)
这道题的关键点是如何保证重建序列的唯一性，其实就是判断`q.size() == 1`，也就是队列的大小是否为1。

还有一点细节需要注意，那就是先要保证重建序列集中的元素为`{1,...,n}`，这里用Set集合来判断。

广度优先搜索：
```java
class Solution {
    public boolean sequenceReconstruction(int[] org, List<List<Integer>> seqs) {
        // 保证重建序列集的完整性，即重建序列中元素能是 {1,...,n}
        Set<Integer> set = new HashSet<>();
        for (List<Integer> list : seqs) {
            for (int ele : list) {
                set.add(ele);
            }
        }
        // 如果set中元素不够，返回 false
        if (set.size() != org.length) return false;
        // 如果元素不匹配，返回 false
        for (int i = 1; i <= org.length; i++) {
            if (!set.contains(i)) {
                return false;
            }
        }
        
        List<Integer>[] graph = buildGraph(org, seqs);

        // 初始化队列
        Queue<Integer> q = new LinkedList<>();
        for (int i = 1; i < indegree.length; i++) {
            if (indegree[i] == 0) {
                q.offer(i);
            }
        }
        
        int count = 0;
        int[] res = new int[org.length];
        while (!q.isEmpty()) {
            // 保证唯一性
            if (q.size() > 1) return false;
            
            int cur = q.poll();
            res[count] = cur; // 记录拓扑排序结果
            count++;
            int nextCount = 0;
            for (int next : graph[cur]) {
                indegree[next]--;
                if (indegree[next] == 0) {
                    q.offer(next);
                    
                }
            }
        }
        
        // 出现环的情况
        if (count != org.length) return false;
        // 比较每一个元素
        for (int i = 0; i < org.length; i++) {
            if (org[i] != res[i]) return false;
        }
        
        return true;
    }
    
    // 建图
    int[] indegree; // 入度数组
    private List<Integer>[] buildGraph(int[] org, List<List<Integer>> seqs) {
        List<Integer>[] graph = new LinkedList[org.length + 1];
        indegree = new int[org.length + 1];
        for (int i = 0; i <= org.length; i++) {
            graph[i] = new LinkedList<>();
        }
        
        for (List<Integer> ele : seqs) {
            if (ele.size() < 2) continue;
            for (int i = 1; i < ele.size(); i++) {
                int from = ele.get(i - 1), to = ele.get(i);
                graph[from].add(to);
                indegree[to]++;
            }
        }
        return graph;
    }
}
```

## 参考资料
* [labuladong的算法小抄](https://labuladong.github.io/algo/)