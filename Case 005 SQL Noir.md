# Workflow - SQL Noir Case #005


## ✨ Introduction to the case

**Case #005: The Silicon Sabotage**
QuantumTech, Miami’s leading technology corporation, was about to unveil its groundbreaking microprocessor called “QuantaX.” Just hours before the reveal, the prototype was destroyed, and all research data was erased. Detectives suspect corporate espionage.

**Objective:**
Find who sabotaged the microprocessor.

---

**Definitions:**
***Primary key*** - Unique identification for rows within a dataset/table
***Foreign key*** - Identifier within a dataset/table which references entry within a different dataset/table
*This is highlighted with the key being highlighted with phrase that helps to identify which foreign dataset the key is referring to*


***Case 5 contains 7 datatables***
- incident_reports
	○ Primary_key: id
- witness_statements
	○ Primary_key: id
	○ Foreign_key: incident_id (incident_reports)
	○ Foreign_key: employee_id (employee_records)
- keycard_access_logs
	○ Primary_key: id
	○ Foreign_key: employee_id (employee_records)
- computer_access_logs
	○ Primary_key: id
	○ Foreign_key: employee_id (employee_records)
- email_logs
	○ Primary_key: id
	○ Foreign_key: sender_employee_id (employee_records)
	○ Foreign_key: recipient_employee_id (employee_records)
- facility_access_logs
	○ Primary_key: id
	○ Foreign_key: employee_id (employee_records)
- employee_records
	○ Primary_key: id




---


## 🔍︎ Investigation ##


So to look at the crime scene, we need to find the location for Quantumtech in *incident_reports*




```sql
select * from incident_reports
where location like "%Quantum%"
```

#### 🎯 Output
| id	|date	|location	|description|
| -- | -- | -- | -- |
|74	|19890421	|QuantumTech HQ	|Prototype destroyed; data erased from servers.|

---


Now it is best to join the *witness_statements* table to the *incident_reports* one so we know the reports that occured with this incident



```sql
select * from witness_statements
join incident_reports on witness_statements.incident_id = incident_reports.id
where incident_reports.location like "%Quantum%"
```

#### 🎯 Output
|id	|incident_id	|employee_id	|statement	|id	|date	|location	|description|
| -- | -- | -- | -- | -- | -- | -- | -- |
|40	|74	|145	|I heard someone mention a server in Helsinki.	|74	|19890421	|QuantumTech HQ	|Prototype destroyed; data erased from servers.|
|59	|74	|134	|I saw someone holding a keycard marked QX- succeeded by a two-digit odd number.	|74	|19890421	|QuantumTech HQ	|Prototype destroyed; data erased from servers.|


---

Initially I have only filtered for computer accesses that are associated with the Helsinki server, but it requires further filtering for the correct data. For this I have used a more complicated filtering method to know better who had access with the keycard described in the second witness statement and to the Helsinki server from the *person* table.


```sql
select * from employee_records
where id in
(
select employee_id from keycard_access_logs
where keycard_code like "%QX-0__" and
SUBSTR(keycard_code, -1, 1) in ('1', '3', '5', '7', '9') 
and access_date = 19890421
)
and
id in
(
select employee_id from computer_access_logs
where server_location like "%Helsinki%" 
and access_date = 19890421
)
```

#### 🎯 Output
|id	|employee_name	|department	|occupation	|home_address|
|--|--|--|--|--|
|99	|Elizabeth Gordon	|Engineering	|Solutions |Architect	|147 Coastal Pine Rd, Doral, FL|


---


Initially I have tried looking at the facility_access_log associated with this employee_id but it did not return any meaningful results. Then I have switched to looking at the *email_logs* associated with this employee



```sql
select * from email_logs
where sender_employee_id = 99
or recipient_employee_id = 99
```

#### 🎯 Output
|id	|sender_employee_id	|recipient_employee_id	|email_date	|email_subject	|email_content|
|--:|--:|:--|--:|:--|:--|
|126	|263	|99	|19890421	|Alarm System Concern	|I noticed something strange with the alarm system. There might be a potential malfunction near the chip. Thought you should check it out to be safe.|


---

As this is our only lead, I decided to continue following along the *sender_employee* 263 so I would be able to see if he had any other suspicious email activity. 

```sql
select * from email_logs
where sender_employee_id = 263
or recipient_employee_id = 263
```


#### 🎯 Output
|id	|sender_employee_id	|recipient_employee_id	|email_date	|email_subject	|email_content|
|--|--|--|--|--|--|
|126	|263	|99	|19890421	|Alarm System Concern	|I noticed something strange with the alarm system. There might be a potential malfunction near the chip. Thought you should check it out to be safe.|
|138	|NULL	|263	|19890421	|Realign Asset Trajectory	|L’s schedule puts her close enough, but we need her inside F18 before 9. Trigger a minor alert or routine checkup to send her in by 8:30. Make sure she logs the visit. That part matters.|
|140	|NULL	|263	|19890421	|Execute Phase Window	|Unlock 18 quietly by 9. He’ll use his own credentials to access it shortly after L leaves. No questions. Just ensure the timing lines up. The trail will lead exactly where it needs to.|

*in this case it was good to not filter out Null values from the filtering*


---


Based on the information received, we shall look at the `facility_logs` for *Facility 18*

```sql
select * from facility_access_logs
where facility_name like "%18%"
```


#### 🎯 Output
|id	|employee_id	|facility_name	|access_date	|access_time|
|--|--|--|--|--|
|59	|290	|Facility 18	|19890421	|12:56|
|74	|99	|Facility 18	|19890421	|08:55|
|81	|297	|Facility 18	|19890421	|09:01|

---

The two entries around 9 am are our suspects, as we know employee 99, Elizabeth Gordon, but we do not know the other employee that shortly after Elizabeth entered the Facility


```sql
select * from employee_records
where id = 297
```


#### 🎯 Output
|id	|employee_name	|department	|occupation	|home_address|
|--|--|--|--|--|
|297	|Hristo Bogoev	|Engineering	|Principal Engineer	|901 Quantum Ocean Way, Key Biscayne, FL|


--- 


This man, **Hristo Bogoev**, is our suspect who has entered and sabotaged the microprocessor.



*Much thanks for Hristo Bogoev, the creator of SQL Noir.*
 

