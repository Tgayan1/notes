#### Cooperatives processes
- Processes which will affected by another processes while executing
- Either shares
	- Same address space -> data and code
	- Only data -> via files and messages
- When cooperative processes happens, it leads to *data inconsistencies* 
#### Race Condition
- Two cooperative process try to access the same data, the final value of the data will depend on the order of accessing the data
##### So, process synchronization is, allow cooperative process to run concurrently while keeping the shared data consistency.

### Problems with Process synchronization
#### Critical Section problem
- Section where a process try to grant the access for shared resource/ data is known as *Critical Section*
- For a complete solution, The solution must consisted with following properties.
	1. **Mutual Exclusion** -> if one process is granted the access, others must wait.
	2. **Progress** -> if resource is available, waiters can grant the access. *for the selection, process that not need the shared resource must not participate*
	3. **Bounded waiting** -> for a process, # of times have to wait for granting access to the shared data must be finite. *eventually everyone who required the access to the shared data must be granted the access.*

```
while (true) {
    entry section        // request permission to enter
	    critical section      // access shared data
    exit section           // release permission
    remainder section    // all other code
}
```

#### Software Based Solution - Peterson  Algorithm
- Doesn't work on Modern computer architectures. 
- Limited only for two processes.
- *Idea*->
	- Between two Processes ($P_i$  and $P_j$ ), there are two shared variables.
		- `int turn` -> *Indicates whose turn is to enter the critical section*
		- `boolean flag[]`-> *indicates the state of entering the critical section(ready or not)*
	- $P_i$ tries to enter the critical section
		- set `flag[i] = true` -> indicates it s ready to enter the critical section
		- set `turn = j` -> give chance to process $P_j$ (humble)
		- if process $P_j$ isn't ready, then $P_i$ will enter to the critical section. If not, it will wait and give chance to $P_j$
		- When the critical section procedure completed, it will exit by `flag[i] = false`
	
		```
		// Process P_i
		do{
			  //entering section
			  flag[i] = true;
			  turn = j;
			  while (flag[j] && turn == j);
			  
			  //critical section
			  
			  //exit section
			  flag[i] = flase;
			  
			  //remainder section
			  
		  } while(true);
		```
		
		  
