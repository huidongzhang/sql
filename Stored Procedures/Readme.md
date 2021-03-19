### What are Stored Procedures?
**Procedures** are a simple way to save queries for later access. 

A **stored procedure** can be defined as a set of SQL statements stored in the MySQL server. 
- For example a SELECT statement that returns all the male actors can be turned into a stored procedure. 

A stored procedure can be invoked using `CALL` statement which executes the query and returns the result.
- When the stored procedures are **called**, MySQL **compiles** the code of the procedure and stores it in the cache. 
- Then the code is executed. 
- Any subsequent calls to the same stored procedure result in execution of the **previously compiled code**.

Just like **functions**, we can pass parameters to stored procedures. Stored procedures have conditional statements and loops, and can call other stored procedures.

There are several **advantages** of using stored procedures. 
1. They reduce the traffic between applications and server because instead of sending queries, only the **name** of the stored procedure is sent to the server. 
2. Since a stored procedure can be called by other procedures, it **reduces the duplication of code**. 
3. **Performance gains** are also achieved by using store procedures because the code can be pre-compiled instead of parsing the query every time. 
4. Stored procedures can be used for **security** by giving users access to certain procedures without giving access to the tables.

Stored procedures have certain **limitations**. 
1. Stored procedures are **difficult to debug** as they get automatically executed and it is not possible to apply checkpoints in the code. 
2. Stored procedures incur a **resource overhead** and may result in **overuse of memory and CPU**. There is no way to rollback a change to a stored procedure, once it is made.
