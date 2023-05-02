Download Link: https://assignmentchef.com/product/solved-comp9311-project-1-sql
<br>
<h1>1. Aims</h1>

This project aims to give you practice in

<ul>

 <li>reading and understanding a moderately large relational schema (MyMyUNSW).</li>

 <li>implementing SQL queries and views to satisfy requests for information.</li>

 <li>The goal is to build some useful data access operations on the MyMyUNSW database. A theme of this project is “dirty data”. As I was building the database, using a collection of reports from UNSW’s information systems and the database for the academic proposal system (MAPPS), I discovered that there were some inconsistencies in parts of the data (e.g. duplicate entries in the table for UNSW buildings, or students who were mentioned in the student data, but had no enrolment records, and, worse, enrolment records with marks and grades for students who did not exist in the student data). I removed most of these problems as I discovered them, but no doubt missed some. Some of the exercises below aim to uncover such anomalies; please explore the database and let me know if you find other anomalies.</li>

</ul>

<ol start="2">

 <li><strong>How to do this project: </strong></li>

</ol>

<ul>

 <li>Read this specification carefully and completely</li>

 <li>Familiarize yourself with the database <strong>schema</strong> (description, SQL schema,  summary)</li>

 <li>Make a private directory for this project, and put a copy of the <strong>sql</strong> template there</li>

 <li>You <strong>must</strong> use the create statements in <strong>sql</strong> when defining your solutions</li>

 <li>Look at the expected outputs in the expected_qX tables loaded as part of the <strong>sql</strong> file</li>

 <li>Solve each of the problems below, and put your completed solutions into <strong>sql</strong></li>

 <li>Check that your solution is correct by verifying against the example outputs and by using the check_qX() functions</li>

 <li>Test that your <strong>sql</strong> file will load <em>without error</em> into a database containing just the original MyMyUNSW data</li>

 <li>Double-check that your <strong>sql</strong> file loads in a <em>single pass</em> into a database containing just the original MyMyUNSW data</li>

 <li>Submit the project via give</li>

 <li>PLpgSQL functions are <strong>not</strong> allowed to use in this project</li>

 <li>For each question, you must output result within 120 seconds on Grieg server.</li>

</ul>

<h1>3. Introduction</h1>

All Universities require a significant information infrastructure in order to manage their affairs. This typically involves a large commercial DBMS installation. UNSW’s student information system sits behind the MyUNSW web site. MyUNSW provides an interface to a PeopleSoft enterprise management system with an underlying Oracle database. This back-end system (Peoplesoft/Oracle) is often called NSS.

UNSW has spent a considerable amount of money ($80M+) on the MyUNSW/NSS system, and it handles much of the educational administration plausibly well. Most people gripe about the quality of the MyUNSW interface, but the system does allow you to carry out most basic enrolment tasks online.

Despite its successes, MyUNSW/NSS still has a number of deficiencies, including:

<ul>

 <li>no waiting lists for course or class enrolment</li>

 <li>no representation for degree program structures</li>

 <li>poor integration with the UNSW Online Handbook</li>

</ul>

The first point is inconvenient, since it means that enrolment into a full course or class becomes a sequence of trial-and-error attempts, hoping that somebody has dropped out just before you attempt to enrol and that no-one else has grabbed the available spot.

The second point prevents MyUNSW/NSS from being used for three important operations that would be extremely helpful to students in managing their enrolment:

<ul>

 <li>finding out how far they have progressed through their degree program, and what remains to be completed</li>

 <li>checking what are their enrolment options for next semester (e.g. get a list of suggested courses)</li>

 <li>determining when they have completed all of the requirements of their degree program and are eligible to graduate</li>

</ul>

NSS contains data about student, courses, classes, pre-requisites, quotas, etc. but does not contain any representation of UNSW’s degree program structures. Without such information in the NSS database, it is not possible to do any of the above three. So, in 2007 the COMP9311 class devised a data model that could represent program requirements and rules for UNSW degrees. This was built on top of an existing schema that represented all of the core NSS data (students, staff, courses, classes, etc.). The enhanced data model was named the MyMyUNSW schema.

The MyMyUNSW database includes information that encompasses the functionality of NSS, the UNSW Online Handbook, and the CATS (room allocation) database. The MyMyUNSW data model, schema and database are described in a separate document.

<h1>4. Setting Up</h1>

To install the MyMyUNSW database under your Grieg server, simply run the following two commands:

$ <strong>createdb proj1</strong>

$ <strong>psql proj1 -f /home/cs9311/web/19T3/proj/proj1/mymyunsw.dump</strong>

If you’ve already set up PLpgSQL in your template1 database, you will get one error message as the database starts to load:

psql:mymyunsw.dump:<em>NN</em>: ERROR:  language “plpgsql” already exists

You can ignore this error message, but any other occurrence of ERROR during the load needs to be investigated.

If everything proceeds correctly, the load output should look something like:

SET

SET

SET

SET SET

psql:mymyunsw.dump:NN: ERROR:  language “plpgsql” already exists

… if PLpgSQL is not already defined,

… the above ERROR will be replaced by CREATE LANGUAGE

SET

SET

SET

CREATE TABLE

CREATE TABLE

… a whole bunch of these

CREATE TABLE

ALTER TABLE

ALTER TABLE

… a whole bunch of these

ALTER TABLE

Apart from possible messages relating to plpgsql, you should get no error messages. The database loading should take less than 60 seconds on Grieg, assuming that Grieg is not under heavy load. (If you leave your project until the last minute, loading the database on Grieg will be considerably slower, thus delaying your work even more. The solution: at least load the database Right Now, even if you don’t start using it for a while.) (Note that the mymyunsw.dump file is 50MB in size; copying it under your home directory or your /srvr directory is not a good idea).

If you have other large databases under your PostgreSQL server on Grieg or you have large files under your /srvr/YOU/ directory, it is possible that you will exhaust your Grieg disk quota. In particular, you will not be able to store two copies of the MyMyUNSW database under your Grieg server. The solution: remove any existing databases before loading your MyMyUNSW database.

If you are running PostgreSQL at home, you can download the files: mymyunsw.dump, proj1.sql to get you started. You can grab the check.sql separately, once it becomes available.

A useful thing to do initially is to get a feeling for what data is actually there. This may help you understand the schema better, and will make the descriptions of the exercises easier to understand.

Look at the schema. Ask some queries. Do it now.

Examples …

$ <strong>psql proj1</strong>

… PostgreSQL welcome stuff … proj1=# <strong>d</strong>

… look at the schema …

proj1=# <strong>select * from Students;</strong>

… look at the Students table …

proj1=# <strong>select p.unswid,p.name from People p join Students s on (p.id=s.id);</strong> … look at the names and UNSW ids of all students … proj1=# <strong>select p.unswid,p.name,s.phone from People p join Staff s on (p.id=s.id);</strong> … look at the names, staff ids, and phone #s of all staff …

proj1=# <strong>select count(*) from Course_Enrolments;</strong> … how many course enrolment records … proj1=# <strong>select * from dbpop();</strong> … how many records in all tables …

proj1=# <strong>select * from transcript(3197893);</strong> … transcript for student with ID 3197893 … proj1=# … etc. etc. etc.

proj1=# <strong>q </strong>




You will find that some tables (e.g. Books, Requirements, etc.) are currently unpopulated; their contents are not needed for this project. You will also find that there are a number of views and functions defined in the database (e.g. dbpop() and transcript() from above), which may or may not be useful in this project.

<strong>Summary on Getting Started </strong>

To set up your database for this project, run the following commands in the order supplied:

$ <strong>createdb  proj1</strong>

$ <strong>psql  proj1  -f  /home/cs9311/web/19T3/proj/proj1/mymyunsw.dump</strong>  $ <strong>psql  proj1</strong>

… run some checks to make sure the database is ok

$ <strong>mkdir  <em>Project1Directory</em></strong>

… make a working directory for Project 1

$ <strong>cp  /home/cs9311/web/19T3/proj/proj1/proj1.sql  <em>Project1Directory </em></strong>




The only error messages produced by these commands should be those noted above. If you omit any of the steps, then things will not work as planned.

<strong>Notes </strong>

<strong>Read these</strong> before you start on the exercises:

<ul>

 <li>the marks reflect the relative difficulty/length of each question</li>

 <li>use the supplied <strong>sql</strong> template file for your work</li>

 <li>you may define as many additional functions and views as you need, provided that (a) the definitions in proj1.sql are preserved, (b) you follow the requirements in each question on what you are allowed to define</li>

 <li>make sure that your queries would work on any instance of the MyMyUNSW schema; don’t customize them to work just on this database; we may test them on a different database instance</li>

 <li>do not assume that any query will return just a single result; even if it phrased as “most” or</li>

</ul>

“biggest”, there may be two or more equally “big” instances in the database

<ul>

 <li>when queries ask for people’s names, use the name field; it’s there precisely to produce displayable names</li>

 <li>when queries ask for student ID, use the unswid field; the People.id field is an internal numeric key and of no interest to anyone outside the database</li>

 <li>unless specifically mentioned in the exercise, the order of tuples in the result does not matter; it can always be adjusted using order by. In fact, our check.sql will order your results automatically for comparison.</li>

 <li>the precise formatting of fields within a result tuple <strong>does</strong> matter; e.g. if you convert a number to a string using to_char it may no longer match a numeric field containing the same value, even though the two fields may look similar</li>

 <li>develop queries in stages; make sure that any sub-queries or sub-joins that you’re using actually work correctly before using them in the query for the final view/function</li>

 <li>You can define either SQL views OR SQL functions to answer the following questions.</li>

 <li>If you meet with error saying something like “cannot change name of view column”, you can drop the view you just created by using command “<strong>drop view VIEWNAME cascade;</strong>” then create your new view again.</li>

</ul>

Each question is presented with a brief description of what’s required. If you want the full details of the expected output, take a look at the expected_qX tables supplied in the checking script.

<h1>5. Tasks</h1>

To facilitate the semi-auto marking, please pack all your SQL solutions into view or function as defined in each problem (see details from the solution template we provided).

<h2>Q1</h2>

Define an SQL view Q1(unswid,longname) that gives the distinct room id and name of any room that is Air-conditioned (refers to the facilities.description). The view should return the following details about each room:

<ul>

 <li>unswid should be taken from unswid field.</li>

 <li>longname should be taken from longname field.</li>

</ul>

<h2>Q2</h2>

Define a SQL view Q2(unswid,name) that displays unswid and name of all the distinct staff who taught the student Hemma Margareta. The view should return the following details about each staff:

<ul>

 <li>unswid should be taken from unswid field.</li>

 <li>name should be taken from name field.</li>

</ul>

<h2>Q3</h2>

Define a SQL view Q3(unswid,name) that gives all the distinct international students who enrolled in COMP9311 and COMP9024 in the same semester and got HD

(Course_enrolments.mark &gt;= 85) in both courses. The view should return the following details about each student:

<ul>

 <li>unswid should be taken from unswid field.</li>

 <li>name should be taken from name field.</li>

</ul>

<h2>Q4</h2>

Define a SQL view Q4(num_student) that gives the number of distinct students who get more HD (Course_enrolments.mark &gt;= 85) than average student. For example, if on average a student gets 3 HDs, we only count students who get more than 3 HDs.

<strong>Note: </strong>

<ul>

 <li>when calculating the average, only consider students who have at least one <strong>not null</strong> mark.</li>

</ul>

<h2>Q5</h2>

Define a SQL view Q5(code,name,semester) that displays the course(s) with the lowest maximum mark in each semester. The view should return the following details about each course:

<ul>

 <li>code should be taken from code field.</li>

 <li>name should be taken from name field.</li>

 <li>semester should be taken from name field.</li>

</ul>

<strong>Note: </strong>

<ul>

 <li>only consider valid courses which have at least <strong>20</strong> <strong>not null</strong> mark records.</li>

 <li>if several courses have the same lowest maximum mark in one semester, return all of them.</li>

 <li>skip the semester with no valid course.</li>

</ul>

<h2>Q6</h2>

<ul>

 <li>Define SQL view Q6(num), which gives the number of distinct local students enrolling in 10S1 in Management stream but never enrolling in any course offered by</li>

</ul>

Faculty of Engineering.

<strong>Note:</strong>

<ul>

 <li>the student IDs are the UNSW ids (i.e. student numbers) defined in the unswid field.</li>

 <li>Do not count duplicate records.</li>

</ul>

<h2>Q7</h2>

Database Systems course admins would like to know the average mark of each semester. Define a SQL view Q7(year, term, average_mark) to help them monitor the average mark each semester. Each tuple in the view should contain the following:

<ul>

 <li>the year (year) of the semester</li>

 <li>the term (term)</li>

 <li>the average mark of students enrolled in this course this semester as numeric(4,2)</li>

</ul>

Database Systems has value ‘Database Systems’ in the Subjects.name field. You can find the information about all the course offerings for a given subject from Courses. You should calculate the average mark of enrolled students for a course offering from the table Course_enrolments.

<strong>Note</strong>:

<ul>

 <li>There are two subjects that share the same name “Database Systems”, and we do not distinguish them in this question. In consequence, you may find more than one course for a single semester. In such case, there is no student enrolling in more than one course.</li>

 <li>When calculating the average marks, only consider <strong>not null</strong> mark records.</li>

 <li>Only consider the semesters which have ‘Database Systems’.</li>

</ul>




<h2>Q8</h2>

The head of school would like to know the performance of students in a set of CSE subjects. A subject in this set has two properties: (1) its subject code must start with “COMP93”, and (2) it must be offered in every major semester (i.e., S1 and S2) from 2004 (inclusive) to 2013 (inclusive). The head of school requests a list of students who failed every subject in the set. We say a student fails a subject if he/she had received a <strong>not null</strong> mark &lt; 50 for at least one course offering of the subject. Define a SQL view Q8(zid, name) for the head of school. Each tuple in the view should contain the following:

<ul>

 <li>zid (‘z’ concatenating with unswid field)</li>

 <li>student name (taken from the name field)</li>

</ul>

<strong>Note:</strong>

<ul>

 <li>For a given subject, the number of course offerings that a student can enroll in is at least one. For example, one may fail the course offering of a subject in 03S1, and then re-enroll in another course offering of the same subject in 04S1.</li>

 <li>Assume there are two subjects in the set, A and B. We only count students who failed both A and B, but not students who failed either A or B.</li>

</ul>




<h2>Q9</h2>

Define SQL view Q9(unswid,name)that gives all the distinct students who are satisfied the following conditions in one program:

<ul>

 <li>enroll a program in BSc (refer to abbrev)</li>

 <li>must pass at least one course in the program in semester 2010 S2.</li>

 <li><strong>average mark</strong> &gt;= 80. <strong>Average mark </strong>means the average mark of all courses a student has passed before 2011(exclusive) in the program.</li>

 <li>the total UOC (refer to uoc) earned in the program should be no less than the required UOC of the program (refer to programs.uoc). A student can only earn the UOC of the courses he/she passed.</li>

</ul>

The view should return the following details about each student:

<ul>

 <li>unswid should be taken from unswid field.</li>

 <li>name should be taken from name field. <strong>Note:</strong></li>

 <li>to pass a course, a student must get at least 50 in that course</li>

</ul>

(Course_enrolments.mark &gt;= 50).

<ul>

 <li>if a student has enrolled into several different programs, you need to calculate the UOC and average mark separately according to different programs. A course belongs to a program if this student enrolls into course and program in a same semester (refer to id).</li>

</ul>




<h2>Q10</h2>

The university is interested in the Lecture Theatre usage status in 2011 S1. So please define SQL view Q10(unswid,longname,num,rank), which gives the distinct room id and name of all the Lecture Theatre with the number of distinct classes that use this theatre and the rank. Theatre rankings are ordered by num from highest to lowest. If there are multiple theatres with the same num, they have the same rank. The view should return the following details about each theatre:

<ul>

 <li>unswid should be taken from unswid field.</li>

 <li>longname should be taken from longname field.</li>

 <li>num counts the total number of distinct classes that use this theatre.</li>

 <li>rank records the rank of the number of distinct classes that use this theatre.</li>

</ul>

<strong>Note: </strong>

<ul>

 <li>if there is no class using a theatre, the num would be 0.</li>

 <li>the ranking is with gaps. i.e., if there are 2 theatres ranked as first, the third theatre will be ranked as third.</li>

</ul>

The proj1.sql file should contain answers to all of the exercises for this project. It should be completely self-contained and able to load in a single pass, so that it can be auto-tested as follows:

<ul>

 <li>a fresh copy of the MyMyUNSW database will be created (using the schema from mymyunsw.dump)</li>

 <li>the data in this database may be <strong>different</strong> from the database that you’re using for testing</li>

 <li>a new check.sql file may be loaded (with expected results appropriate for the database)</li>

 <li>the contents of your proj1.sql file will be loaded</li>

 <li>each checking function will be executed, and the results recorded</li>

</ul>

Before you submit your solution, you should check that it will load correctly for testing by using something like the following operations:

$ <strong>dropdb proj1</strong>                  … remove any existing DB

$ <strong>createdb proj1</strong>                … create an empty database

$ <strong>psql proj1 -f /home/cs9311/web/19T3/proj/proj1/mymyunsw.dump</strong>    … load the MyMyUNSW schema and data

$ <strong>psql proj1 -f /home/cs9311/web/19T3/proj/proj1/check.sql</strong>  … load the checking code

$ <strong>psql proj1 -f proj1.sql</strong>        … load your solution

$ <strong>psql proj1</strong>

<table width="403">

 <tbody>

  <tr>

   <td width="183">proj1=#<strong> select check_q1();</strong>…</td>

   <td width="219">… check your solution to question1</td>

  </tr>

  <tr>

   <td width="183">proj1=# <strong>select check_q5();</strong>…</td>

   <td width="219">… check your solution to question5</td>

  </tr>

  <tr>

   <td width="183">proj1=# <strong>select check_q10();</strong></td>

   <td width="219">… check your solution to question10</td>

  </tr>

  <tr>

   <td width="183">proj1=#<strong> select check_all(); </strong></td>

   <td width="219">… check all your solutions</td>

  </tr>

 </tbody>

</table>

Note: if your database contains any views or functions that are not available in a file somewhere, you should put them into a file before you drop the database.

If your code loads with errors, fix it and repeat the above until it does not.

You must ensure that your proj1.sql file will load correctly (i.e. it has no syntax errors and it contains all of your view definitions in the correct order). If I need to manually fix problems with your proj1.sql file in order to test it (e.g. change the order of some definitions), you will be fined via half of the mark penalty for each problem. In addition, make sure that your queries are reasonably efficient. For each question, you must output result within 120 seconds on Grieg server. This time restriction applies to the execution of the ‘select * from check_Qn()’ calls. For each question, you will be fined via half of the mark penalty if your solution cannot output results within 120 seconds.





