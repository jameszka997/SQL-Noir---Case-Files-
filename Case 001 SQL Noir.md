# Workflow - SQL Noir Case #001


## ✨ Introduction to the case

### Case #001: The Vanishing Briefcase ###

Set in the gritty 1980s, a valuable briefcase has disappeared from the Blue Note Lounge. A witness reported that a man in a trench coat was seen fleeing the scene. Investigate the crime scene, review the list of suspects, and examine interview transcripts to reveal the culprit.


#### Objectives: ####
1. Retrieve the correct crime scene details to gather the key clue.
2. Identify the suspect whose profile matches the witness description.
3. Verify the suspect using their interview transcript.
---

**Definitions:**
***Primary key*** - Unique identification for rows within a dataset/table
***Foreign key*** - Identifier within a dataset/table which references entry within a different dataset/table

*This is highlighted with the key being highlighted with phrase that helps to identify which foreign dataset the key is referring to*


***Case 1 contains 3 datatables***
- crime_scene
		○ Primary_key: id
- suspects
		○ Primary_key: id
- interviews
		○ Primary_key: id
		○ Foreign_key: suspect_id


---


## 🔍︎ Investigation ##

First I have looked at the 'crime_scene' data of the crime location.

```sql
SELECT id, date, type, location, description
FROM crime_scene
WHERE location like "%Blue%"
```



#### 🎯 Output
|id	|date	|type	|location	|description|
|--|--|--|--|--|
|76	|19851120	|theft	|Blue Note Lounge	|A briefcase containing sensitive documents vanished. A witness reported a man in a trench coat with a scar on his left cheek fleeing the scene.|


---

Based on the description, we know that a man with a trench coat and a scar on his left cheek was fleeing the scene, thus I will do filtering for these values in the 'suspects' datatable.


```sql
SELECT *
FROM suspects
WHERE attire LIKE "%trench%"
and scar like "%left%"
```



#### 🎯 Output
|id	|name	|attire	|scar|
|--|--|--|--|
|3	|Frankie Lombardi	|trench coat	|left cheek|
|183	|Vincent Malone|trench coat|left cheek|


---

As we have two suspects, I had the idea to search for both of their interviews based on 'suspect_id'


```sql
SELECT *
FROM interviews
WHERE suspect_id in (3, 183)
```



#### 🎯 Output
|suspect_id	|transcript|
|--|--|
|3	|NULL|
|183	|I wasn’t going to steal it, but I did.|



---


Only suspect #183, **Vincent Malone** had an interview where he has admitted to stealing the briefcase.



*Much thanks for Hristo Bogoev, the creator of SQL Noir.*
