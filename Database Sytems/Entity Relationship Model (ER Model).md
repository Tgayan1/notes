- Main Design Goals are,
	- Avoid Redundancy (Repeat same data in the database)
	- Avoid Incompleteness
### Terminology

1. `entity` -> object instance. distinguish with other objects
2. `entity set` -> set of objects that possess same properties or `attributes`
3. `relationship` -> connection between two or more entities
4. `relationship set` -> set of relations which occurs on two or more entity sets having/not having `attributes`
5. `degree` of a relationship -> Number of entities involved in the relationship (n-ary entities)
	- ***binary*** -> 2 entities 
	- **ternary*** -> 3 entities
6. *Mapping cardinalities* for Binary Relationships
	- number of entities to which another entity can be associated via a relationship set
		1. one to one
		2. one to many
		3. many to one
		4. many to many
		
		![[Pasted image 20260710200638.png|519]]

7. ***Attributes***
	- `domain` -> set of permitted values for an attribute
	- Attribute Types
		- Simple attributes -> attributes cannot be further reduced
		- Composite attributes -> consisted with two or more simple values
		- Single-valued attributes 
		- Multi-valued attributes
		- Derived attributes -> computed using existing attributes
	- `keys` -> set of special attributes 
		- `super key` -> set of one or more attributes whose values uniquely determine each entity
		- `candidte key` -> **minimal super key***
		- `primary key` -> key which is used to identify records.
8. ***Weak Entities***
	- Entity which is dependent on another entity.
		- Properties 
			- Weak entity `totally participated`in the relationship with *identifying entity*.
			- Primary key for `identifying relationship` will contain the primary key of identifying relationship and **discriminating attribute(s)*** of weak entity. 

### Entity Relationship Diagram (ER Diagram)

![[ChatGPT Image Jul 23, 2026, 05_39_33 PM.png]]

## Reduction ER Diagram In to Relational Schema

![[Lab 1.1.png|577]]


- Strong Entities -> Tables
- Weak Entities -> Tables with setting a composite primary key including,
	- *identifying strong entities' primary key* 
	- *discriminators* of the weak entity 
- Relations ->
	- Many-to Many relations -> 
		- Tables with both primary keys composite as a single primary key (**Composite Primary Key**)
	- One-to-Many or Many-to-One relations ->
		- Add primary key of *one side* as an additional attribute to the *many side* (**Foreign Key**) 

```
	  branch(
		  branch_id PRIMARY KEY,
		  location
	  )
	  license(
		  license_no PRIMARY KEY,
		  owner UNIQUE REFERENCES candidate(NIC) NOT NULL,
		  issued_on,
		  expired_on,
		  type,
		  issued_by REFERENCES branch(branch_id) NOT NULL
	  )
	  candidate(
		  NIC PRIMARY KEY,
		  full_name,
		  address
	  )
	  exam(
		  exam_no PRIMARY KEY,
		  type
	  )
	  payment(
		  ref_no PRIMARY KEY,
		  date,
		  amount,
		  for_attempt UNIQUE REFRENCES attempt(attempt_id) NOT NULL
	  )
	  attempt(
		  attempt_id PRIMARY KEY,
		  date,
		  result,
		  candidate_id UNIQUE REFRENCES candidate(NIC) NOT NULL,
		  exam_no UNIQUE REFERENCES exam(exam_no) NOT NULL,
		  branch REFERENCES branch(branch_id) NOT NULL,
	  )
```

- Noticed Corrections in ER Diagram
	1. Full participation of the License on "*Owns*" with candidate
	2. Full participations of the Payment on "*Paid*" with attempt

![[Pasted image 20260808104909.png|695]]

```
department(
    dept_name PRIMARY KEY,
    building,
    budget
)
course(
    course_id PRIMARY KEY,
    title,
    credits,
    dept_name NOT NULL
        REFERENCES department(dept_name)
)
student(
    ID PRIMARY KEY,
    name,
    tot_credit,
    dept_name NOT NULL
        REFERENCES department(dept_name)
)
instructor(
    ID PRIMARY KEY,
    name,
    salary,
    dept_name NOT NULL
        REFERENCES department(dept_name)
)
advisor(
    student_id PRIMARY KEY
        REFERENCES student(ID),
    instructor_id NOT NULL
        REFERENCES instructor(ID)
)
classroom(
    building,
    room_number,
    capacity,
    PRIMARY KEY (building, room_number)
)
time_slot(
    time_slot_id PRIMARY KEY,
    day,
    start_time,
    end_time
)
section(
    course_id,
    section_id,
    semester,
    year,
    building,
    room_number,
    time_slot_id,

    PRIMARY KEY (
        course_id,
        section_id,
        semester,
        year
    ),

    FOREIGN KEY (course_id)
        REFERENCES course(course_id),

    FOREIGN KEY (building, room_number)
        REFERENCES classroom(building, room_number),

    FOREIGN KEY (time_slot_id)
        REFERENCES time_slot(time_slot_id)
)
takes(
    student_id,
    course_id,
    section_id,
    semester,
    year,
    grade,

    PRIMARY KEY (
        student_id,
        course_id,
        section_id,
        semester,
        year
    ),

    FOREIGN KEY (student_id)
        REFERENCES student(ID),

    FOREIGN KEY (
        course_id,
        section_id,
        semester,
        year
    )
        REFERENCES section(
            course_id,
            section_id,
            semester,
            year
        )
)
teaches(
    instructor_id,
    course_id,
    section_id,
    semester,
    year,

    PRIMARY KEY (
        instructor_id,
        course_id,
        section_id,
        semester,
        year
    ),

    FOREIGN KEY (instructor_id)
        REFERENCES instructor(ID),

    FOREIGN KEY (
        course_id,
        section_id,
        semester,
        year
    )
        REFERENCES section(
            course_id,
            section_id,
            semester,
            year
        )
)
prereq(
    course_id,
    prereq_course_id,

    PRIMARY KEY (course_id, prereq_course_id),

    FOREIGN KEY (course_id)
        REFERENCES course(course_id),

    FOREIGN KEY (prereq_course_id)
        REFERENCES course(course_id)
)
```

## Extended ER featuers

| Feature                   | Description                                                                                                                                          | Example                                                                                                                                 |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Specialization**        | Divides an entity set into **subgroups** based on distinct characteristics. Each subgroup may have additional attributes not shared by all entities. | The entity `` `person` `` may be specialized into `` `employee` `` and `` `student` ``.                                                 |
| **Generalization**        | The reverse of specialization — combines multiple entity sets with common attributes into a **higher‑level** entity set.                             | `` `car` ``, `` `truck` ``, and `` `motorcycle` `` generalized into `` `vehicle` ``.                                                    |
| **Attribute Inheritance** | Sub‑entities **inherit attributes** from their parent entity sets.                                                                                   | `` `employee` `` inherits `` `ID` ``, `` `name` ``, `` `street` ``, `` `city` `` from `` `person` ``.                                   |
| **Aggregation**           | Treats a **relationship set** as an entity set so it can participate in other relationships.                                                         | A `` `works_on` `` relationship between `` `employee` `` and `` `project` `` can be aggregated to form a new entity `` `assignment` ``. |
##### EX: Aggregation
![[Pasted image 20260808115703.png]]
```
// Aggregated Entity -> Relationship set which gonna treat as an Entity
// Depend on the participation, this set may be redundant
// 1. Full partitcipation on eval_for -> Drop the proj_guide
// 2. Partial participation on eval_for -> Keep proj_guide (Redundant)

proj_guide(
	inst_id,
	st_id,
	proj_id
	PRIMARY KEY(
		inst_id,
		st_id,
		proj_id
	)
	FOREIGN KEY(inst_id) REFERENCES instructor(ID)
	FOREIGN KEY(st_id) REFERENCES student(ID)
	FOREIGN KEY(proj_id) REFERENCES project(proj_id)
)
eval_for(
	inst_id,
	st_id,
	proj_id,
	evaluation_id
	PRIMARY KEY(
		inst_id,
		st_id,
		proj_id,
		evaluation_id
	)
	FOREIGN KEY(inst_id) REFERENCES instructor(ID)
	FOREIGN KEY(st_id) REFERENCES student(ID)
	FOREIGN KEY(proj_id) REFERENCES project(proj_id)
	FOREIGN KEY(evaluation_id) REFERENCES evaluation(evaluation_id)
)
```
