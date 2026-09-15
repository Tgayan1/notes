
## Normalization
#### Idea -> Obtain a well structured Relation / Database design

## Normalization Terminologies
### 1. Functional Dependency
- **Definition**:
	- Let $R$ be a *legal relational schema* (a relation that satisfies all real-world constraints). And $\alpha$ and $\beta$ represents two sets of *Attributes* such that,
	$$
	\alpha \subseteq R \ \ ,\ \ \beta \subseteq R
	$$
	- **Functional Dependency** $\alpha\rightarrow\beta$ Holds on $R$, *Using a tuple of $\alpha$,* *we can uniquely identify a tuple in $\beta$ .* 
	- Formally,
		 **If there exist two tuples $t_1,t_2$ : which agree on $\alpha$ , Must also agree on $\beta$.**
	 $$
	 t_1[\alpha] == t_2[\alpha] \implies t_1[\beta] == t_2[\beta]
	 $$
![[Pasted image 20260811141806.png|431]]

$R$ -> Instructor_Department(ID, name, salary, dept_name, building, budget)
*Functional Dependencies* ->
- ID -> name
- ID -> name, salary
- dept_name ->building
- dept_name ->building, budget
- ID -> dept_name
- ID, dept_name ->name, salary, building ....

### 2. Closure of Functional Dependencies

Let $F$ be a set of functional dependencies. A relation have functional dependencies that not are given in the $F$ but can logically imply using the set $F$.
- Set which contains all possible functional dependencies which can be logically implied by set $F$ is called the **Closure of** $F$ or $F^+$.
- So technically,  $F \rightarrow F^+$
#### Computation of $F^+$ 

 ![[Pasted image 20260811143302.png|437]]
 Inferring above axioms,![[Pasted image 20260811143410.png|578]]
#### Method - 01 -> 

![[Pasted image 20260811145721.png|461]]

### 3. Closure of Attribute Sets

Let $F$ be a set of functional dependencies defined on $R$. and $\alpha$ is a given attribute set.
**Closure** of $\alpha$ on $F$ is defined as the **complete set of attributes uniquely describable by** $\alpha$ on $F$.  Closure of $\alpha$ is Denoted as $\alpha^+$. 
#### Method ->

![[Pasted image 20260811151759.png|364]]
Ex :-

![[WhatsApp Image 2026-08-11 at 3.56.08 PM 1.jpeg|346]]

### 4. Keys Using Functional Dependencies
#### a. Super key
- **Super Key** -> Set of attributes which can use to identify a row uniquely in a table.
- **Definition** :
	- $K$ is super-key if and only if  $K \rightarrow R$ for a given Relation $R$.
	
 ![[Pasted image 20260811141806.png|349]]

- $K$ can be (ID, name, salary, dept_name, building, budget) or (ID, name) or (ID) for the relation *instructor_department*
#### b. Candidate Key
- **Candidate Key** -> Minimal super key 
- **Definition** :
	- $K$ is a candidate key if and only if $K \rightarrow R$ and,
	- $\not\exists \  \alpha$  such that $\alpha \subset K \  \ and \ \ \alpha \rightarrow R$ (set $K$ cannot reduce furthermore)
- ID attribute is a candidate key {${(ID)} \rightarrow R$ and ID cannot be furthermore reduce.} for the relation *instructor_department*.

EX:- (example...)

![[WhatsApp Image 2026-08-11 at 4.52.47 PM.jpeg|321]]
### 5. Attribute-to-Attribute Functional Dependency

- Every real-world constraints cannot be determine via super keys. Using functional Dependencies, we can capture them
	- ID is a super key (also the candidate key) for *instructor_department* relation.
		- $ID \rightarrow R \implies ID \rightarrow (name,salary)$ {ID is uniquely describes an instructor}
	- ${dept\_name} \rightarrow building$ {One department is located at on building} cannot be captured via super keys (dept_name isn't a super key)  

### 6. Trivial Functional Dependencies

- Set of attributes which uniquely describes about it self or a part of it.
- Formally, $\alpha \rightarrow \beta$ is **trivial** if and only of $\beta \subseteq \alpha$ 
	- $(ID,name) \rightarrow ID$
	- $name \rightarrow name$ 

### 7. Finding Closure of  Functional Dependencies using Closure of Attribute set

- Closure of $F$ or $F^+$ is,
	$$F \rightarrow F^+$$
- This $F^+$ isn't consist the direct derivations of F. $$ F^+ = (Logically \  Derive \  from \ F) \ \ + (Trivial \ Dependencies)$$
- Using the General algorithm It is hard. So we use attribute set approach.
- Method ->
```
R(n)                                 // relational schema
F(k)                                 // functional dependency set
closuer_F = F

gamma_set = permutate_set(R)             // powerset of R ex:R(A,B)
                                     // gamma = {(),(A),(B),(A,B)}
for each attr in gamma:
	closure_attr_set = compute_attrubiute_closuer(attr,F)
	for each attribute in closure_attr_set:
		append_functional_depedency(attr,attribute,closuer_F)
		
return closuer_F
```

- Note that, Practically **We don't Deriving $F^+$**. And note that there is $\emptyset$-**Null Set/Attribute** where no meaning with it. So, mathematically we need to include it on $F^+$ too.  

![[WhatsApp Image 2026-08-12 at 9.58.40 PM.jpeg|385]]

### 8. Canonical Cover

- Simplest set for functional Dependency set.
- **Definition**:
	- Canonical cover for functional dependency $F$; $F_c$ is the set which,
		- Equivalent to $F$
		- For each dependency, ==**RHS** only has single attribute==. And we can't remove any attribute from **LHS** without changing the $F^+$
		- Cannot remove any dependency without changing $F^+$. 
			- that is there isn't *Redundant Dependencies(same one) or transitive dependencies*.

![[WhatsApp Image 2026-08-13 at 8.13.40 AM.jpeg|480]]

### 9. Extraneous Attributes

- Attributes which can be removed from any F.D's LHS or RHS in a F.D set without changing the closure. 

- **Definition:**
	- Let $F$ is a functional Dependency on $R$. let $\alpha,\ \beta$ are attribute sets such that $\alpha\rightarrow\beta$. then let $A$ is an attribute where $A\in R$.
	- Case 1: $A$ is extraneous on $\alpha$ iff $A \in \alpha$ and $$F \rightarrow (F -\{\alpha \rightarrow \beta\}) \cup ((\alpha-A) \rightarrow \beta)  $$
		- That is, If we can derive an strong dependency $(\alpha - A) \rightarrow \beta$ **Without changing the $F^+$**, A is extraneous attribute on $\alpha$.
	- Case 2: $A$ is extraneous on $\beta$ iff $A \in \beta$ and $$F \rightarrow (F -\{\alpha \rightarrow \beta\}) \cup (\alpha \rightarrow (\beta-A))  $$
		- That is, If we can derive an strong dependency $\alpha \rightarrow (\beta - A)$ **Without changing the $F^+$**, A is extraneous attribute on $\beta$.
 
- **Method To test an attribute is extraneous or not**:
	- Case 1: $A$ is extraneous on $\alpha$
		```
		  S = {alpha - A}
		  closure_S = compute_closure_attr(S,F)
		  if (beta is in closuer_S):
			  return A is extraneous
		  return A is not extraneous
		```

	- Case 2: $A$ is extraneous on $\beta$
		```
		  F_hat = (F - functional_dependency(alpha,beta)) union functional_dependency(alpha,beta - A)
		  S = compute_closure(alpha,F_hat)
		  if(A in S):
			  return A is extraneous
		  return A is not extraneous
		```

**Computing Canonical cover with the help of extraneous attributes.**

![[WhatsApp Image 2026-08-13 at 9.27.09 AM.jpeg|413]]
  
## Normalization Theory

- Based on **Functional Dependencies** and **Multivalued Dependencies**.
- Target:
	- Convert Bad relation/database  design in to good one by *lossless decomposing* relations. 
- Meaning : *Multivalued dependency*
	- let $X,Y,Z$ be three sets of attributes. then Multivalued dependency means simply,
		1. $X \rightarrow \rightarrow Y$
		2. $X \rightarrow \rightarrow Y$
		3. $Y$ and $X$ are independent
	- Model R = (NIC, name, car_model) -> (name and car_model has no sense
	- of relation or logical connection.)

#### Decomposing Relations

![[Pasted image 20260813103100.png|565]]
#### Dependency Preservation
- Let R be a relation and lets decompose it $R = R_1,R_2,R_3...,R_i$
- Let F be functional dependencies of R. so, $F = F_1,F_2F_3...,F_n$
- Let $F_i \in F^+$ such that $F_i$ only includes attributes in $R_j \in R$.
- So a a certain decomposition said to be dependency preserving is, all original functional dependencies can be achieved directly without joining each decomposed restations again.
- Formally,$$(F_1 \cup F_2 \cup \dots \cup F_n)^+ = F^+$$
- 
#### Normal Forms

##### 1. First Normal Form - 1NF

- **Requirements :** 
	- Each attribute of a relation has an **atomic** (cannot further decompose the attribute) domain.

![[ChatGPT Image Aug 13, 2026, 12_08_14 PM.png]]
- This method Introduces higher redundancy. But with Higher Forms, we can further reduce the redundancy less than the first original table.

##### 2. Second Normal Form - 2NF

- **Requirements:** 
	1. Must meet the 1NF
		- All attributes must be atomic.
	2. There aren't any **Partial Dependencies** between non-prime attributes and prime attributes.
		- Let $\alpha \ , \beta$ be two attribute sets of $R$ such that $\alpha \rightarrow \beta$ and $\alpha \rightarrow R$. if $\exists  \ \gamma \subset \alpha$ such that $\gamma \rightarrow \beta$ , then $\beta$  has *Partial dependency* on $\alpha$.
		- **"no non-prime attribute is dependent on a part of a candidate key"**

EX:- Course Fee is Redundant.

![[Pasted image 20260813133040.png|428]]

The Candidate Key For Courses Table$$(STUD\_NO,COURSE\_NO) \rightarrow (STUD\_NO,COURSE\_NO,COURSE\_FEE)$$
 Each course has a Price.$$COURSE\_NO \rightarrow COURSE\_FEE$$
 ***Partial Dependency There***.$$COURSE\_NO \ \subset \  (STUD\_NO,COURSE\_NO)$$
**Solution**: Decompose it into Two relations. 
	1. Student_Enrollments 
	2. Course_info.

![[Pasted image 20260813133959.png|530]]

- Student_Enrollments -> 2NF satisfied. ✅
- Course_info -> 2NF satisfied. ✅
- Redundancy resolved!  ✅

#### Third Normal Form - 3NF

- **Requirements:**
	1. Must Hold 2NF
		- No *partial dependencies* must be there with prime and non-prime attributes.
	2. No **Transitive Dependency** must be there with prime and non-prime attributes.
		- *transitivity* :  $A \rightarrow B$ and $B \rightarrow C$ then $A \rightarrow C$
		- For a relation $R$ let $X,Y$ be attribute sets which holds $X \rightarrow Y$. For all *non-trivial functional dependency*, at least one of the following must hold.
			1. $X$ must be a super key.
				- $X \rightarrow R$
				- Avoids any transitivity.
			2. If $X$ isn't  a super key, Then $Y$ must be a candidate key/ part of candidate key.
				- If Y is a prime attribute, we allow the dependency because it involves candidate-key information, and decomposing it solely to eliminate this dependency may unnecessarily disrupt the relation's key structure.

![[ChatGPT Image Aug 18, 2026, 07_58_34 AM.png|653]]

#### Boyce-Codd Normal Form - BCNF

- **Requirements**:
	1. Relation must satisfy 3NF.
		- That is No transitive dependencies are allowed.
	2. For all functional dependencies, LHS must be the super key.
		- That is $\forall \ (x \rightarrow y)$ : $x$ must be the super key.
		- **If the LHS cannot determine the entire row on its own, it cannot be a determinant in this table.**

![[Gemini_Generated_Image_t7q5frt7q5frt7q5.jpg|617]]

