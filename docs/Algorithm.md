# Algorithm

## quick sort reference
```python
import bigarray


def quicksort(a, lo, hi):
	idx = partition(a, lo, hi)
	if idx > lo:
		quicksort(a, lo, idx-1)
	if idx < hi:
		quicksort(a, idx+1, hi)


def partition(a, lo, hi):
	idx = lo
	for i in range(lo, hi):
		if a[i] < a[hi]:
			a[i], a[idx] = a[idx], a[i]
			idx += 1
	a[hi], a[idx] = a[idx], a[hi]
	return idx


def find_k(a, lo, hi, k):
	idx = partition(a, lo, hi)
	if idx < k:
		find_k(a, idx+1, hi, k)
	elif idx > k:
		find_k(a, lo, idx-1, k)
	elif idx == k:
		print a[idx]


if __name__ == '__main__':
	a = [6,5,4,0,2,1,3]
	quicksort(a, 0, len(a)-1)
	print a
	# find_k(a, 0, len(a)-1, 6)
	# print a[6]
```

## bfs
```python
import Queue
from tree import build_tree

def prettify_bfs(node):
	q = Queue.Queue()
	q.put(node)
	while not q.empty():
		node = q.get()
		print node.depth * "--" + node.val
		for child in node.children:
			child.depth = node.depth + 1
			q.put(child)


def bfs(node):
	q = Queue.Queue()
	q.put(node)
	while not q.empty():
		node = q.get()
		print node.val
		for child in node.children:
			q.put(child)


def bfs_recursive(node):
	q = Queue.Queue()
	q.put(node)
	def traverse(q):
		if q.empty():
			return
		node = q.get()
		print node.val
		for child in node.children:
			q.put(child)
		traverse(q)
	traverse(q)

if __name__ == '__main__':
	root = build_tree()
	# bfs(root)
	prettify_bfs(root)
```

## dfs
```python
import Queue
from tree import buil_tree


def prettify_dfs(node, depth):
	print depth * "--" + node.val
	for child in node.children:
		prettify_dfs(child, depth + 1)


def dfs_recursive(node):
	print node.val
	for child in node.children:
		dfs_recursive(child)


def dfs(node):
	stack = Queue.LifoQueue()
	stack.put(node)
	while not stack.empty():
		node = stack.get()
		print node.depth * "--" + node.val
		for child in node.children:
			child.depth = node.depth + 1
			stack.put(child)


if __name__ == '__main__':
	root = buil_tree()
	dfs(root)
```