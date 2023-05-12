## 1. Classic American names
<p><img src="https://assets.datacamp.com/production/project_1441/img/name.jpg" alt="Lots of name tags piled on top of each other." width="600"></p>
<p>Photo by Travis Wise on <a href="https://commons.wikimedia.org/wiki/File:Hello_My_Name_Is_(15283079263).jpg">Wikimedia</a>.</p>
<p>How have American baby name tastes changed since 1920?  Which names have remained popular for over 100 years, and how do those names compare to more recent top baby names? These are considerations for many new parents, but the skills we'll practice while answering these queries are broadly applicable. After all, understanding trends and popularity is important for many businesses, too! </p>
<p>We'll be working with data provided by the United States Social Security Administration, which lists first names along with the number and sex of babies they were given to in each year. For processing speed purposes, we've limited the dataset to first names which were given to over 5,000 American babies in a given year. Our data spans 101 years, from 1920 through 2020.</p>
<h3 id="baby_names"><code>baby_names</code></h3>
<table>
<thead>
<tr>
<th style="text-align:left;">column</th>
<th>type</th>
<th>meaning</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;"><code>year</code></td>
<td>int</td>
<td>year</td>
</tr>
<tr>
<td style="text-align:left;"><code>first_name</code></td>
<td>varchar</td>
<td>first name</td>
</tr>
<tr>
<td style="text-align:left;"><code>sex</code></td>
<td>varchar</td>
<td><code>sex</code> of babies given <code>first_name</code></td>
</tr>
<tr>
<td style="text-align:left;"><code>num</code></td>
<td>int</td>
<td>number of babies of <code>sex</code> given <code>first_name</code> in that <code>year</code></td>
</tr>
</tbody>
</table>
<p>Let's get oriented to American baby name tastes by looking at the names that have stood the test of time!</p>


```sql
%%sql
postgresql:///names
    
-- Select first names and the total babies with that first_name
-- Group by first_name and filter for those names that appear in all 101 years
-- Order by the total number of babies with that first_name, descending
SELECT first_name, sum(num)
from baby_names
group by first_name
having count(first_name) = 101
order by 2 desc
```

    UsageError: Cell magic `%%sql` not found.


## 2. Timeless or trendy?
<p>Wow, it looks like there are a lot of timeless traditionally male names! Elizabeth is holding her own for the female names, too. </p>
<p>Now, let's broaden our understanding of the dataset by looking at all names. We'll attempt to capture the type of popularity that each name in the dataset enjoyed. Was the name classic and popular across many years or trendy, only popular for a few years? Let's find out. </p>


```sql
%%sql

-- Classify first names as 'Classic', 'Semi-classic', 'Semi-trendy', or 'Trendy'
-- Alias this column as popularity_type
-- Select first_name, the sum of babies who have ever had that name, and popularity_type
-- Order the results alphabetically by first_name
SELECT first_name, sum(num),
    case when count(first_name) < 20 THEN 'Trendy'
         when count(first_name) < 50 THEN 'Semi-trendy'
         when count(first_name) < 80 THEN 'Semi-classic'
         else 'Classic' end as popularity_type
FROM baby_names
group by first_name
order by 1
```

    UsageError: Cell magic `%%sql` not found.


## 3. Top-ranked female names since 1920
<p>Did you find your favorite American celebrity's name on the popularity chart? Was it classic or trendy? How do you think the name Henry did? What about Jaxon?</p>
<p>Since we didn't get many traditionally female names in our classic American names search in the first task, let's limit our search to names which were given to female babies. </p>
<p>We can use this opportunity to practice window functions by assigning a rank to female names based on the number of babies that have ever been given that name. What are the top-ranked female names since 1920?</p>


```sql
%%sql

-- RANK names by the sum of babies who have ever had that name (descending), aliasing as 
-- name_rank
-- Select name_rank, first_name, and the sum of babies who have ever had that name
-- Filter the data for results where sex equals 'F'
-- Limit to ten results
select 
    rank() over( order by sum(num) desc) as name_rank,
    first_name,
    sum(num)
from baby_names
where sex='F'
group by first_name
limit 10

```

    UsageError: Cell magic `%%sql` not found.


## 4. Picking a baby name
<p>Perhaps a friend has heard of our work analyzing baby names and would like help choosing a name for her baby, a girl. She doesn't like any of the top-ranked names we found in the previous task. </p>
<p>She's set on a traditionally female name ending in the letter 'a' since she's heard that vowels in baby names are trendy. She's also looking for a name that has been popular in the years since 2015. </p>
<p>Let's see what we can do to find some options for this friend!</p>


```sql
%%sql
-- Select only the first_name column
-- Filter for results where sex is 'F', year is greater than 2015, and first_name ends in 'a'
-- Group by first_name and order by the total number of babies given that first_name
select first_name
from baby_names
where sex='F' AND year > 2015 AND first_name LIKE '%a'
group by first_name
order by sum(num) desc

```

    UsageError: Cell magic `%%sql` not found.


## 5. The Olivia expansion
<p>Based on the results in the previous task, we can see that Olivia is the most popular female name ending in 'A' since 2015. When did the name Olivia become so popular?</p>
<p>Let's explore the rise of the name Olivia with the help of a window function.</p>


```sql
%%sql

-- Select year, first_name, num of Olivias in that year, and cumulative_olivias
-- Sum the cumulative babies who have been named Olivia up to that year; 
-- alias as cumulative_olivias
-- Filter so that only data for the name Olivia is returned.
-- Order by year from the earliest year to most recent
select 
    year,
    first_name,
    num,
    sum(num) OVER
    ( order by year RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW )
    as cumulative_olivias
FROM baby_names
where first_name='Olivia'
```

    UsageError: Cell magic `%%sql` not found.






## 6. Many males with the same name
<p>Wow, Olivia has had a meteoric rise! Let's take a look at traditionally male names now. We saw in the first task that there are nine traditionally male names given to at least 5,000 babies every single year in our 101-year dataset! Those names are classics, but showing up in the dataset every year doesn't necessarily mean that the timeless names were the most popular. Let's explore popular male names a little further.</p>
<p>In the next two tasks, we will build up to listing every year along with the most popular male name in that year. This presents a common problem: how do we find the greatest X in a group? Or, in the context of this problem, how do we find the male name given to the highest number of babies in a year? </p>
<p>In SQL, one approach is to use a subquery. We can first write a query that selects the <code>year</code> and the maximum <code>num</code> of babies given any single male name in that year. For example, in 1989, the male name given to the highest number of babies was given to 65,339 babies. We'll write this query in this task. In the next task, we can use the code from this task as a subquery to look up the <code>first_name</code> that was given to 65,339 babies in 1989… as well as the top male first name for all other years!</p>


```sql
%%sql

-- Select year and maximum number of babies given any one male name in that year, 
-- aliased as max_num
-- Filter the data to include only results where sex equals 'M'
select year, max(num) as max_num
from baby_names
where sex='M'
group by year
```

    UsageError: Cell magic `%%sql` not found.


## 7. Top male names over the years
<p>In the previous task, we found the maximum number of babies given any one male name in each year. Incredibly, the most popular name each year varied from being given to less than 20,000 babies to being given to more than 90,000! </p>
<p>In this task, we find out what that top male name is for each year in our dataset. </p>


```sql
%%sql

-- Select year, first_name given to the largest number of male babies, 
-- and num of babies given that name
-- Join baby_names to the code in the last task as a subquery
-- Order results by year descending
select b.year, first_name, num
from baby_names as b
join (select year, max(num) as max_num
from baby_names
where sex='M'
group by year) as t1
ON t1.max_num=b.num and t1.year=b.year
ORDER BY b.year desc
```

    UsageError: Cell magic `%%sql` not found.


## 8. The most years at number one
<p>Noah and Liam have ruled the roost in the last few years, but if we scroll down in the results, it looks like Michael and Jacob have also spent a good number of years as the top name! Which name has been number one for the largest number of years? Let's use a common table expression to find out. </p>


```sql
%%sql

-- Select first_name and a count of years it was the top name in the last task;
-- alias as count_top_name
-- Use the code from the previous task as a common table expression
-- Group by first_name and order by count_top_name descending
with popular_name_per_year as (
    select b.year, first_name
    from baby_names as b
    join (select year, max(num) as max_num
    from baby_names
    where sex='M'
    group by year) as t1
    ON t1.max_num=b.num and t1.year=b.year
    ORDER BY b.year desc
)

select first_name, count(*) as count_top_name
from popular_name_per_year
group by first_name
order by 2 desc;
```

    UsageError: Cell magic `%%sql` not found.

