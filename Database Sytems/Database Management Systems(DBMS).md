Systems which are equipped with *collection of data - Database* and *set of programs - tools to access data* 
## Problems occurred with file system based data storing technics
- Data Redundancy and Inconsistency
- Data Isolation (several types, hard to manage)
- Data Integrity issues (couldn't maintain constrains properly)
- Data accessing issues (no direct mechanisms, need to develop for each operation)
- Security Issues (unauthorized access for data)
- Concurrency problems 
## Database Systems

### Data View
#### a. Provides Data Abstraction

![[Pasted image 20260710130944.png|341]]
1. View Level
	- Simplify the complexity of data organization, and provide data which only accessible to the *users*.
	- Highest abstraction
2. Logical Level
	- Describes what are the relationships among the data and what is the structure that data is organized logically
	- *Database Administrators* use this layer mainly.
	- Medium abstraction
3. Physical Level
	- Where the the data really lives
	- In-detail description about complex-structured organization
	- Low abstraction

![[Gemini_Generated_Image_x37ym2x37ym2x37y.png|586]]

#### b. Data Modeling
- Methods to define how the data be structured
		a. Entity Relationship Model.
		b. Relational Model.
		c. Semi-Structured Data Model.
		d. Object based Model.
- ![[ChatGPT Image Jul 10, 2026, 01_37_51 PM.png]]

#### c. Database Schemas and instances

**Schema** -> The Design/ Definition/ Structure type definition of the database
**Instance** -> Collection of information stored in the database at a particular time

![[ChatGPT Image Jul 10, 2026, 03_00_27 PM.png|456]]


Note:
- Each abstract level have separate schemas to define the each layers behavior.

| Schema                          | Description                                              |
| ------------------------------- | -------------------------------------------------------- |
| Physical Schema                 | Describes how the data physically stored                 |
| **Logical Schema**              | **Describes what are the relationship among those data** |
| View Level Schema (*subschema*) | Describes how the data should be view based on access    |

#### d. Database Languages
- Two categories
	1. Data-Definition Language (DDL)
	2. Data-Manipulation Language (DML)

1. Data-Definition Language (DDL)
	- Used to specify schemas (definitions)
	- *Note* -> `Data Storage and Definition Language (DSDL)` special type of DDL used for specify storage structures and access methods for database systems which are hidden from the user.
	- DDLs implements `data constrains`.
		- Consistency Constrains (rules for attributes)
		- Integrity Constrains (primary keys, foreign keys)
		- Domain Constrains (domain restriction for attributes)
		- Authorization (insert, read, update, delete)
	- DDL compiler compiles code and store it in *Data Dictionary* Which holds **metadata** that only access by database system.

2. Data-Manipulation Language (DML)
	- Enables access to the database for users to manipulate data (Insert, Read, Update, Delete).
	- Two sub types
		- `Procedural DMLs`  -> Describes What are the data and how to get it from the database (have to manually write algorithms for management.)
		- `Declarative DMLs`  ->Describes only what are the data needed (way of managing will manage by database system itself)
	- `query` is a statement which use to retrieve data from database. `query language` is the portion of DML which used to retrieve data.

- `SQL` Language exhibits all those language aspects.

![[ChatGPT Image Jul 10, 2026, 05_55_10 PM.png|675]]

## Designing Database

![[Gemini_Generated_Image_o5lc4uo5lc4uo5lc.png|697]]

## Database Engine

| Component              | Tasks                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Additional Information                                                                                                                                                                                                                                                                                                          |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Storage Manager        | • Provides the interface between low-level stored data and application programs or queries.              <br>• Stores, retrieves, and updates data in the database.  <br>• Tests for integrity constraints and checks user access authority via the authorization and integrity manager.  <br>• Manages disk space allocation and data structures via the file manager.  <br>• Fetches data from disk to main memory and decides what to cache via the buffer manager.                                                                                        | • Crucial because databases require large storage spaces, often ranging from gigabytes to terabytes or petabytes.  <br>• Implements data files to store the database itself.  <br>• Implements a data dictionary to store metadata and schema details.  <br>• Implements indices to provide fast access to specific data items. |
| Query Processor        | • Simplifies and facilitates user access to data.  <br>• Translates nonprocedural updates and queries into an efficient sequence of physical-level operations.  <br>• Interprets DDL statements and records definitions in the data dictionary via the DDL interpreter.  <br>• Translates DML statements into an evaluation plan of low-level instructions via the DML compiler.  <br>• Performs query optimization by picking the lowest cost evaluation plan from alternatives.  <br>• Executes the low-level instructions via the query evaluation engine. | • Allows users to obtain good performance while working at the view level.  <br>• Relieves users from the burden of understanding physical-level implementation details.                                                                                                                                                        |
| Transaction Management | • Treats a sequence of database accesses as a single logical unit of work.  <br>• Ensures atomicity (all-or-none execution) and durability (persistence after failure) via the recovery manager.  <br>• Performs failure recovery by detecting system failures and restoring the database to its prior state.  <br>• Controls interactions among concurrent transactions to ensure consistency via the concurrency-control manager.                                                                                                                           | • The transaction manager consists of the concurrency-control manager and the recovery manager.  <br>• Requires that transactions do not violate any database-consistency constraints.  <br>• Applied broadly across financial applications, real-time telecommunications, product design, and administrative workflows.        |
![[Pasted image 20260808123118.png|376]]
### Database Users and Administrators Interfaces

#### Users
| Type of User                | Interface                                                  | Capabilities                                                                                     |
| --------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Naïve Users**             | Use predefined interfaces like web or mobile forms.        | Can fill forms and view reports; perform simple tasks such as class registration or data entry.  |
| **Application Programmers** | Use programming tools and languages to build applications. | Write programs that interact with the database; design forms and reports for naïve users.        |
| **Sophisticated Users**     | Use query languages (e.g., SQL) directly.                  | Formulate complex queries, analyze data, and retrieve information without writing full programs. |
| **Specialized Users**       | Use customized interfaces for specific tasks.              | Work with specialized applications like CAD systems or statistical analysis tools.               |
#### Administrators
| Role                        | Responsibilities                                                               |
| --------------------------- | ------------------------------------------------------------------------------ |
| **Design & Implementation** | Define schema, structure, and storage organization.                            |
| **Maintenance & Security**  | Ensure data integrity, manage user access, and perform backups.                |
| **Performance & Recovery**  | Optimize queries, monitor system performance, and restore data after failures. |
