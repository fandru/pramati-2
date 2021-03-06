## SET - 2 

**1. Manager Name, Count of Employees whose start date is after 2013** 

```
SELECT b.name AS  Mgr_Name,count(a.name) AS Emp_count 
FROM employee a, employee b 
WHERE b.emp_id = a.mgr_id 
AND a.joining_date>'01-01-2013' 
GROUP BY b.name 
ORDER BY b.name;
```
``` 
SELECT b.name AS  Mgr_Name,count(a.name) AS Emp_count 
FROM employee a inner join employee b ON b.emp_id = a.mgr_id 
WHERE a.joining_date>'01-01-2013' 
group by b.name 
order by b.name;
```
```
SELECT COALESCE(b.name,'CEO') AS  Mgr_Name,count(a.name) AS Emp_count 
FROM employee a LEFT join employee b ON b.emp_id = a.mgr_id 
WHERE a.joining_date>'01-01-1900' 
GROUP BY b.name 
ORDER BY b.name;
```

**2) Manager Name, Min(emp salary) where manager start date is after 2013**

```
SELECT b.name AS "Mgr_Name",min(a.salary) AS "Emp_salary" 
FROM employee a, employee b 
WHERE b.emp_id = a.mgr_id 
AND b.joining_date>'01-01-2013' 
GROUP BY b.name;
```

**3) Manager name who atleast has two employees started after 2013**

```
SELECT b.name AS  Mgr_Name,count(a.name) AS Emp_count 
FROM employee a, employee b 
WHERE b.emp_id = a.mgr_id 
AND a.joining_date>'01-01-2013' 
GROUP BY b.name 
HAVING count(a.name) >=2;
```
```
SELECT b.name AS Mgr_Name 
FROM employee a, employee b 
WHERE b.emp_id = a.mgr_id 
AND a.joining_date>'01-01-2013' 
GROUP BY b.name 
HAVING count(a.name) >=2;
```

**4) Dept Name, Count of Employees whose start date is after 2013**

```
SELECT d.name,count(e.name) 
FROM employee e,dept d  
WHERE d.dept_id=e.dept_id 
AND e.joining_date>'01-01-2013' 
GROUP BY d.name;
```

**5) Dept Name, Min(emp salary) where manager start date is after 2013**

```
SELECT d.name,min(a.salary) 
FROM employee a,employee b,dept d  
WHERE d.dept_id=a.dept_id 
AND d.dept_id=b.dept_id 
AND b.emp_id=a.mgr_id 
AND  b.joining_date>'01-01-2013' 
GROUP BY d.name;
```

**6) Dept Name who atleast has two employees started after 2013**

```
SELECT d.name,count(e.name) 
FROM employee e,dept d  
WHERE d.dept_id=e.dept_id 
AND e.joining_date>'01-01-2013' 
GROUP BY d.name 
HAVING count(e.name)>=2;
```

**7. Number of employees per year**

```
SELECT COUNT(name) AS emp_count,EXTRACT(year FROM joining_date) 
FROM employee 
GROUP BY EXTRACT(year from joining_date);
```


**8. Number of employees per year, per department**

```
SELECT EXTRACT(year FROM e.joining_date) ,d.name,COUNT(e.emp_id) 
FROM employee e,dept d 
WHERE e.dept_id=d.dept_id 
GROUP BY EXTRACT(year FROM e.joining_date),d.name 
ORDER BY d.name;
```


**9.List of Months (Say, Jan-2018) where number of employees joined is more than 2**

```
SELECT EXTRACT(year FROM joining_date) AS YEAR,EXTRACT(month FROM joining_date) AS Month ,COUNT(emp_id) AS COUNT 
FROM employee GROUP BY EXTRACT(year FROM joining_date),
EXTRACT(month FROM joining_date) 
HAVING COUNT(emp_id)>2;
```


**10. List of Managers who has employees from more than one department**

```
SEELCT m.name 
FROM employee e,employee m,dept d 
WHERE e.mgr_id=m.emp_id 
AND e.dept_id=d.dept_id 
GROUP BY m.name having COUNT(DISTINCT d.name)>1;
```


**11. List of Departments who has atleast two managers**

```
SELECT d.name,COUNT(distinct m.name) 
FROM employee e,employee m,dept d 
WHERE e.mgr_id=m.emp_id 
AND m.dept_id=d.dept_id 
GROUP BY d.name 
HAVING COUNT(DISTINCT m.name)>=2;
```


**12. Employee Names who joined after 2013 and his manager has more than two reportees**

```
SELECT a.name 
FROM employee a,employee b 
WHERE b.emp_id=a.mgr_id 
AND a.joining_date>'01-01-2013'
AND (select count(a.name) 
FROM employee a,employee b 
WHERE b.emp_id = a.mgr_id )>2;
```

## SET - 3

**1. SQL to find the missing ids from dept**

```
SELECT  dept_id +1
FROM dept
WHERE dept_id + 1 NOT IN (SELECT DISTINCT dept_id FROM dept)
AND dept_id + 1 < (select MAX(dept_id) from dept) ;
```

**2. Manager Name, Reportee who joined first (Reportee Name - doj), Reportee who draws less sal (Reportee Name - salary) - window function**

```
SELECT a.name,a.doj AS "Reportee Name - doj",b.sal AS "Reportee Name - salary"
FROM
    (SELECT b.name,CONCAT(a.name,'-',a.joining_date) AS doj 
     FROM employee a,employee b
     WHERE b.emp_id=a.mgr_id 
     AND EXISTS(
         SELECT 1
         FROM employee a_inr
         WHERE a_inr.mgr_id=b.emp_id
         HAVING min(a_inr.joining_date)=a.joining_date
         ) 
) a
INNER JOIN (
    SELECT b.name,CONCAT(a.name,'-',a.salary) AS sal	
    FROM employee a,employee b
    WHERE b.emp_id=a.mgr_id
    AND EXISTS(
             SELECT 1
             FROM employee a_inr
             WHERE a_inr.mgr_id=b.emp_id
             HAVING min(a_inr.salary)=a.salary     
             )
) b
ON a.name=b.name;
```

```
SELECT m1.Manager_name,m1.jd,m2.sd
FROM
    (SELECT DISTINCT m2.name AS Manager_name,
     FIRST_VALUE(CONCAT(m1.name,'-',m1.joining_date)) OVER(
     PARTITION BY m1.mgr_id
     ORDER BY m1.joining_date) AS jd
     FROM employee m1, employee m2
     WHERE m2.emp_id=m1.mgr_id
    )m1 
INNER JOIN
    (SELECT DISTINCT m2.name AS Manager_name,
     FIRST_VALUE(CONCAT(m1.name,'-',m1.salary)) OVER ( 
     PARTITION BY m1.mgr_id 
     ORDER BY m1.salary) AS sd 
     FROM employee m1, employee m2 
     WHERE m2.emp_id=m1.mgr_id
    )m2
ON m1.Manager_name=m2.Manager_name;
```
