# Workflow - SQL Noir Case #003 


## ✨ Introduction to the case

**Case #003: The Miami Marina Murder**
A body was found floating near the docks of Coral Bay Marina in the early hours of August 14, 1986. Your job, detective, is to find the murderer and bring them to justice. This case might require the use of JOINs, wildcard searches, and logical deduction. Get to work, detective.

**Objective:**
Find the murderer. (Start by finding the crime scene and go from there )



**Definitions:**
***Primary key*** - Unique identification for rows within a dataset/table
***Foreign key*** - Identifier within a dataset/table which references entry within a different dataset/table
*This is highlighted with the key being highlighted with phrase that helps to identify which foreign dataset the key is referring to*



**Schema layout and explanation**

<img width="448" height="283" alt="Case 3 schema" src="https://github.com/user-attachments/assets/3c4cdb6a-caae-424d-b709-e9bde0e84fb6" />


***Case 3 contains 6 tables***
- Crime_scene
		○ Primary_key: id
- Person
		○ Primary_key: id
- Confession
		○ Primary_key: id
		○ Foreign_key: person_id
- Interviews
		○ Primary_key: id
		○ Foreign_key: person_id
- Hotel_checkins
		○ Primary_key: id
		○ Foreign_key: person_id
- Surveillance_records
		○ Primary_key: id
		○ Foreign_key: person_id
		○ Foreign_key: hotel_checkins_id
	

## 🔍︎ Investigation ##


As first order of business, the *crime_scene* data was investigated for any clues by looking for 'Coral Bay Marina'


```sql
SELECT id, date, location, description
FROM crime_scene
WHERE location LIKE "%Coral Bay Marina%"
```

#### 🎯 Output
| id	|date	|location	|description|
| -- | -- | -- | -- |
|43	| 19860814	| Coral Bay Marina	| The body of an unidentified man was found near the docks. Two people were seen nearby: one who lives on 300ish "Ocean Drive" and another whose first name ends with "ul" and his last name ends with "ez".


Based on this, two suspects are investigated within the *person* table 

```sql
SELECT id, name, alias, occupation, address
FROM person
WHERE name like "%ul %ez"
or address like "3__ %Ocean Drive%"
```

#### 🎯 Output
|id|	name	|alias	|occupation	|address|
| -- | -- | -- | -- | -- |
|101	|Carlos Mendez	|Los Ojos	|Fisherman	|369 Ocean Drive|
|102	|Raul Gutierrez	|The Cobra	|Nightclub Owner	|45 Sunset Ave|



After knowing the two suspects, their interviews were retrieved from the *interviews* table


#### 🎯 Output
|id	|person_id	|transcript|
|--|--|--|
|101	|101	|I saw someone check into a hotel on August 13. The guy looked nervous.|
|103	|102	|I heard someone checked into a hotel with "Sunset" in the name.|



Initially look into the hotel_checkins with just the information from the suspects to filter down the data, but it yielded no *specific enough result*

```sql
select * from hotel_checkins 
join person on hotel_checkins.person_id = person.id 
where hotel_checkins.hotel_name like '%Sunset%' and 
hotel_checkins.check_in_date = 19860813
```



With the information gathered form the two suspects, we can make a multi-step filter to filter out records that match the information from the two suspects within the *surveillance_records* table
As the table has the 'suspicious_activity' column, and we know our suspect was nervous, we could filter more specifically with that informatoin
This is where we perform our first join between the *person* table and the *hotel_checkins* with the 'person_id' key
As well as filtering out NULL values

```sql
select * from surveillance_records 
where person_id in (
  select hotel_checkins.person_id from hotel_checkins 
  join person on hotel_checkins.person_id = person.id 
  where hotel_checkins.hotel_name like '%Sunset%' and 
  hotel_checkins.check_in_date = 19860813
) and suspicious_activity is not NULL
```



#### 🎯 Output
*first 3 entries are suspicious*
|id	|person_id	|hotel_checkin_id	|suspicious_activity|
|--|--|--|--|
|6	|6	|34	|Spotted entering late at night|
|7	|7	|89	|Seen arguing with an unknown person|
|8	|8	|2	|Left suddenly at 3 AM|



As we have the following 3 people, we would join together the information with the confessions table to see what these people have said (using *person* and *confessions* tables)

'''sql
select person.name, confessions.confession from confessions 
join person on person.id = confessions.person_id
where person_id in (6, 7, 8)
'''

#### 🎯 Output
|name	|confession|
|--|--|
|James Wilson	|I don't know anything about this.|
|Robert Smith	|I was just walking my dog that night.|
|Thomas Brown	|Alright! I did it. I was paid to make sure he never left the marina alive.|



Thus **Thomas Brown** was the murderer.



*Much thanks for Hristo Bogoev, the creator of SQL Noir.*




