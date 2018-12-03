## 二叉树节点定义如下
```python
# -*- encoding=utf-8
class Node(object):
    def __init__(self, val):
        super(Node, self).__init__()
        self.val = val
        self.left = None
        self.right = None
```
## 前序遍历
```python
def preorder(node, res):
    if not node:
        return
    res.append(node.val)
    preorder(node.left, res)
    preorder(node.right, res)

def preorderNonRecursive(root):
    s = []
    res = []
    node = root
    while s or node:
        if node:
            res.append(node.val)
        if not node:
            node = s.pop()
            node = node.right
        elif node.left:
            s.append(node)
            node = node.left
        else:
            node = node.right
    return res
```
## 中序遍历
```python
def inorder(node, res):
    if node.left:
        inorder(node.left, res)
    res.append(node.val)
    if node.right:
        inorder(node.right, res)

def inorderNonRecursive(root):
    node = root
    res = []
    s = []
    while s or node:
        if not node:
            node = s.pop()
            res.append(node.val)
            node = node.right
        elif node.left:
            s.append(node)
            node = node.left
        else:
            res.append(node.val)
            node = node.right
    return res
```
## 后续遍历
```python
def postorder(node, res):
    if not node:
        return
    postorder(node.left, res)
    postorder(node.right, res)
    res.append(node.val)

def postorderNonRecursive(root):
    s = []
    res = []
    node = root
    while s or node:
        if not node:
            node = s.pop()
            res.append(node.val)
            node = node.right
        elif node.left:
            s.append(node)
            node = node.left
        else:
            s.append(node)
            node = node.right
```