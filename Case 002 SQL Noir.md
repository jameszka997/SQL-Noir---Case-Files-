# Workflow - SQL Noir Case #002


## ✨ Introduction to the case



### Case #002: The Stolen Sound ###

In the neon glow of 1980s Los Angeles, the West Hollywood Records store was rocked by a daring theft. A prized vinyl record, worth over $10,000, vanished during a busy evening, leaving the store owner desperate for answers. Vaguely recalling the details, you know the incident occurred on July 15, 1983, at this famous store. Your task is to track down the thief and bring them to justice.



#### Objectives: ####
1. Retrieve the crime scene report for the record theft using the known date and location.
2. Retrieve witness records linked to that crime scene to obtain their clues.
3. Use the clues from the witnesses to find the suspect in the suspects table.
4. Retrieve the suspect's interview transcript to confirm the confession.

---

**Definitions:**
***Primary key*** - Unique identification for rows within a dataset/table
***Foreign key*** - Identifier within a dataset/table which references entry within a different dataset/table

*This is highlighted with the key being highlighted with phrase that helps to identify which foreign dataset the key is referring to*


***Case 2 contains 4 datatables***
- crime_scene
		○ Primary_key: id
- suspects
		○ Primary_key: id
- interviews
		○ Primary_key: id
		○ Foreign_key: suspect_id
- witnesses
		○ Primary_key: id
		○ Foreign_key: crime_scene_id



---


## 🔍︎ Investigation ##

First I have looked at the 'crime_scene' data of the crime location.

```sql
SELECT id, date, type, location, description
FROM crime_scene
WHERE location LIKE "%West Hollywood%"
```



#### 🎯 Output
|id	|date	|type	|location	|description|
|--|--|--|--|--|
|65	|19830715	|theft	|West Hollywood Records	|A prized vinyl record was stolen from the store during a busy evening.|


---


Using the 'crime_scene_id' foreign_key, we can see what witnesses were involved with this crime scene.

```sql
SELECT *
FROM witnesses
WHERE crime_scene_id IN (65)
```



#### 🎯 Output
|id	|crime_scene_id	|clue|
|--|--|--|
|28	|65	|I saw a man wearing a red bandana rushing out of the store.|
|75	|65	|The main thing I remember is that he had a distinctive gold watch on his wrist.|


---

```sql
SELECT *
FROM witnesses
WHERE crime_scene_id IN (65)
```



#### 🎯 Output
|id	|crime_scene_id	|clue|
|--|--|--|
|28	|65	|I saw a man wearing a red bandana rushing out of the store.|
|75	|65	|The main thing I remember is that he had a distinctive gold watch on his wrist.|


---

Now that we have information on the appearance of the suspect, we can use these information to filter the 'suspect' datatable, to have both of these clues.

```sql
SELECT *
FROM suspects
WHERE bandana_color LIKE "%red%"
AND accessory LIKE "%gold watch%"
```



#### 🎯 Output
|id	|name	|bandana_color	|accessory|
|--|--|--|--|
|35	|Tony Ramirez	|red	|gold watch|
|44	|Mickey Rivera	|red	|gold watch|
|97	|Rico Delgado	|red	|gold watch|


---


As we know the id of these 3 suspects, we can look at their interviews in the last datatable.

```sql
SELECT *
FROM interviews
WHERE suspect_id IN (35, 44, 97)
```



#### 🎯 Output
|suspect_id	|transcript|
|--|--|
|35	|I wasn't anywhere near West Hollywood Records that night.|
|44	|I was busy with my music career; I have nothing to do with this theft.|
|97	|I couldn't help it. I snapped and took the record.|


---


With the interviews, we can conclude that suspect 97, **Rico Delgado**, stole the vinyl record.

*Much thanks for Hristo Bogoev, the creator of SQL Noir.*

