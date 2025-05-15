# MySQL Data Cleaning Project  

## Overview  
This project demonstrates how to clean and manage a MySQL database efficiently. It includes exporting an entire database and individual tables, troubleshooting MySQL commands, and organizing structured SQL files.  

## Project Structure  
The following files are included in this repository:  
- `database_dump.sql` → Full MySQL database export  
- `layoffs.sql` → Raw data in layoffs table 
- `layoffs_alter.sql` → same data as layoffs table kept this duplicate for safe side 
- `layoffs_rownum_added.sql` → Data cleaning is done for this table 

## Installation & Setup  
### Prerequisites  
Ensure you have the following installed:  
- **MySQL Server** (download [here](https://dev.mysql.com/downloads/installer/))  
- **Git** (download [here](https://git-scm.com/downloads))  
- A MySQL-compatible client such as **MySQL Workbench**  

### Importing the Database  
To import the `database_dump.sql` file into MySQL:  
```bash
mysql -u root -p database_name < database_dump.sql