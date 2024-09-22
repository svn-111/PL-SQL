# PL-SQL

## SQL Clause Order

### Overview
This document provides the typical order of clauses in a SQL `SELECT` statement, along with an example to illustrate their usage.

### Order of SQL Clauses

The standard order of clauses in a SQL `SELECT` statement is as follows:

1. **SELECT**: Specify the columns to be retrieved.
2. **FROM**: Specify the tables from which to retrieve the data.
3. **JOIN**: (If needed) Join other tables to the primary table.
4. **WHERE**: Filter records based on specified conditions.
5. **GROUP BY**: Group the result set based on one or more columns.
6. **HAVING**: Filter groups based on aggregate conditions.
7. **ORDER BY**: Sort the result set based on one or more columns.
8. **LIMIT / OFFSET**: (If applicable) Limit the number of rows returned.

## Example SQL Query

Here’s a complete example that incorporates all these clauses:

```sql
SELECT 
    e.employee_id, 
    e.first_name, 
    SUM(s.amount) AS total_sales
FROM 
    employees e
JOIN 
    sales s ON e.employee_id = s.employee_id
WHERE 
    e.department_id = 10
GROUP BY 
    e.employee_id, e.first_name
HAVING 
    SUM(s.amount) > 5000
ORDER BY 
    total_sales DESC
LIMIT 10;
```

## What is PL/SQL?
PL/SQL is a procedural language designed to enhance SQL by providing features like loops, conditions, and exception handling. It's commonly used in Oracle databases for creating stored procedures, functions, triggers, and more.

## PL/SQL Block Structure
A PL/SQL block consists of three main sections:

1. **Declaration Section (optional)**: Define variables, constants, and cursors.
2. **Execution Section (mandatory)**: The code that executes SQL queries and performs operations.
3. **Exception Handling Section (optional)**: Handles errors and exceptions.

### Example of a Basic PL/SQL Block
```sql
DECLARE
    -- Declaration section
    v_message VARCHAR2(50);
BEGIN
    -- Execution section
    v_message := 'Hello, PL/SQL!';
    DBMS_OUTPUT.PUT_LINE(v_message);  -- Print the message
EXCEPTION
    -- Exception handling section
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An error occurred.');
END;
```
### Key Points
- **DECLARE**: Used to declare variables.
- **BEGIN**: Marks the start of the execution section.
- **EXCEPTION**: Handles errors that might occur in the execution section.
- **END**: Used to terminate the block.
## Variables and Data Types
You can declare variables in the `DECLARE` section. Some common PL/SQL data types include:

- **NUMBER**: For numeric values.
- **VARCHAR2(size)**: For string data.
- **DATE**: For date and time values.

### Example of Variable Declaration
```sql
DECLARE
    v_name VARCHAR2(30);
    v_age NUMBER(2);
BEGIN
    v_name := 'John Doe';
    v_age := 25;
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_name || ', Age: ' || v_age);
END;
```
In this example, we declare two variables (v_name and v_age), assign values to them, and print them.

## Control Structures
PL/SQL includes control structures like `IF-THEN-ELSE` and `LOOP` to control the flow of execution.

### IF-THEN-ELSE Example
```sql
DECLARE
    v_salary NUMBER := 5000;
BEGIN
    IF v_salary > 4000 THEN
        DBMS_OUTPUT.PUT_LINE('High salary');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Low salary');
    END IF;
END;
```
### LOOP Example
```sql
DECLARE
    i NUMBER := 1;
BEGIN
    WHILE i <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE('Count: ' || i);
        i := i + 1;
    END LOOP;
END;
```

## Exception Handling
Errors are handled in the `EXCEPTION` section using predefined or custom exceptions.

### Example of Exception Handling
```sql
BEGIN
    -- Deliberate error (dividing by zero)
    DECLARE
        v_num NUMBER := 10;
        v_zero NUMBER := 0;
    BEGIN
        v_num := v_num / v_zero;
    EXCEPTION
        WHEN ZERO_DIVIDE THEN
            DBMS_OUTPUT.PUT_LINE('Cannot divide by zero!');
    END;
END;
```


# PL/SQL Cursors

## Cursors
Cursors allow you to fetch and process database rows one at a time. A cursor acts as a pointer to a query result set in PL/SQL.

### Why Use Cursors?
When you execute a SELECT query that returns multiple rows, you can't directly process them all in one go. A cursor allows you to loop through each row individually.

### Types of Cursors
- **Implicit Cursor**: Automatically created for single-row queries like `SELECT INTO`. You don’t have to declare or manage it.
- **Explicit Cursor**: You declare it explicitly when you want to handle multiple rows.

### Explicit Cursor Steps
1. Declare the cursor.
2. Open the cursor (to execute the query).
3. Fetch the data row by row.
4. Close the cursor.

### Example of an Implicit Cursor
```sql
DECLARE
    -- Declare variables to hold the values retrieved
    v_emp_id employees.employee_id%TYPE;
    v_emp_name employees.first_name%TYPE;
BEGIN
    -- Use an implicit cursor to select a single employee
    SELECT employee_id, first_name
    INTO v_emp_id, v_emp_name
    FROM employees
    WHERE employee_id = 1;  -- Assuming you want to fetch employee with ID 1

    -- Print the retrieved employee details
    DBMS_OUTPUT.PUT_LINE('ID: ' || v_emp_id || ', Name: ' || v_emp_name);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No employee found with the specified ID.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An error occurred: ' || SQLERRM);
END;
```
### Example of an Explicit Cursor
```sql
DECLARE
    -- Declare a cursor to select employee_id and first_name from employees table--STEP 1
    CURSOR cur_emp IS
        SELECT employee_id, first_name FROM employees WHERE department_id = 10;
    
    -- Declare variables to store each row's values
    v_emp_id employees.employee_id%TYPE;
    v_emp_name employees.first_name%TYPE;
    
BEGIN
    OPEN cur_emp;  -- Step 2: Open the cursor
    LOOP
        FETCH cur_emp INTO v_emp_id, v_emp_name;  -- Step 3: Fetch each row into variables
        EXIT WHEN cur_emp%NOTFOUND;  -- Exit loop when no more rows are found
        DBMS_OUTPUT.PUT_LINE('ID: ' || v_emp_id || ', Name: ' || v_emp_name);
    END LOOP;
    CLOSE cur_emp;  -- Step 4: Close the cursor
END;
```

## Example of using an explicit cursor

### Overview
This document provides an example of using an explicit cursor in PL/SQL to calculate employee bonuses based on their base salary.

### SQL Table Structure

Assume we have an `employees` table with the following fields:

- **employee_id**: Unique identifier for each employee.
- **first_name**: Employee's first name.
- **base_salary**: Employee's base salary.


```sql
CREATE TABLE employees (
    employee_id NUMBER PRIMARY KEY,
    first_name VARCHAR2(50),
    base_salary NUMBER
);
DECLARE
    -- Declare a cursor to select employee_id and base_salary
    CURSOR cur_emp IS
        SELECT employee_id, first_name, base_salary
        FROM employees;

    -- Declare variables to store each row's values
    v_emp_id employees.employee_id%TYPE;
    v_emp_name employees.first_name%TYPE;
    v_base_salary employees.base_salary%TYPE;
    v_bonus NUMBER;
    
BEGIN
    OPEN cur_emp;  -- Step 2: Open the cursor
    LOOP
        FETCH cur_emp INTO v_emp_id, v_emp_name, v_base_salary;  -- Step 3: Fetch each row into variables
        EXIT WHEN cur_emp%NOTFOUND;  -- Exit loop when no more rows are found
        
        -- Calculate bonus (10% of base salary)
        v_bonus := v_base_salary * 0.10;
        
        -- Print employee details and calculated bonus
        DBMS_OUTPUT.PUT_LINE('Employee: ' || v_emp_name || ', Base Salary: ' || v_base_salary || ', Bonus: ' || v_bonus);
    END LOOP;
    CLOSE cur_emp;  -- Step 4: Close the cursor
END;
```
## PL/SQL Procedures

### Overview
A Procedure is a named PL/SQL block that performs a specific task. It’s stored in the database and can be reused.

### Why Use Procedures?
Procedures allow you to encapsulate business logic, making your code modular and reusable. You can pass data into procedures (via parameters), perform operations, and call them whenever needed.

### Syntax of a Procedure
The basic syntax for creating a procedure is as follows:

```sql
CREATE OR REPLACE PROCEDURE procedure_name (
    parameter_name IN datatype
) IS
BEGIN
    -- Execution block
END;
```
### Procedure Example
```sql
CREATE OR REPLACE PROCEDURE greet_user (p_name IN VARCHAR2) IS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello, ' || p_name || '!');
END;
BEGIN
    greet_user('Alice');  -- Outputs: Hello, Alice!
END;
```

## PL/SQL Functions

### Overview
A Function in PL/SQL is similar to a procedure but is designed to return a value. Functions can be used in SQL queries, expressions, and other PL/SQL code.

### Why Use Functions?
Functions are ideal when you need to perform a calculation or transformation and return a single value. They help encapsulate logic that can be reused throughout your code.

### Syntax of a Function
The basic syntax for creating a function is as follows:

```sql
CREATE OR REPLACE FUNCTION function_name (
    parameter_name IN datatype
) RETURN return_datatype IS
BEGIN
    -- Logic to calculate the value
    RETURN value;
END;
```
### Example of a Function
```sql
CREATE OR REPLACE FUNCTION square_number(p_number IN NUMBER) RETURN NUMBER IS
BEGIN
    RETURN p_number * p_number;
END;
DECLARE
    v_result NUMBER;
BEGIN
    v_result := square_number(4);  -- Calls the function
    DBMS_OUTPUT.PUT_LINE('The square of 4 is: ' || v_result);  -- Outputs: The square of 4 is: 16
END;
```

## PL/SQL Triggers

### Overview
A Trigger is a special PL/SQL block that automatically executes in response to certain events occurring in the database, such as an INSERT, UPDATE, or DELETE operation on a table.

### Why Use Triggers?
Triggers are useful for enforcing business rules automatically. They can perform various tasks such as:
- Logging changes made to data.
- Validating data before it is committed.
- Updating related tables or fields based on certain conditions.

### Syntax of a Trigger
The basic syntax for creating a trigger is as follows:

```sql
CREATE OR REPLACE TRIGGER trigger_name
BEFORE | AFTER INSERT | UPDATE | DELETE ON table_name
FOR EACH ROW
BEGIN
    -- Logic to execute when the trigger fires
END;
```
### Trigger Example
Here’s an example of a trigger that enforces a salary rule before inserting an employee record:

```sql
CREATE OR REPLACE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF :NEW.salary < 1000 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Salary must be at least 1000.');
    END IF;
END;
```
#### Explanation
In this example:

- The trigger before_employee_insert fires before an employee record is inserted into the employees table.
- If the new employee's salary is less than 1000, the trigger raises an error with the message: "Salary must be at least 1000."
#### Key Points
- :NEW refers to the new values being inserted or updated in the table.
- Triggers can be classified as BEFORE (executed before the action) or AFTER (executed after the action) triggers.

## PL/SQL Dynamic SQL

### Overview
Dynamic SQL allows you to build and execute SQL queries dynamically at runtime. This capability is particularly useful when the structure of the SQL statement is not known until the program is executing, such as when table names or column names are determined during runtime.

### Example of Dynamic SQL
The following example demonstrates how to use Dynamic SQL to delete a record from the `employees` table based on a specified `employee_id`.

```sql
DECLARE
    v_sql VARCHAR2(200);
BEGIN
    v_sql := 'DELETE FROM employees WHERE employee_id = 100';
    EXECUTE IMMEDIATE v_sql;  -- Execute the dynamic SQL
END;
```
### Explanation
- In this example, we declare a variable v_sql of type VARCHAR2 to hold our SQL statement.
- The DELETE statement is constructed as a string and assigned to v_sql.
- The EXECUTE IMMEDIATE command is used to execute the SQL statement stored in the variable.
### Key Points
- Dynamic SQL is useful for executing queries where the structure is not predetermined.
- Always validate input to avoid SQL injection attacks when using dynamic SQL.
- EXECUTE IMMEDIATE is used for executing a single SQL statement dynamically.
