- [A Comprehensive Guide to SQL Window Functions](#window-functions)
- [Demystifying Common Table Expressions (CTEs) in SQL](#ctes)
- [The Universal View: All Joins as Filtered Cross Joins](#universal-joins)
- [Understanding Implicit Joins in SQL: The Old‐School Approach](#implicit-joins)
- [Understanding Subqueries in SQL: Types and Use Cases](#subqueries)
- [Unlocking the Power of Recursive CTEs in SQL](#recursive-ctes)
- [What actually happens when you do an inner join?](#inner-join-mechanics)
- [Why do we even need Junction Tables?](#junction-tables)

[window-functions]: #window-functions "A Comprehensive Guide to SQL Window Functions"
[ctes]: #ctes "Demystifying Common Table Expressions (CTEs) in SQL"
[universal-joins]: #universal-joins "The Universal View: All Joins as Filtered Cross Joins"
[implicit-joins]: #implicit-joins "Understanding Implicit Joins in SQL: The Old‐School Approach"
[subqueries]: #subqueries "Understanding Subqueries in SQL: Types and Use Cases"
[recursive-ctes]: #recursive-ctes "Unlocking the Power of Recursive CTEs in SQL"
[inner-join-mechanics]: #inner-join-mechanics "What actually happens when you do an inner join?"
[junction-tables]: # Why do we even need Junction Tables? "Why do we even need Junction Tables?"


# Why do we even need Junction Tables?

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
