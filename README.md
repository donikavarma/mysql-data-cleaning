-- DATA CLEANING

select * from layoffs;

#Firstly creating a duplicate table same as layoffs - this might to do changes to the raw data in real time

create table layoffs_alter like layoffs;
insert layoffs_alter
select * from layoffs;

select * from layoffs_alter;
describe layoffs_alter;

-- 1.Remove Duplicates

with duplicate_cte as
(
	select *,row_number() 
	over( 
			partition by company,location,industry,total_laid_off,percentage_laid_off,
			`date`,stage,country,funds_raised_millions
		) as row_num
	from layoffs_alter
)
delete from duplicate_cte #we cant directly delete here this not gonna work
where row_num>1;

#we need to create another table with the extra column row_num
#to create this i just gone to layoffs_alter table , right click,copy to clipboard

CREATE TABLE `layoffs_rownum_added` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci; #run this table will be created

insert into layoffs_rownum_added
select *,row_number() 
	over( 
			partition by company,location,industry,total_laid_off,percentage_laid_off,
			`date`,stage,country,funds_raised_millions
		) as row_num
	from layoffs_alter;

select*from layoffs_rownum_added
where row_num >1; #i checked here for duplicates

SET SQL_SAFE_UPDATES = 0;
delete from layoffs_rownum_added
where row_num >1; #deleted here 

select * from layoffs_rownum_added;

-- 2. remove any extra columns

alter table layoffs_rownum_added
drop column row_num;

describe layoffs_rownum_added;

-- 3.standardisation of data - checking every row to keep clean data here

#trim
select company,trim(company) from layoffs_rownum_added; #white spaces removed
 
update layoffs_rownum_added
set company = trim(company); #updated the table without white space
 
 select country,trim(trailing '.' from country) as no_fullstop 
 from layoffs_rownum_added; #full stop removed
 
 update layoffs_rownum_added
 set country = trim(trailing '.' from country);
 
 select country from layoffs_rownum_added;
 
#removing different names of same department

select distinct(industry) from layoffs_rownum_added;
 
update layoffs_rownum_added
set industry ="Crypto"
where industry like "Crypto%";

#changing the format of a column

select `date`,
str_to_date(`date` ,'%m/%d/%Y') as format_date from layoffs_rownum_added; #check the date format correctly here
update layoffs_rownum_added
set `date` = str_to_date(`date` ,'%m/%d/%Y');
  
#we changed the data type of date here - recommended to do on duplicate table

alter table layoffs_rownum_added
modify column `date` date; #check on left side for type
  
select `date` from layoffs_rownum_added;
  
-- 4.removing null data or blank data

select * from layoffs_rownum_added 
where total_laid_off is null;

select industry from layoffs_rownum_added
where industry is null or industry = "";

#here we are updating the empty spaces

update layoffs_rownum_added
set industry = null
where industry =""; -- turned everything to null

select t1.industry,t2.industry from layoffs_rownum_added as t1
join layoffs_rownum_added t2
on t1.company = t2.company
and t1.location = t2.location
where t1.industry is null  and t2.industry is not null ; #checking null and not null data

update layoffs_rownum_added as t1
join layoffs_rownum_added as t2
on t1.company = t2.company
set t1.industry = t2.industry
where t1.industry is null
and t2.industry is not null; #changing null data with not null data

select * from layoffs_rownum_added where company="Airbnb"; #run this both industry will be travel one  used to be null

#cleaning null data

delete from layoffs_rownum_added
where total_laid_off is null 
and percentage_laid_off is null;

select * from layoffs_rownum_added;
