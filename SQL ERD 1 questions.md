Given this ERD

<img src="https://github.com/T1mSchneider/T1mSchneider.github.io/blob/master/images/sql1.2.png"/>
<img src="https://github.com/T1mSchneider/T1mSchneider.github.io/blob/master/images/sql1.1.png"/>

Show all of the patients grouped into weight groups.
Show the total amount of patients in each weight group.
Order the list by the weight group decending.

For example, if they weight 100 to 109 they are placed in the 100 weight group, 110-119 = 110 weight group, etc.
```sql
select count(*) as total_patients, (weight / 10) * 10 as weight_group
from patients
group by weight_group
order by weight_group desc
```

Show patient_id, weight, height, isObese from the patients table.

Display isObese as a boolean 0 or 1.
Obese is defined as weight(kg)/(height(m)2) >= 30.
weight is in units kg.
height is in units cm.

```sql
SELECT patient_id, weight, height,
  (CASE
  	WHEN weight/(POWER(height/100.0,2)) >= 30 THEN
      	1
  	ELSE
      	0
  	END) AS isObese
FROM patients
```

Show patient_id, first_name, last_name, and attending doctor's specialty.
Show only the patients who has a diagnosis as 'Epilepsy' and the doctor's first name is 'Lisa'

```sql
select p.patient_id, p.first_name, p.last_name, d.specialty
from patients p
inner join admissions a on p.patient_id = a.patient_id
inner join doctors d on a.attending_doctor_id = d.doctor_id
where a.diagnosis = 'Epilepsy' and d.first_name = 'Lisa'
```
All patients who have gone through admissions, can see their medical documents on our site. Those patients are given a temporary password after their first admission. Show the patient_id and temp_password.
The password must be the following, in order:
1. patient_id
2. the numerical length of patient's last_name
3. year of patient's birth_date

```sql
select distinct(a.patient_id),
CONCAT(p.patient_id, LENGTH(p.last_name), year(p.birth_date)) as temp_password
from patients p
inner join admissions a on p.patient_id = a.patient_id
```

Each admission costs $50 for patients without insurance, and $10 for patients with insurance. All patients with an even patient_id have insurance.
Give each patient a 'Yes' if they have insurance, and a 'No' if they don't have insurance. Add up the admission_total cost for each has_insurance group.

```sql
select
	case
    	when a.patient_id % 2 = 0 then 'Yes'
    	else 'No'
	end as has_insurance,
	sum(case
        	when a.patient_id % 2 = 0 then 10
        	else 50
    	end) as cost_after_insurance
from admissions a
group by has_insurance
```

Show the provinces that has more patients identified as 'M' than 'F'. Must only show full province_name

```sql
SELECT pr.province_name
FROM province_names pr
JOIN patients p ON pr.province_id = p.province_id
WHERE p.gender IN ('M', 'F')
GROUP BY province_name
HAVING SUM(CASE WHEN p.gender = 'M' THEN 1 ELSE 0 END) >
   	SUM(CASE WHEN p.gender = 'F' THEN 1 ELSE 0 END)
```

We are looking for a specific patient. Pull all columns for the patient who matches the following criteria:
- First_name contains an 'r' after the first two letters.
- Identifies their gender as 'F'
- Born in February, May, or December
- Their weight would be between 60kg and 80kg
- Their patient_id is an odd number
- They are from the city 'Kingston'

```sql
select *
from patients
where first_name like '__%r%' and
month(birth_date) in (2,5,12) and
weight between 60 and 80 and
patient_id % 2 = 1 and
city = 'Kingston'
```

Show the percent of patients that have 'M' as their gender. Round the answer to the nearest hundreth number and in percent form.
```sql
select
CONCAT(round( 100.0 * sum(CASE WHEN p.gender = 'M' THEN 1 ELSE 0 END)/ count(*), 2), '%')
as percent_of_male_patients
from patients p
```
For each day display the total amount of admissions on that day. Display the amount changed from the previous date.

```sql
SELECT
 admission_date,
 count(admission_date) as admission_day,
 count(admission_date) - LAG(count(admission_date)) OVER(ORDER BY admission_date) AS admission_count_change
FROM admissions
 group by admission_date
```

Sort the province names in ascending order in such a way that the province 'Ontario' is always on top.

```sql
select province_name
from province_names
order by
	case when province_name = 'Ontario' then 0 else 1 end,
	Province_name
```
We need a breakdown for the total amount of admissions each doctor has started each year. Show the doctor_id, doctor_full_name, specialty, year, total_admissions for that year.

```sql
SELECT
  d.doctor_id as doctor_id,
  CONCAT(d.first_name,' ', d.last_name) as doctor_name,
  d.specialty,
  YEAR(a.admission_date) as selected_year,
  COUNT(*) as total_admissions
FROM doctors as d
  LEFT JOIN admissions as a ON d.doctor_id = a.attending_doctor_id
GROUP BY
  doctor_name,
  selected_year
ORDER BY doctor_id, selected_year
```
