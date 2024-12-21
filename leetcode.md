# 1.树相关

[1372. 二叉树中的最长交错路径](https://leetcode.cn/problems/longest-zigzag-path-in-a-binary-tree/)

已解答

中等

相关标签

提示

给你一棵以 `root` 为根的二叉树，二叉树中的交错路径定义如下：

- 选择二叉树中 **任意** 节点和一个方向（左或者右）。
- 如果前进方向为右，那么移动到当前节点的的右子节点，否则移动到它的左子节点。
- 改变前进方向：左变右或者右变左。
- 重复第二步和第三步，直到你在树中无法继续移动。

交错路径的长度定义为：**访问过的节点数目 - 1**（单个节点的路径长度为 0 ）。

请你返回给定树中最长 **交错路径** 的长度。

```java
class Solution {
    // point 当前方向 0 左，1 右
    //

    // 存在左和右两种状态的极限值 --- [0] [1] 当前节点，从左子数过来的最大值，从右子树过来
    //
    
    int res;

    public List<Integer> dfs(TreeNode root){
        if(root == null){

            List<Integer> r = new ArrayList<>();
            r.add(-1);
            r.add(-1);
            return r;
        }
else{
            List<Integer> l = dfs(root.left);
            List<Integer> r= dfs(root.right);

            List<Integer> now = new ArrayList<>();
            // 左过来的最大值
            now.add(l.get(1) + 1); // 左的右子树值加1
            now.add(r.get(0) + 1); // 右的左子树加1
            res = Math.max(res,Math.max(now.get(0),now.get(1)));
            return now;
        }
        
    }

    public int longestZigZag(TreeNode root) {
        res = Integer.MIN_VALUE;
        dfs(root);
        
        return res;
    }
}
```

