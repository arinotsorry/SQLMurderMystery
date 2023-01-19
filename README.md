# SQLMurderMystery
### This is a document describing my process going through Knight Lab's SQL Murder Mystery exercise found here: https://mystery.knightlab.com/

---
### The murder mystery follows this Schema Diagram:
![image](https://github.com/arinotsorry/SQLMurderMystery/blob/main/Schema%20Diagram.png)

### First, we are given the following info:
- The crime type was a murder
- The murder happened on January 15, 2018 (years are formatted throughout as yyyymmdd, or in this case, 20180115)
- The murder happened in SQL City


## Crime Scene Report
  <details>
    <summary>SQLite to Find Report</summary>

```sql
SELECT description
  FROM crime_scene_report
  WHERE date = 20180115 AND type = 'murder' AND city = 'SQL City'
```
  </details>
  
  <br>
  
  We want to find the description where the type matches 'murder', the city is 'SQL City', and the date is 20180115, from above.
  
  
  <details>
    <summary>Click to see the retrieved crime report</summary>

<br>
  
  > Security footage shows that there were 2 witnesses.
  > The first witness lives at the last house on "Northwestern Dr". 
  > The second witness, named Annabel, lives somewhere on "Franklin Ave"."
  </details>

<br>

## Interviews

From the crime report, we have a couple of witnesses we have to narrow down.

<details>
  <summary>The First Witness</summary>
  
  
### The First Witness
    
  The first witness lives at the last house on "Northwestern Dr."
    
  To find the witness, we have to find all the houses on Northwestern Dr., find the highest house number, 
    and find the person associated with that address.
    
  <details>
    <summary>SQLite to Find Witness Statement</summary>
    
```sql
SELECT name, transcript
  FROM interview LEFT JOIN person ON person_id = id
  WHERE (
    SELECT address_number
      FROM person
      WHERE address_street_name = 'Northwestern Dr'
      ORDER BY address_number DESC
      LIMIT 1
    ) = address_number AND address_street_name = 'Northwestern Dr'
```
  </details>
  
  <br>
  
  The first witness had this to say:
  
  <details>
    <summary>Click to expand the First Witness's Statement</summary>
    
<br>
    
  > “I heard a gunshot and then saw a man run out. 
  > He had a ‘Get Fit Now Gym’ bag. The membership number on the bag started with ‘48Z’. 
  > Only gold members have those bags. The man got into a car with a plate that included ‘H42W’.” - Morty Schapiro
    
  </details>
    
</details>

<br>

<details>
  <summary>The Second Witness</summary>
  
  
### The Second Witness
    
  The second witness is named Annabel, and she lives somewhere on "Franklin Ave"."
    
  To find Annabel, we have to find all the houses on Franklin Ave and cross reference that with the names of the house's occupants.
    
  <details>
    <summary>SQLite to Find Witness Statement</summary>
    
```sql
SELECT name, transcript
    FROM interview LEFT JOIN person ON person_id = id
    WHERE name LIKE 'Annabel %' 
      AND address_street_name = 'Franklin Ave'
```
  </details>
  
  <br>
  
  The second witness had this to say:
  
  <details>
    <summary>Click to expand the Second Witness's Statement</summary>
    
<br>
    
  > “I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.” - Annabel Miller
    
  </details>
    
</details>

<br>

## Summary of Clues Gathered So Far
<details>
  <summary>Clues Gathered from Witness Statements</summary>
  
<br>
  
From the first witness, Morty Schapiro, we learned:
  - The suspect's gym membership number begins with '48Z'
  - The suspect's license plate includes 'H42W'

From the second witness, Annabel Miller, we learned:
  - The suspect was at the gym with Annabel Miller on January 9, 2018 (20180109)
</details>

----
  
<br>

## Clues From Interviews

### Clue <sup>#</sup>1
<details>
  <summary>Details about First Clue</summary>
  
  <br>
  
  ### Gym membership number starts with '48Z'
  
  <br>
  
  We need to get the membership IDs (or names, for readability) of people whose gym membership numbers begin with '48Z'
  
  <br>
  
<details>
  <summary>SQLite for getting names with appropriate membership IDs</summary>
  
```sql
SELECT DISTINCT name
  FROM get_fit_now_member JOIN get_fit_now_check_in ON id = membership_id
  WHERE id LIKE '48Z%'
```
  
  </details>
</details>
<br>

### Clue <sup>#</sup>2
<details>
  <summary>Details about Second Clue</summary>
  
  <br>
  
  ### Car's license plate includes 'H42W'
  
  <br>
  
  We need to get the names of people whose license plate includes 'H42W'
  
  <br>
  
<details>
  <summary>SQLite for getting names associated with matching license plates</summary>
  
```sql
SELECT name, plate_number
  FROM person LEFT JOIN drivers_license ON person.license_id = drivers_license.id
  WHERE plate_number LIKE '%H42W%'
```
  
  </details>
</details>
<br>

### Clue <sup>#</sup>3
<details>
  <summary>Details about Third Clue</summary>
  
  <br>
  
  ### Saw Annabel at the gym
  
  <br>
  
  We need to get the names of people whose time at the gym overlapped with Annabel's time there on January 9<sub>th</sub>, 2018.
  
  If we consider how many different ways someone can encounter Annabel at the gym:
  - arriving before Annabel arrives, leaving after she leaves (suspect's time there encapsulates hers)
  - arriving before Annabel arrives, leaving before she leaves (but after she's arrived) (Annabel sees suspect arrive and leave)
  - arriving after Annabel arrives, leaving before she leaves (Annabel sees suspect leave or end of their workout)
  - arriving after Annabel arrives, leaving after she leaves (Annabel sees suspect arrive or beginning of their workout)
  <br>
  
  These are kind of annoying to check for and have a lot of repetition, though. However, we can consider the scenarios where Annabel won't see the a gym-goer:  

  - if they leave before Annabel arrives
  - if they arrive after Annabel leaves
  
  
  <br>
  
<details>
  <summary>SQLite for getting names of people who overlapped with Annabel's gym time</summary>
  
```sql
SELECT name, membership_id, check_in_time, check_out_time
  FROM get_fit_now_check_in JOIN get_fit_now_member ON membership_id = id
  WHERE check_in_date = 20180109 AND NOT ( 
    check_in_time > (
      SELECT check_out_time
        FROM get_fit_now_check_in JOIN get_fit_now_member ON membership_id = id
        WHERE check_in_date = 20180109 AND name = 'Annabel Miller'
    ) OR check_out_time < (
      SELECT check_in_time
        FROM get_fit_now_check_in JOIN get_fit_now_member ON membership_id = id
        WHERE check_in_date = 20180109 AND name = 'Annabel Miller'
    ))
```
                           
  </details>
</details>
<br>

## Finding Overlap of All the Clues
<details>
  <summary>Who's the Murderer?</summary>
  
<br>
  
We know that the murderer has to satisfy the 3 conditions/clues discussed in the previous section for them to have been the killer.
  
<br>
  
<details>
  <summary>SQLite Statement to Find Murderer</summary>
  
```sql
SELECT person.name
  FROM get_fit_now_check_in
    JOIN get_fit_now_member ON membership_id = get_fit_now_member.id
    LEFT JOIN person ON person_id = person.id
    LEFT JOIN drivers_license ON license_id = drivers_license.id
  WHERE membership_id LIKE '48Z%'
    AND plate_number LIKE '%H42W%'
    AND check_in_date = 20180109
    AND NOT (
      check_in_time > (
        SELECT check_out_time
          FROM get_fit_now_check_in JOIN get_fit_now_member ON membership_id = id
          WHERE check_in_date = 20180109 AND get_fit_now_member.name = 'Annabel Miller'
      )
    OR
      check_out_time < (
        SELECT check_in_time
          FROM get_fit_now_check_in JOIN get_fit_now_member ON membership_id = id
          WHERE check_in_date = 20180109 AND get_fit_now_member.name = 'Annabel Miller' 
       )
    )
```
                        
  </details>
  
  <br>
  
  <details>
    <summary>And the killer is...</summary>
    
### Jeremy Bowers!
    
    
#### But wait! There's more...
    
</details>
</details>
  
-----------------------------------------------------------------------------------------------------
  
## Did we find the real bad guy?

<details>
  <summary>The Killer's Statement</summary>
  
<br>
  
### The Killer's Statement
  
<br>
  
  The fine folks at Night Lab hinted that there may be more to uncover, so let's see what Jeremy has to say.
  
  We can query the `interview` table for his statement:
  
<br>
  
<details>
  <summary>SQLite Query</summary>
  
```sql
SELECT name, transcript
  FROM interview JOIN person ON person_id = id
  WHERE (
    SELECT id
      FROM person
      WHERE name = 'Jeremy Bowers'
  ) = person_id
```

  </details>
  
<br>
  
  <details>
    <summary>What he said</summary>
    
<br>
    
> I was hired by a woman with a lot of money. 
> I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). 
> She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017
    
  </details>
</details>
  
<br>

<details>
  <summary>What we know about the person behind the curtain</summary>
  
  1. She's between 5'5 and 5'7
  2. She has red hair
  3. She has a Tesla Model S
  4. She attended the SQL Symphony Concert 3 times in December 2017
  
  We can find <sup>#</sup>1-3 in the `drivers_license` table and <sup>#</sup>4 in the `facebook_events` table.
  
<details>
  <summary>SQLite for Driver's License Info</summary>
  <br>
  
```sql
SELECT name	
  FROM person JOIN drivers_license ON license_id = drivers_license.id
  WHERE (height >= 65 AND height <= 67)
    AND hair_color = 'red'
    AND car_make = 'Tesla'
    AND car_model = 'Model S'
```
  
  </details>
<details>
  <br>
  To find out who attended the SQL Symphony Concert 3 times in December 2017, we need to divide the date (yyyymmdd) by 100, to get just the year and month. Then, we have to group by name and return the count of how many concerts they attended.
  <br>
  <summary>SQLite for finding who attended 3 Symphony Concerts</summary>
  <br>
  
```sql
SELECT name, COUNT(*) AS times_attended
  FROM facebook_event_checkin JOIN person ON person_id = id
  WHERE event_name = 'SQL Symphony Concert' AND date/100 = 201712
  GROUP BY name
  ORDER BY times_attended DESC
```
                                      
  </details>

  <details>
    <summary>Combined Statement</summary>
    
<br>
    
```sql
SELECT DISTINCT name
	FROM facebook_event_checkin JOIN person ON person_id = person.id
  	  JOIN drivers_license ON license_id = drivers_license.id
	WHERE (height >= 65 AND height <= 67)
  	  AND hair_color = 'red'
  	  AND car_make = 'Tesla'
  	  AND car_model = 'Model S'
     AND (
   	 SELECT COUNT(*) AS times_attended
   	 FROM facebook_event_checkin JOIN person ON person_id = id
   	 WHERE event_name = 'SQL Symphony Concert'
   		 AND date/100 = 201712
   	 GROUP BY name
   	 ORDER BY times_attended DESC
   	) = 3
```
  </details>
    
<details>
  <br>
  <summary>There is only one person who matches that description...</summary>
  
# Miranda Priestly.
  <br>
  Usually, the devil wears Prada, but today the devil wears an orange jumpsuit and a pair of handcuffs 😎.
    </details>
  </details>
