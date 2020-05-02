2. Find those students for whom all of their friends are in different grades from themselves. Return the students' names and grades.
```sql
SELECT h3.NAME, 
       h3.grade 
FROM   (SELECT f1.id1, 
               f1.id2, 
               CASE 
                 WHEN h1.grade = h2.grade THEN 0 
                 ELSE 1 
               END AS same_grade_indicator 
        FROM   friend f1 
               INNER JOIN highschooler h1 
                       ON f1.id1 = h1.id 
               INNER JOIN highschooler h2 
                       ON f1.id2 = h2.id) t1 
       INNER JOIN highschooler h3 
               ON t1.id1 = h3.id 
GROUP  BY t1.id1 
HAVING Sum(t1.same_grade_indicator) = Count(t1.same_grade_indicator) 
ORDER  BY grade, 
          NAME 
```
3. What is the average number of friends per student? (Your result should be just one number.)

```sql
SELECT Avg(f1.counts) 
FROM   (SELECT id1, 
               Count(*) AS counts 
        FROM   friend 
        GROUP  BY id1) f1 
```

4. Find the number of students who are either friends with Cassandra or are friends of friends of Cassandra. Do not count Cassandra, even though technically she is a friend of a friend.

```sql
SELECT Count(DISTINCT id1) 
FROM   (SELECT f1.id1 AS id1, 
               f1.id2 AS id2, 
               f2.id2 
        FROM   friend f1 
               INNER JOIN friend f2 
                       ON f1.id2 = f2.id1 
               INNER JOIN highschooler not_cass 
                       ON f1.id1 = not_cass.id 
               INNER JOIN highschooler h1 
                       ON f1.id2 = h1.id 
               INNER JOIN highschooler h2 
                       ON f2.id2 = h2.id 
        WHERE  not_cass.NAME <> 'Cassandra' 
               AND ( ( h1.NAME = 'Cassandra' 
                       AND h2.NAME <> 'Cassandra' ) 
                      OR ( h1.NAME <> 'Cassandra' 
                           AND h2.NAME = 'Cassandra' ) )) t1 
```

5. Find the name and grade of the student(s) with the greatest number of friends.
```sql
SELECT name, 
       grade 
FROM   highschooler h 
       inner join (SELECT id1, 
                          Count(*) AS counts2 
                   FROM   friend 
                   GROUP  BY id1) f2 
               ON h.id = f2.id1 
WHERE  counts2 == (SELECT Max(f1.counts) 
                   FROM   (SELECT id1, 
                                  Count(*) AS counts 
                           FROM   friend 
                           GROUP  BY id1) f1) 
```