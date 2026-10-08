What I learned about GOTO

GOTO jumps to a label in the program. I used it to classify numbers (A1) and to review salaries (A2). I learned that GOTO can jump out of an IF block, but it cannot jump into one, because this causes the error PLS-00375 (A3). I fixed it by moving the label outside the IF block. I also learned that a label must be followed by a statement, so I used NULL; when nothing else came after it.

GOTO vs normal code

When I rewrote A1 and A2 without GOTO (A4), using IF/ELSIF/ELSE and CONTINUE, the programs gave the same results and were easier to read. I think GOTO should be avoided in most cases. It is only useful in a few situations, such as one shared error exit, as in my payroll validator (C1).

What I learned about functions

A function always returns a value, and I can call it inside a SELECT query (B5). I wrote functions for annual salary (B1), years of service (B2), tax (B3) and department name (B4). I used exception handling so bad input, like an employee that does not exist, gives NULL, UNKNOWN or a clear error message instead of crashing the program.

Challenges
Remembering that a label must be followed by a statement.
Deciding what each function should return for missing or wrong data.
Setting up the database connection in SQL Developer.
Combined task

In C1, I combined all the functions into one payroll validator. It checks that the employee exists, is active, has a salary, a valid department and a valid hire date, and that the tax is less than the salary. It returns VALID or INVALID with the reason.

Conclusion

This assignment helped me understand when to use GOTO, why structured code is better, and how functions make SQL code reusable.
