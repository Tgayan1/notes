![[CS3043 - Advanced SQL (1).pdf]]

Database configuration
- Database URL
- Username
- Password

SQL Exceptions. 
- Database Is busy -> Retrying in degradation manner (10s,30s etc...)

Statement Object
- Carrying out the Query to Database And Retrieve the data or throws error.

Name vs Position foe getting result attributes 
- Name is preferred. If we change the database structure, and used position, we have to edit whole code.

Prepared statements
- Prevent SQL Injection
- Prepared statements will caches by the database.

Database metadata
- Describes about the database
	- Row counts
	- Column count
	- Tables

Embedded SQL
- Embedded SQL is best when *performance, reliability, and tight integration with procedural code matter* especially in **banking, insurance, healthcare, and telecom batch systems**.

 **Function** vs **Procedure** in SQL:

| Feature                 | **Function**                                                                               | **Procedure**                                                                        |
| ----------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| **Purpose**             | Used mainly for **calculations** and returning a **single value** (scalar) or a **table**. | Used to perform **a set of actions** (insert, update, delete, select, etc.).         |
| **Return Value**        | Must **return a value** (scalar or table).                                                 | May **return zero, one, or multiple values** via `OUT` parameters or result sets.    |
| **Usage in SQL**        | Can be used **inside SQL statements** (e.g., `SELECT`, `WHERE`).                           | Cannot be directly used inside SQL statements; must be called with `EXEC` or `CALL`. |
| **Transaction Control** | Cannot use `COMMIT` or `ROLLBACK` inside.                                                  | Can use `COMMIT` and `ROLLBACK`.                                                     |
| **Parameters**          | Only **input parameters** allowed.                                                         | Can have **input**, **output**, and **input-output** parameters.                     |
| **Error Handling**      | Limited error handling (depends on DBMS).                                                  | Full error handling with `TRY...CATCH` (in SQL Server) or `EXCEPTION` (in PL/SQL).   |
| **Performance**         | Generally faster for computations.                                                         | More flexible for complex business logic.                                            |
| **Compilation**         | Compiled and stored in the database.                                                       | Compiled and stored in the database.                                                 |

---

### **Example in SQL Server**

**Function:**
```sql
# Function:
CREATE FUNCTION dbo.GetTotalPrice(@qty INT, @price DECIMAL(10,2)) RETURNS DECIMAL(10,2) AS BEGIN     RETURN @qty * @price; END;

# Usage:
SELECT dbo.GetTotalPrice(5, 20.00) AS TotalPrice;

#########################################################################

# Procedure:
CREATE PROCEDURE dbo.UpdateStock     @productId INT,     @qtySold INT AS BEGIN     UPDATE Products     SET Stock = Stock - @qtySold     WHERE ProductID = @productId; END;

# Usage:
EXEC dbo.UpdateStock @productId = 101, @qtySold = 3;

```

✅ **Rule of Thumb**:

- Use a **Function** when you need a **value** inside a query.
- Use a **Procedure** when you need to **perform actions** or **modify data**.
