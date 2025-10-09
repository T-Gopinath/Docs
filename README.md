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

________________________________________________________________________________
DDL - Data Definition Language.
  +  Create
  +  ALTER
  +  DROP
  +  RENAME

DML - Data Manipulation Language
  + Select
  + Insert 
  + Update
  + Delete

DCL - Data Control Language (TCL)
  + Grant
  + Revoke

______________________________________________________________________________________

Defference between **Having** and **WHERE** clause

Having works on groupby
Where class works on table's (before groupby)

**SELECT NAME,SECTION ON FROM TABLE GROUP BY NAME, SECTION hAVING COUNT(*) > 1**
________________________________________________________________________________________

#### JOIN

  An SQL join is Used to combine data from two or more tables, based on a common field between them.

##### Student_Table
  
|Enroll_No| Student_Name| Adress|
:------- | :---------- |:----------:
100 | geek1|geeksqaz1|

##### Student_Course

|CourseID| ENROLLNO|
:------- |:----------:
|1|100|


**SELECT StudentCouse.CourseID, Student.Student_Name from Student 
  INNER join STUDENT_COURSE ON Student_Course.EnrollmetNo == Student.EnrollNo
  ORDERby StudentCours.CoursID**

___________________________________________________________________________________________________________

Q) WHAT is identity
  Identity is a column that automatically generates numaric values
___________________________________________________________________________________________________________

Q) What is a view in SQL ? how to create one.

+ A View is a virtual table based on the result of an SQL Statment 
+ We can create using Create view syntax

   **_ CREATE VIEW view_name AS_ SELECT Column_name(s) from table where condition**

+ View take very little space to store the data base conditions only the definition of a view, not a copy of
  all the data which is present in a table.

+ Views can represent a subset of the data contained in a table. Consequently, a view limit the degree of expose of the underlaying tables to the outer world.
+ A given user many have permission to query the view, While dened access to the rest of the base table.
+ Views can join and simplify multiple table into a Single Virtual tables
+ The view can be used to hide some of the coloumn from the table.
_______________________________________________________________________________________________________________________________

Q) What is Trigger ? 

  + Trigger is a code that associated with Insert,Update or Delete Operations. The code is executed automatically whenever the associated query is executed on table.
  + Triggers can be useful to maintain integrity in database.
    
   ##### PL/SQL on Triggers
    + Update Emp table such that if an updation is done in Dept_table then salary of all employee of the departments should be incremented by same amount

    **CREATE TRIGGER update_trig
      AFTER UPDATE ON Dept
      for EACH ROW
        DECLARE
          CUROSR emp_cur IS SELECT * FROM EMP;
          BEGIN
            FOR i IN emp_cur loop
              IF i.dept.No new.dept_no then
                DBMS_OUTPUT.PUT - LINE(i.emp_no)
                update emp
                  SET sal = i.sal + 100;
                  where emp-no=i-emp-no;
              ENDif
            END
          END
    **
_______________________________________________________________________________________
**What is StoredProcedure**
  + A StoredProcedur is like a function that contains set of operatios compoled toghather.
  + It contains set of operation that are commonly used in an application to do some common database taks.
  
  **
    PL/SQL Store Procedure
    
    CREATE/REPLACE
    PROCEDURE Procedure_name[(parameter1,parameter2)]
    is
    [declaration-secion]
    Begin
      [execution-Section]
      [EXCEPTION]
    END

    CREATE OR REPLACE PROCEDURE  insertuser(id IN NUMBER,name IN VARCHAR)
    IS
    BEGING
      INSERT INTO USER VALUES(id,name);
    END  

  PL/SQL Store Procedure

  CREATE OR REPLACE FUNCTION ADD(n1 in Number, n2 in number)
  return number

  IS N3 NUMBER(8);
  BEGIN
    n3=n1+n2;
  END;
  
  **
  
  #### Difference between S.P and Triggers

  +Triggers can't be called directly.
  + They can only associate d with queries.
_______________________________________________________________________________________

#### INDEX
  + Is a data structure that improves the speed of data retrival operations on a data base
  **Create Index Index_name ON Tablename(coloumn1,column2, column2)**

##### What are clustered and non-clustered index ?
  **Cluster** Clustered index is the index according to which data physically stored on disk. therefore, only one clustered index can e created on agiven database table.
 **Non-clustered index**  Don't define physical ordiering of data, but logical ordering. Typically, a tree is created whose leaf point to disk records
      B-Tree or B+ Tree are used for this purpose

   **Types of indexes**
   
       + Clustered
       + Nonclustered
       + Unique
       + Full_text
  ___________________________________________________________________________________________________________________

  ###CURSOR
    + Cursors are database objects used to maniplate data in aset on a row-by-row basis.
    + We can also fetch cursor rows and perform operation on them in a loop just like using looping mechanisam.

    **
      DECLEARE cussor_name CURSOR FOR
        SELECT COLOUMN1,COLOUMN1,....  FROM TABLE_NAME WHERE CONDITION
      OPEN CURSOR_NAME 
        FETCH NEXT FROM CURSOR_NAME INTO var1,var2,
         WHILE @@ FETCH_STATUS = 0
          BEGING
            FETCH NEXT CURSOR_NAME INTO VAR1,VAR2;
          END
      CLOSE CURSOR_NAME
    **
______________________________________________________________________________________________________________________________
### SUBQUERY
  + is a query with in another query, also known as a nested query
  + A subquery is used to return data that will be used in the main query as a conditionto further restrict the data to be restricted.
  + Subqueries are used with the SELECT,INSERT,UPDATE,DELETE.

    #### EXISS
    
      + Operator is used to test for the existance of any record in a subquery.
      + The exists operator returns true if subquery return one or more reqcords.
__________________________________________________________________________________________________________________________________

### correlated subquery
   
   + A correlated subquery is a subquery in SQL that references columns from its containing (outer) query, causing it to be evaluated once for each row processed by the outer query.
   +  Because it uses values from the outer query, the subquery's results can change for each row, and this repeated execution can significantly impact query performance.
   +  Correlated subqueries are used to compare data across rows, such as finding employees whose salary is higher than their department's average 
    
 **
   SELECT
      employee_id,
      salary,
      department_id
  FROM
      employees AS outer_emp
  WHERE
      salary > (
          SELECT AVG(salary)
          FROM employees AS inner_emp
          WHERE inner_emp.department_id = outer_emp.department_id
      );
 **
  
  







