### Uninformed Algorithms

#### 1.Breadth First Search - Complete and Optimal when all edges are equal

```
Graph:
	root:A
	struct:
		A:B,C
		B:D,E
		C:F

GetPath(map,leaf):
	current = leaf
	path = Array()
	while(current != null):
		path.pushBack(current)
		current = map(current)
	path.reverse()
	return path

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
				
	return GetPath(predecessor,dest)
			
```

#### 2. Depth First Search - Complete but Not optimal (gives the first result)

```
Graph:
	root:A
	struct:
		A:B,C
		B:D,E
		C:F

GetPath(map,leaf):
	current = leaf
	path = Array()
	while(current != null):
		path.pushBack(current)
		current = map(current)
	path.reverse()
	return path
		
DepthFirstSearch(Graph,dest):
	s = Stack()
	visited = Set()
	predecessor = Map()
	
	predecessor.add(Graph.root,null)
	s.push(Graph.root)
	visited.add(Graph.root)
	
	while(!s.isEmpty()):
		root = s.pop()  
		                          
		if root == dest:
			break
			
		for each neb of Graph.struct[root]:
			if(!visited.has(neb)):
				visited.add(neb)
				s.push(neb)
				predecessor.add(neb,root)
				
	return GetPath(predecessor,dest)	
			
```

#### 3.Dijkastra Algorithm - Complete Optimal for Uneven edges

```
Graph:
	root:A
	struct:
		A: (B,w1),(C,w2)
		B: (D,w3),(E,w4)
		C: (F,w5)
		
Dijkastra(Graph,Dest):
	
```
