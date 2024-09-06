- [A Comprehensive Guide to SQL Window Functions](#a-comprehensive-guide-to-sql-window-functions)
- [Demystifying Common Table Expressions (CTEs) in SQL](#demystifying-common-table-expressions-ctes-in-sql)
- [The Universal View: All Joins as Filtered Cross Joins](#the-universal-view-all-joins-as-filtered-cross-joins)
- [Understanding Implicit Joins in SQL: The Old‐School Approach](#understanding-implicit-joins-in-sql-the-old-school-approach)
- [Understanding Subqueries in SQL: Types and Use Cases](#understanding-subqueries-in-sql-types-and-use-cases)
- [Unlocking the Power of Recursive CTEs in SQL](#unlocking-the-power-of-recursive-ctes-in-sql)
- [What actually happens when you do an inner join?](#What-actually-happens-when-you-do-an-inner join?)
- [Why do we even need Junction Tables?](#why-do-we-even-need-junction-tables)


# Why do we even need Junction Tables?
[junction-tables]: #junction-tables "Why do we even need Junction Tables?"

Imagine you're creating a database for Greenwood High School. You start with two simple tables:

Students:
| StudentID | Name | Address |
|-----------|------|---------|
| 1 | John Smith | 123 Maple St, Greenwood |
| 2 | Emma Johnson | 456 Oak Ave, Greenwood |

Courses:
| CourseID | Name |
|----------|------|
| 101 | Algebra |
| 102 | Biology |

The principal asks, "How do we know which students are in which courses?"

## Attempt 1: Adding Courses to the Student Table

Your first thought is to add a course column to the student table:

Students:
| StudentID | Name | Address | Course |
|-----------|------|---------|--------|
| 1 | John Smith | 123 Maple St, Greenwood | Algebra |
| 2 | Emma Johnson | 456 Oak Ave, Greenwood | Algebra |

This works until Emma decides to also take Biology.

## Attempt 2: Multiple Course Columns

To solve this, you add multiple course columns:

Students:
| StudentID | Name | Address | Course1 | Course2 |
|-----------|------|---------|---------|---------|
| 1 | John Smith | 123 Maple St, Greenwood | Algebra | NULL |
| 2 | Emma Johnson | 456 Oak Ave, Greenwood | Algebra | Biology |

This seems to work, but what if a new student, Michael, wants to take three courses? You'd need to keep adding columns.

## Attempt 3: Repeating Student Rows

You try a different approach:

Students:
| StudentID | Name | Address | Course |
|-----------|------|---------|--------|
| 1 | John Smith | 123 Maple St, Greenwood | Algebra |
| 2 | Emma Johnson | 456 Oak Ave, Greenwood | Algebra |
| 2 | Emma Johnson | 456 Oak Ave, Greenwood | Biology |

This allows students to take multiple courses, but now you have repeated data. What happens if Emma moves to 789 Pine Rd? You'd have to update multiple rows, risking inconsistencies if you miss one.

## Attempt 4: Adding Students to Course Table

You try the reverse approach:

Courses:
| CourseID | Name | Student1 | Address1 | Student2 | Address2 |
|----------|------|----------|----------|----------|----------|
| 101 | Algebra | John Smith | 123 Maple St, Greenwood | Emma Johnson | 456 Oak Ave, Greenwood |
| 102 | Biology | Emma Johnson | 456 Oak Ave, Greenwood | NULL | NULL |

This works for now, but what happens when a third student wants to join Algebra? You'd need to add more columns.

## The Solution: Junction Table

After much frustration, you discover the concept of a junction table. You keep your original tables:

Students:
| StudentID | Name | Address |
|-----------|------|---------|
| 1 | John Smith | 123 Maple St, Greenwood |
| 2 | Emma Johnson | 456 Oak Ave, Greenwood |

Courses:
| CourseID | Name |
|----------|------|
| 101 | Algebra |
| 102 | Biology |

And create a new table to manage enrollments:

Enrollments (Junction Table):
| StudentID | CourseID |
|-----------|----------|
| 1 | 101 |
| 2 | 101 |
| 2 | 102 |

Now, let's see how this handles different scenarios:

| Scenario | Generic Action |
|----------|----------------|
| A new student, Michael Lee, joins the school and enrolls in Biology | 1. Add a new record to the Students table<br>2. Add a new record to the Enrollments table |
| A new course, Chemistry, is added, and John enrolls | 1. Add a new record to the Courses table<br>2. Add a new record to the Enrollments table |
| Emma decides to take Chemistry as well | Add a new record to the Enrollments table |
| Find all students in Algebra | 1. Look up the CourseID for Algebra<br>2. Query the Enrollments table for all records with this CourseID<br>3. Use the resulting StudentIDs to query the Students table |
| Find all courses John is taking | 1. Look up John's StudentID<br>2. Query the Enrollments table for all records with this StudentID<br>3. Use the resulting CourseIDs to query the Courses table |
| Emma moves to 789 Pine Rd, Greenwood | Update a single record in the Students table |

This solution handles all scenarios elegantly, without needing to modify table structures or repeat data. Note how changing Emma's address now only requires updating a single row in the Students table, regardless of how many courses she's enrolled in.

## Extended Junction Table

We can easily extend the junction table to include more information:

Enrollments (Extended):
| StudentID | CourseID | EnrollmentDate |
|-----------|----------|----------------|
| 1 | 101 | 2023-09-01 |
| 2 | 101 | 2023-09-01 |
| 2 | 102 | 2023-09-02 |

This allows you to track when each student enrolled in each course, without affecting the structure of the Students or Courses tables.

## Conclusion

The junction table solves the many-to-many relationship problem by allowing:
1. Students to enroll in any number of courses
2. Courses to have any number of students
3. Easy addition of new students, courses, or enrollments
4. Efficient querying of relationships in both directions
5. Additional enrollment-specific data to be stored
6. Simple updates to student or course information without risking data inconsistencies

This approach provides flexibility, eliminates redundancy, and allows the database to grow with Greenwood High School's needs.

# What actually happens when you do an inner join?

SQL joins are a fundamental concept in relational databases, allowing us to combine data from multiple tables. In this post, we'll explore different types of joins, starting from basic inner joins and progressing to more advanced concepts like non-equi joins and self joins.

## 1. Inner Joins: The Basics

Let's start with a simple scenario using employees and departments. Consider these two tables:

**Employees:**
| id | name | department_id |
|----|------|---------------|
| 1  | Alice| 1             |
| 2  | Bob  | 2             |
| 3  | Charlie | 1          |
| 4  | Diana| 3             |
| 5  | Eva  | 2             |
| 6  | Frank| 4             |

**Departments:**
| id | name    |
|----|---------|
| 1  | HR      |
| 2  | IT      |
| 3  | Finance |

To get a list of employees with their department names, we can use a basic inner join:

```sql
SELECT Employees.name AS employee, Departments.name AS department
FROM Employees
INNER JOIN Departments ON Employees.department_id = Departments.id;
```

This join operation:
1. Starts with the `Employees` table.
2. Looks for matching `department_id` in the `Departments` table.
3. Combines matching data into a single row in the result.
4. Excludes rows without a match.

The result:
| employee | department |
|----------|------------|
| Alice    | HR         |
| Bob      | IT         |
| Charlie  | HR         |
| Diana    | Finance    |
| Eva      | IT         |

Key takeaways:
- Only employees with matching departments are included.
- It provides more readable data by showing department names instead of IDs.
- Multiple employees in the same department each get their own row.
- The order of tables in `FROM` and `JOIN` doesn't affect the final result.

## 2. Non-Equi Joins: Beyond Equality

Non-equi joins use comparison operators other than equality (=) in the join condition. Let's look at an example using a salary grade scenario:

**Employees:**
| id | name    | department_id | salary |
|----|---------|---------------|--------|
| 1  | Alice   | 1             | 55000  |
| 2  | Bob     | 2             | 62000  |
| 3  | Charlie | 1             | 48000  |
| 4  | Diana   | 3             | 71000  |
| 5  | Eva     | 2             | 59000  |

**Salary_Grades:**
| grade | min_salary | max_salary |
|-------|------------|------------|
| A     | 65000      | 80000      |
| B     | 55000      | 64999      |
| C     | 45000      | 54999      |

To match employees with their salary grade:

```sql
SELECT e.name, e.salary, sg.grade
FROM Employees e
INNER JOIN Salary_Grades sg ON e.salary BETWEEN sg.min_salary AND sg.max_salary;
```

This join:
1. Starts with the `Employees` table.
2. Finds `Salary_Grades` rows where the employee's salary is between `min_salary` and `max_salary`.
3. Combines matching data into the result.

Result:
| name    | salary | grade |
|---------|--------|-------|
| Alice   | 55000  | B     |
| Bob     | 62000  | B     |
| Charlie | 48000  | C     |
| Diana   | 71000  | A     |
| Eva     | 59000  | B     |

Non-equi joins allow for more flexible conditions and can be used for categorization or finding relationships based on ranges.

## 3. Self Joins with Non-Equi Conditions: Creating Pairs

Self joins occur when a table is joined with itself. Combined with non-equi conditions, they can be powerful for data analysis. Let's use our Employees table to find pairs of employees where one earns more than another:

**Employees:**
| id | name    | salary |
|----|---------|--------|
| 1  | Alice   | 55000  |
| 2  | Bob     | 62000  |
| 3  | Charlie | 48000  |
| 4  | Diana   | 71000  |
| 5  | Eva     | 59000  |

Here's the SQL query using a self join with a simple non-equi condition:

```sql
SELECT e1.name AS employee1, e1.salary AS salary1, 
       e2.name AS employee2, e2.salary AS salary2
FROM Employees e1
JOIN Employees e2 ON e1.salary > e2.salary;
```

This join operation works as follows:

1. It starts with the Employees table (aliased as e1).
2. For each employee in e1, it looks for other employees (e2) where:
   - e1's salary is greater than e2's salary
3. When this condition is met, it creates a row in the result pairing these employees.

The result would be:

| employee1 | salary1 | employee2 | salary2 |
|-----------|---------|-----------|---------|
| Bob       | 62000   | Alice     | 55000   |
| Bob       | 62000   | Charlie   | 48000   |
| Diana     | 71000   | Bob       | 62000   |
| Diana     | 71000   | Eva       | 59000   |
| Diana     | 71000   | Alice     | 55000   |
| Diana     | 71000   | Charlie   | 48000   |
| Eva       | 59000   | Alice     | 55000   |
| Eva       | 59000   | Charlie   | 48000   |
| Alice     | 55000   | Charlie   | 48000   |

Key points about this self join with a simple non-equi condition:

1. It allows us to compare each employee's salary with every other employee's salary.
2. The non-equi condition (`>`) lets us find all pairs where one employee earns more than another.
3. This query includes comparisons of employees with themselves, which may or may not be desired depending on the specific analysis needs.
4. The result set is not ordered in any specific way, as the focus is on the relationship between the salaries rather than their absolute values.
5. This type of query can be useful for analyzing salary distributions, identifying potential pay gaps, or understanding the overall salary structure within an organization.

This example demonstrates how combining self joins with even a simple non-equi condition can uncover relationships within a single table, providing valuable insights for data analysis. It's particularly useful when you need to compare elements within a dataset based on inequalities.

## Conclusion

SQL joins are a powerful tool for data analysis and manipulation. From basic inner joins to complex self joins with non-equi conditions, understanding these concepts allows you to extract meaningful insights from relational databases. As you become more comfortable with these techniques, you'll find yourself able to answer increasingly complex questions about your data.

Remember, the key to mastering SQL joins is practice. Try these examples on your own datasets and experiment with different conditions to see what insights you can uncover!


artifact https://claude.site/artifacts/59c1e32c-7398-47c6-ab96-22bf0c5be477
