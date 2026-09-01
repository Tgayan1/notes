![[CS3043 - Advanced SQL (1).pdf]]

Database configurations
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