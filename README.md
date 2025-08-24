
Project Overview

Project Title: Library Management System  

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

Objectives

1. Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
3. CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.

Project Structure

CREATE DATABASE library_db;

drop table if exists books;
create table books(
	isbn varchar(50) primary key , 
	book_title varchar (100),
	category varchar(100),
	rental_price float,
	status varchar(20),
	author varchar (50),
	publisher varchar(100)
);

select*from books;

drop table if exists branch;
create table branch(
	branch_id varchar(10)primary key,
	manager_id  varchar(10),
	branch_address varchar(100),
	contact_no int 
	);
	
select*from branch;

drop table if exists employees;
create table employees(
	emp_id varchar(10) primary key,
	emp_name varchar(50),
	position varchar(10),
	salary int,
	branch_id varchar(10),
	foreign key(branch_id)references branch(branch_id) 
);

select*from employees;

drop table if exists issued_status;
create table issued_status(
	issued_id varchar(10) primary key,
	issued_member_id varchar(10),
	issued_book_name varchar(50),
	issued_date date,
	issued_book_isbn varchar(50),
	issued_emp_id varchar(10),
	foreign key(issued_emp_id) references employees(emp_id),
	foreign key(issued_member_id) references members(member_id),
	foreign key(issued_book_isbn) references books(isbn)
	);
	
select*from issued_status;

drop table if exists members;
create table members(
	member_id varchar(10) primary key,
	member_name varchar(50),
	member_address varchar(100),
	reg_date date
);

select*from members;

drop table if exists return_status;
create table return_status(
	return_id varchar(10),
	issued_id varchar(10),
	return_book_name varchar(50),
	return_date date,
	return_book_isbn varchar(50),
	foreign key(return_book_isbn) REFERENCES books(isbn)
);

select*from  return_status ;


SELECT * FROM books;
SELECT * FROM branch;
SELECT * FROM employees;
SELECT * FROM issued_status;
SELECT * FROM return_status;
SELECT * FROM members;

-- Project Task

-- Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic',
--6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

insert into books(isbn,book_title,category,rental_price,status,author,publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic',6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;

-- Task 2: Update an Existing Member's Address
update members
set member_address = '125 Main St'
WHERE member_id = 'C101';
SELECT * FROM members;

-- Task 3: Delete a Record from the Issued Status Table 
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
delete from issued_status
where issued_id = 'IS121' ;

-- Task 4: Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by
--the employee with emp_id = 'E101'.
select issued_book_name,issued_emp_id from issued_status
where issued_emp_id = 'E101';

-- Task 5: List Members Who Have Issued More Than One Book
--Objective: Use GROUP BY to find members who have issued more than one book.
SELECT ist.issued_emp_id, e.emp_name 
FROM issued_status ist
JOIN
employees e
ON e.emp_id = ist.issued_emp_id
GROUP BY 1, 2
HAVING COUNT(ist.issued_id) > 1;


-- Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results 
-- each book and total book_issued_cnt**
drop table if exists book_cnts;
CREATE TABLE book_cnts
AS    
SELECT 
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) as no_issued
FROM books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn = b.isbn
GROUP BY  b.isbn,
    b.book_title;
SELECT * FROM book_cnts;

-- Task 7. Retrieve All Books in a Specific Category:
select * from books
where category = 'Classic';

-- Task 8: Find Total Rental Income by Category:
select category , sum(rental_price) as total_income
from books
group by category;

--Task 9: List Members Who Registered in the Last 180 Days:
select* from members
where reg_date>=current_date-interval '180days';

-- task 10 List Employees with Their Branch Manager's Name and their branch details:
select e1.emp_name , e1.position , e1.salary , b.*
from employees e1
join
branch b
on b.branch_id = e1.branch_id
join
employees e2
on b.manager_id = e2.emp_id;

 --Task 11. Create a Table of Books with Rental Price Above a Certain Threshold.
create table expensive_books as 
select * from books
where rental_price > 7.00;

 select*from expensive_books;
 
 --Task 12: Retrieve the List of Books Not Yet Returned
SELECT * FROM 
issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;

--Task 13: Identify Members with Overdue Books 
--Write a query to identify members who have overdue books (assume a 30-day return period).
--Display the member's_id, member's name, book title, issue date, and days overdue.
select 
iss.issued_member_id , m.member_name , b.book_title , iss.issued_date , r.return_date,
(current_date - issued_date) - 30 as overdue_date
from
issued_status as iss
join
members as m 
on iss.issued_member_id = m.member_id
join
books as b 
on iss.issued_book_isbn = b.isbn
left join
return_status as r
ON r.issued_id = iss.issued_id
WHERE 
r.return_date is null 
and 
(current_date - iss.issued_date) >30
order by 1;


--Task 14: CTAS: Create a Table of Active Members
--Use the CREATE TABLE AS (CTAS) statement to create a new table active_members 
--containing members who have issued at least one book in the last 2 months.
create table Active_members as 
select distinct  m.member_id , m. member_name , iss.issued_book_name 
 from members as m 
 join
 issued_status as iss 
 on m.member_id = iss.issued_member_id
 where iss.issued_date >=( current_date-interval '2 months ');

--Task 15: Find Employees with the Most Book Issues Processed
--Write a query to find the top 3 employees who have processed the most book issues
--Display the employee name, number of books processed, and their branch.
select distinct e.emp_id , e.emp_name, b.branch_id, COUNT(i.issued_id) as no_book_issued 
from employees e 
join 
issued_status i
on e.emp_id = i.issued_emp_id
join
branch b 
on e.branch_id = b.branch_id
group by e.emp_id ,e.emp_name,b.branch_id
order by  no_book_issued desc 
limit 3 ;


## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.


