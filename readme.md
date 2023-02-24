## Basic queries for postgresql

#### Retrieve everything from a table
```sql
select * from table_name
```

#### Retrieve specific columns from a table
```sql
select name, age from table_name
```

#### Control which rows are retrieved
```sql
select * from table_name where age > 18

select sid, name, courses from table_name where age > 18 and studentfee < 5000
```

#### Basic string searches
```sql
select * from table_name where course like '%software%' 
```

#### Matching against multiple possible values (get records where age is either 18 or 20)
```sql
select * from table_name where studentage in (18,20)
```

#### Classify results into buckets (add extra column in the result based on your calculations)
```sql
select name, 
	case when (age > 18) then
		'adult'
	else
		'not an adult' 
	end	as classification
	from table_name
```

#### Working with dates (Retrive all users whose join date is after August 2012)
```sql
select firstname, joindate from table_name where joindate > '2012-08-31'
```

#### Removing duplicates, and ordering results
```sql
select distinct surname 
  from table_name 
order by surname asc 
limit 10
```

#### Combining results from multiple queries
```sql
select surname from table_name 
union 
select name from table_name
```

#### Simple aggregation (Max() or Min() functions to get the latest or oldest joindate)
```sql
select MAX(joindate) as latest from table_name
```

#### More aggregation (Max() or Min() functions to get other properties along with the latest or oldest joindate)
```sql
select firstname, surname , joindate from table_name 
  where joindate = (select max(joindate) from table_name)
```
