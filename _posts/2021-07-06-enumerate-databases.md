---
title: Enumerate Databases
layout: post
author: hyprcub
tags: cheatsheet
---
In many cases you'll need to dump the content of some databases. Here is how to do it for three common database types.

Also, for privilege escalation purpose or just information gathering, you may need to read the content of some files using different privileges (SUID or `sudo`).

# References


# MySQL

## Enumerate Databases[^1]

| Command                  | Description                          |
| ---                      | ---                                  |
| `show databases;`        | list databases                       |
| `use [database];`        | select a database                    |
| `show tables;`           | list tables in the selected database |
| `select * from [table];` | show table content                   |

## Read a File

`show load_file('[FILENAME]') as result;`

# PostgreSQL

## Enumerate Databases[^2]

| Command                  | Description                          |
| ---                      | ---                                  |
| `\list`                  | list databases                       |
| `\c [database]`          | select a database                    |
| `\d`                     | list tables in the selected database |
| `select * from [table];` | show table content                   |

## Read a File

| Command                        | Description                      |
| ---                            | ---                              |
| `create table demo(t text);`   | create a table                   |
| `copy demo from '[FILENAME]';` | insert file content in the table |
| `select * from demo;`          | show the content                 |

# SQLite[^3]

## Enumerate Databases

| Command                  | Description                 |
| ---                      | ---                         |
| `sqlite3 [file.db]`      | open the database `file.db` |
| `.tables`                | show tables                 |
| `select * from [table];` | show table content          |
|                          |                             |

## Read a File

| Command                         | Description                             |
| ---                             | ---                                     |
| `sqlite3 mydb.db`               | create or use an existing database file |
| `create table mytable(t TEXT);` | create a table with one column          |
| `.import [filename] mytable;`   | import the file content                 |
| `select * from mytable`         | show the content of the import          |


[^1]: [https://book.hacktricks.xyz/pentesting/pentesting-mysql](https://book.hacktricks.xyz/pentesting/pentesting-mysql)
[^2]: [https://www.postgresqltutorial.com/postgresql-show-tables](https://www.postgresqltutorial.com/postgresql-show-tables)
[^3]: [https://sqlite.org/cli.html](https://sqlite.org/cli.html)
