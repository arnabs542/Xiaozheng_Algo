# Closest Binary Search Tree Value II

_update Sep 1, 2017 0:09_

[LeetCode](https://leetcode.com/problems/closest-binary-search-tree-value-ii/description/)

Given a non-empty binary search tree and a target value, find k values in the BST that are closest to the target.

**Note:** Given target value is a floating point. You may assume k is always valid, that is: k ≤ total nodes. You are guaranteed to have only one unique set of k values in the BST that are closest to the target.

**Follow up:** Assume that the BST is balanced, could you solve it in less than O\(n\) runtime \(where n = total nodes\)?

### Basic Idea:

**基本思路：** 最简单的思路当然是inorder遍历一遍 BST，维持一个size为 k 的最大堆，key 是 `abs(val-target)` ，沿途更新（每次淘汰堆顶距离target最大的val）。但是这样的做法当然配不上这样一道 Hard 难度的题目。要想体会这道题目的深深恶意，必须考虑低于 O\(n\) 的解法；

这里附上随手写的这种做法的 Python code:

```python
    class Solution(object):
        def closestKValues(self, root, target, k):
            """
            :type root: TreeNode
            :type target: float
            :type k: int
            :rtype: List[int]
            """
            def inorder(root):
                if not root: return
                inorder(root.left)
                heapq.heappush(pq, (-abs(root.val - target), root.val))
                if len(pq) > k:
                    heapq.heappop(pq)
                inorder(root.right)

            pq = []
            inorder(root)
            return [t[1] for t in pq]
```

**更快的方法：** 我们注意到事实上这道题目很像一道`二分法 + 2 pointers`的题 [K Closest Numbers In Sorted Array](https://will-gxz.gitbooks.io/xiaozheng_algo/content/binary-search/k-closest-numbers-in-sorted-array.html), 在这道题目中，我们先用二分法找到最近的位置，然后用 2 pointers 向两边扩展，耗时 O\(logN + k\)。

那么在现在这道题中，无疑非常相似，不同之处在于使用双指针没有那么方便，因为我们需要持续地获取 predecessor 和 successor，而题目中的 TreeNode 没有 parent 指针。

于是，一种思路浮现在脑海，我们可以写两个函数，分别获取某节点的 predecessor 和 successor，每次耗时最坏情况下为 O\(logN\), 而我们共需要找 k 次，总计耗时 O\(klogN\)，一般来说这是优于 O\(n\) 的，所以我最先写了如下代码：

Java Code:

```java
    class Solution {
        public List<Integer> closestKValues(TreeNode root, double target, int k) {
            // 先找到距离target最近的
            TreeNode target_node = root;
            TreeNode temp = root;

            for (;;) {
                if (target < temp.val && temp.left != null) {
                    temp = temp.left;
                } else if (target > temp.val && temp.right != null) {
                    temp = temp.right;
                } else {
                    break;
                }
                if (Math.abs(temp.val - target) < Math.abs(target_node.val - target)) target_node = temp;
            }

            List<Integer> res = new ArrayList<>();
            // 用类似于 2 pointers 的算法
            int count = 1;
            res.add(target_node.val);
            TreeNode pred = getPred(root, target_node);
            TreeNode succ = getSuc(root, target_node);
            while (count < k) {
                if (pred == null) {
                    res.add(succ.val);
                    succ = getSuc(root, succ);
                    count++;
                } else if (succ == null) {
                    res.add(pred.val);
                    pred = getPred(root, pred);
                    count++;
                }
                else if (Math.abs(pred.val - target) < Math.abs(succ.val - target)) {
                    res.add(pred.val);
                    pred = getPred(root, pred);
                    count++;
                } else {
                    res.add(succ.val);
                    succ = getSuc(root, succ);
                    count++;
                }
            }
            return res;
        }

        private TreeNode getPred(TreeNode root, TreeNode target) {
            // 有左子树，则predecessor在左子树，最大的
            if (target.left != null) {
                target = target.left;
                while (target.right != null) {
                    target = target.right;
                }
                return target;
            }
            // 否则，是第一个左祖先
            TreeNode curr = root;
            TreeNode ret = null;
            while (curr != target) {
                if (target.val < curr.val) {
                    curr = curr.left;
                } else {
                    ret = curr;
                    curr = curr.right;
                }
            }
            return ret;
        }
        private TreeNode getSuc(TreeNode root, TreeNode target) {
            // 有右孩子，则successor一定在右子树
            if (target.right != null) {
                target = target.right;
                while (target.left != null) {
                    target = target.left;
                }
                return target;
            }
            // 否则是第一个右祖先
            TreeNode curr = root;
            TreeNode ret = null;
            while (curr != target) {
                if (target.val < curr.val) {
                    ret = curr;
                    curr = curr.left;
                } else {
                    curr = curr.right;
                }
            }
            return ret;
        }
    }
```

**优化：** 接下来我们注意到之前的code中，每次 get predecessor 和 successor 的过程中最耗时的就是从root开始向下找最近的左祖先或右祖先，每次调用都要运行一遍，毫无疑问是重复的，于是我们就想能否优化这一步骤，答案是可以。

对successor 和 predecessor 分别维持一个stack，这里用 successor 举例：

当我们有如下BST的时候： 

![](../../.gitbook/assets/WechatIMG30%20%281%29.jpg)

经过优化，获取 predecessor 和 successor 的操作只有一开始的initialize stack 是 O\(logN\)的，之后每次获取事实上都是 O\(1\), 因此，时间复杂度变为 O\(k + logN\).

优化之后的Code：

```java
class Solution {
    Deque<TreeNode> sucStack;
    Deque<TreeNode> predStack;
    TreeNode root;
    public List<Integer> closestKValues(TreeNode root, double target, int k) {
        this.root = root;
        // find closest node first
        TreeNode temp = root, closest = root;
        for (;;) {
            if (temp.val < target) {
                if (temp.right == null) break;
                temp = temp.right;
            } else {
                if (temp.left == null) break;
                temp = temp.left;
            }
            if (Math.abs(temp.val - target) < Math.abs(closest.val - target)) {
                closest = temp;
            }
        }

        // init sucStack and predStack for closest node
        initSucStack(closest);
        initPredStack(closest);

        // two pointers find k closest
        List<Integer> ret = new ArrayList<>();
        ret.add(closest.val);
        k--;
        Integer suc = getNextSuc(), pred = getNextPred();
        while (k > 0) {
            if (suc == null) {
                ret.add(pred);
                pred = getNextPred();
            } else if (pred == null) {
                ret.add(suc);
                suc = getNextSuc();
            } else if (Math.abs(target - suc) < Math.abs(target - pred)) {
                ret.add(suc);
                suc = getNextSuc();
            } else {
                ret.add(pred);
                pred = getNextPred();
            }
            k--;
        }
        return ret;
    }

    private void initSucStack(TreeNode node) {
        this.sucStack = new LinkedList<>();
        // add right parents
        TreeNode temp = root;
        while (temp != node) {
            if (temp.val < node.val) {
                temp = temp.right;
            } else {
                sucStack.addLast(temp);
                temp = temp.left;
            }
        }
        // add node's right child, then left all the way
        temp = node.right;
        while (temp != null) {
            sucStack.addLast(temp);
            temp = temp.left;
        }
    }

    private void initPredStack(TreeNode node) {
        this.predStack = new LinkedList<>();
        // add left parents
        TreeNode temp = root;
        while (temp != node) {
            if (temp.val < node.val) {
                predStack.addLast(temp);
                temp = temp.right;
            } else {
                temp = temp.left;
            }
        }
        // add node's left child, and all the way right
        temp = node.left;
        while (temp != null) {
            predStack.addLast(temp);
            temp = temp.right;
        }
    }

    private Integer getNextSuc() {
        if (sucStack.isEmpty()) return null;
        TreeNode ret = sucStack.removeLast();
        TreeNode temp = ret.right;
        while (temp != null) {
            sucStack.addLast(temp);
            temp = temp.left;
        }
        return ret.val;
    }

    private Integer getNextPred() {
        if (predStack.isEmpty()) return null;
        TreeNode ret = predStack.removeLast();
        TreeNode temp = ret.left;
        while (temp != null) {
            predStack.addLast(temp);
            temp = temp.right;
        }
        return ret.val;
    }
}
```

_update Dec 25, 2017 4:14_

### 一点感想

  这道题可以说是非常复杂了，但是如果将其分为几个部分，分别实现起来其实并不复杂;  
  首先我们想到的就是 binary search + 2 pointers 的在数组中找 k closest elements 的 O\(logn + k\) 的算法，并且想把它用在BST中，于是我们必须要解决找到 1st closest 之后如何向左右遍历的问题;  
  接下来自然会想到写两个函数：`getSuccessor(), getPredecessor()`，但是这样的话总时间复杂度为O\(logn + klogn\);  
  为了优化之前的方法，我们建了两个 stack：`sucStack, predStack`，用来加速对于 `successor 和 predecessor` 查找的速度，这样一来每找一次的时间复杂度从 `O(logN)` 降到 `O(1)`，总时间复杂度降为 `O(k + logN)`;

**Python Code:**

```python
class Solution:
    def closestKValues(self, root, target, k):
        """
        :type root: TreeNode
        :type target: float
        :type k: int
        :rtype: List[int]
        """
        def initSucStack(node):
            stack = []
            temp = root
            # add right parent
            while temp is not node:
                if temp.val < node.val:
                    temp = temp.right
                else:
                    stack.append(temp)
                    temp = temp.left
            # add node's right child and left all the way
            temp = node.right
            while temp:
                stack.append(temp)
                temp = temp.left
            return stack


        def initPredStack(node):
            stack = []
            temp = root
            # add left parent
            while temp is not node:
                if temp.val < node.val:
                    stack.append(temp)
                    temp = temp.right
                else:
                    temp = temp.left
            # add node's left child and all the way right
            temp = node.left
            while temp:
                stack.append(temp)
                temp = temp.right
            return stack


        def getNextSuc():
            if not sucStack:
                return None
            ret = sucStack.pop()
            temp = ret.right
            while temp:
                sucStack.append(temp)
                temp = temp.left
            return ret.val


        def getNextPred():
            if not predStack:
                return None
            ret = predStack.pop()
            temp = ret.left
            while temp:
                predStack.append(temp)
                temp = temp.right
            print(ret.val)
            return ret.val



        # find closest first
        temp = root
        node = root
        while True:
            if temp.val < target:
                if not temp.right: break
                temp = temp.right
            else:
                if not temp.left: break
                temp = temp.left
            if abs(temp.val - target) < abs(node.val - target):
                node = temp

        # initialize sucstack and predstack for node
        sucStack = initSucStack(node)
        predStack = initPredStack(node)

        # two pointers
        ret = []
        ret.append(node.val)
        k -= 1
        suc, pred = getNextSuc(), getNextPred()
        while k > 0:
            if suc is None:
                ret.append(pred)
                pred = getNextPred()
            elif pred is None:
                ret.append(suc)
                suc = getNextSuc()
            elif abs(target - suc) < abs(target - pred):
                ret.append(suc)
                suc = getNextSuc()
            else:
                ret.append(pred)
                pred = getNextPred()
            k -= 1

        return ret
```

_udpate Jan 25,2018 10:23_

## Update: 270. Closest Binary Search Tree Value

[LeetCode](https://leetcode.com/problems/closest-binary-search-tree-value/description/)

Given a non-empty binary search tree and a target value, find the value in the BST that is closest to the target.

**Note:**  
Given target value is a floating point. You are guaranteed to have only one unique value in the BST that is closest to the target.

* **Basic Idea:**

  这是前一道题的 easy 版本。首先我们要明确，如果一个BST中的元素 E 有左右孩子，距离它最近的元素一定在其孩子子树中，如果没有左孩子，则距离其最近的比它小的元素为其最近一个左祖先，如果没有右孩子，则距离其最近的比它大的元素一定是其最近一个右祖先。所以我们可以肯定只要沿着搜索路径向下搜索，一定可以经过距离target最近的元素。  

  **时间复杂度：** `O(height)`

  * **Java Code:**

    ```java
    class Solution {
       public int closestValue(TreeNode root, double target) {
           int ret = Integer.MAX_VALUE;
           double minDiff = (double)Integer.MAX_VALUE * 100;
           TreeNode curr = root;
           while (curr != null) {
               double diff = Math.abs((double)curr.val - target);
               if (diff < minDiff) {
                   ret = curr.val;
                   minDiff = diff;
               }
               if (curr.val < target) {
                   curr = curr.right;
               } else {
                   curr = curr.left;
               }
           }
           return ret;
       }
    }
    ```

_update 2018-06-10 11:48:294_

## Update C++ Solution

* **C++ PriorityQueue Solution**

  ```cpp
  class Solution {
      template<class T>
      void helper(TreeNode* root, T& pq, double target, int k) {
          if (root == nullptr) return;
          helper(root->left, pq, target, k);
          pq.push(make_pair(abs(root->val - target), root->val));
          if (pq.size() > k) pq.pop();
          helper(root->right, pq, target, k);
      }
  public:
      vector<int> closestKValues(TreeNode* root, double target, int k) {
          auto comp = [](pair<double, int> a, pair<double, int> b)->bool{ return a.first < b.first; };
          priority_queue<pair<double, int>, vector<pair<double, int>>, decltype(comp)> pq(comp);
          helper(root, pq, target, k);

          vector<int> res;
          while (! pq.empty()) {
              pair<double, int> _pair = pq.top();
              pq.pop();
              cout << _pair.second << endl;
              res.push_back(_pair.second);
          }
          return res;
      }
  };
  ```

* **一种更快的方法**

  这是已知最快的方法，达到了 `O(n)` 的时间复杂度。（虽然对于给定排序数组找距离target最近k个数的问题可以用 binary search 做到 `O(logN + k)`， 但由于这里我们需要 `O(n)` 的时间遍历整个 BST，所以我们可以直接进行一个 inorder traversal，然后同时获取答案）

**基本思路：**  
inorder traversal，相当于从小到大遍历 sorted array。维持一个 deque，每次遇到新的数的时候，看 `deque[0]` 和 当前数字 curr 谁距离target更近，如果 curr 更近，则 移除 deque 最左边的数字，将 curr 插入到 deque 最右。这样，最终就可以得到距离 target 最近的 k 个数。这种方法其实相当于利用已经排序数组的性质手动维持了一个 priority queue，而这个 priority queue 一定是已经排序的。

```cpp
class Solution {
    void inorder(TreeNode* root, deque<int>& q, int k, double target) {
        if (root == nullptr) return;
        inorder(root->left, q, k, target);
        if (q.empty() || q.size() < k || abs(q.front() - target) > abs(root->val - target)) {
            q.push_back(root->val);
        }
        if (q.size() > k) q.pop_front();
        inorder(root->right, q, k, target);
    }
public:
    vector<int> closestKValues(TreeNode* root, double target, int k) {
        deque<int> q;
        inorder(root, q, k, target);
        vector<int> res;
        for (int n : q) res.push_back(n);
        return res;
    }
};
```

