---
id: 7026
title: 'Book review (and recommendation): Learn T-SQL Querying'
date: '2022-02-13T18:02:41+01:00'
author: 'Marco Siccardi'
excerpt: 'I just finished reading "Learn T-SQL Querying" written by two Microsoft employees (Pedro Lopes and Pam Lahoud) who are working on the database engine of Microsoft''s SQL Server. They are providing a very deep look into writing efficient queries and tools that can make the life of a developer easier.'
layout: post
permalink: /book-review-and-recommendation-learn-t-sql-querying/
image: /assets/img/2022/02/LearnTSQLTitle.png
categories:
    - Books
tags:
    - Azure
    - Book
    - 'book review'
    - 'free ebook'
    - optimize
    - query
    - Review
    - SQL
    - 'SQL Server'
    - Troubleshooting
    - TSQL
---

### The Fundamentals

In the first section of the book, the authors give us an overview about the anatomy of a query (going very deep) and how SQL Server (also on Azure) processes queries. They also explain how SQL Server optimizes queries and how different versions of SQL Server are processing them differently, which can result in different performances for complex queries.

### <span id="dos-and-donts">Dos and Don’ts</span>

The second section of the book is guiding us through how Query Execution plans work and how they can help to write more efficient SQL queries. The section has a bunch of tips that developers can use in their everyday life with databases. The perhaps most important part in this section are the two chapters about T-SQL antipatterns. This is the section where I learned the most throughout the book.

### Troubleshooting and tools

The last section of the book shows a lot of troubleshooting techniques and how to use them properly. Microsoft’s SQL Server Management Tools itself comes with a bunch of such tools, and the book helps not only to find them, but also to use them correctly. On top, there are also references to some Open Source tools that can be helpful at times. The book closes with the Query Tuning Assistant, which is the recommended tool to perform SQL server updates.

### Conclusion

I got the book because my current job requires me to write efficient T-SQL code for the interfaces I am developing. The book already helped me during the reading time to better understand what I am doing and how I can optimize my own queries. It will be one of my reference books in future when it comes to troubleshooting and query performance with SQL. Long story short, if you are working regularly with Microsoft SQL Server (also on Azure), you should have this book in your (digital) bookshelf.

### Book metadata

- Published May 2019
- ISBN 9781789348811
- [PacktPub Free E-book](https://www.packtpub.com/free-ebook/Learn-T-SQL-Querying/9781789348811)