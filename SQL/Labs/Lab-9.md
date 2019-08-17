#Lab 9. Date Manipulation
This Lab introduces recipes for searching and modifying dates. Queries involving dates are very common. Thus, you need to know how to think when working with dates, and you need to have a good understanding of the functions that your RDBMS platform provides for manipulating them. The recipes in this Lab form an important foundation for future work as you move on to more complex queries involving not only dates, but times too.

Before getting into the recipes, I want to reinforce the concept (that I mentioned in the Preface) of using these solutions as guidelines to solving your specific problems. Try to think “big picture.” For example, if a recipe solves a problem for the current month, keep in mind that you may be able to use the recipe for any month (with minor modifications), not just the month used in the recipe. Again, I want you to use these recipes as guidelines, not as the absolute final option. There’s no possible way the Labs can contain an answer for all your problems, but if you understand what is presented here, modifying these solutions to fit your needs is trivial. I also urge you to consider alternative versions of the solutions I’ve provided. For instance, if I solve a problem using one particular function provided by your RDBMS, it is worth the time and effort to find out if there is an alternative—maybe one that is more or less efficient than what is presented here. Knowing what options you have will make you a better SQL programmer.

__TIP__
__The recipes presented in this Lab use simple date data types. If you are using more complex date data types you will need to adjust the solutions accordingly.__

##9.1. Determining if a Year Is a Leap Year
####PROBLEM
You want to determine whether or not the current year is a leap year.

####SOLUTION
If you’ve worked on SQL for some time, there’s no doubt that you’ve come across several techniques for solving this problem. Just about all the solutions I’ve encountered work well, but the one presented in this recipe is probably the simplest. This solution simply checks the last day of February; if it is the 29th then the current year is a leap year.

DB2
Use the recursive WITH clause to return each day in February. Use the aggregate function MAX to determine the last day in February.

	   with x (dy,mth)
	     as (
	 select dy, month(dy)
	   from (
	 select (current_date -
	          dayofyear(current_date) days +1 days)
	           +1 months as dy
	   from t1
	        ) tmp1
	  union all
	 select dy+1 days, mth
	   from x
	  where month(dy+1 day) = mth
	 )
	 select max(day(dy))
	   from x
Oracle
Use the function LAST_DAY to find the last day in February:

 select to_char(
          last_day(add_months(trunc(sysdate,'y'),1)),
         'DD')
   from t1
PostgreSQL
Use the function GENERATE_SERIES to return each day in February, then use the aggregate function MAX to find the last day in February:

	 select max(to_char(tmp2.dy+x.id,'DD')) as dy
	   from (
	 select dy, to_char(dy,'MM') as mth
	   from (
	 select cast(cast(
	             date_trunc('year',current_date) as date)
	                        + interval '1 month' as date) as dy
	   from t1
	        ) tmp1
	        ) tmp2, generate_series (0,29) x(id)
	  where to_char(tmp2.dy+x.id,'MM') = tmp2.mth
MySQL
Use the function LAST_DAY to find the last day in February:

	select day(
	       last_day(
	       date_add(
	       date_add(
	       date_add(current_date,
	                interval -dayofyear(current_date) day),
	                interval 1 day),
	                interval 1 month))) dy
	  from t1
SQL Server
Use the recursive WITH clause to return each day in February. Use the aggregate function MAX to determine the last day in February:

	   with x (dy,mth)
	     as (
	 select dy, month(dy)
	   from (
	 select dateadd(mm,1,(getdate()-datepart(dy,getdate()))+1) dy
	   from t1
	        ) tmp1
	  union all
	 select dateadd(dd,1,dy), mth
	   from x
	  where month(dateadd(dd,1,dy)) = mth
	 )
	 select max(day(dy))
	   from x
####DISCUSSION
DB2
The inline view TMP1 in the recursive view X returns the first day in February by:

Starting with the current date

Using DAYOFYEAR to determine the number of days into the current year that the current date represents

Subtracting that number of days from the current date to get December 31 of the prior year, and then adding one to get to January 1 of the current year

Adding one month to get to February 1

The result of all this math is shown below:

	 select (current_date
	           dayofyear(current_date) days +1 days) +1 months as dy
	   from t1

	DY
	-----------
	01-FEB-2005
The next step is to return the month of the date returned by inline view TMP1 by using the MONTH function:

	select dy, month(dy) as mth
	  from (
	select (current_date
	          dayofyear(current_date) days +1 days) +1 months as dy
	  from t1
	       ) tmp1

	DY          MTH
	----------- ---
	01-FEB-2005   2
The results presented thus far provide the start point for the recursive operation that generates each day in February. To return each day in February, repeatedly add one day to DY until you are no longer in the month of February. A portion of the results of the WITH operation is shown below:

	  with x (dy,mth)
	    as (
	select dy, month(dy)
	  from (
	select (current_date -
	         dayofyear(current_date) days +1 days) +1 months as dy
	  from t1
	       ) tmp1
	 union all
	 select dy+1 days, mth
	   from x
	  where month(dy+1 day) = mth
	 )
	 select dy,mth
	   from x

	DY          MTH
	----------- ---
	01-FEB-2005   2
	…
	10-FEB-2005   2
	…
	28-FEB-2005   2
The final step is to use the MAX function on the DY column to return the last day in February; if it is the 29th, you are in a leap year.

Oracle
The first step is to find the beginning of the year using the TRUNC function:

	select trunc(sysdate,'y')
	  from t1

	DY
	-----------
	01-JAN-2005
Because the first day of the year is January 1st, the next step is to add one month to get to February 1st:

	select add_months(trunc(sysdate,'y'),1) dy
	  from t1

	DY
	-----------
	01-FEB-2005
The next step is to use the LAST_DAY function to find the last day in February:

	select last_day(add_months(trunc(sysdate,'y'),1)) dy
	  from t1

	DY
	-----------
	28-FEB-2005
The final step (which is optional) is to use TO_CHAR to return either 28 or 29.

PostgreSQL
The first step is to examine the results returned by inline view TMP1. Use the DATE_TRUNC function to find the beginning of the current year and cast that result as a DATE:

	select cast(date_trunc('year',current_date) as date) as dy
	  from t1

	DY
	-----------
	01-JAN-2005
The next step is to add one month to the first day of the current year to get the first day in February, casting the result as a date:

	select cast(cast(
	            date_trunc('year',current_date) as date)
	                       + interval '1 month' as date) as dy
	  from t1

	DY
	-----------
	01-FEB-2005
Next, return DY from inline view TMP1 along with the numeric month of DY. Return the numeric month by using the TO_CHAR function:

	select dy, to_char(dy,'MM') as mth
	   from (
	 select cast(cast(
	             date_trunc('year',current_date) as date)
	                        + interval '1 month' as date) as dy
	   from t1
	        ) tmp1

	DY          MTH
	----------- ---
	01-FEB-2005   2
The results shown thus far comprise the result set of inline view TMP2. Your next step is to use the extremely useful function GENERATE_SERIES to return 29 rows (values 1 through 29). Every row returned by GENERATE_SERIES (aliased X) is added to DY from inline view TMP2. Partial results are shown below:

	select tmp2.dy+x.id as dy, tmp2.mth
	  from (
	select dy, to_char(dy,'MM') as mth
	  from (
	select cast(cast(
	            date_trunc('year',current_date) as date)
	                       + interval '1 month' as date) as dy
	  from t1
	       ) tmp1
	       ) tmp2, generate_series (0,29) x(id)
	 where to_char(tmp2.dy+x.id,'MM') = tmp2.mth

	DY          MTH
	----------- ---
	01-FEB-2005  02
	…
	10-FEB-2005  02
	…
	28-FEB-2005  02
The final step is to use the MAX function to return the last day in February. The function TO_CHAR is applied to that value and will return either 28 or 29.

MySQL
The first step is to find the first day of the current year by subtracting from the current date the number of days it is into the year, and then adding one day. Do all of this with the DATE_ADD function:

	select date_add(
	       date_add(current_date,
	                interval -dayofyear(current_date) day),
	                interval 1 day) dy
	  from t1

	DY
	-----------
	01-JAN-2005
Then add one month again using the DATE_ADD function:

	select date_add(
	       date_add(
	       date_add(current_date,
	                interval -dayofyear(current_date) day),
	                interval 1 day),
	                interval 1 month) dy
	  from t1

	DY
	-----------
	01-FEB-2005
Now that you’ve made it to February, use the LAST_DAY function to find the last day of the month:

	select last_day(
	       date_add(
	       date_add(
	       date_add(current_date,
	                interval -dayofyear(current_date) day),
	                interval 1 day),
	                interval 1 month)) dy
	  from t1

	DY
	-----------
	28-FEB-2005
The final step (which is optional) is to use the DAY function to return either a 28 or 29.

SQL Server
This solution uses the recursive WITH clause to generate each day in February. The first step is to find the first day of February. To do this, find the first day of the current year by subtracting from the current date the number of days it is into the year, and then adding one day. Once you have the first day of the current year, use the DATEADD function to add one month to advance to the first day of February:

	select dateadd(mm,1,(getdate()-datepart(dy,getdate()))+1) dy
	  from t1

	DY
	-----------
	01-FEB-2005
Next, return the first day of February along with the numeric month for February:

	select dy, month(dy) mth
	  from (
	select dateadd(mm,1,(getdate()-datepart(dy,getdate()))+1) dy
	  from t1
	       ) tmp1

	DY          MTH
	----------- ---
	01-FEB-2005   2
Then use the recursive capabilities of the WITH clause to repeatedly add one day to DY from inline view TMP1 until you are no longer in February (partial results shown below):

	  with x (dy,mth)
	    as (
	select dy, month(dy)
	  from (
	select dateadd(mm,1,(getdate()-datepart(dy,getdate()))+1) dy
	  from t1
	       ) tmp1
	 union all
	select dateadd(dd,1,dy), mth
	  from x
	 where month(dateadd(dd,1,dy)) = mth
	 )
	select dy,mth from x

	DY          MTH
	----------- ---
	01-FEB-2005  02
	…
	10-FEB-2005  02
	…
	28-FEB-2005  02
Now that you can return each day in February, the final step is to use the MAX function to see if the last day is the 28th or 29th. As an optional last step, you can use the DAY function to return a 28 or 29, rather than a date.

##9.2. Determining the Number of Days in a Year
####PROBLEM
You want to count the number of days in the current year.

####SOLUTION
The number of days in the current year is the difference between the first day of the next year and the first day of the current year (in days). For each solution the steps are:

Find the first day of the current year.

Add one year to that date (to get the first day of the next year).

Subtract the current year from the result of Step 2.

The solutions differ only in the built-in functions that you use to perform these steps.

DB2
Use the function DAYOFYEAR to help find the first day of the current year, and use DAYS to find the number of days in the current year:

	select days((curr_year + 1 year)) - days(curr_year)
	  from (
	select (current_date -
	        dayofyear(current_date) day +
	         1 day) curr_year
	  from t1
	       ) x
Oracle
Use the function TRUNC to find the beginning of the current year, and use ADD_ MONTHS to then find the beginning of next year:

	 selectadd_months(trunc(sysdate,'y'),12) - trunc(sysdate,'y')
	   from dual
PostgreSQL
Use the function DATE_TRUNC to find the beginning of the current year. Then use interval arithmetic to determine the beginning of next year:

	 select cast((curr_year + interval '1 year') as date) - curr_year
	   from (
	 select cast(date_trunc('year',current_date) as date) as curr_year
	   from t1
	        ) x
MySQL
Use ADDDATE to help find the beginning of the current year. Use DATEDIFF and interval arithmetic to determine the number of days in the year:

	1 select datediff((curr_year + interval 1 year),curr_year)
	2   from (
	3 select adddate(current_date,-dayofyear(current_date)+1) curr_year
	4   from t1
	5        ) x
SQL Server
Use the function DATEADD to find the first day of the current year. Use DATEDIFF to return the number of days in the current year:

	1 select datediff(d,curr_year,dateadd(yy,1,curr_year))
	2   from (
	3 select dateadd(d,-datepart(dy,getdate())+1,getdate()) curr_year
	4   from t1
	5        ) x
####DISCUSSION
DB2
The first step is to find the first day of the current year. Use DAYOFYEAR to determine how many days you are into the current year. Subtract that value from the current date to get the last day of last year, and then add 1:

	select (current_date
	        dayofyear(current_date) day +
	         1 day) curr_year
	  from t1

	CURR_YEAR
	-----------
	01-JAN-2005
Now that you have the first day of the current year, just add one year to it; this gives you the first day of next year. Then subtract the beginning of the current year from the beginning of next year.

Oracle
The first step is to find the first day of the current year, which you can easily do by invoking the built-in TRUNC function and passing ‘Y’ as the second argument (thereby truncating the date to the beginning of the year):

	select select trunc(sysdate,'y') curr_year
	  from dual

	CURR_YEAR
	-----------
	01-JAN-2005
Then add one year to arrive at the first day of the next year. Finally, subtract the two dates to find the number of days in the current year.

PostgreSQL
Begin by finding the first day of the current year. To do that, invoke the DATE_ TRUNC function as follows:

	select cast(date_trunc('year',current_date) as date) as curr_year
	  from t1

	CURR_YEAR
	-----------
	01-JAN-2005
You can then easily add a year to compute the first day of next year. Then all you need to do is to subtract the two dates. Be sure to subtract the earlier date from the later date. The result will be the number of days in the current year.

MySQL
Your first step is to find the first day of the current year. Use DAYOFYEAR to find how many days you are into the current year. Subtract that value from the current date, and add 1:

	select adddate(current_date,-dayofyear(current_date)+1) curr_year
	  from t1

	CURR_YEAR
	-----------
	01-JAN-2005
Now that you have the first day of the current year, your next step is to add one year to it to get the first day of next year. Then subtract the beginning of the current year from the beginning of the next year. The result is the number of days in the current year.

SQL Server
Your first step is to find the first day of the current year. Use DATEADD and DATEPART to subtract from the current date the number of days into the year the current date is, and add 1:

	select dateadd(d,-datepart(dy,getdate())+1,getdate()) curr_year
	  from t1

	CURR_YEAR
	-----------
	01-JAN-2005
Now that you have the first day of the current year, your next step is to add one year to it get the first day of the next year. Then subtract the beginning of the current year from the beginning of the next year. The result is the number of days in the current year.

##9.3. Extracting Units of Time from a Date
####PROBLEM
You want to break the current date down into six parts: day, month, year, second, minute, and hour. You want the results to be returned as numbers.

####SOLUTION
My use of the current date is arbitrary. Feel free to use this recipe with other dates. In Lab 1, I mention the importance of learning and taking advantage of the built-in functions provided by your RDBMS; this is especially true when it comes to working with dates. There are different ways of extracting units of time from a date than those presented in this recipe, and it would benefit you to experiment with different techniques and functions.

DB2
DB2 implements a set of built-in functions that make it easy for you to extract portions of a date. The function names HOUR, MINUTE, SECOND, DAY, MONTH, and YEAR conveniently correspond to the units of time you can return: if you want the day use DAY, hour use HOUR, etc. For example:

	 select    hour( current_timestamp ) hr,
	         minute( current_timestamp ) min,
	         second( current_timestamp ) sec,
	            day( current_timestamp ) dy,
	          month( current_timestamp ) mth,
	            year( current_timestamp ) yr
	   from t1

	  HR   MIN   SEC    DY   MTH    YR
	---- ----- ----- ----- ----- -----
	  20    28    36    15     6  2005
Oracle
Use functions TO_CHAR and TO_NUMBER to return specific units of time from a date:

	  select to_number(to_char(sysdate,'hh24')) hour,
	         to_number(to_char(sysdate,'mi')) min,
	         to_number(to_char(sysdate,'ss')) sec,
	         to_number(to_char(sysdate,'dd')) day,
	         to_number(to_char(sysdate,'mm')) mth,
	         to_number(to_char(sysdate,'yyyy')) year
	   from dual

	  HOUR   MIN   SEC   DAY   MTH  YEAR
	  ---- ----- ----- ----- ----- -----
	    20    28    36    15     6  2005
PostgreSQL
Use functions TO_CHAR and TO_NUMBER to return specific units of time from a date:

	 select to_number(to_char(current_timestamp,'hh24'),'99') as hr,
	        to_number(to_char(current_timestamp,'mi'),'99') as min,
	        to_number(to_char(current_timestamp,'ss'),'99') as sec,
	        to_number(to_char(current_timestamp,'dd'),'99') as day,
	        to_number(to_char(current_timestamp,'mm'),'99') as mth,
	        to_number(to_char(current_timestamp,'yyyy'),'9999') as yr
	   from t1
	
	   HR   MIN   SEC   DAY   MTH    YR
	 ---- ----- ----- ----- ----- -----
	 20      28    36    15     6  2005
MySQL
Use the DATE_FORMAT function to return specific units of time from a date:

	 select date_format(current_timestamp,'%k') hr,
	        date_format(current_timestamp,'%i') min,
	        date_format(current_timestamp,'%s') sec,
	        date_format(current_timestamp,'%d') dy,
	        date_format(current_timestamp,'%m') mon,
	        date_format(current_timestamp,'%Y') yr
	   from t1

	  HR   MIN   SEC   DAY   MTH    YR
	---- ----- ----- ----- ----- -----
	  20    28    36    15     6  2005
SQL Server
Use the function DATEPART to return specific units of time from a date:

	 select datepart( hour, getdate()) hr,
	        datepart( minute,getdate()) min,
	        datepart( second,getdate()) sec,
	        datepart( day, getdate()) dy,
	        datepart( month, getdate()) mon,
	        datepart( year, getdate()) yr
	   from t1

	  HR   MIN   SEC   DAY   MTH    YR
	---- ----- ----- ----- ----- -----
	  20    28    36    15     6  2005
####DISCUSSION
There’s nothing fancy in these solutions; just take advantage of what you’re already paying for. Take the time to learn the date functions available to you. This recipe only scratches the surface of the functions presented in each solution. You’ll find that each of the functions takes many more arguments and can return more information than what this recipe provides you.

##9.4. Determining the First and Last Day of a Month
####PROBLEM
You want to determine the first and last days for the current month.

####SOLUTION
The solutions presented here are for finding first and last days for the current month. Using the current month is arbitrary. With a bit of adjustment, you can make the solutions work for any month.

DB2
Use the DAY function to return the number of days into the current month the current date represents. Subtract this value from the current date, and then add 1 to get the first of the month. To get the last day of the month, add one month to the current date, then subtract from it the value returned by the DAY function as applied to the current date:

	 select (date(current_date) - day(date(current_date)) day + 1 day) firstday,
	        (date(current_date)+1 month - day(date(current_date)+1 month) day) lastday
	   from t1
Oracle
Use the function TRUNC to find the first of the month, and the function LAST_DAY to find the last day of the month:

	 select trunc(sysdate,'mm') firstday,
	        last_day(sysdate) lastday
	   from dual
__TIP__
_Using TRUNC as decribed here will result in the loss of any time-of-day component, whereas LAST_DAY will preserve the time of day._

PostgreSQL
Use the DATE_TRUNC function to truncate the current date to the first of the current month. Once you have the first day of the month, add one month and subtract one day to find the end of the current month:

	 select firstday,
	        cast(firstday + interval '1 month'
	                      - interval '1 day' as date) as lastday
	   from (
	 select cast(date_trunc('month',current_date) as date) as firstday
	   from t1
	        ) x
MySQL
Use the DATE_ADD and DAY functions to find the number of days into the month the current date is. Then subtract that value from the current date and add 1 to find the first of the month. To find the last day of the current month, use the LAST_DAY function:

	 select date_add(current_date,
	                 interval -day(current_date)+1 day) firstday,
	        last_day(current_date) lastday
	   from t1
SQL Server
Use the DATEADD and DAY functions to find the number of days into the month represented by the current date. Then subtract that value from the current date and add 1 to find the first of the month. To get the last day of the month, add one month to the current date, and then subtract from that result the value returned by the DAY function applied to the current date, again using the functions DAY and DATEADD:

	 select dateadd(day,-day(getdate())+1,getdate()) firstday,
	        dateadd(day,
	                -day(dateadd(month,1,getdate())),
	                dateadd(month,1,getdate())) lastday
	   from t1
####DISCUSSION
DB2
To find the first day of the month simply find the numeric value of the current day of the month then subtract this from the current date. For example, if the date is March 14th, the numeric day value is 14. Subtracting 14 days from March 14th gives you the last day of the month in February. From there, simply add one day to get to the first of the current month. The technique to get the last day of the month is similar to that of the first; subtract the numeric day of the month from the current date to get the last day of the prior month. Since we want the last day of the current month (not the last day of the prior month), we need to add one month to the current date.

Oracle
To find the first day of the current month, use the TRUNC function with “mm” as the second argument to “truncate” the current date down to the first of the month. To find the last day of the current month, simply use the LAST_DAY function.

PostgreSQL
To find the first day of the current month, use the DATE_TRUNC function with “month” as the second argument to “truncate” the current date down to the first of the month. To find the last day of the current month, add one month to the first day of the month, and then subtract one day.

MySQL
To find the first day of the month, use the DAY function. The DAY function conveniently returns the day of the month for the date passed. If you subtract the value returned by DAY(CURRENT_DATE) from the current date, you get the last day of the prior month; add one day to get the first day of the current month. To find the last day of the current month, simply use the LAST_DAY function.

SQL Server
To find the first day of the month, use the DAY function. The DAY function conveniently returns the day of the month for the date passed. If you subtract the value returned by DAY(GETDATE()) from the current date, you get the last day of the prior month; add one day to get the first day of the current month. To find the last day of the current month, use the DATEADD function. Add one month to the current date, then subtract from it the value returned by DAY(GETDATE()) to get the last day of the current month. Add one month to the current date, then subtract from it the value returned by DAY(DATEADD(MONTH,1,GETDATE())) to get the last day of the current month.

##9.5. Determining All Dates for a Particular Weekday Throughout a Year
####PROBLEM
You want to find all the dates in a year that correspond to a given day of the week. For example, you may wish to generate a list of Fridays for the current year.

####SOLUTION
Regardless of vendor, the key to the solution is to return each day for the current year and keep only those dates corresponding to the day of the week that you care about. The solution examples retain all the Fridays.

DB2
Use the recursive WITH clause to return each day in the current year. Then use the function DAYNAME to keep only Fridays:

	   with x (dy,yr)
	     as (
	 select dy, year(dy) yr
	   from (
	 select (current_date -
	          dayofyear(current_date) days +1 days) as dy
	   from t1
	        ) tmp1
	  union all
    select dy+1 days, yr
      from x
     where year(dy +1 day) = yr
    )
    select dy
      from x
     where dayname(dy) = 'Friday'
Oracle
Use the recursive CONNECT BY clause to return each day in the current year. Then use the function TO_CHAR to keep only Fridays:

   with x
     as (
 select trunc(sysdate,'y')+level-1 dy
   from t1
   connect by level <=
      add_months(trunc(sysdate,'y'),12)-trunc(sysdate,'y')
 )
 select *
   from x
     where to_char( dy, 'dy') = 'fri'
PostgreSQL
Use the function GENERATE_SERIES to return each day in the current year. Then use the function TO_CHAR to keep only Fridays:

	 select cast(date_trunc('year',current_date) as date)
	        + x.id as dy
	   from generate_series (
	         0,
	         ( select cast(
	                  cast(
	            date_trunc('year',current_date) as date)
	                       + interval '1 years' as date)
	                       - cast(
	                   date_trunc('year',current_date) as date) )-1
	         ) x(id)
	  where to_char(
	           cast(
	     date_trunc('year',current_date)
	                as date)+x.id,'dy') = 'fri'
MySQL
Use the pivot table T500 to return each day in the current year. Then use the function DAYNAME to keep only Fridays:

	 select dy
	   from (
	 select adddate(x.dy,interval t500.id-1 day) dy  
	   from (
	 select dy, year(dy) yr
	   from (
	 select adddate(
	        adddate(current_date,
	                interval -dayofyear(current_date) day),
	                interval 1 day ) dy
	   from t1
	        ) tmp1
	        ) x,
	        t500
	  where year(adddate(x.dy,interval t500.id-1 day)) = x.yr
	        ) tmp2
	  where dayname(dy) = 'Friday'
SQL Server
Use the recursive WITH clause to return each day in the current year. Then use the function DAYNAME to keep only Fridays:

	   with x (dy,yr)
	     as (
	 select dy, year(dy) yr
	   from (
	 select getdate()-datepart(dy,getdate())+1 dy
	   from t1
	        ) tmp1
	  union all
	 select dateadd(dd,1,dy), yr
	   from x
	  where year(dateadd(dd,1,dy)) = yr
	 )
	 select x.dy
	   from x
	  where datename(dw,x.dy) = 'Friday'
	 option (maxrecursion 400)
####DISCUSSION
DB2
To find all the Fridays in the current year, you must be able to return every day in the current year. The first step is to find the first day of the year by using the DAYOFYEAR function. Subtract the value returned by DAYOFYEAR(CURRENT_DATE) from the current date to get December 31 of the prior year, and then add 1 to get the first day of the current year:

	select (current_date
	         dayofyear(current_date) days +1 days) as dy
	  from t1

	DY
	-----------
	01-JAN-2005
Now that you have the first day of the year, use the WITH clause to repeatedly add one day to the first day of the year until you are no longer in the current year. The result set will be every day in the current year (a portion of the rows returned by the recursive view X is shown below):

	 with x (dy,yr)
	   as (
	select dy, year(dy) yr
	  from (
	select (current_date
	         dayofyear(current_date) days +1 days) as dy
	  from t1
	        ) tmp1
	union all
	select dy+1 days, yr
	  from x
	 where year(dy +1 day) = yr
	)
	select dy
	  from x

	DY
	-----------
	01-JAN-2005
	…
	15-FEB-2005
	…
	22-NOV-2005
	…
	31-DEC-2005
The final step is to use the DAYNAME function to keep only rows that are Fridays.

Oracle
To find all the Fridays in the current year, you must be able to return every day in the current year. Begin by using the TRUNC function to find the first day of the year:

select trunc(sysdate,'y') dy
	  from t1

	DY
	-----------
	01-JAN-2005
Next, use the CONNECT BY clause to return every day in the current year (to understand how to use CONNECT BY to generate rows, see “Generating Consecutive Time and Numeric Values” in Lab 13).

__TIP__
_As an aside, this recipe uses the WITH clause, but you can also use an inline view._

At the time of this writing, Oracle’s WITH clause is not meant for recursive operations (unlike the case with DB2 and SQL Server); recursive operations are done using CONNECT BY. A portion of the result set returned by view X is shown below:

	 with x
	   as (
	select trunc(sysdate,'y')+level-1 dy
	from t1
	 connect by level <=
	    add_months(trunc(sysdate,'y'),12)-trunc(sysdate,'y')
	)
	select *
	from x

	DY
	-----------
	01-JAN-2005
	…
	15-FEB-2005
	…
	22-NOV-2005
	…
	31-DEC-2005
The final step is to use the TO_CHAR function to keep only Fridays.

PostgreSQL
To find all the Fridays in the current year, you must be able to return a row for every day in the current year. To do that, use the GENERATE_SERIES function. The start and end values to be returned by GENERATE_SERIES are 0 and the number of days in the current year minus 1. The first parameter passed to GENERATE_SERIES is 0, while the second is a query that determines the number of days in the current year (because you are adding to the first day of the current year, you actually want to add 1 less than the number of days in the current year, so as to not spill over into the next year). The result returned by the second parameter of the GENERATE_SERIES function is shown below:

	select cast(
	       cast(
	 date_trunc('year',current_date) as date)
	            + interval '1 years' as date)
	            -cast(
	        date_trunc('year',current_date) as date)-1 as cnt
	  from t1

	CNT
	---
	364
Keeping in mind the result set above, the call to GENERATE_SERIES in the FROM clause will look like this: GENERATE_SERIES ( 0, 364 ). If you are in a leap year, such as 2004, the second parameter would be 365.

The next step after generating a list of dates in the year is to add the values returned by GENERATE_SERIES to the first day of the current year. A portion of the results is shown below:

	select cast(date_trunc('year',current_date) as date)
	       + x.id as dy
	  from generate_series (
	       0,
	       ( select cast(
	                cast(
	          date_trunc('year',current_date) as date)
	                     + interval '1 years' as date)
	                     -cast(
	                 date_trunc('year',current_date) as date) )-1
	       ) x(id)

	DY
	-----------
	01-JAN-2005
	…
	15-FEB-2005
	…
	22-NOV-2005
	…
	31-DEC-2005
The final step is to use the TO_CHAR function to keep only the Fridays.

MySQL
To find all the Fridays in the current year, you must be able to return every day in the current year. The first step is to find the first day of the year by using the DAYOF-YEAR function. Subtract the value returned by DAYOFYEAR(CURRENT_DATE) from the current date, and then add 1 to get the first day of the current year:

	select adddate(
	       adddate(current_date,
	               interval -dayofyear(current_date) day),
	               interval 1 day ) dy
	  from t1

	DY
	-----------
	01-JAN-2005
Then use table T500 to generate enough rows to return each day in the current year. You can do this by adding each value of T500.ID to the first day of the year until you break out of the current year. Partial results of this operation are shown below:

	select adddate(x.dy,interval t500.id-1 day) dy
	  from (
	select dy, year(dy) yr
	  from (
	select adddate(
	       adddate(current_date,
	               interval -dayofyear(current_date) day),
	               interval 1 day ) dy
	  from t1
	        ) tmp1
	        ) x,
	        t500
	 where year(adddate(x.dy,interval t500.id-1 day)) = x.yr

	DY
	-----------
	01-JAN-2005
	…
	15-FEB-2005
	…
	22-NOV-2005
	…
	31-DEC-2005
The final step is to use the DAYNAME function to keep only Fridays.

SQL Server
To find all the Fridays in the current year, you must be able to return every day in the current year. The first step is to find the first day of the year by using the DATEPART function. Subtract the value returned by DATEPART(DY,GETDATE()) from the current date, and then add 1 to get the first day of the current year:

	select getdate()-datepart(dy,getdate())+1 dy
	  from t1

	DY
	-----------
	01-JAN-2005
Now that you have the first day of the year, use the WITH clause and the DATEADD function to repeatedly add one day to the first day of the year until you are no longer in the current year. The result set will be every day in the current year (a portion of the rows returned by the recursive view X is shown below):

	with x (dy,yr)
	  as (
	select dy, year(dy) yr
	  from (
	select getdate()-datepart(dy,getdate())+1 dy
	  from t1
	       ) tmp1
	 union all
	select dateadd(dd,1,dy), yr
	  from x
	 where year(dateadd(dd,1,dy)) = yr
	)
	select x.dy
	  from x
	option (maxrecursion 400)

	DY
	-----------
	01-JAN-2005
	…
	15-FEB-2005
	…
	22-NOV-2005
	…
	31-DEC-2005
Finally, use the DATENAME function to keep only rows that are Fridays. For this solution to work, you must set MAXRECURSION to at least 366 (the filter on the year portion of the current year, in recursive view X, guarantees you will never generate more than 366 rows).

##9.6. Determining the Date of the First and Last Occurrence of a Specific Weekday in a Month
####PROBLEM
You want to find, for example, the first and last Mondays of the current month.

####SOLUTION
The choice to use Monday and the current month is arbitrary; you can use the solutions presented in this recipe for any weekday and any month. Because each weekday is seven days apart from itself, once you have the first instance of a weekday, you can add 7 days to get the second and 14 days to get the third. Likewise, if you have the last instance of a weekday in a month, you can subtract 7 days to get the third and subtract 14 days to get the second.

DB2
Use the recursive WITH clause to generate each day in the current month and use a CASE expression to flag all Mondays. The first and last Mondays will be the earliest and latest of the flagged dates:

	   with x (dy,mth,is_monday)
	     as (
	 select dy,month(dy),
	        case when dayname(dy)='Monday'
	              then 1 else 0
	        end
	   from (
	 select (current_date-day(current_date) day +1 day) dy
	   from t1
	        ) tmp1
	  union all
	 select (dy +1 day), mth,
	        case when dayname(dy +1 day)='Monday'
	             then 1 else 0
	        end
	   from x
	  where month(dy +1 day) = mth
	 )
	 select min(dy) first_monday, max(dy) last_monday
	   from x
	  where is_monday = 1
Oracle
Use the functions NEXT_DAY and LAST_DAY, together with a bit of clever date arithmetic, to find the first and last Mondays of the current month:

	select next_day(trunc(sysdate,'mm')-1,'MONDAY') first_monday,
	       next_day(last_day(trunc(sysdate,'mm'))-7,'MONDAY') last_monday
	  from dual
PostgreSQL
Use the function DATE_TRUNC to find the first day of the month. Once you have the first day of the month, you can use simple arithmetic involving the numeric values of weekdays (Sun–Sat is 1–7) to find the first and last Mondays of the current month:

	 select first_monday,
	        case to_char(first_monday+28,'mm')
	             when mth then first_monday+28
	                      else first_monday+21
	        end as last_monday
	   from (
	 select case sign(cast(to_char(dy,'d') as integer)-2)
	             when 0
	             then dy
	             when -1
	             then dy+abs(cast(to_char(dy,'d') as integer)-2)
	             when 1
	             then (7-(cast(to_char(dy,'d') as integer)-2))+dy
	        end as first_monday,
	        mth
	   from (
	 select cast(date_trunc('month',current_date) as date) as dy,
	        to_char(current_date,'mm') as mth
	   from t1
	        ) x
	        ) y
MySQL
Use the ADDDATE function to find the first day of the month. Once you have the first day of the month, you can use simple arithmetic on the numeric values of weekdays (Sun–Sat is 1–7) to find the first and last Mondays of the current month:

	 select first_monday,
	        case month(adddate(first_monday,28))
	             when mth then adddate(first_monday,28)
	                      else adddate(first_monday,21)
	        end last_monday
	  from (
	 select case sign(dayofweek(dy)-2)
	             when 0 then dy
	             when -1 then adddate(dy,abs(dayofweek(dy)-2))
	             when 1 then adddate(dy,(7-(dayofweek(dy)-2)))
	        end first_monday,
	        mth
	   from (
	 select adddate(adddate(current_date,-day(current_date)),1) dy,
	        month(current_date) mth
	   from t1
	        ) x
	        ) y
SQL Server
Use the recursive WITH clause to generate each day in the current month, and then use a CASE expression to flag all Mondays. The first and last Mondays will be the earliest and latest of the flagged dates:

	   with x (dy,mth,is_monday)
	     as (
	 select dy,mth,
	        case when datepart(dw,dy) = 2
	             then 1 else 0
	        end
	   from (
	 select dateadd(day,1,dateadd(day,-day(getdate()),getdate())) dy,
	        month(getdate()) mth
	   from t1
	        ) tmp1
	  union all
	 select dateadd(day,1,dy),
	        mth,
	        case when datepart(dw,dateadd(day,1,dy)) = 2
	             then 1 else 0
	        end
	   from x
	  where month(dateadd(day,1,dy)) = mth
	 )
	 select min(dy) first_monday,
	        max(dy) last_monday
	   from x
	  where is_monday = 1
####DISCUSSION
DB2 and SQL Server
DB2 and SQL Server use different functions to solve this problem, but the technique is exactly the same. If you eyeball both solutions you’ll see the only difference between the two is the way dates are added. This discussion will cover both solutions, using the DB2 solution’s code to show the results of intermediate steps.

__TIP__
_If you do not have access to the recursive WITH clause in the version of SQL Server or DB2 that you are running, you can use the PostgreSQL technique instead._

The first step in finding the first and last Mondays of the current month is to return the first day of the month. Inline view TMP1 in recursive view X finds the first day of the current month by first finding the current date, specifically, the day of the month for the current date. The day of the month for the current date represents how many days into the month you are (e.g., April 10th is the 10th day of the April). If you subtract this day of the month value from the current date, you end up at the last day of the previous month (e.g., subtracting 10 from April 10th puts you at the last day of March). After this subtraction, simply add one day to arrive at the first day of the current month:

	select (current_date-day(current_date) day +1 day) dy
	  from t1

	DY
	-----------
	01-JUN-2005
Next, find the month for the current date using the MONTH function and a simple CASE expression to determine whether or not the first day of the month is a Monday:

	select dy, month(dy) mth,
	        case when dayname(dy)='Monday'
	             then 1 else 0
	        end is_monday
	  from  (
	select  (current_date-day(current_date) day +1 day) dy
	  from  t1
	        ) tmp1

	DY          MTH  IS_MONDAY
	----------- --- ----------
	01-JUN-2005   6          0
Then use the recursive capabilities of the WITH clause to repeatedly add one day to the first day of the month until you’re no longer in the current month. Along the way, you will use a CASE expression to determine which days in the month are Mondays (Mondays will be flagged with “1”). A portion of the output from recursive view X is shown below:

	with x (dy,mth,is_monday)
	     as (
	 select dy,month(dy) mth,
	        case when dayname(dy)='Monday'
	             then 1 else 0
	        end is_monday
	   from (
	 select (current_date-day(current_date) day +1 day) dy
	   from t1
	        ) tmp1
	  union all
	 select (dy +1 day), mth,
	        case when dayname(dy +1 day)='Monday'
	             then 1 else 0
	        end
	   from x
	  where month(dy +1 day) = mth
	 )
	 select *
	   from x

	DY          MTH  IS_MONDAY
	----------- --- ----------
	01-JUN-2005   6          0
	02-JUN-2005   6          0
	03-JUN-2005   6          0
	04-JUN-2005   6          0
	05-JUN-2005   6          0
	06-JUN-2005   6          1
	07-JUN-2005   6          0
	08-JUN-2005   6          0
	…
Only Mondays will have a value of 1 for IS_MONDAY, so the final step is to use the aggregate functions MIN and MAX on rows where IS_MONDAY is 1 to find the first and last Mondays of the month.

Oracle
The function NEXT_DAY makes this problem easy to solve. To find the first Monday of the current month, first return the last day of the prior month via some date arithmetic involving the TRUNC function:

	select trunc(sysdate,'mm')-1 dy
	  from dual

	DY
	-----------
	31-MAY-2005
Then use the NEXT_DAY function to find the first Monday that comes after the last day of the previous month (i.e., the first Monday of the current month):

	select next_day(trunc(sysdate,'mm')-1,'MONDAY') first_monday
	  from dual

	FIRST_MONDAY
	------------
	06-JUN-2005
To find the last Monday of the current month, start by returning the first day of the current month by using the TRUNC function:

	select trunc(sysdate,'mm') dy
	  from dual

	DY
	-----------
	01-JUN-2005
The next step is to find the last week (the last seven days) of the month. Use the LAST_DAY function to find the last day of the month, and then subtract seven days:

	select last_day(trunc(sysdate,'mm'))-7 dy
	  from dual

	DY
	-----------
	23-JUN-2005
If it isn’t immediately obvious, you go back seven days from the last day of the month to ensure that you will have at least one of any weekday left in the month. The last step is to use the function NEXT_DAY to find the next (and last) Monday of the month:

	select next_day(last_day(trunc(sysdate,'mm'))-7,'MONDAY') last_monday
	  from dual

	LAST_MONDAY
	-----------
	27-JUN-2005
PostgreSQL and MySQL
PostgreSQL and MySQL also share the same solution approach. The difference is in the functions that you invoke. Despite their lengths, the respective queries are extremely simple; little overhead is involved in finding the first and last Mondays of the current month.

The first step is to find the first day of the current month. The next step is to find the first Monday of the month. Since there is no function to find the next date for a given weekday, you need to use a little arithmetic. The CASE expression beginning on line 7 (of either solution) evaluates the difference between the numeric value for the weekday of the first day of the month and the numeric value corresponding to Monday. Given that the function TO_CHAR (PostgresSQL), when called with the ‘D’ or ‘d’ format, and the function DAYOFWEEK (MySQL) will return a numeric value from 1 to 7 representing days Sunday to Saturday; Monday is always represented by 2. The first test evaluated by CASE is the SIGN of the numeric value of the first day of the month (whatever it may be) minus the numeric value of Monday (2). If the result is 0, then the first day of the month falls on a Monday and that is the first Monday of the month. If the result is–1, then the first day of the month falls on a Sunday and to find the first Monday of the month simply add the difference in days between 2 and 1 (numeric values of Monday and Sunday, respectively) to the first day of the month.

__TIP__
_If you are having trouble understanding how this works, forget the weekday names and just do the math. For example, say you happen to be starting on a Tuesday and you are looking for the next Friday. When using TO_CHAR with the ‘d’ format, or DAYOFWEEK, Friday is 6 and Tuesday is 3. To get to 6 from 3, simply take the difference (6–3 = 3) and add it to the smaller value ((6–3) + 3 = 6). So, regardless of the actual dates, if the numeric value of the day you are starting from is less than the numeric value of the day you are searching for, adding the difference between the two dates to the date you are starting from will get you to the date you are searching for.

If the result from SIGN is 1, then the first day of the month falls between Tuesday and Saturday (inclusive). When the first day of the month has a numeric value greater than 2 (Monday), subtract from 7 the difference between the numeric value of the first day of the month and the numeric value of Monday (2), and then add that value to the first day of the month. You will have arrived at the day of the week that you are after, in this case Monday._

__TIP__
_Again, if you are having trouble understanding how this works, forget the weekday names and just do the math. For example, suppose you want to find the next Tuesday and you are starting from Friday. Tuesday (3) is less than Friday (6). To get to 3 from 6 subtract the difference between the two values from 7 (7–( |3–6| ) = 4) and add the result (4) to the start day Friday. (The vertical bars in |3-6| generate the absolute value of that difference.) Here, you’re not adding 4 to 6 (which will give you 10), you are adding four days to Friday, which will give you the next Tuesday._

The idea behind the CASE expression is to create a sort of a “next day” function for PostgreSQL and MySQL. If you do not start with the first day of the month, the value for DY will be the value returned by CURRENT_DATE and the result of the CASE expression will return the date of the next Monday starting from the current date (unless CURRENT_DATE is a Monday, then that date will be returned).

Now that you have the first Monday of the month, add either 21 or 28 days to find the last Monday of the month. The CASE expression in lines 2–5 determines whether to add 21 or 28 days by checking to see whether 28 days takes you into the next month. The CASE expression does this through the following process:

It adds 28 to the value of FIRST_MONDAY.

Using either TO_CHAR (PostgreSQL) or MONTH, the CASE expression extracts the name of the current month from result of FIRST_MONDAY + 28.

The result from Step 2 is compared to the value MTH from the inline view. The value MTH is the name of the current month as derived from CURRENT_ DATE. If the two month values match, then the month is large enough for you to need to add 28 days, and the CASE expression returns FIRST_MONDAY + 28. If the two month values do not match, then you do not have room to add 28 days, and the CASE expression returns FIRST_MONDAY + 21 days instead. It is convenient that our months are such that 28 and 21 are the only two possible values you need worry about adding.

__TIP__
_You can extend the solution by adding 7 and 14 days to find the second and third Mondays of the month, respectively._

##9.7. Creating a Calendar
####PROBLEM
You want to create a calendar for the current month. The calendar should be formatted like a calendar you might have on your desk seven columns across, (usually) five rows down.

####SOLUTION
Each solution will look a bit different, but they all solve the problem the same way: return each day for the current month, and then pivot on the day of the week for each week in the month to create a calendar.

There are different formats available for calendars. For example, the Unix cal command formats the days from Sunday to Saturday. The examples in this recipe are based on ISO weeks, so the Monday through Friday format is the most convenient to generate. Once you become comfortable with the solutions, you’ll see that reformatting however you like is simply a matter of modifying the values assigned by the ISO week before pivoting.

__TIP__
_As you begin to use different types of formatting with SQL to create readable output, you will notice your queries becoming longer. Don’t let those long queries intimidate you; the queries presented for this recipe are extremely simple once broken down and run piece by piece._

DB2
Use the recursive WITH clause to return every day in the current month. Then pivot on the day of the week using CASE and MAX:

	   with x(dy,dm,mth,dw,wk)
	   as (
	 select (current_date -day(current_date) day +1 day) dy,
	         day((current_date -day(current_date) day +1 day)) dm,
	         month(current_date) mth,
	         dayofweek(current_date -day(current_date) day +1 day) dw,
	         week_iso(current_date -day(current_date) day +1 day) wk
	   from t1
	  union all
	 select dy+1 day, day(dy+1 day), mth,
	         dayofweek(dy+1 day), week_iso(dy+1 day)
	   from x
	  where month(dy+1 day) = mth
	  )
	  select max(case dw when 2 then dm end) as Mo,
	         max(case dw when 3 then dm end) as Tu,
	         max(case dw when 4 then dm end) as We,
	         max(case dw when 5 then dm end) as Th,
	         max(case dw when 6 then dm end) as Fr,
	         max(case dw when 7 then dm end) as Sa,
	         max(case dw when 1 then dm end) as Su
	   from x
	  group by wk
	  order by wk
Oracle
Use the recursive CONNECT BY clause to return each day in the current month. Then pivot on the day of the week using CASE and MAX:

	  with x
	    as (
	 select *
	   from (
	 select to_char(trunc(sysdate,'mm')+level-1,'iw') wk,
	        to_char(trunc(sysdate,'mm')+level-1,'dd') dm,
	        to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw,
	        to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth,
	        to_char(sysdate,'mm') mth
	   from dual
	  connect by level <= 31
	        )
	  where curr_mth = mth
	 )
	 select max(case dw when 2 then dm end) Mo,
	        max(case dw when 3 then dm end) Tu,
	        max(case dw when 4 then dm end) We,
	        max(case dw when 5 then dm end) Th,
	        max(case dw when 6 then dm end) Fr,
	        max(case dw when 7 then dm end) Sa,
	        max(case dw when 1 then dm end) Su
	   from x
	  group by wk
	  order by wk
PostgreSQL
Use the function GENERATE_SERIES to return every day in the current month. Then pivot on the day of the week using MAX and CASE:

	 select max(case dw when 2 then dm end) as Mo,
	        max(case dw when 3 then dm end) as Tu,
	        max(case dw when 4 then dm end) as We,
	        max(case dw when 5 then dm end) as Th,
	        max(case dw when 6 then dm end) as Fr,
	        max(case dw when 7 then dm end) as Sa,
	        max(case dw when 1 then dm end) as Su
	   from (
	 select *
	   from (
	 select cast(date_trunc('month',current_date) as date)+x.id,
	        to_char(
	           cast(
	     date_trunc('month',current_date)
	                as date)+x.id,'iw') as wk,
	        to_char(
	           cast(
	     date_trunc('month',current_date)
	                as date)+x.id,'dd') as dm,
	        cast(
	     to_char(
	        cast(
	   date_trunc('month',current_date)
	                 as date)+x.id,'d') as integer) as dw,
	         to_char(
	            cast(
	     date_trunc('month',current_date)
	                 as date)+x.id,'mm') as curr_mth,
	         to_char(current_date,'mm') as mth
	   from generate_series (0,31) x(id)
	        ) x
	  where mth = curr_mth
	        ) y
	  group by wk
	  order by wk
Mysql
Use table T500 to return each day in the current month. Then pivot on the day of the week using MAX and CASE:

	 select max(case dw when 2 then dm end) as Mo,
	        max(case dw when 3 then dm end) as Tu,
	        max(case dw when 4 then dm end) as We,
	        max(case dw when 5 then dm end) as Th,
	        max(case dw when 6 then dm end) as Fr,
	        max(case dw when 7 then dm end) as Sa,
	        max(case dw when 1 then dm end) as Su
	   from (
	 select date_format(dy,'%u') wk,
	        date_format(dy,'%d') dm,
	        date_format(dy,'%w')+1 dw
	   from (
	 select adddate(x.dy,t500.id-1) dy,
	        x.mth
	   from (
	 select adddate(current_date,-dayofmonth(current_date)+1) dy,
	        date_format(
	            adddate(current_date,
	                    -dayofmonth(current_date)+1),
	                    '%m') mth
	    from t1
	         ) x,
	           t500
	  where t500.id <= 31
	    and date_format(adddate(x.dy,t500.id-1),'%m') = x.mth
	        ) y
	        ) z
	  group by wk
	  order by wk
SQL Server
Use the recursive WITH clause to return every day in the current month. Then pivot on the day of the week using CASE and MAX:

	   with x(dy,dm,mth,dw,wk)
	     as (
	 select dy,
	        day(dy) dm,
	        datepart(m,dy) mth,
	        datepart(dw,dy) dw,
	        case when datepart(dw,dy) = 1
	             then datepart(ww,dy)-1
	             else datepart(ww,dy)
	        end wk
	   from (
	 select dateadd(day,-day(getdate())+1,getdate()) dy
	   from t1
	        ) x
	  union all
	  select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth,
	         datepart(dw,dateadd(d,1,dy)),
	         case when datepart(dw,dateadd(d,1,dy)) = 1
	              then datepart(wk,dateadd(d,1,dy))-1
	              else datepart(wk,dateadd(d,1,dy))
	         end
	    from x
	   where datepart(m,dateadd(d,1,dy)) = mth
	 )
	 select max(case dw when 2 then dm end) as Mo,
	        max(case dw when 3 then dm end) as Tu,
	        max(case dw when 4 then dm end) as We,
	        max(case dw when 5 then dm end) as Th,
	        max(case dw when 6 then dm end) as Fr,
	        max(case dw when 7 then dm end) as Sa,
	        max(case dw when 1 then dm end) as Su
	   from x
	  group by wk
	  order by wk
####DISCUSSION
DB2
The first step is to return each day in the month for which you want to create a calendar. Do that using the recursive WITH clause (if you don’t have WITH available, you can use a pivot table, such as T500, as in the MySQL solution). Along with each day of the month (alias DM) you will need to return different parts of each date: the day of the week (alias DW), the current month you are working with (alias MTH), and the ISO week for each day of the month (alias WK). The results of the recursive view X prior to recursion taking place (the upper portion of the UNION ALL) are shown below:

	select (current_date -day(current_date) day +1 day) dy,
	       day((current_date -day(current_date) day +1 day)) dm,
	       month(current_date) mth,
	       dayofweek(current_date -day(current_date) day +1 day) dw,
	       week_iso(current_date -day(current_date) day +1 day) wk
	  from t1

	DY          DM MTH         DW WK
	----------- -- --- ---------- --
	01-JUN-2005 01  06          4 22
The next step is to repeatedly increase the value for DM (move through the days of the month) until you are no longer in the current month. As you move through each day in the month, you will also return the day of the week that each day is, and which ISO week the current day of the month falls into. Partial results are shown below:

	with x(dy,dm,mth,dw,wk)
	  as (
	select (current_date -day(current_date) day +1 day) dy,
	       day((current_date -day(current_date) day +1 day)) dm,
	       month(current_date) mth,
	       dayofweek(current_date -day(current_date) day +1 day) dw,
	       week_iso(current_date -day(current_date) day +1 day) wk
	  from t1
	 union all
	 select dy+1 day, day(dy+1 day), mth,
	        dayofweek(dy+1 day), week_iso(dy+1 day)
	   from x
	  where month(dy+1 day) = mth
	)
	select *
	  from x

	DY          DM MTH         DW WK
	----------- -- --- ---------- --
	01-JUN-2005 01 06           4 22
	02-JUN-2005 02 06           5 22
	…
	21-JUN-2005 21 06           3 25
	22-JUN-2005 22 06           4 25
	…
	30-JUN-2005 30 06           5 26
What you are returning at this point are: each day for the current month, the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun–Sat), and the two-digit ISO week each day falls into. With all this information available, you can use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown below:

	with x(dy,dm,mth,dw,wk)
	  as (
	select (current_date -day(current_date) day +1 day) dy,
	       day((current_date -day(current_date) day +1 day)) dm,
	       month(current_date) mth,
	       dayofweek(current_date -day(current_date) day +1 day) dw,
	       week_iso(current_date -day(current_date) day +1 day) wk
	  from t1
	 union all
	 select dy+1 day, day(dy+1 day), mth,
	        dayofweek(dy+1 day), week_iso(dy+1 day)
	   from x
	  where month(dy+1 day) = mth
	 )
	 select wk,
	        case dw when 2 then dm end as Mo,
	        case dw when 3 then dm end as Tu,
	        case dw when 4 then dm end as We,
	        case dw when 5 then dm end as Th,
	        case dw when 6 then dm end as Fr,
	        case dw when 7 then dm end as Sa,
	        case dw when 1 then dm end as Su
	   from x

	WK MO TU WE TH FR SA SU
	-- -- -- -- -- -- -- --
	22       01
	22          02
	22             03
	22                04
	22                   05
	23 06
	23    07
	23       08
	23          09
	23             10
	23                11
	23                   12
As you can see from the partial output, every day in each week is returned as a row. What you want to do now is to group the days by week, and then collapse all the days for each week into a single row. Use the aggregate function MAX, and group by WK (the ISO week) to return all the days for a week as one row. To properly format the calendar and ensure that the days are in the right order, order the results by WK. The final output is shown below:

	with x(dy,dm,mth,dw,wk)
	  as (
	select (current_date -day(current_date) day +1 day) dy,
	       day((current_date -day(current_date) day +1 day)) dm,
	       month(current_date) mth,
	       dayofweek(current_date -day(current_date) day +1 day) dw,
	       week_iso(current_date -day(current_date) day +1 day) wk
	  from t1
	 union all
	 select dy+1 day, day(dy+1 day), mth,
	        dayofweek(dy+1 day), week_iso(dy+1 day)
	   from x
	  where month(dy+1 day) = mth
	)
	select max(case dw when 2 then dm end) as Mo,
	       max(case dw when 3 then dm end) as Tu,
	       max(case dw when 4 then dm end) as We,
	       max(case dw when 5 then dm end) as Th,
	       max(case dw when 6 then dm end) as Fr,
	       max(case dw when 7 then dm end) as Sa,
	       max(case dw when 1 then dm end) as Su
	  from x
	 group by wk
	 order by wk

	MO TU WE TH FR SA SU
	-- -- -- -- -- -- --
	      01 02 03 04 05
	06 07 08 09 10 11 12
	13 14 15 16 17 18 19
	20 21 22 23 24 25 26
	27 28 29 30
Oracle
Begin by using the recursive CONNECT BY clause to generate a row for each day in the month for which you wish to generate a calendar. If you aren’t running at least Oracle9i Database, you can’t use CONNECT BY this way. Instead, you can use a pivot table, such as T500 in the MySQL solution.

Along with each day of the month, you will need to return different bits of information for each day: the day of the month (alias DM), the day of the week (alias DW), the current month you are working with (alias MTH), and the ISO week for each day of the month (alias WK). The results of the WITH view X for the first day of the current month are shown below:

	select trunc(sysdate,'mm') dy,
	       to_char(trunc(sysdate,'mm'),'dd') dm,
	       to_char(sysdate,'mm') mth,
	       to_number(to_char(trunc(sysdate,'mm'),'d')) dw,
	       to_char(trunc(sysdate,'mm'),'iw') wk
	  from dual

	DY          DM MT         DW WK
	----------- -- -- ---------- --
	01-JUN-2005 01 06          4 22
The next step is to repeatedly increase the value for DM (move through the days of the month) until you are no longer in the current month. As you move through each day in the month, you will also return the day of the week for each day and the ISO week into which the current day falls. Partial results are shown below (the full date for each day is added below for readability):

	with x
	  as (
	select *
	  from (
	select trunc(sysdate,'mm')+level-1 dy,
	       to_char(trunc(sysdate,'mm')+level-1,'iw') wk,
	       to_char(trunc(sysdate,'mm')+level-1,'dd') dm,
	       to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw,
	       to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth,
	       to_char(sysdate,'mm') mth
	  from dual
	 connect by level <= 31
	       )
	 where curr_mth = mth
	)
	select *
	  from x

	DY          WK DM         DW CU MT
	----------- -- -- ---------- -- --
	01-JUN-2005 22 01          4 06 06
	02-JUN-2005 22 02          5 06 06
	…
	21-JUN-2005 25 21          3 06 06
	22-JUN-2005 25 22          4 06 06
	…
	30-JUN-2005 26 30          5 06 06
What you are returning at this point is one row for each day of the current month. In that row you have: the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun–Sat), and the two-digit ISO week number. With all this information available, you can use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown below:

	with x
	  as (
	select *
	  from (
	select trunc(sysdate,'mm')+level-1 dy,
	       to_char(trunc(sysdate,'mm')+level-1,'iw') wk,
	       to_char(trunc(sysdate,'mm')+level-1,'dd') dm,
	       to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw,
	       to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth,
	       to_char(sysdate,'mm') mth
	  from dual
	 connect by level <= 31
	       )
	 where curr_mth = mth
	)
	select wk,
	       case dw when 2 then dm end as Mo,
	       case dw when 3 then dm end as Tu,
	       case dw when 4 then dm end as We,
	       case dw when 5 then dm end as Th,
	       case dw when 6 then dm end as Fr,
	       case dw when 7 then dm end as Sa,
	       case dw when 1 then dm end as Su
	  from x

	WK MO TU WE TH FR SA SU
	-- -- -- -- -- -- -- --
	22       01
	22          02
	22             03
	22                04
	22                  05
	23 06
	23    07
	23       08
	23          09
	23             10
	23               11
	23                 12
As you can see from the partial output, every day in each week is returned as a row, but the day number is in one of seven columns corresponding to the day of the week. Your task now is to consolidate the days into one row for each week. Use the aggregate function MAX and group by WK (the ISO week) to return all the days for a week as one row. To ensure the days are in the right order, order the results by WK. The final output is shown below:

	with x
	  as (
	select *
	  from (
	select to_char(trunc(sysdate,'mm')+level-1,'iw') wk,
	       to_char(trunc(sysdate,'mm')+level-1,'dd') dm,
	       to_number(to_char(trunc(sysdate,'mm')+level-1,'d')) dw,
	       to_char(trunc(sysdate,'mm')+level-1,'mm') curr_mth,
	       to_char(sysdate,'mm') mth
	  from dual
	 connect by level <= 31
	       )
	 where curr_mth = mth
	)
	select max(case dw when 2 then dm end) Mo,
	       max(case dw when 3 then dm end) Tu,
	       max(case dw when 4 then dm end) We,
	       max(case dw when 5 then dm end) Th,
	       max(case dw when 6 then dm end) Fr,
	       max(case dw when 7 then dm end) Sa,
	       max(case dw when 1 then dm end) Su
	  from x
	 group by wk
	 order by wk

	MO TU WE TH FR SA SU
	-- -- -- -- -- -- --
	   01 02 03 04 05
	06 07 08 09 10 11 12
	13 14 15 16 17 18 19
	20 21 22 23 24 25 26
	27 28 29 30
PostgreSQL
Use the GENERATE_SERIES function to return one row for each day in the month. If your version of PostgreSQL doesn’t support GENERATE_SERIES, then query a pivot table as shown in the MySQL solution.

For each day of the month, return the following information: the day of the month (alias DM), the day of the week (alias DW), the current month you are working with (alias MTH), and the ISO week for each day of the month (alias WK). The formatting and explicit casting makes this solution tough on the eyes, but it’s really quite simple. Partial results from inline view X are shown below:

	select cast(date_trunc('month',current_date) as date)+x.id as dy,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'iw') as wk,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'dd') as dm,
	       cast(
	    to_char(
	       cast(
	 date_trunc('month',current_date)
	               as date)+x.id,'d') as integer) as dw,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'mm') as curr_mth,
	       to_char(current_date,'mm') as mth
	  from generate_series (0,31) x(id)

	DY          WK DM         DW CU MT
	----------- -- -- ---------- -- --
	01-JUN-2005 22 01          4 06 06
	02-JUN-2005 22 02          5 06 06
	…
	21-JUN-2005 25 21          3 06 06
	22-JUN-2005 25 22          4 06 06
	…
	30-JUN-2005 26 30          5 06 06
Notice that as you move through each day in the month, you will also return the day of the week and the ISO week number. To ensure you return days only for the month you are interested in, return only rows where CURR_MTH = MTH (the month each day belongs to should be the month the current date belongs to). What you are returning at this point is, for each day for the current month: the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun – Sat), and the two-digit ISO week. Your next step is to use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown below:

	select case dw when 2 then dm end as Mo,
	       case dw when 3 then dm end as Tu,
	       case dw when 4 then dm end as We,
	       case dw when 5 then dm end as Th,
	       case dw when 6 then dm end as Fr,
	       case dw when 7 then dm end as Sa,
	       case dw when 1 then dm end as Su
	  from (
	select *
	  from (
	select cast(date_trunc('month',current_date) as date)+x.id,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'iw') as wk,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	                as date)+x.id,'dd') as dm,
	       cast(
	    to_char(
	       cast(
	 date_trunc('month',current_date)
	               as date)+x.id,'d') as integer) as dw,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'mm') as curr_mth,
	       to_char(current_date,'mm') as mth
	  from generate_series (0,31) x(id)
	       ) x
	 where mth = curr_mth
	       ) y
	
	WK MO TU WE TH FR SA SU
	-- -- -- -- -- -- -- --
	22       01
	22          02
	22             03
	22                04
	22                   05
	23 06
	23    07
	23       08
	23          09
	23             10
	23                11
	23                   12
As you can see from the partial output, every day in each week is returned as a row, and each day number falls into the column corresponding to its day of the week. Your job now is to collapse the days into one row for each week. To that end, use the aggregate function MAX and group the rows by WK (the ISO week). The result will be all the days for each week returned as one row as you would see on a calendar. To ensure the days are in the right order, order the results by WK. The final output is shown below:

	select max(case dw when 2 then dm end) as Mo,
	       max(case dw when 3 then dm end) as Tu,
	       max(case dw when 4 then dm end) as We,
	       max(case dw when 5 then dm end) as Th,
	       max(case dw when 6 then dm end) as Fr,
	       max(case dw when 7 then dm end) as Sa,
	       max(case dw when 1 then dm end) as Su
	  from (
	select *
	  from (
	select cast(date_trunc('month',current_date) as date)+x.id,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'iw') as wk,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'dd') as dm,
	       cast(
	    to_char(
	       cast(
	 date_trunc('month',current_date)
	               as date)+x.id,'d') as integer) as dw,
	       to_char(
	          cast(
	    date_trunc('month',current_date)
	               as date)+x.id,'mm') as curr_mth,
	       to_char(current_date,'mm') as mth
	  from generate_series (0,31) x(id)
	       ) x
	 where mth = curr_mth
	       ) y
	 group by wk
	 order by wk

	MO TU WE TH FR SA SU
	-- -- -- -- -- -- --
	      01 02 03 04 05
	06 07 08 09 10 11 12
	13 14 15 16 17 18 19
	20 21 22 23 24 25 26
	27 28 29 30
MySQL
The first step is to return a row for each day in the month for which you want to create a calendar. To that end, query against table T500. By adding each value returned by T500 to the first day of the month, you can return each day in the month.

For each date, you will need to return the following bits of information: the day of the month (alias DM), the day of the week (alias DW), the current month you are working with (alias MTH), and the ISO week for each day of the month (alias WK). Inline view X returns the first day of the current month along with the two-digit numeric value for the current month. Results are shown below:

	select adddate(current_date,-dayofmonth(current_date)+1) dy,
	       date_format(
	       adddate(current_date,
	               -dayofmonth(current_date)+1),
	               '%m') mth
	  from t1

	DY          MT
	----------- --
	01-JUN-2005 06
The next step is to move through the month, starting from the first day and returning each day in the month. Notice that as you move through each day in the month, you will also return the corresponding day of the week and ISO week number. To ensure you return days only for the month you are interested in, return only rows where the month of the day returned is equal to the current month (the month each day belongs to should be the month the current date belongs to). A portion of the rows from inline view Y is shown below:

	select date_format(dy,'%u') wk,
	       date_format(dy,'%d') dm,
	       date_format(dy,'%w')+1 dw
	  from (
	select adddate(x.dy,t500.id-1) dy,
	       x.mth
	  from (
	select adddate(current_date,-dayofmonth(current_date)+1) dy,
	       date_format(
	           adddate(current_date,
	           -dayofmonth(current_date)+1),
	           '%m') mth
	  from t1
	       ) x,
	         t500
	 where t500.id <= 31
	   and date_format(adddate(x.dy,t500.id-1),'%m') = x.mth
	       ) y

	WK DM         DW
	-- -- ----------
	22 01          4
	22 02          5
	…
	25 21          3
	25 22          4
	…
	26 30          5
For each day for the current month you now have: the two-digit numeric day of the month (DM), the one-digit day of the week (DW), and the two-digit ISO week number (WK). Using this information, you can write a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown below:

	select case dw when 2 then dm end as Mo,
	       case dw when 3 then dm end as Tu,
	       case dw when 4 then dm end as We,
	       case dw when 5 then dm end as Th,
	       case dw when 6 then dm end as Fr,
	       case dw when 7 then dm end as Sa,
	       case dw when 1 then dm end as Su
	  from (
	select date_format(dy,'%u') wk,
	       date_format(dy,'%d') dm,
	       date_format(dy,'%w')+1 dw
	  from (
	select adddate(x.dy,t500.id-1) dy,
	       x.mth
	  from (
	select adddate(current_date,-dayofmonth(current_date)+1) dy,
	       date_format(
	           adddate(current_date,
	                   -dayofmonth(current_date)+1),
	                   '%m') mth
	   from t1
	        ) x,
	          t500
	  where t500.id <= 31
	    and date_format(adddate(x.dy,t500.id-1),'%m') = x.mth
	        ) y
	        ) z

	WK MO TU WE TH FR SA SU
	-- -- -- -- -- -- -- --
	22       01
	22          02
	22             03
	22                04
	22                   05
	23 06
	23    07
	23       08
	23          09
	23             10
	23                11
	23                   12
As you can see from the partial output, every day in each week is returned as a row. Within each row, the day number falls into the column corresponding to the appropriate weekday. Now you need to consolidate the days into one row for each week. To do that, use the aggregate function MAX, and group the rows by WK (the ISO week). To ensure the days are in the right order, order the results by WK. The final output is shown below:

	select max(case dw when 2 then dm end) as Mo,
	       max(case dw when 3 then dm end) as Tu,
	       max(case dw when 4 then dm end) as We,
	       max(case dw when 5 then dm end) as Th,
	       max(case dw when 6 then dm end) as Fr,
	       max(case dw when 7 then dm end) as Sa,
	       max(case dw when 1 then dm end) as Su
	  from (
	select date_format(dy,'%u') wk,
	       date_format(dy,'%d') dm,
	       date_format(dy,'%w')+1 dw
	  from (
	select adddate(x.dy,t500.id-1) dy,
	       x.mth
	  from (
	select adddate(current_date,-dayofmonth(current_date)+1) dy,
	       date_format(
	           adddate(current_date,
	                   -dayofmonth(current_date)+1),
	                   '%m') mth
	  from t1
	      ) x,
	         t500
	 where t500.id <= 31
	  and date_format(adddate(x.dy,t500.id-1),'%m') = x.mth
	      ) y
	      ) z
	 group by wk
	 order by wk

	MO TU WE TH FR SA SU
	-- -- -- -- -- -- --
	      01 02 03 04 05
	06 07 08 09 10 11 12
	13 14 15 16 17 18 19
	20 21 22 23 24 25 26
	27 28 29 30
SQL Server
Begin by returning one row for each day of the month. You can do that using the recursive WITH clause. Or, if your version of SQL Server doesn’t support recursive WITH, you can use a pivot table in the same manner as the MySQL solution. For each row that you return, you will need the following items: the day of the month (alias DM), the day of the week (alias DW), the current month you are working with (alias MTH), and the ISO week for each day of the month (alias WK). The results of the recursive view X prior to recursion taking place (the upper portion of the UNION ALL) are shown below:

	select dy,
	       day(dy) dm,
	       datepart(m,dy) mth,
	       datepart(dw,dy) dw,
	       case when datepart(dw,dy) = 1
	            then datepart(ww,dy)-1
	            else datepart(ww,dy)
	       end wk
	  from (
	select dateadd(day,-day(getdate())+1,getdate()) dy
	  from t1
	       ) x

	DY          DM MTH         DW WK
	----------- -- --- ---------- --
	01-JUN-2005  1   6          4 23
Your next step is to repeatedly increase the value for DM (move through the days of the month) until you are no longer in the current month. As you move through each day in the month, you will also return the day of the week and the ISO week number. Partial results are shown below:

	  with x(dy,dm,mth,dw,wk)
	    as (
	select dy,
	       day(dy) dm,
	       datepart(m,dy) mth,
	       datepart(dw,dy) dw,
	       case when datepart(dw,dy) = 1
	            then datepart(ww,dy)-1
	            else datepart(ww,dy)
	       end wk
	  from (
	select dateadd(day,-day(getdate())+1,getdate()) dy
	  from t1
	       ) x
	 union all
	 select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth,
	        datepart(dw,dateadd(d,1,dy)),
	        case when datepart(dw,dateadd(d,1,dy)) = 1
	             then datepart(wk,dateadd(d,1,dy))-1
	             else datepart(wk,dateadd(d,1,dy))
	        end
	  from x
	 where datepart(m,dateadd(d,1,dy)) = mth
	)
	select *
	  from x

	DY          DM MTH         DW WK
	----------- -- --- ---------- --
	01-JUN-2005 01 06           4 23
	02-JUN-2005 02 06           5 23
	…
	21-JUN-2005 21 06           3 26
	22-JUN-2005 22 06           4 26
	…
	30-JUN-2005 30 06           5 27
You now have, for each day in the current month: the two-digit numeric day of the month, the two-digit numeric month, the one-digit day of the week (1–7 for Sun– Sat), and the two-digit ISO week number.

Now, use a CASE expression to determine which day of the week each value of DM (each day of the month) falls into. A portion of the results is shown below:

	  with x(dy,dm,mth,dw,wk)
	    as (
	select dy,
	       day(dy) dm,
	       datepart(m,dy) mth,
	       datepart(dw,dy) dw,
	       case when datepart(dw,dy) = 1
	            then datepart(ww,dy)-1
	            else datepart(ww,dy)
	       end wk
	  from (
	select dateadd(day,-day(getdate())+1,getdate()) dy
	  from t1
	       ) x
	 union all
	 select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth,
	        datepart(dw,dateadd(d,1,dy)),
	        case when datepart(dw,dateadd(d,1,dy)) = 1
	             then datepart(wk,dateadd(d,1,dy))-1
	             else datepart(wk,dateadd(d,1,dy))
	        end
	   from x
	  where datepart(m,dateadd(d,1,dy)) = mth
	)
	select case dw when 2 then dm end as Mo,
	       case dw when 3 then dm end as Tu,
	       case dw when 4 then dm end as We,
	       case dw when 5 then dm end as Th,
	       case dw when 6 then dm end as Fr,
	       case dw when 7 then dm end as Sa,
	       case dw when 1 then dm end as Su
	  from x

	WK MO TU WE TH FR SA SU
	-- -- -- -- -- -- -- --
	22       01
	22          02
	22             03
	22                04
	22                   05
	23 06
	23    07
	23       08
	23          09
	23             10
	23                11
	23                   12
Every day in each week is returned as a separate row. In each row, the column containing the day number corresponds to the day of the week. You now need to consolidate the days for each week into one row. Do that by grouping the rows by WK (the ISO week) and applying the MAX function to the different columns. The results will be in calendar format as shown below:

	with x(dy,dm,mth,dw,wk)
	    as (
	select dy,
	       day(dy) dm,
	       datepart(m,dy) mth,
	       datepart(dw,dy) dw,
	       case when datepart(dw,dy) = 1
	            then datepart(ww,dy)-1
	            else datepart(ww,dy)
	       end wk
	  from (
	select dateadd(day,-day(getdate())+1,getdate()) dy
	  from t1
	       ) x
	 union all
	 select dateadd(d,1,dy), day(dateadd(d,1,dy)), mth,
	        datepart(dw,dateadd(d,1,dy)),
	        case when datepart(dw,dateadd(d,1,dy)) = 1
	             then datepart(wk,dateadd(d,1,dy))-1
	             else datepart(wk,dateadd(d,1,dy))
	        end
	   from x
	  where datepart(m,dateadd(d,1,dy)) = mth
	)
	select max(case dw when 2 then dm end) as Mo,
	       max(case dw when 3 then dm end) as Tu,
	       max(case dw when 4 then dm end) as We,
	       max(case dw when 5 then dm end) as Th,
	       max(case dw when 6 then dm end) as Fr,
	       max(case dw when 7 then dm end) as Sa,
	       max(case dw when 1 then dm end) as Su
	  from x
	 group by wk
	 order by wk

	MO TU WE TH FR SA SU
	-- -- -- -- -- -- --
	      01 02 03 04 05
	06 07 08 09 10 11 12
	13 14 15 16 17 18 19
	20 21 22 23 24 25 26
	27 28 29 30
##9.8. Listing Quarter Start and End Dates for the Year
####PROBLEM
You want to return the start and end dates for each of the four quarters of a given year.

####SOLUTION
There are four quarters to a year, so you know you will need to generate four rows. After generating the desired number of rows, simply use the date functions supplied by your RDBMS to return the quarter the start and end dates fall into. Your goal is to produce the following result set (one again, the choice to use the current year is arbitrary):

	QTR Q_START     Q_END
	--- ----------- -----------
	  1 01-JAN-2005 31-MAR-2005
	  2 01-APR-2005 30-JUN-2005
	  3 01-JUL-2005 30-SEP-2005
	  4 01-OCT-2005 31-DEC-2005
DB2
Use table EMP and the window function ROW_NUMBER OVER to generate four rows. Alternatively, you can use the WITH clause to generate rows (as many of the recipes do), or you can query against any table with at least four rows. The following solution uses the ROW_NUMBER OVER approach:

	 select quarter(dy-1 day) QTR,
	        dy-3 month Q_start,
	        dy-1 day Q_end
	   from (
	 select (current_date -
	          (dayofyear(current_date)-1) day
	            + (rn*3) month) dy
	   from (
	 select row_number()over() rn
	   from emp
	  fetch first 4 rows only
	         ) x
	         ) y
Oracle
Use the function ADD_MONTHS to find the start and end dates for each quarter. Use ROWNUM to represent the quarter the start and end dates belong to. The following solution uses table EMP to generate four rows.

	 select rownum qtr,
	        add_months(trunc(sysdate,'y'),(rownum-1)*3) q_start,
	        add_months(trunc(sysdate,'y'),rownum*3)-1 q_end
	   from emp
	  where rownum <= 4
PostgreSQL
Use the function GENERATE_SERIES to generate the required four quarters. Use the DATE_TRUNC function to truncate the dates generated for each quarter down to year and month. Use the TO_CHAR function to determine which quarter the start and end dates belong to:

	 select to_char(dy,'Q') as QTR,
	        date(
	          date_trunc('month',dy)-(2*interval '1 month')
	        ) as Q_start,
	        dy as Q_end
	   from (
	 select date(dy+((rn*3) * interval '1 month'))-1 as dy
	   from (
	 select rn, date(date_trunc('year',current_date)) as dy
	   from generate_series(1,4) gs(rn)
	        ) x
	        ) y
MySQL
Use table T500 to generate four rows (one for each quarter). Use functions DATE_ ADD and ADDDATE to create the start and end dates for each quarter. Use the QUARTER function to determine which quarter the start and end dates belong to:

	 select quarter(adddate(dy,-1)) QTR,
	date_add(dy,interval -3 month) Q_start,
	        adddate(dy,-1) Q_end
	   from (
	 select date_add(dy,interval (3*id) month) dy
	   from (
	 select id,
	        adddate(current_date,-dayofyear(current_date)+1) dy
	  from t500
	 where id <= 4
	        ) x
	        ) y
SQL Server
Use the recursive WITH clause to generate four rows. Use the function DATEADD to find the start and end dates. Use the function DATEPART to determine which quarter the start and end dates belong to:

	  with x (dy,cnt)
	    as (
	 select dateadd(d,-(datepart(dy,getdate())-1),getdate()),
	        1
	   from t1
	  union all
	 select dateadd(m,3,dy), cnt+1
	   from x
	  where cnt+1 <= 4
	 )
	 select datepart(q,dateadd(d,-1,dy)) QTR,
	        dateadd(m,-3,dy) Q_start,
	        dateadd(d,-1,dy) Q_end
	   from x
	 order by 1
####DISCUSSION
DB2
The first step is to generate four rows (with values 1 through 4) for each quarter in the year. Inline view X uses the window function ROW_NUMBER OVER and the FETCH FIRST clause to return only four rows from EMP. The results are shown below:

	select row_number()over() rn
	  from emp
	 fetch first 4 rows only

	RN
	--
	 1
	 2
	 3
	 4
The next step is to find the first day of the year, then add n months to it, where n is three times RN (you are adding 3, 6, 9, and 12 months to the first day of the year). The results are shown below:

	select (current_date
	        (dayofyear(current_date)-1) day
	          + (rn*3) month) dy 
	  from (
	select row_number()over() rn
	  from emp
	 fetch first 4 rows only
	       ) x

	DY
	-----------
	01-APR-2005
	01-JUL-2005
	01-OCT-2005
	01-JAN-2005
At this point, the values for DY are one day after the end date for each quarter. The next step is to get the start and end dates for each quarter. Subtract one day from DY to get the end of each quarter, and subtract three months from DY to get the start of each quarter. Use the QUARTER function on DY-1 (the end date for each quarter) to determine which quarter the start and end dates belong to.

Oracle
The combination of ROWNUM, TRUNC, and ADD_MONTHS makes this solution very easy. To find the start of each quarter simply add n months to the first day of the year, where n is (ROWNUM-1)*3 (giving you 0,3,6,9). To find the end of each quarter add n months to the first day of the year, where n is ROWNUM*3, and subtract one day. As an aside, when working with quarters, you may also find it useful to use TO_CHAR and/or TRUNC with the ‘q’ formatting option.

PostgreSQL
The first step is to truncate the current date to the first day of the year using the DATE_TRUNC function. Next, add n months, where n is RN (the values returned by GENERATE_SERIES) times three, and subtract one day. The results are shown below:

	select date(dy+((rn*3) * interval '1 month'))-1 as dy
	  from (
	select rn, date(date_trunc('year',current_date)) as dy
	  from generate_series(1,4) gs(rn)
	       ) x

	DY
	-----------
	31-MAR-2005
	30-JUN-2005
	30-SEP-2005
	31-DEC-2005
Now that you have the end dates for each quarter, the final step is to find the start date by subtracting two months from DY then truncating to the first day of the month by using the DATE_TRUNC function. Use the TO_CHAR function on the end date for each quarter (DY) to determine which quarter the start and end dates belong to.

MySQL
The first step is to find the first day of the year by using functions ADDDATE and DAYOFYEAR, then adding n months to the first day of the year, where n is T500.ID times three, by using the DATE_ADD function. The results are shown below:

	select date_add(dy,interval (3*id) month) dy
	  from (
	select id,
	       adddate(current_date,-dayofyear(current_date)+1) dy
	  from t500
	 where id <= 4
	       ) x
 	DY
	-----------
	01-APR-2005
	01-JUL-2005
	01-OCT-2005
	01-JAN-2005
At this point the dates are one day after the end of each quarter; to find the end of each quarter, simply subtract one day from DY. The next step is to find the start of each quarter by subtracting three months from DY. Use the QUARTER function on the end date of each quarter to determine which quarter the start and end dates belong to.

SQL Server
The first step is to find the first day of the year, then recursively add n months, where n is three times the current iteration (there are four iterations, therefore, you are adding 3*1 months, 3*2 months, etc.), using the DATEADD function. The results are shown below:

	with x (dy,cnt)
	   as (
	select dateadd(d,-(datepart(dy,getdate())-1),getdate()),
	       1
	  from t1
	 union all
	select dateadd(m,3,dy), cnt+1
	  from x
	 where cnt+1 <= 4
	)
	select dy
	  from x

	DY
	-----------
	01-APR-2005
	01-JUL-2005
	01-OCT-2005
	01-JAN-2005
The values for DY are one day after the end of each quarter. To get the end of each quarter, simply subtract one day from DY by using the DATEADD function. To find the start of each quarter, use the DATEADD function to subtract three months from DY. Use the DATEPART function on the end date for each quarter to determine which quarter the start and end dates belong to.

##9.9. Determining Quarter Start and End Dates for a Given Quarter
####PROBLEM
When given a year and quarter in the format of YYYYQ (four-digit year, one-digit quarter), you want to return the quarter’s start and end dates.

####SOLUTION
The key to this solution is to find the quarter by using the modulus function on the YYYYQ value. (As an alternative to modulo, since the year format is four digits, you can simply substring out the last digit to get the quarter.) Once you have the quarter, simply multiply by 3 to get the ending month for the quarter. In the solutions that follow, inline view X will return all four year and quarter combinations. The result set for inline view X is as follows:

	select 20051 as yrq from t1 union all
	select 20052 as yrq from t1 union all
	select 20053 as yrq from t1 union all
	select 20054 as yrq from t1
	    YRQ
	-------
	  20051
	  20052
	  20053
	  20054
DB2
Use the function SUBSTR to return the year from inline view X. Use the MOD function to determine which quarter you are looking for:

	 select (q_end-2 month) q_start,
	        (q_end+1 month)-1 day q_end
	   from (
	 select date(substr(cast(yrq as char(4)),1,4) ||'-'||
	        rtrim(cast(mod(yrq,10)*3 as char(2))) ||'-1') q_end
	   from (
	 select 20051 yrq from t1 union all
	 select 20052 yrq from t1 union all
	 select 20053 yrq from t1 union all
	 select 20054 yrq from t1
	        ) x
	        ) y
Oracle
Use the function SUBSTR to return the year from inline view X. Use the MOD function to determine which quarter you are looking for:

	 select add_months(q_end,-2) q_start,
	        last_day(q_end) q_end
	   from (
	 select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') q_end
	   from (
	 select 20051 yrq from dual union all
	 select 20052 yrq from dual union all
	 select 20053 yrq from dual union all
	 select 20054 yrq from dual
	        ) x
	        ) y
PostgreSQL
Use the function SUBSTR to return the year from the inline view X. Use the MOD function to determine which quarter you are looking for:

	 select date(q_end-(2*interval '1 month')) as q_start,
	        date(q_end+interval '1 month'-interval '1 day') as q_end
	   from (
	 select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') as q_end
	   from (
	 select 20051 as yrq from t1 union all
	 select 20052 as yrq from t1 union all
	 select 20053 as yrq from t1 union all
	 select 20054 as yrq from t1
	        ) x
	        ) y
MySQL
Use the function SUBSTR to return the year from the inline view X. Use the MOD function to determine which quarter you are looking for:

	 select date_add(
	         adddate(q_end,-day(q_end)+1),
	                interval -2 month) q_start,
	        q_end
	   from (
	 select last_day(
	     str_to_date(
	          concat(
	          substr(yrq,1,4),mod(yrq,10)*3),'%Y%m')) q_end
	   from (
	 select 20051 as yrq from t1 union all
	 select 20052 as yrq from t1 union all
	 select 20053 as yrq from t1 union all
	 select 20054 as yrq from t1
	        ) x
	        ) y
SQL Server
Use the function SUBSTRING to return the year from the inline view X. Use the modulus function (%) to determine which quarter you are looking for:

	 select dateadd(m,-2,q_end) q_start,
	        dateadd(d,-1,dateadd(m,1,q_end)) q_end
	   from (
	 select cast(substring(cast(yrq as varchar),1,4)+'-'+
	        cast(yrq%10*3 as varchar)+'-1' as datetime) q_end
	   from (
	 select 20051 as yrq from t1 union all   
	 select 20052 as yrq from t1 union all   
	 select 20052 as yrq from t1 union all     
	 select 20054 as yrq from t1
	        ) x
	        ) y
####DISCUSSION
DB2
The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by 3 to get the end month for the quarter. The results are shown below:

	select substr(cast(yrq as char(4)),1,4) yr,
	       mod(yrq,10)*3 mth
	  from (
	select 20051 yrq from t1 union all
	select 20052 yrq from t1 union all
	select 20053 yrq from t1 union all
	select 20054 yrq from t1
	       ) x
	YR      MTH
	---- ------
	2005      3
	2005      6
	2005      9
	2005     12
At this point you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator “||” to glue together the year and month, then use the DATE function to convert to a date:

	select date(substr(cast(yrq as char(4)),1,4) ||'-'||
	       rtrim(cast(mod(yrq,10)*3 as char(2))) ||'-1') q_end
	  from (
	select 20051 yrq from t1 union all
	select 20052 yrq from t1 union all
	select 20053 yrq from t1 union all
	select 20054 yrq from t1
	       ) x

	Q_END
	-----------
	01-MAR-2005
	01-JUN-2005
	01-SEP-2005
	01-DEC-2005
The values for Q_END are the first day of the last month of each quarter. To get to the last day of the month add one month to Q_END, then subtract one day. To find the start date for each quarter subtract two months from Q_END.

Oracle
The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by 3 to get the end month for the quarter. The results are shown below:

	select substr(yrq,1,4) yr, mod(yrq,10)*3 mth
	  from (
	select 20051 yrq from dual union all
	select 20052 yrq from dual union all
	select 20053 yrq from dual union all
	select 20054 yrq from dual
	       ) x
	YR      MTH
	---- ------
	2005      3
	2005      6
	2005      9
	2005     12
At this point you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator “||” to glue together the year and month, then use the TO_DATE function to convert to a date:

	select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') q_end
	  from (
	select 20051 yrq from dual union all
	select 20052 yrq from dual union all
	select 20053 yrq from dual union all
	select 20054 yrq from dual
	       ) x
	Q_END
	-----------
	01-MAR-2005
	01-JUN-2005
	01-SEP-2005
	01-DEC-2005
The values for Q_END are the first day of the last month of each quarter. To get to the last day of the month use the LAST_DAY function on Q_END. To find the start date for each quarter subtract two months from Q_END using the ADD_MONTHS function.

PostgreSQL
The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by 3 to get the end month for the quarter. The results are shown below:

	select substr(yrq,1,4) yr, mod(yrq,10)*3 mth
	  from (
	select 20051 yrq from dual union all
	select 20052 yrq from dual union all
	select 20053 yrq from dual union all
	select 20054 yrq from dual
	       ) x
	YR        MTH
	----  -------
	2005        3
	2005        6
	2005        9
	2005       12
At this point you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator “||” to glue together the year and month, then use the TO_ DATE function to convert to a date:

	select to_date(substr(yrq,1,4)||mod(yrq,10)*3,'yyyymm') q_end
	  from (
	select 20051 yrq from dual union all
	select 20052 yrq from dual union all
	select 20053 yrq from dual union all
	select 20054 yrq from dual
	       ) x

	Q_END
	-----------
	01-MAR-2005
	01-JUN-2005
	01-SEP-2005
	01-DEC-2005
The values for Q_END are the first day of the last month of each quarter. To get to the last day of the month add one month to Q_END and subtract one day. To find the start date for each quarter subtract two months from Q_END. Cast the final result as dates.

MySQL
The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTR function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by 3 to get the end month for the quarter. The results are shown below:

	select substr(yrq,1,4) yr, mod(yrq,10)*3 mth
	  from (
	select 20051 yrq from dual union all
	select 20052 yrq from dual union all
	select 20053 yrq from dual union all
	select 20054 yrq from dual
	       ) x

	YR      MTH
	---- ------
	2005      3
	2005      6
	2005      9
	2005     12
At this point you have the year and end month for each quarter. Use those values to construct a date, specifically, the last day of each quarter. Use the CONCAT function to glue together the year and month, then use the STR_TO_DATE function to convert to a date. Use the LAST_DAY function to find the last day for each quarter:

	select last_day(
	    str_to_date(
	         concat(
	         substr(yrq,1,4),mod(yrq,10)*3),'%Y%m')) q_end
	  from (
	select 20051 as yrq from t1 union all
	select 20052 as yrq from t1 union all
	select 20053 as yrq from t1 union all
	select 20054 as yrq from t1
	       ) x

	Q_END
	-----------
	31-MAR-2005
	30-JUN-2005
	30-SEP-2005
	31-DEC-2005
Because you already have the end of each quarter, all that’s left is to find the start date for each quarter. Use the DAY function to return the day of the month the end of each quarter falls on, and subtract that from Q_END using the ADDDATE function to give you the end of the prior month; add one day to bring you to the first day of the last month of each quarter. The last step is to use the DATE_ADD function to subtract two months from the first day of the last month of each quarter to get you to the start date for each quarter.

SQL Server
The first step is to find the year and quarter you are working with. Substring out the year from inline view X (X.YRQ) using the SUBSTRING function. To get the quarter, use modulus 10 on YRQ. Once you have the quarter, multiply by 3 to get the end month for the quarter. The results are shown below:

	select substring(yrq,1,4) yr, yrq%10*3 mth
	  from (
	select 20051 yrq from dual union all
	select 20052 yrq from dual union all
	select 20053 yrq from dual union all
	select 20054 yrq from dual
	       ) x

	YR        MTH
	----   ------
	2005        3
	2005        6
	2005        9
	2005       12
At this point, you have the year and end month for each quarter. Use those values to construct a date, specifically, the first day of the last month for each quarter. Use the concatenation operator “+” to glue together the year and month, then use the CAST function to convert to a date:

	select cast(substring(cast(yrq as varchar),1,4)+'-'+
	       cast(yrq%10*3 as varchar)+'-1' as datetime) q_end
	  from (
	select 20051 yrq from t1 union all
	select 20052 yrq from t1 union all
	select 20053 yrq from t1 union all
	select 20054 yrq from t1
	       ) x

	Q_END
	-----------
	01-MAR-2005
	01-JUN-2005
	01-SEP-2005
	01-DEC-2005
The values for Q_END are the first day of the last month of each quarter. To get to the last day of the month add one month to Q_END and subtract one day using the DATEADD function. To find the start date for each quarter subtract two months from Q_END using the DATEADD function.

##9.10. Filling in Missing Dates
####PROBLEM
You need to generate a row for every date (or every month, week, or year) within a given range. Such rowsets are often used to generate summary reports. For example, you want to count the number of employees hired every month of every year in which any employee has been hired. Examining the dates of all the employees hired, there have been hirings from 1980 to 1983:

	select distinct
	       extract(year from hiredate) as year
	  from emp

	YEAR
	-----
	 1980
	 1981
	 1982
	 1983
You want to determine the number of employees hired each month from 1980 to 1983. A portion of the desired result set is shown below:

	MTH          NUM_HIRED
	----------- ----------
	01-JAN-1981          0
	01-FEB-1981          2
	01-MAR-1981          0
	01-APR-1981          1
	01-MAY-1981          1
	01-JUN-1981          1
	01-JUL-1981          0
	01-AUG-1981          0
	01-SEP-1981          2
	01-OCT-1981          0
	01-NOV-1981          1
	01-DEC-1981          2
####SOLUTION
The trick here is that you want to return a row for each month even if no employee was hired (i.e., the count would be zero). Because there isn’t an employee hired every month between 1980 and 1983, you must generate those months yourself, and then outer join to table EMP on HIREDATE (truncating the actual HIREDATE to its month, so it can match the generated months when possible).

DB2
Use the recursive WITH clause to generate every month (the first day of each month from January 1, 1980, to December 1, 1983). Once you have all the months for the required range of dates, outer join to table EMP and use the aggregate function COUNT to count the number of hires for each month:

	   with x (start_date,end_date)
	     as (
	 select (min(hiredate)
	          dayofyear(min(hiredate)) day +1 day) start_date,
	        (max(hiredate)
	          dayofyear(max(hiredate)) day +1 day) +1 year end_date
	   from emp
	  union all
	 select start_date +1 month, end_date
	   from x
	  where (start_date +1 month) < end_date
	 )
	 select x.start_date mth, count(e.hiredate) num_hired
	   from x left join emp e
	     on (x.start_date = (e.hiredate-(day(hiredate)-1) day))
	  group by x.start_date
	  order by 1
Oracle
Use the CONNECT BY clause to generate each month between 1980 and 1983. Then outer join to table EMP and use the aggregate function COUNT to count the number of employees hired in each month. If you are on Oracle8i Database and earlier, the ANSI outer join is not available to you, nor is the ability to use CONNECT BY as a row generator; a simple workaround is to use a traditional pivot table (like the one used in the MySQL solution). Following as an Oracle solution using Oracle’s outer-join syntax:

	   with x
	     as (
	 select add_months(start_date,level-1) start_date
	   from (
	 select min(trunc(hiredate,'y')) start_date,
	        add_months(max(trunc(hiredate,'y')),12) end_date
	   from emp
	        )
	  connect by level <= months_between(end_date,start_date)
	 )
	 select x.start_date MTH, count(e.hiredate) num_hired
	   from x, emp e
	  where x.start_date = trunc(e.hiredate(+),'mm')
	  group by x.start_date
	  order by 1
and here is a second Oracle solution, this time using the ANSI syntax:

	   with x
	     as (
	 select add_months(start_date,level-1) start_date
	   from (
	 select min(trunc(hiredate,'y')) start_date,
	        add_months(max(trunc(hiredate,'y')),12) end_date
	   from emp
	        )
	  connect by level <= months_between(end_date,start_date)
	 )
	 select x.start_date MTH, count(e.hiredate) num_hired
	   from x left join emp e
	     on (x.start_date = trunc(e.hiredate,'mm'))
	  group by x.start_date
	  order by 1
PostgreSQL
To improve readability, this solution uses a view, named V, to return the number of months between the first day of the first month of the year the first employee was hired and the first day of the last month of the year the most recent employee was hired. Use the value returned by view V as the second value passed to the function GENERATE_SERIES, so that the correct number of months (rows) are generated. Once you have all the months for the required range of dates, outer join to table EMP and use the aggregate function COUNT to count the number of hires for each month:

	create view v
	as
	select cast(
	        extract(year from age(last_month,first_month))*12-1
	          as integer) as mths
	  from (
	select cast(date_trunc('year',min(hiredate)) as date) as first_month,
	       cast(cast(date_trunc('year',max(hiredate))
	             as date) + interval '1 year'
	             as date) as last_month
	  from emp
	       ) x


	select y.mth, count(e.hiredate) as num_hired
	  from (
	select cast(e.start_date + (x.id * interval '1 month')
	         as date) as mth
	  from generate_series (0,(select mths from v)) x(id),
	       ( select cast(
	                 date_trunc('year',min(hiredate))
	                   as date) as start_date
	           from emp ) e
	        ) y left join emp e
	     on (y.mth = date_trunc('month',e.hiredate))
	 group by y.mth
	 order by 1
MySQL
Use the pivot table T500 to generate each month between 1980 and 1983. Then outer join to table EMP and use the aggregate function COUNT to count the number of employees hired for each month:

	select z.mth, count(e.hiredate) num_hired
	  from (
	select date_add(min_hd,interval t500.id-1 month) mth
	  from (
	select min_hd, date_add(max_hd,interval 11 month) max_hd
	  from (
	select adddate(min(hiredate),-dayofyear(min(hiredate))+1) min_hd,
	       adddate(max(hiredate),-dayofyear(max(hiredate))+1) max_hd
	  from emp
	        ) x
	        ) y,
	        t500
	 where date_add(min_hd,interval t500.id-1 month) <= max_hd
	       ) z left join emp e
	    on (z.mth = adddate(
	               date_add(
	               last_day(e.hiredate),interval -1 month),1))
	 group by z.mth
	 order by 1
SQL Server
Use the recursive WITH clause to generate every month (the first day of each month from January 1, 1980, to December 1, 1983). Once you have all the months for the required range of dates, outer join to table EMP and use the aggregate function COUNT to count the number of hires for each month:

	  with x (start_date,end_date)
	    as (
	select (min(hiredate)
	         datepart(dy,min(hiredate))+1) start_date,
	       dateadd(yy,1,
	        (max(hiredate)
	         datepart(dy,max(hiredate))+1)) end_date
	  from emp
	 union all
	 select dateadd(mm,1,start_date), end_date
	   from x
	  where dateadd(mm,1,start_date) < end_date
	 )
	 select x.start_date mth, count(e.hiredate) num_hired
	   from x left join emp e
	     on (x.start_date =
	            dateadd(dd,-day(e.hiredate)+1,e.hiredate))
	 group by x.start_date
	 order by 1
####DISCUSSION
DB2
The first step is to generate every month (actually the first day of each month) from 1980 to 1983. Start using the DAYOFYEAR function on the MIN and MAX HIREDATEs to find the boundary months:

	select (min(hiredate)
	         dayofyear(min(hiredate)) day +1 day) start_date,
	       (max(hiredate)
	         dayofyear(max(hiredate)) day +1 day) +1 year end_date
	  from emp

	START_DATE  END_DATE
	----------- -----------
	01-JAN-1980 01-JAN-1984
Your next step is to repeatedly add months to START_DATE to return all the months necessary for the final result set. The value for END_DATE is one day more than it should be. This is OK. As you recursively add months to START_DATE, you can stop before you hit END_DATE. A portion of the months created is shown below:

	with x (start_date,end_date)
	  as (
	select (min(hiredate)
	         dayofyear(min(hiredate)) day +1 day) start_date,
	       (max(hiredate)
	         dayofyear(max(hiredate)) day +1 day) +1 year end_date
	  from emp
	 union all
	select start_date +1 month, end_date
	  from x
	 where (start_date +1 month) < end_date
	)
	select *
	  from x

	START_DATE  END_DATE
	----------- -----------
	01-JAN-1980 01-JAN-1984
	01-FEB-1980 01-JAN-1984
	01-MAR-1980 01-JAN-1984
	…
	01-OCT-1983 01-JAN-1984
	01-NOV-1983 01-JAN-1984
	01-DEC-1983 01-JAN-1984
At this point, you have all the months you need, and you can simply outer join to EMP.HIREDATE. Because the day for each START_DATE is the first of the month, truncate EMP.HIREDATE to the first day of its month. Finally, use the aggregate function COUNT on EMP.HIREDATE.

Oracle
The first step is to generate the first day of every for every month from 1980 to 1983. Start by using TRUNC and ADD_MONTHS together with the MIN and MAX HIREDATE values to find the boundary months:

	select min(trunc(hiredate,'y')) start_date,
	       add_months(max(trunc(hiredate,'y')),12) end_date
	  from emp

	START_DATE  END_DATE
	----------- -----------
	01-JAN-1980 01-JAN-1984
Then repeatedly add months to START_DATE to return all the months necessary for the final result set. The value for END_DATE is one day more than it should be, which is OK. As you recursively add months to START_DATE, you can stop before you hit END_DATE. A portion of the months created is shown below:

	with x as (
	select add_months(start_date,level-1) start_date
	  from (
	select min(trunc(hiredate,'y')) start_date,
	       add_months(max(trunc(hiredate,'y')),12) end_date
	  from emp
	       )
	 connect by level <= months_between(end_date,start_date)
	)
	select *
	  from x

	START_DATE
	-----------
	01-JAN-1980
	01-FEB-1980
	01-MAR-1980
	…
	01-OCT-1983
	01-NOV-1983
	01-DEC-1983
At this point, you have all the months you need; simply outer join to EMP.HIREDATE. Because the day for each START_DATE is the first of the month, truncate EMP.HIREDATE to the first day of the month it is in. The final step is to use the aggregate function COUNT on EMP.HIREDATE.

PostgreSQL
This solution uses the function GENERATE_SERIES to return the months you need. If you do not have the GENERATE_SERIES function available, you can use a pivot table as in the MySQL solution. The first step is to understand view V. View V simply finds the number of months you’ll need to generate by finding the boundary dates for the range. Inline view X in view V uses the MIN and MAX HIREDATEs to find the start and end boundary dates and is shown below:

	select cast(date_trunc('year',min(hiredate)) as date) as first_month,
	       cast(cast(date_trunc('year',max(hiredate))
	             as date) + interval '1 year'
	             as date) as last_month
	  from emp

	FIRST_MONTH LAST_MONTH
	----------- -----------
	01-JAN-1980 01-JAN-1984
The value for LAST_MONTH is actually one day more than it should be. This is fine, as you can just subtract 1 when you calculate the months between these two dates. The next step is to use the AGE function to find the difference between the two dates in years, then multiply by 12 (and remember, subtract by 1!):

	select cast(
	        extract(year from age(last_month,first_month))*12-1
	          as integer) as mths
	  from (
	select cast(date_trunc('year',min(hiredate)) as date) as first_month,
	       cast(cast(date_trunc('year',max(hiredate))
	             as date) + interval '1 year'
	             as date) as last_month
	  from emp
	       ) x

	MTHS
	----
	  47
Use the value returned by view V as the second parameter of GENERATE_SERIES to return the number of months you need. Your next step is then to find your start date. You’ll repeatedly add months to your start date to create your range of months. Inline view Y uses the DATE_TRUNC function on the MIN(HIREDATE) to find the start date, and uses the values returned by GENERATE_SERIES to add months. Partial results are shown below:

	select cast(e.start_date + (x.id * interval '1 month')
	        as date) as mth
	  from generate_series (0,(select mths from v)) x(id),
	       ( select cast(
	                 date_trunc('year',min(hiredate))
	                   as date) as start_date
	           from emp
	       )  e

	MTH
	-----------
	01-JAN-1980
	01-FEB-1980
	01-MAR-1980
	…
	01-OCT-1983
	01-NOV-1983
	01-DEC-1983
Now that you have each month you need for the final result set, outer join to EMP. HIREDATE and use the aggregate function COUNT to count the number of hires for each month.

MySQL
First, find the boundary dates by using the aggregate functions MIN and MAX along with the DAYOFYEAR and ADDDATE functions. The result set shown below is from inline view X:

	select adddate(min(hiredate),-dayofyear(min(hiredate))+1) min_hd,
	       adddate(max(hiredate),-dayofyear(max(hiredate))+1) max_hd
	  from emp

	MIN_HD      MAX_HD
	----------- -----------
	01-JAN-1980 01-JAN-1983
Next, increment MAX_HD to the last month of the year:

	select min_hd, date_add(max_hd,interval 11 month) max_hd
	  from (
	select adddate(min(hiredate),-dayofyear(min(hiredate))+1) min_hd,
	       adddate(max(hiredate),-dayofyear(max(hiredate))+1) max_hd
	  from emp
	       ) x

	MIN_HD      MAX_HD
	----------- -----------
	01-JAN-1980 01-DEC-1983
Now that you have the boundary dates, add months to MIN_HD up to and including MAX_HD by using pivot table T500 to generate the rows you need. A portion of the results is shown below:

	select date_add(min_hd,interval t500.id-1 month) mth
	  from (
	select min_hd, date_add(max_hd,interval 11 month) max_hd
	  from (
	select adddate(min(hiredate),-dayofyear(min(hiredate))+1) min_hd,
	       adddate(max(hiredate),-dayofyear(max(hiredate))+1) max_hd
	  from emp
	       ) x
	       ) y,
	       t500
	where date_add(min_hd,interval t500.id-1 month) <= max_hd
	MTH
	-----------
	01-JAN-1980
	01-FEB-1980
	01-MAR-1980
	…
	01-OCT-1983
	01-NOV-1983
	01-DEC-1983
Now that you have all the months you need for the final result set, outer join to EMP.HIREDATE (be sure to truncate EMP.HIREDATE to the first day of the month) and use the aggregate function COUNT on EMP.HIREDATE to count the number of hires in each month.

SQL Server
Begin by generating every month (actually, the first day of each month) from 1980 to 1983. Then find the boundary months by applying the DAYOFYEAR function to the MIN and MAX HIREDATEs:

	select (min(hiredate) -
	         datepart(dy,min(hiredate))+1) start_date,
 	       dateadd(yy,1,
	        (max(hiredate) -
	         datepart(dy,max(hiredate))+1)) end_date
	  from emp

	START_DATE  END_DATE
	----------- -----------
	01-JAN-1980 01-JAN-1984
Your next step is to repeatedly add months to START_DATE to return all the months necessary for the final result set. The value for END_DATE is one day more than it should be, which is OK, as you can stop recursively adding months to START_DATE before you hit END_DATE. A portion of the months created is shown below:

	with x (start_date,end_date)
	  as (
	select (min(hiredate) -
	         datepart(dy,min(hiredate))+1) start_date,
	       dateadd(yy,1,
	        (max(hiredate) -
	         datepart(dy,max(hiredate))+1)) end_date
	  from emp
	 union all
	select dateadd(mm,1,start_date), end_date
	  from x
	 where dateadd(mm,1,start_date) < end_date
	)
	select *
	  from x

	START_DATE  END_DATE
	----------- -----------
	01-JAN-1980 01-JAN-1984
	01-FEB-1980 01-JAN-1984
	01-MAR-1980 01-JAN-1984
	…
	01-OCT-1983 01-JAN-1984
	01-NOV-1983 01-JAN-1984
	01-DEC-1983 01-JAN-1984
At this point, you have all the months you need. Simply outer join to EMP.HIREDATE. Because the day for each START_DATE is the first of the month, truncate EMP.HIREDATE to the first day of the month. The final step is to use the aggregate function COUNT on EMP.HIREDATE.

##9.11. Searching on Specific Units of Time
####PROBLEM
You want to search for dates that match a given month, or day of the week, or some other unit of time. For example, you want to find all employees hired in February or December, as well as employees hired on a Tuesday.

####SOLUTION
Use the functions supplied by your RDBMS to find month and weekday names for dates. This particular recipe can be useful in various places. Consider, if you wanted to search HIREDATEs but wanted to ignore the year by extracting the month (or any other part of the HIREDATE you are interested in), you can do so. The example solutions to this problem search by month and weekday name. By studying the date formatting functions provided by your RDBMS, you can easily modify these solutions to search by year, quarter, combination of year and quarter, month and year combination, etc.

DB2 and MySQL
Use the functions MONTHNAME and DAYNAME to find the name of the month and weekday an employee was hired, respectively:

	 select ename
	   from emp
	 where monthname(hiredate) in ('February','December')
	    or dayname(hiredate) = 'Tuesday'
Oracle and PostgreSQL
Use the function TO_CHAR to find the names of the month and weekday an employee was hired. Use the function RTRIM to remove trailing whitespaces:

	 select ename
	   from emp
	 where rtrim(to_char(hiredate,'month')) in ('february','december')
	    or rtrim(to_char(hiredate,'day')) = 'tuesday'
SQL Server
Use the function DATENAME to find the names of the month and weekday an employee was hired:

	 select ename
	   from emp
	 where datename(m,hiredate) in ('February','December')
	    or datename(dw,hiredate) = 'Tuesday'
####DISCUSSION
The key to each solution is simply knowing which functions to use and how to use them. To verify what the return values are, put the functions in the SELECT clause and examine the output. Listed below is the result set for employees in DEPTNO 10 (using SQL Server syntax):

	select ename,datename(m,hiredate) mth,datename(dw,hiredate) dw
	  from emp
	 where deptno = 10

	ENAME   MTH        DW
	------  ---------  -----------
	CLARK   June       Tuesday
	KING    November   Tuesday
	MILLER  January    Saturday
Once you know what the function(s) return, finding rows using the functions shown in each of the solutions is easy.

##9.12. Comparing Records Using Specific Parts of a Date
####PROBLEM
You want to find which employees have been hired on the same month and weekday. For example, if an employee was hired on Monday, March 10, 1988, and another employee was hired on Monday, March 2, 2001, you want those two to come up as a match since the day of week and month match. In table EMP, only three employees meet this requirement. You want to return the following result set:

	MSG
	------------------------------------------------------
	JAMES was hired on the same month and weekday as FORD
	SCOTT was hired on the same month and weekday as JAMES
	SCOTT was hired on the same month and weekday as FORD
####SOLUTION
Because you want to compare one employee’s HIREDATE with the HIREDATE of the other employees, you will need to self join table EMP. That makes each possible combination of HIREDATEs available for you to compare. Then, simply extract the weekday and month from each HIREDATE and compare.

DB2
After self joining table EMP, use the function DAYOFWEEK to return the numeric day of the week. Use the function MONTHNAME to return the name of the month:

	 select a.ename ||
	        ' was hired on the same month and weekday as '||
	        b.ename msg
	   from emp a, emp b
	 where (dayofweek(a.hiredate),monthname(a.hiredate)) =
	       (dayofweek(b.hiredate),monthname(b.hiredate))
	   and a.empno < b.empno
	 order by a.ename
Oracle and PostgreSQL
After self joining table EMP, use the TO_CHAR function to format the HIREDATE into weekday and month for comparison:

	 select a.ename ||
	        ' was hired on the same month and weekday as '||
	        b.ename as msg
	   from emp a, emp b
	 where to_char(a.hiredate,'DMON') =
	       to_char(b.hiredate,'DMON')
	   and a.empno < b.empno
	 order by a.ename
MySQL
After self joining table EMP, use the DATE_FORMAT function to format the HIREDATE into weekday and month for comparison:

	 select concat(a.ename,
	        ' was hired on the same month and weekday as ',
	        b.ename) msg
	   from emp a, emp b
	  where date_format(a.hiredate,'%w%M') =
	        date_format(b.hiredate,'%w%M')
	    and a.empno < b.empno
	 order by a.ename
SQL Server
After self joining table EMP, use the DATENAME function to format the HIREDATE into weekday and month for comparison:

	 select a.ename +
	        ' was hired on the same month and weekday as '+
	        b.ename msg
	  from emp a, emp b
	 where datename(dw,a.hiredate) = datename(dw,b.hiredate)
	   and datename(m,a.hiredate) = datename(m,b.hiredate)
	   and a.empno < b.empno
	 order by a.ename
####DISCUSSION
The only difference between the solutions is the date function used to format the HIREDATE. I’m going to use the Oracle/PostgreSQL solution in this discussion (because it’s the shortest to type out), but the explanation holds true for the other solutions as well.

The first step is to self join EMP so that each employee has access to the other employees’ HIREDATEs. Consider the results of the query below (filtered for SCOTT):

	select a.ename as scott, a.hiredate as scott_hd,
	       b.ename as other_emps, b.hiredate as other_hds
	  from emp a, emp b
	 where a.ename = 'SCOTT'
	  and a.empno != b.empno

	SCOTT      SCOTT_HD    OTHER_EMPS OTHER_HDS
	---------- ----------- ---------- -----------
	SCOTT      09-DEC-1982 SMITH      17-DEC-1980
	SCOTT      09-DEC-1982 ALLEN      20-FEB-1981
	SCOTT      09-DEC-1982 WARD       22-FEB-1981
	SCOTT      09-DEC-1982 JONES      02-APR-1981
	SCOTT      09-DEC-1982 MARTIN     28-SEP-1981
	SCOTT      09-DEC-1982 BLAKE      01-MAY-1981
	SCOTT      09-DEC-1982 CLARK      09-JUN-1981
	SCOTT      09-DEC-1982 KING       17-NOV-1981
	SCOTT      09-DEC-1982 TURNER     08-SEP-1981
	SCOTT      09-DEC-1982 ADAMS      12-JAN-1983
	SCOTT      09-DEC-1982 JAMES      03-DEC-1981
	SCOTT      09-DEC-1982 FORD       03-DEC-1981
	SCOTT      09-DEC-1982 MILLER     23-JAN-1982
By self-joining table EMP, you can compare SCOTT’s HIREDATE to the HIREDATE of all the other employees. The filter on EMPNO is so that SCOTT’s HIREDATE is not returned as one of the OTHER_HDS. The next step is to use your RDBMS’s supplied date formatting function(s) to compare the weekday and month of the HIREDATEs and keep only those that match:

	select a.ename as emp1, a.hiredate as emp1_hd,
	       b.ename as emp2, b.hiredate as emp2_hd
	  from emp a, emp b
	 where to_char(a.hiredate,'DMON') =
	       to_char(b.hiredate,'DMON')
	   and a.empno != b.empno
	 order by 1

	EMP1       EMP1_HD     EMP2       EMP2_HD
	---------- ----------- ---------- -----------
	FORD       03-DEC-1981 SCOTT      09-DEC-1982
	FORD       03-DEC-1981 JAMES      03-DEC-1981
	JAMES      03-DEC-1981 SCOTT      09-DEC-1982
	JAMES      03-DEC-1981 FORD       03-DEC-1981

	SCOTT      09-DEC-1982 JAMES      03-DEC-1981
	SCOTT      09-DEC-1982 FORD       03-DEC-1981
At this point, the HIREDATEs are correctly matched, but there are six rows in the result set rather than the three in the “Problem” section of this recipe. The reason for the extra rows is the filter on EMPNO. By using “not equals” you do not filter out the reciprocals. For example, the first row matches FORD and SCOTT and the last row matches SCOTT and FORD. The six rows in the result set are technically accurate but redundant. To remove the redundancy use “less than” (the HIREDATEs are removed to bring the intermediate queries closer to the final result set):

	select a.ename as emp1, b.ename as emp2
	  from emp a, emp b
	 where to_char(a.hiredate,'DMON') =
	       to_char(b.hiredate,'DMON')
	   and a.empno < b.empno
	 order by 1

	EMP1       EMP2
	---------- ----------
	JAMES      FORD
	SCOTT      JAMES
	SCOTT      FORD
The final step is to simply concatenate the result set to form the message.

##9.13. Identifying Overlapping Date Ranges
####PROBLEM
You want to find all instances of an employee starting a new project before ending an existing project. Consider table EMP_PROJECT:

	select *
	  from emp_project

	EMPNO ENAME      PROJ_ID PROJ_START  PROJ_END
	----- ---------- ------- ----------- -----------
	7782  CLARK            1 16-JUN-2005 18-JUN-2005
	7782  CLARK            4 19-JUN-2005 24-JUN-2005
	7782  CLARK            7 22-JUN-2005 25-JUN-2005
	7782  CLARK           10 25-JUN-2005 28-JUN-2005
	7782  CLARK           13 28-JUN-2005 02-JUL-2005
	7839  KING             2 17-JUN-2005 21-JUN-2005
	7839  KING             8 23-JUN-2005 25-JUN-2005
	7839  KING            14 29-JUN-2005 30-JUN-2005
	7839  KING            11 26-JUN-2005 27-JUN-2005
	7839  KING             5 20-JUN-2005 24-JUN-2005
	7934  MILLER           3 18-JUN-2005 22-JUN-2005
	7934  MILLER          12 27-JUN-2005 28-JUN-2005
	7934  MILLER          15 30-JUN-2005 03-JUL-2005
	7934  MILLER           9 24-JUN-2005 27-JUN-2005
	7934  MILLER           6 21-JUN-2005 23-JUN-2005
Looking at the results for employee KING, you see that KING began PROJ_ID 8 before finishing PROJ_ID 5 and began PROJ_ID 5 before finishing PROJ_ID 2. You want to return the following result set:

	EMPNO ENAME      MSG
	----- ---------- --------------------------------
	7782  CLARK      project 7 overlaps project 4
	7782  CLARK      project 10 overlaps project 7
	7782  CLARK      project 13 overlaps project 10
	7839  KING       project 8 overlaps project 5
	7839  KING       project 5 overlaps project 2
	7934  MILLER     project 12 overlaps project 9
	7934  MILLER     project 6 overlaps project 3
####SOLUTION
The key here is to find rows where PROJ_START (the date the new project starts) occurs on or after another project’s PROJ_START date and on or before that other project’s PROJ_END date. To begin, you need to be able to compare each project with each other project (for the same employee). By self joining EMP_PROJECT on employee, you generate every possible combination of two projects for each employee. To find the overlaps, simply find the rows where PROJ_START for any PROJ_ID falls between PROJ_START and PROJ_END for another PROJ_ID by the same employee.

DB2, PostgreSQL, and Oracle
Self join EMP_PROJECT. Then use the concatenation operator “||” to construct the message that explains which projects overlap:

	select a.empno,a.ename,
	       'project '||b.proj_id||
	       ' overlaps project '||a.proj_id as msg
	  from emp_project a,
	       emp_project b
	 where a.empno = b.empno
	   and b.proj_start >= a.proj_start
	   and b.proj_start <= a.proj_end
	   and a.proj_id != b.proj_id
MySQL
Self join EMP_PROJECT. Then use the CONCAT function to construct the message that explains which projects overlap:

	 select a.empno,a.ename,
	        concat('project ',b.proj_id,
	         ' overlaps project ',a.proj_id) as msg
	   from emp_project a,
	        emp_project b
	  where a.empno = b.empno
	    and b.proj_start >= a.proj_start
	    and b.proj_start <= a.proj_end
	    and a.proj_id != b.proj_id
SQL Server
Self join EMP_PROJECT. Then use the concatenation operator "+” to construct the message that explains which projects overlap:

	select a.empno,a.ename,
	       'project '+b.proj_id+
	       ' overlaps project '+a.proj_id as msg
	  from emp_project a,
	       emp_project b
	 where a.empno = b.empno
	   and b.proj_start >= a.proj_start
	   and b.proj_start <= a.proj_end
	   and a.proj_id != b.proj_id
####DISCUSSION
The only difference between the solutions lies in the string concatenation, so one discussion using the DB2 syntax will cover all three solutions. The first step is a self join of EMP_PROJECT so that the PROJ_START dates can be compared amongst the different projects. The output of the self join for employee KING is shown below. You can observe how each project can “see” the other projects:

	select a.ename,
	       a.proj_id as a_id,
	       a.proj_start as a_start,
	       a.proj_end as a_end,
	       b.proj_id as b_id,
	       b.proj_start as b_start
	  from emp_project a,
	       emp_project b
	 where a.ename = 'KING'
	   and a.empno = b.empno
	   and a.proj_id != b.proj_id
	order by 2

	ENAME  A_ID  A_START     A_END       B_ID  B_START
	------ ----- ----------- ----------- ----- -----------
	KING       2 17-JUN-2005 21-JUN-2005     8 23-JUN-2005
	KING       2 17-JUN-2005 21-JUN-2005    14 29-JUN-2005
	KING       2 17-JUN-2005 21-JUN-2005    11 26-JUN-2005
	KING       2 17-JUN-2005 21-JUN-2005     5 20-JUN-2005
	KING       5 20-JUN-2005 24-JUN-2005     2 17-JUN-2005
	KING       5 20-JUN-2005 24-JUN-2005     8 23-JUN-2005
	KING       5 20-JUN-2005 24-JUN-2005    11 26-JUN-2005
	KING       5 20-JUN-2005 24-JUN-2005    14 29-JUN-2005
	KING       8 23-JUN-2005 25-JUN-2005     2 17-JUN-2005
	KING       8 23-JUN-2005 25-JUN-2005    14 29-JUN-2005
	KING       8 23-JUN-2005 25-JUN-2005     5 20-JUN-2005
	KING       8 23-JUN-2005 25-JUN-2005    11 26-JUN-2005
	KING      11 26-JUN-2005 27-JUN-2005     2 17-JUN-2005
	KING      11 26-JUN-2005 27-JUN-2005     8 23-JUN-2005
	KING      11 26-JUN-2005 27-JUN-2005    14 29-JUN-2005
	KING      11 26-JUN-2005 27-JUN-2005     5 20-JUN-2005
	KING      14 29-JUN-2005 30-JUN-2005     2 17-JUN-2005
	KING      14 29-JUN-2005 30-JUN-2005     8 23-JUN-2005
	KING      14 29-JUN-2005 30-JUN-2005     5 20-JUN-2005
	KING      14 29-JUN-2005 30-JUN-2005    11 26-JUN-2005
As you can see from the result set above, the self join makes finding overlapping dates easy; simply return each row where B_START occurs between A_START and A_END. If you look at the WHERE clause on lines 7 and 8 of the solution:

	and b.proj_start >= a.proj_start
	and b.proj_start <= a.proj_end
it is doing just that. Once you have the required rows, constructing the messages is just a matter of concatenating the return values.

Oracle users can use the window function LEAD OVER to avoid the self join, if the maximum number of projects per employee is fixed. This can come in handy if the self join is expensive for your particular results (if the self join requires more resources than the sorts needed for LEAD OVER). For example, consider the alternative for employee KING using LEAD OVER:

	select empno,
	       ename,
	       proj_id,
	       proj_start,
	       proj_end,
	       case
	       when lead(proj_start,1)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       when lead(proj_start,2)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       when lead(proj_start,3)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       when lead(proj_start,4)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       end is_overlap
	  from emp_project
	 where ename = 'KING'

	EMPNO ENAME  PROJ_ID PROJ_START  PROJ_END    IS_OVERLAP
	----- ------ ------- ----------- ----------- ----------
	7839  KING         2 17-JUN-2005 21-JUN-2005          5
	7839  KING         5 20-JUN-2005 24-JUN-2005          8
	7839  KING         8 23-JUN-2005 25-JUN-2005
	7839  KING        11 26-JUN-2005 27-JUN-2005
	7839  KING        14 29-JUN-2005 30-JUN-2005
Because the number of projects is fixed at five for employee KING, you can use LEAD OVER to move examine the dates of all the projects without a self join. From here, producing the final result set is easy. Simply keep the rows where IS_OVERLAP is not NULL:

	select empno,ename,
	       'project '||is_overlap||
	       ' overlaps project '||proj_id msg
	  from (
	select empno,
	       ename,
	       proj_id,
	       proj_start,
	       proj_end,
	       case
	       when lead(proj_start,1)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       when lead(proj_start,2)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       when lead(proj_start,3)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       when lead(proj_start,4)over(order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(order by proj_start)
	       end is_overlap
	  from emp_project
	 where ename = 'KING'
	       )
	 where is_overlap is not null

	EMPNO ENAME  MSG
	----- ------ --------------------------------
	7839  KING   project 5 overlaps project 2
	7839  KING   project 8 overlaps project 5
To allow the solution to work for all employees (not just KING), partition by ENAME in the LEAD OVER function:

	select empno,ename,
	       'project '||is_overlap||
	       ' overlaps project '||proj_id msg
	  from (
	select empno,
	       ename,
	       proj_id,
	       proj_start,
	       proj_end,
	       case
	       when lead(proj_start,1)over(partition by ename
	                                       order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(partition by ename
	                                  order by proj_start)
	       when lead(proj_start,2)over(partition by ename
	                                       order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(partition by ename
	                                  order by proj_start)
	       when lead(proj_start,3)over(partition by ename
	                                       order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(partition by ename
	                                  order by proj_start)
	       when lead(proj_start,4)over(partition by ename
	                                       order by proj_start)
	            between proj_start and proj_end
	       then lead(proj_id)over(partition by ename
	                                  order by proj_start)
	       end is_overlap
	 from emp_project
	      )
	where is_overlap is not null

	EMPNO ENAME  MSG
	----- ------ -------------------------------
	7782  CLARK  project 7 overlaps project 4
	7782  CLARK  project 10 overlaps project 7
	7782  CLARK  project 13 overlaps project 10
	7839  KING   project 5 overlaps project 2
	7839  KING   project 8 overlaps project 5
	7934  MILLER project 6 overlaps project 3
	7934  MILLER project 12 overlaps project 9
