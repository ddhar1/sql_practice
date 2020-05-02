1. Find the names of all students who are friends with someone named Gabriel.
```sql
select h2.name as name from highschooler h2
inner join ( select id1 from friend
    where id2 in ( select id from highschooler h 
                    where h.name = "Gabriel" ) ) f1
                    on h2.id = f1.id1

```

2. For every student who likes someone 2 or more grades younger than themselves, return that student's name and grade, and the name and grade of the student they like.

```sql
select liker_name, liker_grade, likee_name, likee_grade from 
( select id1, liker.name as liker_name, liker.grade as liker_grade, id2, likee.name as likee_name, likee.grade as likee_grade from likes l
inner join highschooler liker on l.id1 = liker.id
inner join highschooler likee on l.id2 = likee.id ) l1
where (liker_grade - likee_grade >= 2 ) 
```
3. For every pair of students who both like each other, return the name and grade of both students. Include each pair only once, with the two names in alphabetical order.

```sql
select distinct h1.name, h1.grade, h2.name, h2.grade from 
(select l1.id1 as l1_id1, l1.id2 as l1_id2, l2.id2 as l2_id2 from likes l1
inner join likes l2 on l1.id2 = l2.id1 ) l3
inner join highschooler h1 on l3.l1_id1 = h1.id
inner join highschooler h2 on l3.l1_id2 = h2.id
where l1_id1 = l2_id2 and h1.name < h2.name

```

4. Find all students who do not appear in the Likes table (as a student who likes or is liked) and return their names and grades. Sort by grade, then by name within each grade.

```sql
select name, grade from ( select name, grade, l1.id1 as l1_id1, l2.id2 as l2_id2 from highschooler h
left join likes l1 on h.id = l1.id1
left join likes l2 on h.id = l2.id2 ) t1
where l1_id1 is null and l2_id2 is NULL
```

5. For every situation where student A likes student B, but we have no information about whom B likes (that is, B does not appear as an ID1 in the Likes table), return A and B's names and grades.

```sql
select h1.name, h1.grade, h2.name, h2.grade from likes l1
left join likes l2 on l1.id2 = l2.id1
inner join highschooler h1 on l1.id1 = h1.id
inner join highschooler h2 on l1.id2 = h2.id
where l2.id1 is null
```

6. Find names and grades of students who only have friends in the same grade. Return the result sorted by grade, then by name within each grade.
```sql
select h3.name, h3.grade from 
( select f1.id1, f1.id2, case when h1.grade = h2.grade then 0 else 1 end as same_grade_indicator from friend f1
inner join highschooler h1 on f1.id1 = h1.id
inner join highschooler h2 on f1.id2 = h2.id ) t1
inner join highschooler h3 on t1.id1 = h3.id
group by t1.id1
having sum(t1.same_grade_indicator) = 0
order by grade, name
```

7. For each student A who likes a student B where the two are not friends, find if they have a friend C in common (who can introduce them!). For all such trios, return the name and grade of A, B, and C.
```sql
select a.name, a.grade, b.name, b.grade, c.name, c.grade from 
    ( select l.id1 as id1, l.id2 as id2, f1.id1 as f1_id1, f1.id2 as f1_id2
        from likes l
        left join friend f1 on l.id1 = f1.id1 and l.id2 = f1.id2 ) t1
inner join friend f2 
    on t1.id1 = f2.id1
inner join friend f3 
    on t1.id2 = f3.id1
inner join highschooler a on t1.id1 = a.id
inner join highschooler b on t1.id2 = b.id
inner join highschooler c on f2.id2 = c.id
where t1.f1_id1 is null and t1.f1_id2 is null and f2.id2 = f3.id2
```

8. Find the difference between the number of students in the school and the number of different first names.
```sql
select count(*) - count( distinct name) as difference from highschooler
```

9. Find the name and grade of all students who are liked by more than one other student.
```sql
select name, grade from highschooler
where id in ( select id2 from likes
group by id2
having count(*) > 1)
```

