# SQLMurderMystery
### This is a document describing my process going through Knight Lab's SQL Murder Mystery exercise found here: https://mystery.knightlab.com/

---
### The murder mystery follows this Schema Diagram:
![image](https://github.com/arinotsorry/SQLMurderMystery/blob/main/Schema%20Diagram.png)

### First, we are given the following info:
- The crime type was a murder
- The murder happened on January 15, 2018 (years are formatted throughout as yyyymmdd, or in this case, 20180115)
- The murder happened in SQL City

### Crime Scene Report
  <details>
    <summary>Click to see SQLite Statement</summary>

```sql
SELECT description
  FROM crime_scene_report
  WHERE date = 20180115 AND type = 'murder' AND city = 'SQL City'
```
  </details>
  
  We want to find the description where the type matches 'murder', the city is 'SQL City', and the date is 20180115, from above.
  
  
  <details>
    <summary>Click to see the crime report</summary>

  
  > Security footage shows that there were 2 witnesses.
  > The first witness lives at the last house on "Northwestern Dr". 
  > The second witness, named Annabel, lives somewhere on "Franklin Ave"."
  </details>


  
