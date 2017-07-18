## Validate Binary Search Tree
_update Jul 14, 2017 18:13_

---
[LintCode](http://www.lintcode.com/en/problem/validate-binary-search-tree/)

Given a binary tree, determine if it is a valid binary search tree (BST).

Assume a BST is **defined** as follows:

The left subtree of a node contains only nodes with keys less than the node's key.
The right subtree of a node contains only nodes with keys greater than the node's key.
Both the left and right subtrees must also be binary search trees.
A single node tree is a BST

    Example
    An example:
    
      2
     / \
    1   4
       / \
      3   5
    The above binary tree is serialized as {2,1,4,#,#,3,5} (in level order).
    
#### Basic Idea:
利用定义，每一个子树都需要是vlaid bst。对于每个root，需要保证root.val 大于左子树中的最大值，小于右子树的最小值。由此，我们可以定义一个ReturnType，其中包含`boolean isValid, max, min`三个变量。
（需要注意的是，当node的值是Integer.MAX_VALUE的时候，会造成边界判定不准，可以转成(long)Integer.MAX_VALUE + 1之类来处理）

#### Java Code:
```java
    public class Solution {
        /**
         * @param root: The root of binary tree.
         * @return: True if the binary tree is BST, or false
         */
        // 使用分治法
        private class Return {
            boolean isValid = true;
            long min = Integer.MAX_VALUE;
            long max = Integer.MIN_VALUE; 
            public Return(boolean b, long min, long max) {
                isValid = b;
                this.min = min;
                this.max = max;
            }
        }
        public boolean isValidBST(TreeNode root) {
            if (root == null) return true;
            Return ret = helper(root);
            return ret.isValid;
        }
        private Return helper(TreeNode root) {
            if (root == null) return new Return(true, (long)Integer.MAX_VALUE + 1, (long)Integer.MIN_VALUE - 1);
            if (root.left == null && root.right == null) {
                return new Return(true, root.val, root.val);
            } 
            Return left = helper(root.left);
            Return right = helper(root.right);
            boolean flag = true;
            long max = Math.max(left.max, Math.max(right.max, root.val));
            long min = Math.min(left.min, Math.min(right.min, root.val));
            if (! left.isValid || ! right.isValid || root.val >= right.min || root.val <= left.max) {
                flag = false;
            }
            return new Return(flag, min, max);
        }
    }
```