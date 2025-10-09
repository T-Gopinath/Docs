Q) Write a query to select Nth maximum salary from EMP table 
(or) Write a query to find 2nd, 3rd max salary from EMP table 
(or) Write a query to find 10 highest salary 
(Or) Write a query to find 4th highest salary (without analytical function)  

Answer:  We can achieve this by using the correlated sub query. In the below example 
we are getting the 5th highest salary without using the Analytical function.  

**select * from emp emp1 where (5-1) = ( select count(distinct(emp2.sal)) from 
emp emp2 where emp2.sal > emp1.sal )**
____________________________________________________________________________________________
  
 Q)  _In the below example we are getting the 5th highest salary by using the 
Analytical function._ 

**select * from ( select e.*, DENSE_RANK() over (order by sal DESC) RN from 
emp e ) where RN=5** 

**Select maximum N salaries from EMP Table** 
___________________________________________________________________________________________________
Q) Write a query to select top N salaries from the EMP table 
(or) Write a query to select maximum N salaries from the EMP table 

Answer: We can achieve this by using the DENSE_RANK Analytical function. In the 
below example we are getting the TOP 5 salaries from the EMP table.

**select * from ( select e.*, DENSE_RANK() over (order by sal DESC) RN from emp e ) 
where RN <=5**
_________________________________________________________________________________________

Q) Select top N salaries from each Department of EMP table 
Write a query to select top N salaries from each department of the EMP table 
(or) Write a query to select maximum N salaries from each department of the 
EMP table 

Answer: We can achieve this by using the DENSE_RANK Analytical function. In 
the below example we are getting the TOP 3 salaries for each department of 
the EMP table. 

**select * from ( select e.*, DENSE_RANK() over (partition by deptno order by sal 
DESC) RN from emp e ) where RN <=3** 
________________________________________________________________________________________

Q) Select/Delete duplicate rows from EMP table 

**select * from emp where rowid not in ( select min(rowid) from emp group by 
empno );**  
**delete from emp where rowid not in ( select min(rowid) from emp group by 
empno );** 

________________________________________________________________________________________
Q) Same salary query 

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

Q) Odd/Even rows Question... 
Write a query to display even/odd number rows from a table.  
We can achieve this by using the ROWNUM pseudo column.  

**select * from (select empno, ename, sal, rownum rn from emp order by empno ) where 
mod (rn, 2) <> 0 order by rn**
____________________________________________________________________________________

Q) More than 2 employees Question 

Write a query to display the employee information, who have more than 2 
employees under a manager

Answer: We can achieve this by using the COUNT analytical function. 

**select * from ( SELECT e.*, count(mgr) over (partition by mgr) as cnt from emp 
e ) where cnt >= 2** 

________________________________________________________________________________________
Q) Maximum salary without using functions


Write a query to find the maximum salary from the EMP table 
without using functions. 

Answer: We can achieve this by using the SELF joins.

**select * from emp where sal not in ( select A.sal from emp A, emp B 
where A.sal < B.sal )** 
_____________________________________________________________________________________________
Q) Find the number of rows in a table without using COUNT function

Answer: Write a query to find the number of rows in a table without using COUNT 
function..  
**SELECT MAX(rn) FROM ( SELECT ROW_NUMBER() OVER(ORDER BY empno 
DESC) as rn FROM emp )** 
_____________________________________________________________________________________________

Q) Find the LAST inserted record in a table 

Answer: If you want the last record inserted, you need to have a timestamp or 
sequence number assigned to each record as they are inserted and then you 
can use the below query…

**select * from t where TIMESTAMP_COLUMN = (select max(timestamp_column) from T) and rownum = 1;**

____________________________________________________________________________________________

Q) Select LAST n records from a table

Write a query to select the last N records from a table… 
Or Explain the below query…

**select * from emp minus select * from emp where rownum <= (select count(*) - &n from emp);**

__________________________________________________________________________________________
Q) Write a query to find the employees who are working in the company for the 
past 5 years. 
Answer: We can achieve this using the ADD_MONTHS function.  

**select * from emp where hiredate < add_months(sysdate,-60)**
__________________________________________________________________________________________

Q) Find duplicate record in a table 

**SELECT name, section from tbl GROUP BY name , section Having count(*) > 1
**
__________________________________________________________________________________________
Q) Find student whose marks are greater than average marks.
  ** SELECT student_name,marks from Student WHERE marks > (select avg(marks) FROM student **
_________________________________________________________________________________________

Primary_key| Unique-Key|
:------- | :----------: 
|Primary key can't have Null Values | Unique key allows NULL values
|Only one P.K in a table | Having multiple U.K

__________________________________________________________________________________________
## Normalization

  + The process of removing the **redundent data**, by **spliting up the table** in to well defined fashion is called normalization.
  + To reduce redundancy
  + Minimizing the insertion, Deletion and Update Anomalies.

  ### FirstNormal Form
  ### Second Normat form
  ### Third Normal Form

  #### FirstNormal Form
    A relation is in ** 1st normal ** form if it does not contain any multi-valued or composite attribures.
    By 1st normal form, if underlaying domains contans atomic values only.
    
|First_name| Last_name| Knowledge|
:------- | :---------- |:----------:
Thomas | lingaya|java,c++,PHP


##### After 1NCF
|First_name| Last_name| Knowledge|
:------- | :---------- |:----------:
Thomas | lingaya|java|
Thomas | lingaya|PHP|
Thomas | lingaya|C++|

### Second Normal Form
  + if a relation does not contain any partial dependency.
  + if every monkey values fully depends on P.K.

|Id| Last_name| IDProff|Profession|
:------- | :---------- |:----------|:----------:
1 | Muller|3|Professior-1
2 | Meier|2|Professior-1
3 | Tobler|1|Professior-1

#### After 2NF

##### Student Table
|Id| Last_name| 
|:----------|:----------:
1 | Muller|
2 | Meier|
3 | Tobler|

##### Professior Table
|IdProf| Professor| 
|:----------|:----------:
1 | Schmid|
2 | Borner|
3 | Bornasconi|
                                                                               
### Third Normal Form
  + Removing transitive dependency.
  + A relation is in the third normal form, if it does not contain any transitive dependency.
  + If every non-key attribures is non-transtivity dependds on the P.K.
  + A is functionally depends on B, B is functionally depends on C, C is transitively depends on A via B.

|Book_Id| Genre_ID| Genre Type|Price|
:------- | :---------- |:----------|:----------:
1 | 1|3|gardning|25-99

+ Here book_id determines Genre_ID and Genre_ID determines Genre_Type. So BookID determine Genere_ID

##### BookTable

|Book_Id| Genre_ID|Price|                           
:------- | :---------- |:----------:
1|1|25|

##### GENRE Table

|Genre_ID| Genre_Type|
:------- | :----------:
|3|gardning|

    




