# Populating Next Right Pointers in Each Node

## Populating Next Right Pointers in Each Node

_update Sep 1, 2017 23:10_

[LeetCode](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/description/)

Given a binary tree

```c
    struct TreeLinkNode {
      TreeLinkNode *left;
      TreeLinkNode *right;
      TreeLinkNode *next;
    }
```

Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to NULL.

Initially, all next pointers are set to NULL.

**Note:**

You may only use constant extra space. You may assume that it is a perfect binary tree \(ie, all leaves are at the same level, and every parent has two children\).

**For example,**

```text
Given the following perfect binary tree,
         1
       /  \
      2    3
     / \  / \
    4  5  6  7
After calling your function, the tree should look like:
         1 -> NULL
       /  \
      2 -> 3 -> NULL
     / \  / \
    4->5->6->7 -> NULL
```

### Basic Idea:

[这里](https://siddontang.gitbooks.io/leetcode-solution/content/tree/populating_next_right_pointers_in_each_node.html) 有一个不错的分析。

题目要求 O\(1\)space, 说明不能用 BFS。注意到我们可以利用构建好的 next 指针在同层中移动，利用这条性质可以避免 extra space；

同时，因为所给树是一个完美满二叉树，我们注意到如下规律：

```java
    node.left.next = node.right
    node.right.next = node.next.left
```

### Java Code:

```java
public class Solution {
    public void connect(TreeLinkNode root) {
        TreeLinkNode first = root;
        // 用一个first沿着左边边线向下遍历
        while (first != null) {
            // 用curr和一个内层循环，横向遍历每层
            TreeLinkNode curr = first;
            while (curr.left != null) {
                curr.left.next = curr.right;
                if (curr.next == null) break;
                curr.right.next = curr.next.left;
                curr = curr.next;
            }
            first = first.left;
        }
    }
}
```

## Populating Next Right Pointers in Each Node II

[LeetCode](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/description/)

Given a binary tree

```text
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```

Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to `NULL`.

Initially, all next pointers are set to `NULL`.

**Follow up:**

* You may only use constant extra space.
* Recursive approach is fine, you may assume implicit stack space does not count as extra space for this problem.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/02/15/117_sample.png)

```text
Input: root = [1,2,3,4,5,null,7]
Output: [1,#,2,3,#,4,5,7,#]
Explanation: Given the above binary tree (Figure A), your function should populate each next pointer to point to its next right node, just like in Figure B. The serialized output is in level order as connected by the next pointers, with '#' signifying the end of each level.
```

**Constraints:**

* The number of nodes in the given tree is less than `6000`.
* `-100 <= node.val <= 100`

### Basic Idea:

推广到任意二叉树后，前面的解法就不成立了，需要进行修正。这道题目中，任务变成了已知上层的next关系，推导下层的next关系；

这次，不再无脑使用：

```java
    node.left.next = node.right
    node.right.next = node.next.left
```

我们还需要加入判断。对于first，不再无脑往left走，而是每层循环之前将下一层的first设为null，用本层第一个child更新下一层的first。用变量lastNode跟踪之前一个visit过的node，即当前需要为其设置next的node。

**Java Code:**

```java
// update Jul 14, 2021
class Solution {
    public Node connect(Node root) {
        Node head = root;
        while (head != null) {
            Node curr = head; // 在当前head同一层向右移动
            Node prev = null; // 在下一层
            Node nextHead = null;
            while (curr != null) {
                if (curr.left != null) {
                    if (nextHead == null) nextHead = curr.left;
                    if (prev != null) prev.next = curr.left;
                    prev = curr.left;
                }
                if (curr.right != null) {
                    if (nextHead == null) nextHead = curr.right;
                    if (prev != null) prev.next = curr.right;
                    prev = curr.right;
                }
                curr = curr.next;
            }
            head = nextHead;
        }
        return root;
    }
}
```

_update 2018-06-16 01:42:31_

### Update C++ Code for the Follow Up Question:

用C++实现，优化了判断的逻辑，更好理解。

```cpp
  class Solution {
  public:
      void connect(TreeLinkNode *root) {
          TreeLinkNode* head = root;
          while (head) {
              TreeLinkNode* node = head;
              head = nullptr;
              TreeLinkNode* prev = nullptr;
              while (node) {
                  if (node->left) {
                      if (! head) head = node->left;
                      if (! prev) prev = node->left;
                      else {
                          prev->next = node->left;
                          prev = node->left;
                      }
                  }
                  if (node->right) {
                      if (! head) head = node->right;
                      if (! prev) prev = node->right;
                      else {
                          prev->next = node->right;
                          prev = node->right;
                      }
                  }
                  node = node->next;
              }
          }
      }
  };
```

