### Uninformed Algorithms

#### 1.Breadth First Search

```
Graph:
	root:A
	struct:
		A:B,C
		B:D,E
		C:F

BreadthFirstSearch(Graph,dest):
	q = Queue()
	predecessor = Map()
	visited = Set()
	q.push(Graph.root)
	while(!q.isEmpty()):
		root = q.front()
		
		if root == dest:
			break
			
		q.pop()
		visited.add(root)
		predecessor.add(root,null)
		
		for each neb of Graph.struct[root]:
			if (!visited.has(neb)):
				visited.add(neb)
				predecessor.add(neb,root)
				q.push(neb)
	return predecessor
			
```
