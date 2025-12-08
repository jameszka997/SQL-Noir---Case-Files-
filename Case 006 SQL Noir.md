# Workflow - SQL Noir Case #006


## ✨ Introduction to the case

**Case #006: The Vanishing Diamond**
At Miami’s prestigious Fontainebleau Hotel charity gala, the famous “Heart of Atlantis” diamond necklace suddenly disappeared from its display.

**Objective:**
Find who stole the diamond.

---

**Definitions:**
***Primary key*** - Unique identification for rows within a dataset/table
***Foreign key*** - Identifier within a dataset/table which references entry within a different dataset/table
*This is highlighted with the key being highlighted with phrase that helps to identify which foreign dataset the key is referring to*


***Case 6 contains 6 datatables***
- crime_scene
		○ Primary_key: id
- guest
		○ Primary_key: id
- witness_statements
		○ Primary_key: id
		○ Foreign_key: guest_id
- attire_registry
		○ Primary_key: id
		○ Foreign_key: guest_id
- marina_rentals
		○ Primary_key: id
		○ Foreign_key: renter_guest_id (guest_id)
- final_interviews
		○ Primary_key: id
		○ Foreign_key: guest_id



---


## 🔍︎ Investigation ##

As a start, I have looked at the *crime_scene* table for the Hotel


```sql
select * from crime_scene
where location like "%Fontainebleau Hotel%"
```

#### 🎯 Output
| id	|date	|location	|description|
| -- | -- | -- | -- |
|48	|19870520	|Fontainebleau Hotel	|The Heart of Atlantis necklace disappeared. Many guests were questioned but only two of them gave valuable clues. One of them is a really famous actor. The other one is a woman who works as a consultant for a big company and her first name ends with "an".|


---

Based on the description given for the two people who were named at the crime_scene, the *person* table is investigated to find out more about them

```sql
select * from crime_scene
where location like "%Fontainebleau Hotel%"
```

#### 🎯 Output
*the list is a bit longer but upon reading the entries we can find our people of interest
|id	|name	|occupation	|invitation_code|
| -- | -- | -- | -- |
|129	|Clint Eastwood	|Actor	|VIP-G|
|116	|Vivian Nair	|Consultant	|VIP-R|


---

Their witness statements are interesting, and to show this I have joined together the *guest* and *witness_statements* tables to filter for them as well as see the name of the guest



```sql
select witness_statements.guest_id, witness_statements.clue, guest.name from witness_statements
join guest on witness_statements.guest_id = guest.id
where guest.occupation like '%Actor%' or 
(guest.occupation like '%consultant%' and guest.name like '%an %')
```

#### 🎯 Output
|guest_id	|clue	|name|
| -- | -- | -- |
|116	|I saw someone holding an invitation ending with "-R". He was wearing a navy suit and a white tie.	|Vivian Nair|
|129	|I overheard someone say, "Meet me at the marina, dock 3.	|Clint Eastwood|


---


As the two witnesses gave two statements, I need to perform two joins of *attire_registry* and *marine_rentals* so we can find the guest the clues point toward



```sql
select marina_rentals.dock_number, guest.invitation_code, marina_rentals.rental_date, marina_rentals.boat_name, guest.name, attire_registry.note, guest.id from guest
join attire_registry on guest.id = attire_registry.guest_id
join marina_rentals on guest.id = marina_rentals.renter_guest_id
where attire_registry.note like "%navy suit, white tie%"
and guest.invitation_code like "%-R"
and dock_number = 3
```

#### 🎯 Output
|dock_number	|invitation_code	|rental_date	|boat_name	|name	|note	|id|
| -- | -- | -- | -- | -- | -- | -- |
|3	|VIP-R	|19870520	|Coastal Spirit	|Mike Manning	|navy suit, white tie	|105|


---


Our prime suspect now is *Mike Manning*. Shall look at his 'final_interview' entry to see what he has said.

```sql
select * from final_interviews
where guest_id = 105
```


#### 🎯 Output
|id	|guest_id	|confession|
|--|--|--|
|105	|105	|I was the one who took the crystal. I guess I need a lawyer now?|


Thus, the person who has stolen the crystal was Mike Manning.


*Much thanks for Hristo Bogoev, the creator of SQL Noir.*
