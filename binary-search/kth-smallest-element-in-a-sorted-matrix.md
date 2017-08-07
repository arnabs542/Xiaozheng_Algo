## Kth Smallest Element in a Sorted Matrix
_update Aug 4,2017 17:00_

---
[LeetCode](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/description/)

Given a n x n matrix where each of the rows and columns are sorted in ascending order, find the kth smallest element in the matrix.

Note that it is the kth smallest element in the sorted order, not the kth distinct element.

**Example:**

      matrix = [
         [ 1,  5,  9],
         [10, 11, 13],
         [12, 13, 15]
      ],
      k = 8,
      
      return 13.

**Note: **

You may assume k is always valid, 1 ? k ? n2.

#### Basic Idea:
**思路1：**
利用 priority queue 的性质，我们先把第一行的数字 offer，然后做如下事情 k-1 次：

将当前最小 poll，然后将它下面的元素 offer。

之后，queue.peek 就是第 k 小的数了。

这个方法的原理是我们每次都保证不在 queue 中的数字比 queue 中的都大，那么我们经过 k-1 次的poll之后，一定会得到第 k 小的元素。

代码如下：
java：
```java
    // 为了保存坐标，新建了 wrapper，注意override compareTo的细节
    class Node implements Comparable<Node> {
        int val, x, y;
        public Node(int val, int x, int y) {
            this.val = val;
            this.x = x;
            this.y = y;
        }
        @Override
        public int compareTo(Node that) {
            return this.val - that.val;
        }
    } 
    
    public class Solution {
    
        public int kthSmallest(int[][] matrix, int k) {
            PriorityQueue<Node> heapq = new PriorityQueue();
            for (int i = 0; i < matrix[0].length; ++i) {
                heapq.offer(new Node(matrix[0][i], 0, i));
            }
            for (int i = 0; i < k - 1; ++i) {
                Node peek = heapq.poll();
                int x = peek.x;
                int y = peek.y;
                if (x == matrix.length - 1) {
                    continue;
                }
                heapq.offer(new Node(matrix[x + 1][y], x + 1, y));
            }
            return heapq.peek().val;
        }
    }
```

python:
```python
    class Solution(object):
        def kthSmallest(self, matrix, k):
            """
            :type matrix: List[List[int]]
            :type k: int
            :rtype: int
            """
            heap = []
            for i in range(len(matrix[0])):
                heapq.heappush(heap, (matrix[0][i], 0, i))
            for i in range(k - 1):
                peek = heapq.heappop(heap)
                r = peek[1]
                c = peek[2]
                if r == len(matrix) - 1: # 如果已经在最下面一行
                    continue
                heapq.heappush(heap, (matrix[r + 1][c], r + 1, c))
            return heap[0][0]
```

**思路2：**
使用二分法对答案进行二分，然后判断估计的答案在matrix中的排名，判断时可以对每行进行二分。
```java
    // binary search
    // 采用二分答案的方法，初始 p = min(matrix), r = max(matrix)，每次对于猜测的答案mid
    // 在matrix中找mid的排名，
    public class Solution {
        public int kthSmallest(int[][] matrix, int k) {
    		int n = matrix.length;
    		int L = matrix[0][0], R = matrix[n - 1][n - 1];
    		while (L < R) {
    			int mid = L + ((R - L) >> 1);
    			int temp = 0;
    			for (int i = 0; i < n; i++) temp += binary_search(matrix[i], n, mid);
    			if (temp < k) L = mid + 1;
    			else R = mid;
    		}
    		return L;
    	}
    	
    	private int binary_search(int[] row,int R,int x){
    	    int L = 0;
    	    while (L < R){
    	        int mid = (L + R) / 2;
    	        if(row[mid] <= x) L = mid + 1;
    	        else R = mid;
    	    }
    	    return L;
    	}
    }
```



