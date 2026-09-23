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
	route = Array()
	visited = Set()
	q.push(Graph.root)
	while(!q.isEmpty()):
		root = q.front()
		
		if root == dest:
			break
			
		q.pop()
		visited.add(root)
		route.pushBack(root)
		
		for each neb of Graph.struct[root]:
			if (!visited.has(neb)):
				visited.add(neb)
				q.push(neb)
	return route
			
```
