![[ch9_main_memory.pdf]]


#### Paging

- Physical -> Frame -> fixed size -> powers of 2
- Logical -> Page
- Page <------ Frames => mapping by => page table
	- frames doesn't have to contiguous.
- Reduce external Fragmentations
- Cannot Solve internal Fragmentations
	- worst case -> use only one byte form each frame
	- average case -> half of a frame size
	- minimizing it -> growing page size
- Page  -> "Page number" + "page offset"
- "page number" -> **page table** -> frame's address


#### Page table Implementation
- Page table -> Reside in Main memory -> (Base address, Length) -> Stores in Registers
- CPU issues address -> look up in page table and decode physical access -> go to the decoded address
- Two memory access Problem
	- TLB / Associative memory
		- Separate cache for page table lie normal cache

- Effective Access Time
$${EAT = Hit \ \ Ratio * (access \ \ time) + (1 - Hit \ \ Ratio) * (access \ \ time * 2) }$$
- Slowdown -> $$\frac{EAT - access \ \ time}{access \ \ time}$$
  ?? what is read-write page??
  ?? Hierarchical -> set association??

- Page size is 4KB.Page offset is 12 bits. Determine
	1. 32bit addressing system with two level
	2. 64 bit addressing with two level
	3. 64 addressing with three level