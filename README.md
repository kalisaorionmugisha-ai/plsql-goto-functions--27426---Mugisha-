# PL/SQL GOTO Statements and Functions


## 1. Repository Structure

```
plsql-goto-functions-<studentID>-<firstname>/
├── README.md
├── .gitignore
├── 00_setup/
│   └── create_tables.sql
├── 01_goto/
│   ├── A1_number_classifier.sql
│   ├── A2_salary_review.sql
│   ├── A3_illegal_goto.sql
│   └── A4_rewrite_no_goto.sql
├── 02_functions/
│   ├── B1_fn_annual_salary.sql
│   ├── B2_fn_years_of_service.sql
│   ├── B3_fn_calculate_tax.sql
│   ├── B4_fn_dept_name.sql
│   └── C1_fn_validate_payroll.sql
├── 03_tests/
│   ├── B5_functions_in_select.sql
│   ├── test_functions.sql
│   └── test_validate_payroll.sql
├── screenshots/
│   ├── A1_output.png
│   ├── A2_output.png
│   ├── A3_error_and_fix.png
│   ├── A4_output.png
│   ├── B5_select_output.png
│   └── C1_output.png
└── docs/
    └── REFLECTION.md
```

## 2. How to Run

1. Run `00_setup/create_tables.sql` (Run Script, F5).
2. Run the functions in `02_functions/` in the order B1, B2, B3, B4, C1.
3. Run the programs in `01_goto/` (A1, A2, A3, A4).
4. Run the test files in `03_tests/`.
5. Verify the results and the screenshots.

In SQL Developer, enable **View > Dbms Output** so `DBMS_OUTPUT` text is displayed.

---

## 3. Setup: Tables and Data

File: `00_setup/create_tables.sql`

Two tables are used. `departments` holds department names and `employees` holds salary, hire date and department. The data deliberately includes problem rows to test the validator: employee 5 has no salary, employee 6 has a future hire date, employee 7 has no department.

```sql
BEGIN
  EXECUTE IMMEDIATE 'DROP TABLE employees CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/
BEGIN
  EXECUTE IMMEDIATE 'DROP TABLE departments CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

CREATE TABLE departments (
  dept_id    NUMBER(4)     PRIMARY KEY,
  dept_name  VARCHAR2(50)  NOT NULL
);

CREATE TABLE employees (
  emp_id          NUMBER(6)     PRIMARY KEY,
  first_name      VARCHAR2(30)  NOT NULL,
  last_name       VARCHAR2(30)  NOT NULL,
  monthly_salary  NUMBER(10,2),
  hire_date       DATE,
  dept_id         NUMBER(4) REFERENCES departments(dept_id),
  status          VARCHAR2(10)  DEFAULT 'ACTIVE'
);

INSERT INTO departments VALUES (10, 'Finance');
INSERT INTO departments VALUES (20, 'Human Resources');
INSERT INTO departments VALUES (30, 'IT');
INSERT INTO departments VALUES (40, 'Marketing');

INSERT INTO employees VALUES (1, 'Alice',  'Uwase',    450000, DATE '2015-03-01', 10, 'ACTIVE');
INSERT INTO employees VALUES (2, 'Bob',    'Mugisha',  120000, DATE '2021-07-15', 20, 'ACTIVE');
INSERT INTO employees VALUES (3, 'Carine', 'Ineza',    800000, DATE '2010-01-10', 30, 'ACTIVE');
INSERT INTO employees VALUES (4, 'David',  'Habimana', 60000,  DATE '2024-02-20', 30, 'ACTIVE');
INSERT INTO employees VALUES (5, 'Eric',   'Nkusi',    NULL,   DATE '2019-09-09', 40, 'ACTIVE');
INSERT INTO employees VALUES (6, 'Fiona',  'Keza',     300000, DATE '2030-01-01', 10, 'ACTIVE');
INSERT INTO employees VALUES (7, 'Grace',  'Mutoni',   250000, DATE '2018-05-05', NULL, 'ACTIVE');

COMMIT;

SELECT * FROM departments;
SELECT * FROM employees;
```

---

## 4. Part A - GOTO

### A1 - Number Classifier

File: `01_goto/A1_number_classifier.sql`

Classifies a number as negative, zero, positive even or positive odd using `GOTO` and labels.

```sql
SET SERVEROUTPUT ON

DECLARE
  v_number NUMBER := 7;   -- test with: -5, 0, 7, 10
BEGIN
  DBMS_OUTPUT.PUT_LINE('Number tested: ' || v_number);

  IF v_number < 0 THEN
    GOTO negative_number;
  ELSIF v_number = 0 THEN
    GOTO zero_number;
  ELSIF MOD(v_number, 2) = 0 THEN
    GOTO even_number;
  ELSE
    GOTO odd_number;
  END IF;

  <<negative_number>>
  DBMS_OUTPUT.PUT_LINE('Result: NEGATIVE number');
  GOTO end_program;

  <<zero_number>>
  DBMS_OUTPUT.PUT_LINE('Result: ZERO');
  GOTO end_program;

  <<even_number>>
  DBMS_OUTPUT.PUT_LINE('Result: POSITIVE EVEN number');
  GOTO end_program;

  <<odd_number>>
  DBMS_OUTPUT.PUT_LINE('Result: POSITIVE ODD number');

  <<end_program>>
  DBMS_OUTPUT.PUT_LINE('Classification finished.');
END;
/
```

**Output (v_number = 7):**

```
Number tested: 7
Result: POSITIVE ODD number
Classification finished.
```

A1 :<img width="1135" height="1632" alt="a1" src="https://github.com/user-attachments/assets/fe6edaac-c2ef-4b7c-8462-f95ddaff95cd" />


**Explanation:** Each branch of the `IF` jumps to a label. Every outcome ends by jumping to `end_program` so the other outcomes are skipped.

### A2 - Salary Review

File: `01_goto/A2_salary_review.sql`

Reviews each employee's monthly salary. `GOTO` sends each employee to the right outcome and skips employees with no salary.

```sql
SET SERVEROUTPUT ON

DECLARE
  CURSOR c_emp IS
    SELECT emp_id, first_name, last_name, monthly_salary
    FROM   employees
    ORDER  BY emp_id;
  v_msg VARCHAR2(100);
BEGIN
  DBMS_OUTPUT.PUT_LINE('===== SALARY REVIEW =====');

  FOR r IN c_emp LOOP

    IF r.monthly_salary IS NULL THEN
      GOTO skip_employee;
    END IF;

    IF r.monthly_salary < 100000 THEN
      GOTO low_salary;
    ELSIF r.monthly_salary < 500000 THEN
      GOTO medium_salary;
    ELSE
      GOTO high_salary;
    END IF;

    <<low_salary>>
    v_msg := 'LOW - eligible for 10% raise';
    GOTO print_result;

    <<medium_salary>>
    v_msg := 'MEDIUM - eligible for 5% raise';
    GOTO print_result;

    <<high_salary>>
    v_msg := 'HIGH - no raise this cycle';
    GOTO print_result;

    <<skip_employee>>
    DBMS_OUTPUT.PUT_LINE(r.emp_id || ' ' || r.first_name || ' ' || r.last_name
                         || ' : SKIPPED (no salary on record)');
    GOTO next_employee;

    <<print_result>>
    DBMS_OUTPUT.PUT_LINE(r.emp_id || ' ' || r.first_name || ' ' || r.last_name
                         || ' (' || r.monthly_salary || ') : ' || v_msg);

    <<next_employee>>
    NULL;
  END LOOP;

  DBMS_OUTPUT.PUT_LINE('===== REVIEW COMPLETE =====');
END;
/
```

**Output:**

```
===== SALARY REVIEW =====
1 Alice Uwase (450000) : MEDIUM - eligible for 5% raise
2 Bob Mugisha (120000) : MEDIUM - eligible for 5% raise
3 Carine Ineza (800000) : HIGH - no raise this cycle
4 David Habimana (60000) : LOW - eligible for 10% raise
5 Eric Nkusi : SKIPPED (no salary on record)
6 Fiona Keza (300000) : MEDIUM - eligible for 5% raise
7 Grace Mutoni (250000) : MEDIUM - eligible for 5% raise
===== REVIEW COMPLETE =====
```

A2 output: <img width="1476" height="1728" alt="A2" src="https://github.com/user-attachments/assets/6cea8778-05b7-4891-b790-6f23d4ef39a1" />


**Explanation:** The `GOTO` statements jump out of `IF` blocks to labels in the enclosing loop body, which is legal. `NULL;` follows the last label because a label must be followed by an executable statement.

### A3 - Illegal GOTO and Fix

File: `01_goto/A3_illegal_goto.sql`

PL/SQL does not allow a `GOTO` to jump **into** an `IF`, loop or nested block.

**Step 1 - Illegal version:**

```sql
SET SERVEROUTPUT ON

DECLARE
  v_salary NUMBER := 200000;
BEGIN
  GOTO inside_if;                      -- ILLEGAL: label is inside an IF block

  IF v_salary > 100000 THEN
    <<inside_if>>
    DBMS_OUTPUT.PUT_LINE('Inside the IF block');
  END IF;
END;
/
```

**Error:**

```
PLS-00375: illegal GOTO statement; this GOTO cannot branch to label 'INSIDE_IF'
```

**Step 2 - Fixed version:**

```sql
DECLARE
  v_salary NUMBER := 200000;
BEGIN
  IF v_salary > 100000 THEN
    GOTO high_salary;                  -- legal: jumps OUT of the IF to an outer label
  END IF;

  DBMS_OUTPUT.PUT_LINE('Salary is not high.');
  GOTO finish;

  <<high_salary>>
  DBMS_OUTPUT.PUT_LINE('Salary is high (fixed version works).');

  <<finish>>
  DBMS_OUTPUT.PUT_LINE('Done.');
END;
/
```

**Output:**

```
Salary is high (fixed version works).
Done.
```

A3 output: <img width="1472" height="1718" alt="a3" src="https://github.com/user-attachments/assets/4d42120f-3aee-40fc-a690-f4d3fbb46531" />


**Explanation:** The fix moves the label to the same level as the `GOTO` statement, so the program jumps out of the `IF` (allowed) instead of into it (not allowed).

### A4 - Rewrite Without GOTO

File: `01_goto/A4_rewrite_no_goto.sql`

A1 and A2 rewritten with `IF/ELSIF/ELSE` and `CONTINUE`.

```sql
SET SERVEROUTPUT ON

-- A1 rewritten
DECLARE
  v_number NUMBER := 7;
BEGIN
  DBMS_OUTPUT.PUT_LINE('Number tested: ' || v_number);

  IF v_number < 0 THEN
    DBMS_OUTPUT.PUT_LINE('Result: NEGATIVE number');
  ELSIF v_number = 0 THEN
    DBMS_OUTPUT.PUT_LINE('Result: ZERO');
  ELSIF MOD(v_number, 2) = 0 THEN
    DBMS_OUTPUT.PUT_LINE('Result: POSITIVE EVEN number');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Result: POSITIVE ODD number');
  END IF;

  DBMS_OUTPUT.PUT_LINE('Classification finished.');
END;
/

-- A2 rewritten
DECLARE
  v_msg VARCHAR2(100);
BEGIN
  DBMS_OUTPUT.PUT_LINE('===== SALARY REVIEW (no GOTO) =====');

  FOR r IN (SELECT emp_id, first_name, last_name, monthly_salary
            FROM employees ORDER BY emp_id) LOOP

    IF r.monthly_salary IS NULL THEN
      DBMS_OUTPUT.PUT_LINE(r.emp_id || ' ' || r.first_name || ' ' || r.last_name
                           || ' : SKIPPED (no salary on record)');
      CONTINUE;
    END IF;

    IF r.monthly_salary < 100000 THEN
      v_msg := 'LOW - eligible for 10% raise';
    ELSIF r.monthly_salary < 500000 THEN
      v_msg := 'MEDIUM - eligible for 5% raise';
    ELSE
      v_msg := 'HIGH - no raise this cycle';
    END IF;

    DBMS_OUTPUT.PUT_LINE(r.emp_id || ' ' || r.first_name || ' ' || r.last_name
                         || ' (' || r.monthly_salary || ') : ' || v_msg);
  END LOOP;

  DBMS_OUTPUT.PUT_LINE('===== REVIEW COMPLETE =====');
END;
/
```

**Output:** the same results as A1 and A2.

```
Number tested: 7
Result: POSITIVE ODD number
Classification finished.

===== SALARY REVIEW (no GOTO) =====
1 Alice Uwase (450000) : MEDIUM - eligible for 5% raise
2 Bob Mugisha (120000) : MEDIUM - eligible for 5% raise
3 Carine Ineza (800000) : HIGH - no raise this cycle
4 David Habimana (60000) : LOW - eligible for 10% raise
5 Eric Nkusi : SKIPPED (no salary on record)
6 Fiona Keza (300000) : MEDIUM - eligible for 5% raise
7 Grace Mutoni (250000) : MEDIUM - eligible for 5% raise
===== REVIEW COMPLETE =====
```

A4 output: <img width="1406" height="1640" alt="a4" src="https://github.com/user-attachments/assets/7c87e515-9294-4179-a8d1-00aab2deab7b" />


**Explanation:** The structured version gives identical output but reads top to bottom. `CONTINUE` replaces the `GOTO` that skipped employees.

---

## 5. Part B - Functions

### B1 - fn_annual_salary

File: `02_functions/B1_fn_annual_salary.sql`

Returns monthly salary x 12. Returns NULL if the employee does not exist or has no salary.

```sql
CREATE OR REPLACE FUNCTION fn_annual_salary (
  p_emp_id IN employees.emp_id%TYPE
) RETURN NUMBER
IS
  v_monthly employees.monthly_salary%TYPE;
BEGIN
  SELECT monthly_salary
  INTO   v_monthly
  FROM   employees
  WHERE  emp_id = p_emp_id;

  IF v_monthly IS NULL THEN
    RETURN NULL;
  END IF;

  RETURN v_monthly * 12;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RETURN NULL;
  WHEN OTHERS THEN
    RETURN NULL;
END fn_annual_salary;
/
```

### B2 - fn_years_of_service

File: `02_functions/B2_fn_years_of_service.sql`

Returns complete years since the hire date. Returns -1 for a future hire date and NULL if the employee is not found.

```sql
CREATE OR REPLACE FUNCTION fn_years_of_service (
  p_emp_id IN employees.emp_id%TYPE
) RETURN NUMBER
IS
  v_hire employees.hire_date%TYPE;
BEGIN
  SELECT hire_date
  INTO   v_hire
  FROM   employees
  WHERE  emp_id = p_emp_id;

  IF v_hire IS NULL THEN
    RETURN NULL;
  ELSIF v_hire > SYSDATE THEN
    RETURN -1;
  END IF;

  RETURN TRUNC(MONTHS_BETWEEN(SYSDATE, v_hire) / 12);
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RETURN NULL;
  WHEN OTHERS THEN
    RETURN NULL;
END fn_years_of_service;
/
```

### B3 - fn_calculate_tax

File: `02_functions/B3_fn_calculate_tax.sql`

Progressive tax on a monthly salary. Raises error -20001 for NULL or negative input.

| Monthly salary | Tax |
|---|---|
| 0 - 60,000 | 0% |
| 60,001 - 100,000 | 20% of the amount above 60,000 |
| 100,001 - 200,000 | 8,000 + 30% of the amount above 100,000 |
| Above 200,000 | 38,000 + 35% of the amount above 200,000 |

```sql
CREATE OR REPLACE FUNCTION fn_calculate_tax (
  p_salary IN NUMBER
) RETURN NUMBER
IS
  v_tax NUMBER;
BEGIN
  IF p_salary IS NULL OR p_salary < 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'Invalid salary: must be a non-negative number.');
  END IF;

  IF p_salary <= 60000 THEN
    v_tax := 0;
  ELSIF p_salary <= 100000 THEN
    v_tax := (p_salary - 60000) * 0.20;
  ELSIF p_salary <= 200000 THEN
    v_tax := 8000 + (p_salary - 100000) * 0.30;
  ELSE
    v_tax := 38000 + (p_salary - 200000) * 0.35;
  END IF;

  RETURN ROUND(v_tax, 2);
END fn_calculate_tax;
/
```

### B4 - fn_dept_name

File: `02_functions/B4_fn_dept_name.sql`

Returns the department name, or `UNKNOWN` if the id is NULL or not found.

```sql
CREATE OR REPLACE FUNCTION fn_dept_name (
  p_dept_id IN departments.dept_id%TYPE
) RETURN VARCHAR2
IS
  v_name departments.dept_name%TYPE;
BEGIN
  IF p_dept_id IS NULL THEN
    RETURN 'UNKNOWN';
  END IF;

  SELECT dept_name
  INTO   v_name
  FROM   departments
  WHERE  dept_id = p_dept_id;

  RETURN v_name;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RETURN 'UNKNOWN';
  WHEN OTHERS THEN
    RETURN 'UNKNOWN';
END fn_dept_name;
/
```

### Function Tests

File: `03_tests/test_functions.sql`

```sql
SET SERVEROUTPUT ON

DECLARE
  PROCEDURE show (p_label VARCHAR2, p_value VARCHAR2) IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE(RPAD(p_label, 45) || ' => ' || NVL(p_value, 'NULL'));
  END;
BEGIN
  DBMS_OUTPUT.PUT_LINE('===== B1 fn_annual_salary =====');
  show('Emp 1 (450000 x 12 = 5,400,000)', fn_annual_salary(1));
  show('Emp 5 (salary NULL)',            fn_annual_salary(5));
  show('Emp 999 (does not exist)',       fn_annual_salary(999));

  DBMS_OUTPUT.PUT_LINE('===== B2 fn_years_of_service =====');
  show('Emp 3 (hired 2010-01-10)',       fn_years_of_service(3));
  show('Emp 4 (hired 2024-02-20)',       fn_years_of_service(4));
  show('Emp 6 (future hire date -> -1)', fn_years_of_service(6));
  show('Emp 999 (does not exist)',       fn_years_of_service(999));

  DBMS_OUTPUT.PUT_LINE('===== B3 fn_calculate_tax =====');
  show('Salary 50,000  (expect 0)',      fn_calculate_tax(50000));
  show('Salary 80,000  (expect 4,000)',  fn_calculate_tax(80000));
  show('Salary 150,000 (expect 23,000)', fn_calculate_tax(150000));
  show('Salary 450,000 (expect 125,500)',fn_calculate_tax(450000));
  BEGIN
    show('Salary -1 (expect error)',     fn_calculate_tax(-1));
  EXCEPTION WHEN OTHERS THEN
    show('Salary -1 error caught',       SQLERRM);
  END;
  BEGIN
    show('Salary NULL (expect error)',   fn_calculate_tax(NULL));
  EXCEPTION WHEN OTHERS THEN
    show('Salary NULL error caught',     SQLERRM);
  END;

  DBMS_OUTPUT.PUT_LINE('===== B4 fn_dept_name =====');
  show('Dept 10',                        fn_dept_name(10));
  show('Dept 30',                        fn_dept_name(30));
  show('Dept 99 (not found)',            fn_dept_name(99));
  show('Dept NULL',                      fn_dept_name(NULL));
END;
/
```

**Results:**

| Test | Result |
|---|---|
| `fn_annual_salary(1)` | 5400000 |
| `fn_annual_salary(5)` (null salary) | NULL |
| `fn_annual_salary(999)` | NULL |
| `fn_years_of_service(3)` | 16 |
| `fn_years_of_service(4)` | 2 |
| `fn_years_of_service(6)` (future date) | -1 |
| `fn_calculate_tax(50000)` | 0 |
| `fn_calculate_tax(80000)` | 4000 |
| `fn_calculate_tax(150000)` | 23000 |
| `fn_calculate_tax(450000)` | 125500 |
| `fn_calculate_tax(-1)` / `(NULL)` | ORA-20001: Invalid salary: must be a non-negative number. |
| `fn_dept_name(10)` / `(30)` | Finance / IT |
| `fn_dept_name(99)` / `(NULL)` | UNKNOWN |

### B5 - Functions in SQL

File: `03_tests/B5_functions_in_select.sql`

```sql
SELECT e.emp_id,
       e.first_name || ' ' || e.last_name         AS employee,
       e.monthly_salary,
       fn_annual_salary(e.emp_id)                 AS annual_salary,
       fn_years_of_service(e.emp_id)              AS years_of_service,
       fn_calculate_tax(NVL(e.monthly_salary, 0)) AS monthly_tax,
       fn_dept_name(e.dept_id)                    AS department
FROM   employees e
ORDER  BY e.emp_id;

-- Functions in WHERE and ORDER BY
SELECT emp_id, first_name, fn_annual_salary(emp_id) AS annual_salary
FROM   employees
WHERE  fn_annual_salary(emp_id) > 3000000
ORDER  BY fn_annual_salary(emp_id) DESC;

-- Functions with aggregation
SELECT fn_dept_name(dept_id) AS department,
       COUNT(*)              AS headcount,
       SUM(fn_annual_salary(emp_id)) AS total_annual_payroll
FROM   employees
GROUP  BY fn_dept_name(dept_id)
ORDER  BY department;
```

**Output of the first query:**

| EMP_ID | EMPLOYEE | MONTHLY_SALARY | ANNUAL_SALARY | YEARS_OF_SERVICE | MONTHLY_TAX | DEPARTMENT |
|---|---|---|---|---|---|---|
| 1 | Alice Uwase | 450000 | 5400000 | 11 | 125500 | Finance |
| 2 | Bob Mugisha | 120000 | 1440000 | 5 | 14000 | Human Resources |
| 3 | Carine Ineza | 800000 | 9600000 | 16 | 248000 | IT |
| 4 | David Habimana | 60000 | 720000 | 2 | 0 | IT |
| 5 | Eric Nkusi | (null) | (null) | 7 | 0 | Marketing |
| 6 | Fiona Keza | 300000 | 3600000 | -1 | 73000 | Finance |
| 7 | Grace Mutoni | 250000 | 3000000 | 8 | 55500 | UNKNOWN |

B5 output: <img width="2485" height="1758" alt="b5" src="https://github.com/user-attachments/assets/72c3ecf4-19ff-4d63-8abb-16a2d56c1865" />


**Explanation:** The functions are called per row in the select list, in `WHERE`, in `ORDER BY` and inside `GROUP BY`. `NVL` is used for the tax column because `fn_calculate_tax` rejects NULL.

---

## 6. Part C - Combined Task

### C1 - Payroll Validator

File: `02_functions/C1_fn_validate_payroll.sql`

Combines B1 to B4. It checks that the employee exists, is active, has a positive salary, has a valid department, has a valid hire date, and that tax is lower than salary. It returns `VALID (...)` or `INVALID: <reason>`. A single error-exit label uses `GOTO`, which is a legitimate use.

```sql
CREATE OR REPLACE FUNCTION fn_validate_payroll (
  p_emp_id IN employees.emp_id%TYPE
) RETURN VARCHAR2
IS
  v_count   NUMBER;
  v_salary  employees.monthly_salary%TYPE;
  v_dept_id employees.dept_id%TYPE;
  v_status  employees.status%TYPE;
  v_years   NUMBER;
  v_tax     NUMBER;
  v_error   VARCHAR2(200) := NULL;
BEGIN
  -- 1. Employee must exist
  SELECT COUNT(*) INTO v_count FROM employees WHERE emp_id = p_emp_id;
  IF v_count = 0 THEN
    v_error := 'Employee ' || p_emp_id || ' not found';
    GOTO report_error;
  END IF;

  SELECT monthly_salary, dept_id, status
  INTO   v_salary, v_dept_id, v_status
  FROM   employees
  WHERE  emp_id = p_emp_id;

  -- 2. Must be active
  IF v_status <> 'ACTIVE' THEN
    v_error := 'Employee is not ACTIVE (status=' || v_status || ')';
    GOTO report_error;
  END IF;

  -- 3. Salary must exist and be positive
  IF v_salary IS NULL OR v_salary <= 0 THEN
    v_error := 'Missing or non-positive salary';
    GOTO report_error;
  END IF;

  -- 4. Must belong to a valid department
  IF fn_dept_name(v_dept_id) = 'UNKNOWN' THEN
    v_error := 'No valid department assigned';
    GOTO report_error;
  END IF;

  -- 5. Hire date must be valid (not in the future)
  v_years := fn_years_of_service(p_emp_id);
  IF v_years IS NULL OR v_years < 0 THEN
    v_error := 'Invalid hire date';
    GOTO report_error;
  END IF;

  -- 6. Tax must be calculable and less than salary
  v_tax := fn_calculate_tax(v_salary);
  IF v_tax >= v_salary THEN
    v_error := 'Tax (' || v_tax || ') is not less than salary';
    GOTO report_error;
  END IF;

  RETURN 'VALID (annual=' || fn_annual_salary(p_emp_id)
         || ', tax/month=' || v_tax
         || ', years=' || v_years
         || ', dept=' || fn_dept_name(v_dept_id) || ')';

  <<report_error>>
  RETURN 'INVALID: ' || v_error;

EXCEPTION
  WHEN OTHERS THEN
    RETURN 'INVALID: unexpected error - ' || SQLERRM;
END fn_validate_payroll;
/
```

### C1 Test

File: `03_tests/test_validate_payroll.sql`

```sql
SET SERVEROUTPUT ON

BEGIN
  FOR r IN (SELECT emp_id FROM employees ORDER BY emp_id) LOOP
    DBMS_OUTPUT.PUT_LINE('Employee ' || r.emp_id || ': ' || fn_validate_payroll(r.emp_id));
  END LOOP;
  DBMS_OUTPUT.PUT_LINE('Employee 999: ' || fn_validate_payroll(999));
END;
/

SELECT emp_id, fn_validate_payroll(emp_id) AS payroll_check
FROM   employees
ORDER  BY emp_id;
```

**Output:**

```
Employee 1: VALID (annual=5400000, tax/month=125500, years=11, dept=Finance)
Employee 2: VALID (annual=1440000, tax/month=14000, years=5, dept=Human Resources)
Employee 3: VALID (annual=9600000, tax/month=248000, years=16, dept=IT)
Employee 4: VALID (annual=720000, tax/month=0, years=2, dept=IT)
Employee 5: INVALID: Missing or non-positive salary
Employee 6: INVALID: Invalid hire date
Employee 7: INVALID: No valid department assigned
Employee 999: INVALID: Employee 999 not found
```

C1 output: <img width="1483" height="1715" alt="c1" src="https://github.com/user-attachments/assets/f25800ae-68c4-4d6c-af50-c859d7d83862" />


---

## 7. Reflection (C2)



*What I learned about GOTO
GOTO jumps to a label in the program. I used it to classify numbers and to review salaries. I learned it can jump out of an IF block, but it cannot jump into one, because that gives the error PLS-00375. I fixed this by moving the label outside the IF.

GOTO vs normal code
When I rewrote the programs without GOTO, using IF/ELSIF and CONTINUE, they gave the same results and were easier to read. I think GOTO should be avoided in most cases. It is only useful for something like one shared error exit, as in my payroll validator.

What I learned about functions
A function always returns a value, and I can use it inside a SELECT query. I wrote functions for annual salary, years of service, tax and department name. I used exception handling so that bad input, like an employee that doesn’t exist, gives NULL, UNKNOWN or a clear error message instead of crashing.
