## Course Schedule
_update Jul 18, 2017 14:11_

---
[lintcode](http://www.lintcode.com/en/problem/course-schedule/)

There are a total of n courses you have to take, labeled from 0 to n - 1.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: [0,1]

Given the total number of courses and a list of prerequisite pairs, is it possible for you to finish all courses?

    Example
    Given n = 2, prerequisites = [[1,0]]
    Return true
    
    Given n = 2, prerequisites = [[1,0],[0,1]]
    Return false

#### Basic Idea:
这个问题就是问一个有向图能否被 topological sort，即有向图中是否有环。首先把input的形式转换为adjacent list的形式，然后用bfs和indegree count（Kahn's Algorithm）的方法检查能否topological sort。

#### Java Code:
```java
    public class Solution {
        /**
         * @param numCourses a total of n courses
         * @param prerequisites a list of prerequisite pairs
         * @return true if can finish all courses or false
         */
        public boolean canFinish(int numCourses, int[][] prerequisites) {
            if (numCourses < 2) return true;
            Map<Integer, Set<Integer>> graph = initGraph(numCourses, prerequisites);
            // 得到adjacent list形式的图之后，用BFS进行topological sort，看是否可行
                // 先count indegree
            Map<Integer, Integer> count = new HashMap<>();
            for (int node : graph.keySet()) {
                count.put(node, 0);
            }
            for (int node : graph.keySet()) {
                for (int neighbor : graph.get(node)) {
                    count.put(neighbor, count.get(neighbor) + 1);
                }
            }
            // bfs
            Deque<Integer> queue = new LinkedList<>();
            for (int node : graph.keySet()) {
                if (count.get(node) == 0) queue.addFirst(node);
            }
            while (! queue.isEmpty()) {
                int node = queue.removeLast();
                for (int neighbor : graph.get(node)) {
                    count.put(neighbor, count.get(neighbor) - 1);
                    if (count.get(neighbor) == 0) queue.addFirst(neighbor);
                }
            }
            return Collections.max(count.values()) <= 0;
        }
        private Map<Integer, Set<Integer>> initGraph(int n, int[][] edges) {
            Map<Integer, Set<Integer>> graph = new HashMap<>();
            for (int i = 0; i < n; ++i) {
                graph.put(i, new HashSet<Integer>());
            }
            for (int[] edge : edges) {
                int u = edge[0], v = edge[1];
                graph.get(u).add(v);
            }
            return graph;
        }
    }
```

#### Python Code:
```python
    class Solution:
        # @param {int} numCourses a total of n courses
        # @param {int[][]} prerequisites a list of prerequisite pairs
        # @return {boolean} true if can finish all courses or false
        
        # 这就是一个问图能否topological sort的问题，但是所给信息的形式是node + edges
        # 的形式，我们还需要先将其转换为adjacent list的形式，即（Map<node, Set<neighbor>>）
        def canFinish(self, numCourses, prerequisites):
            def initGraph(n, edges):
                graph = {}
                for i in range(n):
                    graph[i] = set()
                for edge in edges:
                    u, v = edge[0], edge[1]
                    graph[u].add(v)
                return graph
                
            if numCourses < 2: return True
            graph = initGraph(numCourses, prerequisites)
            # count indegree
            count = {}
            for i in range(numCourses):
                count[i] = 0
            for node in graph:
                for neighbor in graph[node]:
                    count[neighbor] += 1
            # bfs, topological sort
            queue = collections.deque()
            for node in graph:
                if count[node] == 0:
                    queue.appendleft(node)
            while queue:
                node = queue.pop()
                for neighbor in graph[node]:
                    count[neighbor] -= 1
                    if count[neighbor] == 0:
                        queue.appendleft(neighbor)
            
            return not [node for node in graph if count[node] > 0]
```