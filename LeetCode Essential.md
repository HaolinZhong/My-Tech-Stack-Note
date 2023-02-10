# General

- 注意判空



# Binary Tree

## 遍历

- 合理使用stack, queue等结构
- 最常见的遍历: 
  - 循环条件: `栈不为空 || node指针不为空 `



## Tree Comparison

### Serialization

- uniquely identify a binary tree with 1 traversal:
  - use `#` to represent `null` nodes (enough for identification)
- add `^` before node values (to avoid mismatching due to the string value of the node value, like `2 -> 2##` and `22 -> 22##` ) for comparison
- string matching: KMP, or use built in `indexOf` method



### Hash Tree

- how to hash a tree:
  - 对于`null`, 返回其值为`3` (任何质数都可以)
  - left subtree hash 值左移一个固定数字, right subtree hash 值左移1
  - 两值相加, 再加上当前node的值, 即得到树的hash值