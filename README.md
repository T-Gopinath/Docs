Write a query to select Nth maximum salary from EMP table 
(or) Write a query to find 2nd, 3rd max salary from EMP table 
(or) Write a query to find 10 highest salary 
(Or) Write a query to find 4th highest salary (without analytical function)  

  We can achieve this by using the correlated sub query. In the below example 
we are getting the 5th highest salary without using the Analytical function.  

**select * from emp emp1 where (5-1) = ( select count(distinct(emp2.sal)) from 
emp emp2 where emp2.sal > emp1.sal )**
____________________________________________________________________________________________
  
  _In the below example we are getting the 5th highest salary by using the 
Analytical function._ 

**select * from ( select e.*, DENSE_RANK() over (order by sal DESC) RN from 
emp e ) where RN=5** 

**Select maximum N salaries from EMP Table** 
___________________________________________________________________________________________________
Write a query to select top N salaries from the EMP table 
(or) Write a query to select maximum N salaries from the EMP table 

Answer: We can achieve this by using the DENSE_RANK Analytical function. In the 
below example we are getting the TOP 5 salaries from the EMP table.

**select * from ( select e.*, DENSE_RANK() over (order by sal DESC) RN from emp e ) 
where RN <=5**
_________________________________________________________________________________________

Select top N salaries from each Department of EMP table 
Write a query to select top N salaries from each department of the EMP table 
(or) Write a query to select maximum N salaries from each department of the 
EMP table 

Answer: We can achieve this by using the DENSE_RANK Analytical function. In 
the below example we are getting the TOP 3 salaries for each department of 
the EMP table. 

**select * from ( select e.*, DENSE_RANK() over (partition by deptno order by sal 
DESC) RN from emp e ) where RN <=3** 
________________________________________________________________________________________

Select/Delete duplicate rows from EMP table 
**select * from emp where rowid not in ( select min(rowid) from emp group by 
empno );**  
**delete from emp where rowid not in ( select min(rowid) from emp group by 
empno );** 

________________________________________________________________________________________
Same salary query 

Write a query to select only those employee information who are earning the 
same salary?  
Answer: We can achieve this in at least 3 ways… 

1 st way 
**select e1.* from emp e1,emp e2 where e1.sal=e2.sal and e1.ename <> e2.ename**
2 nd way 
**select * from emp where sal in (select sal from emp group by sal having count(sal)>=2 );**
3 rd way  
**SELECT * FROM ( SELECT e.*, count(*) Over (Partition BY sal ORDER BY sal) cnt 
FROM emp e ) WHERE cnt>=2;**

__________________________________________________________________________________

Odd/Even rows Question... 
Write a query to display even/odd number rows from a table.  
We can achieve this by using the ROWNUM pseudo column.  

**select * from (select empno, ename, sal, rownum rn from emp order by empno ) where 
mod (rn, 2) <> 0 order by rn**
____________________________________________________________________________________

More than 2 employees Question 

Write a query to display the employee information, who have more than 2 
employees under a manager

We can achieve this by using the COUNT analytical function. 

**select * from ( SELECT e.*, count(mgr) over (partition by mgr) as cnt from emp 
e ) where cnt >= 2** 

________________________________________________________________________________________
Maximum salary without using functions


Write a query to find the maximum salary from the EMP table 
without using functions. 

We can achieve this by using the SELF joins.

**select * from emp where sal not in ( select A.sal from emp A, emp B 
where A.sal < B.sal )** 
_____________________________________________________________________________________________
Find the number of rows in a table without using COUNT function

Write a query to find the number of rows in a table without using COUNT 
function..  
**SELECT MAX(rn) FROM ( SELECT ROW_NUMBER() OVER(ORDER BY empno 
DESC) as rn FROM emp )** 
_____________________________________________________________________________________________

Find the LAST inserted record in a table 

If you want the last record inserted, you need to have a timestamp or 
sequence number assigned to each record as they are inserted and then you 
can use the below query…

**select * from t where TIMESTAMP_COLUMN = (select max(timestamp_column) from T) and rownum = 1;**____________________________________________________________________________________________

Select LAST n records from a table

Write a query to select the last N records from a table… 
Or Explain the below query…

**select * from emp minus select * from emp where rownum <= (select count(*) - &n from emp);**__________________________________________________________________________________________
Write a query to find the employees who are working in the company for the 
past 5 years. 
Answer: We can achieve this using the ADD_MONTHS function.  

**select * from emp where hiredate < add_months(sysdate,-60)**__________________________________________________________________________________________



