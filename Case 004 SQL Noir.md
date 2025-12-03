# Workflow - SQL Noir Case #004


## ✨ Introduction to the case

**Case #004: The Midnight Masquerade Murder**
On October 31, 1987, at a Coconut Grove mansion masked ball, Leonard Pierce was found dead in the garden. Can you piece together all the clues to expose the true murderer?

**Objective:**
Reveal the true murderer of this complex case.

---

**Definitions:**
***Primary key*** - Unique identification for rows within a dataset/table
***Foreign key*** - Identifier within a dataset/table which references entry within a different dataset/table
*This is highlighted with the key being highlighted with phrase that helps to identify which foreign dataset the key is referring to*



**Schema layout**
*apply picture of schema later*

***Case 3 contains 6 tables***
- Person
		○ Primary_key: id
- Witness_statements
		○ Primary_key: id
		○ Foreign_key: witness_id (person)
		○ Foreign_key: crime_scene_id 
- Hotel_checkins
		○ Primary_key: id
		○ Foreign_key: person_id
- Surveillance_records
		○ Primary_key: id
		○ Foreign_key: hotel_checkin_id
- Phone_records
		○ Primary_key: id
		○ Foreign_key: caller_id (person)
		○ Foreign_key: recipient_id (person)
- Final_interviews
		○ Primary_key: id
		○ Foreign_key: person_id
- Vehicle_registry
		○ Primary_key: id
		○ Foreign_key: person_id
- Catering_orders
		○ Primary_key: id
        ○ Foreign_key: person_id
	

## 🔍︎ Investigation ##

Looking at the crime scene as first order of business for *Coconut Grove*



```sql
select * from crime_scene
where location like "%Coconut Grove%"
```

#### 🎯 Output
| id	|date	|location	|description|
| -- | -- | -- | -- |
|75	|19871031	|Miami Mansion, Coconut Grove	|During a masked ball, a body was found in the garden. Witnesses mentioned a hotel booking and suspicious phone activity.|

---


Witness mentioned a hotel booking and suspicious phone activity
I shall do a join between *crime_scene_id* and the *witness_statements* table

```sql
select crime_scene.id, crime_scene.location, witness_statements.id, witness_statements.crime_scene_id, witness_statements.witness_id, witness_statements.clue from crime_scene
join witness_statements on crime_scene.id = witness_statements.crime_scene_id
where crime_scene.id in (75)
```



#### 🎯 Output
|id	|location	|id	|crime_scene_id	|witness_id	|clue|
|--|--|--|--|--|--|
|75	|Miami Mansion, Coconut Grove	|83	|75	|37|	I overheard a booking at The Grand Regency.|
|75	|Miami Mansion, Coconut Grove	|89	|75	|42	|I noticed someone at the front desk discussing Room 707 for a reservation made yesterday.|


---

So now we can see that the hotel booking was made at the Grand Regency, and reservation to Room 707
I will link the *witness_id* to the *person* table to know who the witnesses were

```sql
select person.id, person.name, person.occupation, person.address, witness_statements.witness_id, witness_statements.clue from person
join witness_statements on person.id = witness_statements.witness_id
where person.id in (37, 42)
```




#### 🎯 Output
|id	|name	|occupation	|address	|witness_id	|clue|
|--|--|--|--|--|--|
|37	|Steven Nelson	|Doctor	|294 Cedar Place	|37	|I overheard a booking at The Grand Regency.|
|42	|Sharon Phillips	|Marketing Manager	|849 Ashwood Court	|42	|I noticed someone at the front desk discussing Room 707 for a reservation made yesterday.|


 ---

 
 

Based on the output, my idea was to filter the *hotel_reservation* data with the information we have gathered before. Then additionally joining this with the surveillance_records if any information was found with anybody who matches the hotel_checkin filter criterias

```sql
select surveillance_records.id, surveillance_records.hotel_checkin_id, 
surveillance_records.note from surveillance_records 
join hotel_checkins on surveillance_records.hotel_checkin_id = hotel_checkins.id
where hotel_checkins.hotel_name like '%The Grand Regency%' and 
hotel_checkins.room_number in (707) and
hotel_checkins.check_in_date in (19871030) and
surveillance_records.note is not NULL
```

#### 🎯 Output
*multiple rows are filtered but upon observation we can find the correct row*
|id	|hotel_checkin_id	|note|
|--|--|--|
|119	|119	|Subject was overheard yelling on a phone: "Did you kill him?|

---

Then, I would check the corresponding hotel_checkin additionally to know the person_id

```sql
select * from hotel_checkins
where id = 119
```


#### 🎯 Output
|id	|person_id	|hotel_name	|check_in_date	|room_number|
|-|-|-|-|-|
|119	|11	|The Grand Regency	|19871030	|707|


---

The person_id is 11 for the hotel_checkin, and I would do a search of the *caller_id* and *recipient_id* that have 11 within the *phone_records* table


```sql
select * from phone_records 
where caller_id in (11)
or recipient_id in (11)
```


#### 🎯 Output
|id	|caller_id	|recipient_id	|call_date	|call_time	|note|
|-|-|-|-|-|-|
|117	|11	|58	|19871030	|23:30	|Why did you kill him, bro? You should have left the carpenter do it himself!|

---

We can look up the recipient id 58, in both caller_id as well to see if he had any other phone calls

```sql
select * from phone_records 
where caller_id in (11, 58)
or recipient_id in (11, 58)
```


#### 🎯 Output
|id	|caller_id	|recipient_id	|call_date	|call_time	|note|
|-|-|-|-|-|-|
|117	|11	|58	|19871030	|23:30	|Why did you kill him, bro? You should have left the carpenter do it himself!|
|163	|133	|58	|19871030	|22:15	|I will do it. Only if you give me that nice Lambo of yours.|


---


Knowing we are looking for a lambo, and the person is a carpenter, we can do a join with the *person* and *vehicle_registry* datatables so we can filter for the car type and occupation 

```sql
select * from vehicle_registry
join person on vehicle_registry.person_id = person.id
where car_make like "%Lambo%"
and occupation like "%Carpenter%"
```


#### 🎯 Output
|id	|person_id	|plate_number	|car_make	|car_model	|id	|name	|occupation	|address|
|-|-|-|-|-|-|-|-|-|
|41	|97	|EFG901	|Lamborghini	|Countach	|97	|Marco Santos	|Carpenter	|112 Forestwood Way|


---

Then we can take a look at the *final_interview* of his to see what he pleaded


```sql
select * from final_interviews
where person_id in (97)
```


#### 🎯 Output
|id	|person_id	|confession|
|-|-|-|
|97	|97	|I ordered the hit. It was me. You caught me.|

---
The person who has ordered the murder was **Marco Santos**.







*Much thanks for Hristo Bogoev, the creator of SQL Noir.*




