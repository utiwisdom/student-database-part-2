# 🎓 Student Database — Part 2 (Advanced SQL + Bash Reporting)

![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)
![Bash](https://img.shields.io/badge/Scripting-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![SQL](https://img.shields.io/badge/Query-SQL-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![freeCodeCamp](https://img.shields.io/badge/freeCodeCamp-Relational%20Database-0A0A23?style=for-the-badge&logo=freecodecamp&logoColor=white)

---

## 📌 Project Overview

Part 2 of the [Student Database project](../student-database-postgresql-bash) — building on the normalized 4-table PostgreSQL schema from Part 1 by adding **advanced SQL querying** and a **Bash reporting script** (`student_info.sh`).

Where Part 1 focused on **loading** data (ETL: CSV → normalized DB), Part 2 focuses on **querying** that data: filtering, pattern matching, aggregating, sorting, limiting, and — most importantly — **joining multiple tables together**.

The result is a single Bash script that prints a full report about the students, majors, and courses in the database.

---

## 🚀 What I Built

- A Bash script (`student_info.sh`) with **11 report sections**
- Advanced `WHERE` filters (numeric, text, `NULL`, boolean logic with parentheses)
- Pattern matching with `LIKE`, `ILIKE`, `%`, and `_`
- Sorting with `ORDER BY` (single and multi-column, `ASC`/`DESC`)
- Row limiting with `LIMIT`
- Aggregate functions: `MIN`, `MAX`, `SUM`, `AVG`, `COUNT`, `DISTINCT`
- Rounding with `ROUND`, `CEIL`, `FLOOR`
- Grouping with `GROUP BY` + filtering groups with `HAVING`
- **All four JOIN types:** `INNER`, `LEFT`, `RIGHT`, `FULL`
- Table aliasing with `AS` and column joining shorthand with `USING`
- Multi-table joins (3–4 tables at once)

---

## 📂 Project Structure

```text
student-database-part-2/
│
├── student_info.sh              # The reporting script (main deliverable)
├── students.sql                 # Database dump (from Part 1)
├── README.md
├── pgadmin-students-table.png   # Screenshot: students table in pgAdmin
└── hello.png                    # Screenshot: query verification
```

---

## 🗄️ Database (From Part 1)

The script queries a normalized 4-table schema:

```text
majors ──1:N──► students
majors ──N:M──► majors_courses ◄──N:M── courses
```

| Table | Purpose |
|-------|---------|
| `students` | Student name, major, GPA |
| `majors` | Unique academic majors |
| `courses` | Unique courses offered |
| `majors_courses` | Junction: which courses belong to which majors |

**See [Part 1](../student-database-postgresql-bash) for the schema definition, the ETL script, and the dump.**

---

## 🧠 SQL Concepts Covered

| # | Report Section | SQL Concept |
|---|----------------|-------------|
| 1 | 4.0 GPA students | `WHERE` with `=` |
| 2 | Courses before 'D' | `WHERE` with `<` on text |
| 3 | High/low GPA students | `WHERE` + `AND` + `OR` + parentheses |
| 4 | Name pattern matching | `LIKE`, `ILIKE`, `%`, `_` |
| 5 | Students without a major | `IS NULL` + `AND`/`OR` |
| 6 | Sorted, limited courses | `LIKE` + `ORDER BY DESC` + `LIMIT` |
| 7 | Average GPA | `AVG` + `ROUND` |
| 8 | Students per major | `GROUP BY` + `HAVING` + `COUNT` + `AVG` |
| 9 | Majors with/without students | `FULL JOIN` + `USING` |
| 10 | Courses nobody takes | `DISTINCT` + multi-table `FULL JOIN` |
| 11 | Courses with one student | `INNER JOIN` + `GROUP BY` + `HAVING` |

---

## 🔍 The Reporting Script

**`student_info.sh`** — a single Bash script that prints 11 sections of reports:

```bash
#!/bin/bash

# Info about my computer science students from students database

PSQL="psql -X --username=freecodecamp --dbname=students --no-align --tuples-only -c"

echo -e "\n~~ My Computer Science Students ~~\n"

# 1. Students with a 4.0 GPA
echo -e "\nFirst name, last name, and GPA of students with a 4.0 GPA:"
echo "$($PSQL "SELECT first_name, last_name, gpa FROM students WHERE gpa = 4.0")"

# 2. Courses before 'D'
echo -e "\nAll course names whose first letter is before 'D' in the alphabet:"
echo "$($PSQL "SELECT course FROM courses WHERE course < 'D'")"

# 3. R+ last name + extreme GPA
echo -e "\nFirst name, last name, and GPA of students whose last name begins with an 'R' or after and have a GPA greater than 3.8 or less than 2.0:"
echo "$($PSQL "SELECT first_name, last_name, gpa FROM students WHERE last_name >= 'R' AND (gpa > 3.8 OR gpa < 2.0)")"

# 4. Case-insensitive pattern match
echo -e "\nLast name of students whose last name contains a case insensitive 'sa' or have an 'r' as the second to last letter:"
echo "$($PSQL "SELECT last_name FROM students WHERE last_name ILIKE '%sa%' OR last_name LIKE '%r_'")"

# 5. No major + D name or high GPA
echo -e "\nFirst name, last name, and GPA of students who have not selected a major and either their first name begins with 'D' or they have a GPA greater than 3.0:"
echo "$($PSQL "SELECT first_name, last_name, gpa FROM students WHERE major_id IS NULL AND (first_name LIKE 'D%' OR gpa > 3.0)")"

# 6. Filtered + sorted + limited
echo -e "\nCourse name of the first five courses, in reverse alphabetical order, that have an 'e' as the second letter or end with an 's':"
echo "$($PSQL "SELECT course FROM courses WHERE course LIKE '_e%' OR course LIKE '%s' ORDER BY course DESC LIMIT 5")"

# 7. Average GPA
echo -e "\nAverage GPA of all students rounded to two decimal places:"
echo "$($PSQL "SELECT ROUND(AVG(gpa), 2) FROM students")"

# 8. Grouped stats per major
echo -e "\nMajor ID, total number of students in a column named 'number_of_students', and average GPA rounded to two decimal places in a column name 'average_gpa', for each major ID in the students table having a student count greater than 1:"
echo "$($PSQL "SELECT major_id, COUNT(*) AS number_of_students, ROUND(AVG(gpa), 2) AS average_gpa FROM students WHERE major_id IS NOT NULL GROUP BY major_id HAVING COUNT(*) > 1")"

# 9. Majors not taken (FULL JOIN)
echo -e "\nList of majors, in alphabetical order, that either no student is taking or has a student whose first name contains a case insensitive 'ma':"
echo "$($PSQL "SELECT major FROM students FULL JOIN majors USING(major_id) WHERE student_id IS NULL OR first_name ILIKE '%ma%' ORDER BY major")"

# 10. Unique courses nobody takes (multi-table FULL JOIN)
echo -e "\nList of unique courses, in reverse alphabetical order, that no student or 'Obie Hilpert' is taking:"
echo "$($PSQL "SELECT DISTINCT(course) FROM students FULL JOIN majors USING(major_id) FULL JOIN majors_courses USING(major_id) FULL JOIN courses USING(course_id) WHERE student_id IS NULL OR first_name = 'Obie' AND last_name = 'Hilpert' ORDER BY course DESC")"

# 11. Courses with one student (INNER JOIN)
echo -e "\nList of courses, in alphabetical order, with only one student enrolled:"
echo "$($PSQL "SELECT course FROM students INNER JOIN majors_courses USING(major_id) INNER JOIN courses USING(course_id) GROUP BY course HAVING COUNT(*) = 1 ORDER BY course")"
```

---

## ▶️ How to Run It

### Prerequisites

- PostgreSQL installed and running
- A `students` database (restore from `students.sql` if needed)
- A PostgreSQL user with access

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/utiwisdom/student-database-part-2.git
cd student-database-part-2

# 2. Start PostgreSQL
sudo service postgresql start

# 3. Restore the database (if not already present)
psql -U postgres < students.sql

# 4. Make the script executable
chmod +x student_info.sh

# 5. Run the report
./student_info.sh
```

### Expected Output

The script prints 11 sections, each with a heading and its query results.

---

## 🖼️ Screenshots

![Students table in pgAdmin](IM_2.PNG)
![Query verification](IM_1.PNG  )

---

## 💭 Business Value

This project demonstrates the **query side** of data engineering:

- **Reporting** — turning a normalized schema into human-readable reports
- **Joins** — combining data across `1:N` and `N:M` relationships
- **Aggregation** — computing per-group statistics (`GROUP BY` + `HAVING`)
- **Filtering** — extracting exactly the rows a business question needs

The same patterns power:
- **FinTech:** "Total transaction volume per account this month"
- **HealthTech:** "Average patient wait time per department"
- **E-commerce:** "Top 10 products by revenue, grouped by category"

Every one of those is a `JOIN` + `GROUP BY` + `HAVING` + `ORDER BY` away — exactly what this script does.

---

## 📚 Technologies Used

- PostgreSQL 12
- Bash (shell scripting)
- SQL — `SELECT`, `WHERE`, `LIKE`/`ILIKE`, `IS NULL`, `ORDER BY`, `LIMIT`, `GROUP BY`, `HAVING`, all four `JOIN` types, aggregates (`AVG`, `COUNT`, `MIN`, `MAX`, `SUM`), `DISTINCT`, `ROUND`/`CEIL`/`FLOOR`, aliases (`AS`, `USING`)

---

## 🎯 Skills Demonstrated

- Advanced SQL Querying
- Multi-Table Joins (INNER, LEFT, RIGHT, FULL)
- Aggregate Functions and Grouping
- Pattern Matching (`LIKE` / `ILIKE` with `%` and `_`)
- NULL Handling (`IS NULL`, `IS NOT NULL`)
- Boolean Logic in SQL (`AND`, `OR`, parentheses)
- Sorting and Pagination (`ORDER BY`, `LIMIT`)
- Column and Table Aliasing (`AS`, `USING`)
- Bash Scripting for Reporting
- Database Restore from Dump (`psql < dump.sql`)

---

## 👨‍💻 Author

**Wisdom Oghenevwede Uti**

Aspiring Data Engineer | ALX Data Science Learner | Secondary School Teacher

Building skills in Docker, Python, SQL, Cloud Computing, Data Engineering, and Modern Data Platforms.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/uti-wisdom-286602228/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/utiwisdom)
[![Portfolio](https://img.shields.io/badge/Portfolio-Visit-FF5722?style=for-the-badge&logo=google-chrome&logoColor=white)](https://datascienceportfol.io/wisdomuti8)

---

## 🙏 Acknowledgments

Built as part of the [freeCodeCamp Relational Database Certification](https://www.freecodecamp.org/learn/relational-database/) — **Learn SQL by Building a Student Database: Part 2**.

Part 1 (the ETL script + schema): [student-database-postgresql-bash](https://github.com/utiwisdom/student-database-postgresql-bash)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
