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

#### Inner join (get the result by combining two tables based on the join-condition)
Gets the starttime of bookings of David Farrell and join the two tables based on the id
```sql
select starttime from table1 bookings
	inner join table2 members
	on members.id = bookings.id
		where members.firstname = 'David'
		and members.surname = 'Farrell
```

Gets the booking start time and facilities name that contains ```Tennis Court``` and starttime = ```2012-09-21``` in ascending order
```sql
select bookings.starttime as start, facilities.name as name 
	from table_bookings bookings
	inner join table_facilities facilities 
	on facilities.facid = bookings.facid
		where facilities.name like '%Tennis Court%' and 
		bookings.starttime >= '2012-09-21' and 
		bookings.starttime < '2012-09-22'
		order by start asc
```

Inner join on same table (members who have recommended other members)
```sql
select distinct mems1.firstname, mems1.surname from members mems1
	inner join members mems2
	on mems1.memid = mems2.recommendedby
	order by mems1.surname asc
```

Inner join on same table (members who have recommended other members)
```sql
select distinct mems1.firstname, mems1.surname from members mems1
	inner join members mems2
	on mems1.memid = mems2.recommendedby
	order by mems1.surname asc
```

Left outer join on the same table (to get the members who have recommended other members along with members who do not have any recommendations)
```sql
select mems1.firstname as memfname, mems1.surname as memsname,
	mems2.firstname as recfname, mems2.surname as recsname  from members mems1
	left outer join members mems2
	on mems2.memid = mems1.recommendedby
	order by memsname, memfname asc
```

#### Multiple inner joins
List of members who have booked tennis court facility
```sql
select distinct (mem.firstname || ' ' || mem.surname) as member, fcl.name as facility from members mem
	inner join bookings bookings
	on mem.memid = bookings.memid
		inner join facilities fcl 
		on bookings.facid = fcl.facid
			where fcl.name like '%Tennis Court%'
			order by member, facility asc
```

List of members who have booked any facility on ```2012-09-14``` and their cost is more than $30
```sql
select (mem.firstname || ' ' || mem.surname) as member, 
	fcl.name as facility,
	case when (mem.memid = 0)
		then bkn.slots * fcl.guestcost
		else bkn.slots * fcl.membercost
		end as cost
from cd.members mem
	inner join cd.bookings bkn
	on mem.memid = bkn.memid
		inner join cd.facilities fcl
		on bkn.facid = fcl.facid
			where bkn.starttime >= '2012-09-14' and bkn.starttime < '2012-09-15' and (
				(mem.memid = 0 and bkn.slots * fcl.guestcost > 30) or
			  	(mem.memid != 0 and bkn.slots * fcl.membercost > 30)
			)
			order by cost desc
```

#### Sub queries
List of members who have recommended other members
```sql
select distinct (mem.firstname || ' ' || mem.surname) as member, (
  	select (rec.firstname || ' ' || rec.surname) from members rec 
  		where mem.recommendedby = rec.memid
  ) from members as mem
  	order by member asc
```

List of members who have booked any facility on ```2012-09-14``` and their cost is more than $30
```sql
select member, facility, cost from (
	select
  		(mem.firstname || ' ' || mem.surname) as member,
		fsc.name as facility,
  		case when mem.memid = 0
  			then bkn.slots * fsc.guestcost
  			else bkn.slots * fsc.membercost
  		end as cost
  		from members mem
		inner join bookings bkn
  		on mem.memid = bkn.memid
  			inner join facilities fsc
  			on bkn.facid = fsc.facid
  				where bkn.starttime >= '2012-09-14' and
  					bkn.starttime < '2012-09-15'
) as bookings
	where cost > 30
	order by cost desc
```

#### Insert queries
Simple insert
```sql
insert into facilities
    (facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
    values(9, 'Spa', 20, 30, 100000, 800)
```

Multiple insert
```sql
insert into facilities
    (facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
    values
        (9, 'Spa', 20, 30, 100000, 800),
        (10, 'Squash Court 2', 3.5, 17.5, 5000, 80);
```

Insert calculated data
```sql
insert into facilities
    (facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
    select (select max(facid) from facilities)+1, 'Spa', 20, 30, 100000, 800;
```

#### Update Query
Simple update
```sql
update facilities fcl
    set initialoutlay=10000
    where fcl.name like '%Tennis Court 2%'
```

Update multiple columns
```sql
update facilities fcl
    set membercost=6, guestcost=30
    where fcl.name like '%Tennis Court%'
```

Update calculated data
```sql
update facilities fcl1
    set membercost=fcl2.membercost + (fcl1.membercost * 0.1),
        guestcost=fcl2.guestcost + (fcl1.guestcost * 0.1)
    from (select * from facilities where facid = 0) as fcl2
    where fcl1.facid = 1
```

#### Delete Query
Simple delete
```sql
delete from bookings
```

Delete conditionally
```sql
delete from members where memid = 37
```

Delete based on calculated data
```sql
delete from members where memid not in (select memid from bookings)
```
