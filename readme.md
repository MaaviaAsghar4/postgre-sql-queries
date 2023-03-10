## Basic queries for postgresql

### All the queries below are taken from the following [site](https://pgexercises.com/)

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
from members mem
	inner join bookings bkn
	on mem.memid = bkn.memid
		inner join facilities fcl
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

#### Aggregation
Count the number of facilities
```sql
select count(*) from facilities
```

Count the number of expensive facilities
```sql
select count(*) from facilities
	where guestcost >= 10
```

Count the number of recommendations each member makes
```sql
select recommendedby, count(recommendedby) from members
	where recommendedby is not null
	group by recommendedby
	order by recommendedby
```

List the total slots booked per facility
```sql
select facid, sum(slots) as "Total Slots" from bookings
	group by facid
	order by facid
```

List the total slots booked per facility in a given month
```sql
select facid, sum(slots) as "Total Slots" from bookings
	where starttime >= '2012-09-01' and starttime < '2012-10-01'
	group by facid 
	order by "Total Slots"
```

List the total slots booked per facility per month
```sql
select facid, extract(MONTH from starttime) as "month", sum(slots) as "Total Slots" 
	from bookings
		where extract(YEAR from starttime) = '2012'
		group by facid, month
	order by facid, month
```

Find the count of members who have made at least one booking
```sql
select count(distinct memid) from bookings
```

List facilities with more than 1000 slots booked
```sql
select facid, sum(slots) as "Total Slots" from bookings 
	group by facid
	having sum(slots) > 1000
	order by facid
```

Find the total revenue of each facility
```sql
select fcl.name as name, 
	sum(bkn.slots * case when (bkn.memid = 0)
			then fcl.guestcost
			else fcl.membercost
		end) as revenue
	from bookings bkn
		inner join facilities fcl
		on bkn.facid = fcl.facid
	group by fcl.name
	order by revenue
```

Find facilities with a total revenue less than 1000
```sql
select name, revenue from (
	select fcl.name as name, sum(bkn.slots *
	  	case when (bkn.memid = 0)
				then fcl.guestcost
				else fcl.membercost
			end
	  ) as revenue from bookings bkn
  		inner join facilities fcl
  			on bkn.facid = fcl.facid
  			group by fcl.name
) as table_name
	where revenue < 1000
		order by revenue asc
```

Output the facility id that has the highest number of slots booked
```sql
select facid, sum(slots) as "Total slots" from bookings
	group by facid
		having sum(slots) = (
		  select max(sum2.totalslots) from (
			select sum(slots) as totalslots from bookings
				group by facid
		  ) as sum2
		)
```

List the total slots booked per facility per month
```sql
select facid, extract(month from starttime) as month, sum(slots) from bookings
	where extract(year from starttime) = '2012'
		group by rollup (facid, month)
		order by facid
```

List the total hours booked per named facility
```sql
select fcl.facid, fcl.name, trim(to_char((sum(bkn.slots)/2.0), '9999999999999999D99')) as "Total Hours" from facilities fcl
	inner join bookings bkn
	on bkn.facid = fcl.facid
		group by fcl.facid, fcl.name
		order by fcl.facid
```

List each member's first booking after September 1st 2012
```sql
select mem.surname, mem.firstname, mem.memid, min(bkn.starttime) from members mem
	inner join bookings bkn
	on bkn.memid = mem.memid
		where bkn.starttime > '2012-09-01'
		group by mem.memid, mem.surname, mem.firstname
		order by memid
```

Produce a list of member names, with each row containing the total member count
```sql
select count(*) over(), firstname, surname from members
	order by joindate
```


Produce a numbered list of members
```sql
select row_number() over(order by joindate), firstname, surname 
	from members
	order by joindate
```

Output the facility id that has the highest number of slots booked, again
```sql
select facid, total from (
	select facid, sum(slots) total, rank() over (order by sum(slots) desc) rank
  		from bookings
  		group by facid
) as ranked
	where rank = 1 
```

Rank members by (rounded) hours used
```sql
select mem.firstname, mem.surname, 
	((sum(bkn.slots)+10)/20)*10 as hours,
	rank() over (order by ((sum(bkn.slots)+10)/20)*10 desc) as rank
	from members mem
		inner join bookings bkn
			on mem.memid = bkn.memid
		group by mem.memid
order by rank, surname, firstname
```

Find the top three revenue generating facilities
```sql
select name, rank from (
	select fcl.name as name, rank() over (order by sum(case
				when memid = 0 then slots * fcl.guestcost
				else slots * membercost
			end) desc) as rank
		from bookings bkn
		inner join facilities fcl
			on bkn.facid = fcl.facid
		group by fcl.name
	) as subq
	where rank <= 3
order by rank
```

Classify facilities by value
```sql
select name, case when class=1 then 'high'
		when class=2 then 'average'
		else 'low'
		end revenue
	from (
		select fcl.name as name, ntile(3) over (order by sum(case
				when memid = 0 then slots * fcl.guestcost
				else slots * membercost
			end) desc) as class
		from bookings bkn
		inner join facilities fcl
			on bkn.facid = fcl.facid
		group by fcl.name
	) as subq
order by class, name
```

Calculate the payback time for each facility
```sql
select name, 
	(initialoutlay / (monthlyrevenue - monthlymaintenance)) as months 
from (
  select fcl.name as name, 
  	fcl.initialoutlay as initialoutlay,
  	fcl.monthlymaintenance as monthlymaintenance,
  	sum(case when memid = 0
		then slots * fcl.guestcost
		else slots * fcl.membercost
		end
	)/3 as monthlyrevenue
  from bookings bkn
  	inner join facilities fcl
  	on bkn.facid = fcl.facid
  group by fcl.facid
) as subq
order by name
```

Calculate a rolling average of total revenue
```sql
select dategen.date, 
	(
	  select sum(
			case when memid = 0 then slots * guestcost
			else slots * membercost
			end
		) as rev
	  from bookings bkn 
	  inner join facilities fcl
	  	on bkn.facid = fcl.facid
	  where bkn.starttime > dategen.date - interval '14 days'
	  	and bkn.starttime < dategen.date + interval '1 day'
	)/15 as revenue
	from (
		select cast(generate_series(timestamp '2012-08-01', 
								   '2012-08-31', '1 day') as date) as date  
	) as dategen
order by dategen.date
```

#### Date (Timestamp)
Get a timestamp from a date
```sql
select timestamp '2012-08-31 01:00:00'
```

Subtract timestamps from each other
```sql
select timestamp '2012-08-31 01:00:00' - timestamp '2012-07-30 01:00:00' as interval
```

Generate a list of all the dates in October 2012
```sql
select generate_series(timestamp '2012-10-01', timestamp '2012-10-31', '1 day') 
```

Get the day of the month from a timestamp
```sql
select extract(day from timestamp '2012-08-31')
```

Work out the number of seconds between timestamps
```sql
select extract(epoch from(timestamp '2012-09-02 00:00:00' - '2012-08-31 01:00:00'))
```

Work out the number of days in each month of 2012
```sql
select extract(month from cal.month) as month,
	(cal.month + interval '1 month') - cal.month as length
	from (
		  select generate_series(timestamp '2012-01-01', timestamp '2012-12-01', interval '1 month') as month
	) cal
order by month
```

Work out the number of days remaining in the month
```sql
select (date_trunc('month', ts.tests) + interval '1 month')
	- date_trunc('day', ts.tests) as remaining
from (select timestamp '2012-02-11 01:00:00' as tests) ts
```

Work out the end time of bookings
```sql
select starttime, starttime + slots*(interval '30 minutes') endtime
	from bookings
	order by endtime desc, starttime desc
	limit 10
```

Return a count of bookings for each month
```sql
select date_trunc('month', starttime) as month, count(*)
	from bookings
	group by month
	order by month
```

Work out the utilisation percentage for each facility by month
```sql
select name, month, 
	round((100*slots)/
		cast(
			25*(cast((month + interval '1 month') as date)
			- cast (month as date)) as numeric),1) as utilisation
	from  (
		select facs.name as name, date_trunc('month', starttime) as month, sum(slots) as slots
			from bookings bks
			inner join facilities facs
				on bks.facid = facs.facid
			group by facs.facid, month
	) as inn
order by name, month         
```


#### String operations
Format the names of members
```sql
select (surname || ', ' || firstname) as name from members        
```

Find facilities by a name prefix
```sql
select * from facilities
	where name like 'Tennis%'     
```

Perform a case-insensitive search
```sql
select * from facilities where name ilike 'tennis%'    
```

Find telephone numbers with parentheses
```sql
select memid, telephone from members
	where telephone ~ '[()]'    
```

Pad zip codes with leading zeroes
```sql
select lpad(cast(zipcode as varchar(5)), 5, '0') from members 
```

Count the number of members whose surname starts with each letter of the alphabet
```sql
select substr(surname, 1,1) as letter, count(*) as count
	from members
		group by letter
		order by letter
```

Clean up telephone numbers
```sql
select memid, translate(telephone, '-,() ', '') as telephone from members
```
```sql
select memid, regexp_replace(telephone, '[^0-9]', '', 'g') as telephone
    from members
    order by memid;
```


#### Recursive Queries
Find the upward recommendation chain for member ID 27
```sql
with recursive recommenders(recommender) as (
	select recommendedby from members where memid = 27
	union all
	select mems.recommendedby from recommenders recs
		inner join members mems
		on mems.memid = recs.recommender  
)

select recs.recommender, mems.firstname, mems.surname
	from recommenders recs
	inner join members mems
		on recs.recommender = mems.memid
order by memid desc
```

Find the downward recommendation chain for member ID 1
```sql
with recursive recommendeds(memid) as (
	select memid from members where recommendedby = 1
  	union all
  	select mems.memid from recommendeds recs
  		inner join members mems
  			on mems.recommendedby = recs.memid
)

select recs.memid, mems.firstname, mems.surname
	from recommendeds recs
	inner join members mems
		on recs.memid = mems.memid
order by memid
```

CTE that can return the upward recommendation chain for any member
```sql
with recursive recommenders(recommender, member) as (
	select recommendedby, memid
		from members
	union all
	select mems.recommendedby, recs.member
		from recommenders recs
		inner join members mems
			on mems.memid = recs.recommender
)
select recs.member member, recs.recommender, mems.firstname, mems.surname
	from recommenders recs
	inner join members mems		
		on recs.recommender = mems.memid
	where recs.member = 22 or recs.member = 12
order by recs.member asc, recs.recommender desc    
```
