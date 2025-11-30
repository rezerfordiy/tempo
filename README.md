
## Вариант 2
15(Покупатели) <->> 14(отгрузки) <<-> 02(детали)

15(<u>c_id</u>, name, address) \
14(<u>st_id</u>, <u>doc_id</u>, c_id, d_id, dim, amount, date) \
02(<u>d_id</u>, type, name, dim, plan) 

### RA
```bash
p100 = [d_id][plan > 100](02)
customers = [c_id](15)
bad = [c_id, d_id][st_id != 5](14)
result = [c_id](customres • p100 - bad)
```

### RIC
find { c.c_id | c ∈ 15 } \
  ∃ (d ∈ 02) d.plan > 100 \
  ∧ (∀ (s ∈ 14) \
      (s.d_id = d.d_id ∧ c.c_id = s.c_id) \ 
      ⇒ s.st_id = 5) 
### SQL
``` sql
WITH p100 AS (
    SELECT d_id FROM 02 WHERE plan > 100
),
bad AS (
    SELECT DISTINCT d_id, c_id
    FROM 14
    WHERE st_id <> 5
)
SELECT DISTINCT c.c_id
FROM 15 c
CROSS JOIN p100
WHERE NOT EXISTS (
    SELECT 1 
    FROM bad 
    WHERE bad.d_id = p100.d_id AND bad.c_id = c.c_id
)
```
### ORM
``` python
p100 = [d.d_id for d in 02 if d.plan > 100]
bad = set()
for s in 14:
    if s.st_id != 5:
        bad.add((s.d_id, s.c_id))

res = set()
for c in 15:
    for d_id in p100:
        if (d_id, c.c_id) not in bad:
            res.add(c.c_id)
return res
```













## Вариант 3
07a(профессии) <->> 10(личный состав) <<-> 07в(участки)

07а(<u>prof_id</u>, name) \
10(<u>worker_id</u>,ceh_id, space_id, prof_id, power, family, name) \
07в(<u>ceh_id</u>, <u>space_id</u>, name, master_id) 

### RA
```bash
s7 = [ceh_id, space_id][ceh_id = 7](07в)
profs = [prof_id](07a)
bad = [prof_id, ceh_id, space_id][power <= 4](10)
result = [prof_id]((profs • s7) - bad)
```

### RIC
find { p.prof_id | p ∈ 07a } \
  ∃ (s ∈ 07в) s.ceh_id = 7 \
  ∧ (∀ (w ∈ 10) \
      (w.prof_id = p.prof_id ∧ w.ceh_id = s.ceh_id ∧ w.space_id = s.space_id) \
      ⇒ w.power > 4)
### SQL
``` sql

WITH s7 AS (
    SELECT DISTINCT ceh_id, space_id
    FROM 07в
    WHERE ceh_id = 7
),
bad AS (
    SELECT DISTINCT prof_id, ceh_id, space_id 
    FROM 10
    WHERE power <= 4
)
SELECT DISTINCT p.prof_id
FROM 07a p
CROSS JOIN s7
WHERE NOT EXISTS(
        SELECT 1 
        FROM bad
        WHERE p.prof_id = bad.prof_id AND
            s7.ceh_id = bad.ceh_id AND
            s7.space_id = bad.space_id            
    )
```
### ORM
``` python
s7 = [(s.ceh_id, s.space_id) for s in 07в if s.ceh_id = 7]
bad = set()
for w in 10:
    if w.power <= 4:
        bad.add((w.prof_id, w.ceh_id, w.space_id))

res = set()
for p in 07a:
    for s in s7:
        if (p.prof_id, *s) not in bad:
            res.add(p.prof_id)
return res
```

---

## Вариант 5
02(Детали) <->> 16в(Наличие деталей) <<-> 14(отгрузки)

02(<u>d_id</u>, type, name, dim, plan) \
16в(<u>st_id</u>, <u>d_id</u>, dim, amount, lastdate) \
14(<u>st_id</u>, <u>doc_id</u>, c_id, d_id, dim, amount, date) 
### RA
```bash
i100 = [st_id, d_id][amount>100](16в)
bad = [st_id, d_id][amount<=200](14)
good = i100 - bad
result = [d_id](good)
```

### RIC
find { d.d_id | d ∈ 02 } \
∃ (i ∈ 16в) i.d_id = d.d_id ∧ i.amount > 100 \
∧ ( ∀ (o ∈ 14) \
(o.st_id = i.st_id ∧ o.d_id = d.d_id) \
⇒ o.amount > 200)
### SQL
``` sql
WITH i100 AS (
    SELECT DISTINCT st_id, d_id
    FROM 16в
    WHERE amount > 100
),
bad AS (
    SELECT DISTINCT d_id, st_id
    FROM 14
    WHERE amount <= 200
)
SELECT DISTINCT i.d_id
FROM i100 i
WHERE NOT EXISTS (
    SELECT 1
    FROM bad 
    WHERE bad.st_id = i.st_id AND bad.d_id = i.d_id
)
```
### ORM
``` python
i100 = set()
for i in 16в:
    if i.amount > 100:
        i100.add((i.st_id, i.d_id))

bad = set()
for o in 14:
    if o.amount <= 200:
        bad.add((o.st_id, o.d_id))

result = set()
for (st_id, d_id) in i100:
    if (st_id, d_id) not in bad:
        result.add(d_id)

return result
```

---

## Вариант 10
07в(участки) <->> 08(программа) <<-> 02(Детали)

07в(<u>ceh_id</u>, <u>space_id</u>, name, master_id) \
08(<u>d_id</u>, <u>ceh_id</u>, <u>space_id</u>, <u>year</u>, <u>month</u>, plan) \
02(<u>d_id</u>, type, name, dim, plan) 

### RA
```bash
d100 = [d_id][plan > 100](02)
places = [ceh_id, space_id](07в)
bad = [d_id, ceh_id, space_id][plan <= 100 and year = 2025](08)
result = [ceh_id, space_id]((d100 • places) - bad)
```

### RIC
find { place.ceh_id, place.space_id | place ∈ 07в } \
∃ (d ∈ 02) d.plan > 100 \
∧ ( ∀ (p ∈ 08) \
(p.ceh_id = place.ceh_id ∧ p.space_id = place.space_id ∧ p.d_id = d.d_id ∧ p.year = 2025) \
⇒ p.plan > 100)
### SQL
``` sql

WITH d100 AS (
    SELECT DISTINCT d_id
    FROM 02
    WHERE plan > 100
),
bad AS (
    SELECT DISTINCT d_id, ceh_id, space_id
    FROM 08
    WHERE amount <= 100 AND year = 2025
)
SELECT DISTINCT place.ceh_id, place.space_id
FROM 17в place
CROSS JOIN d100
WHERE NOT EXISTS(
        SELECT 1 
        FROM bad
        WHERE place.ceh_id = bad.ceh_id AND
            place.space_id = bad.space_id AND
            d100.d_id = bad.d_id
)
```
### ORM
``` python
d100 = [d.d_id for d in 02 if d.plan > 100]
bad = set()
for p in 08:
    if p.amount <= 100 and p.year = 2025:
        bad.add((p.d_id, p.ceh_id, p.space_id))

res = set()
for place in 17в:
    for d in d100:
        if (d.d_id, place.ceh_id, place.space_id) not in bad:
            res.add((place.ceh_id, place.space_id))
return res
```

---
## Вариант 11

16в(Наличие деталей) <->> 13б(Учет поставок деталей) <<-> 09в(Поставки детале)

16в(<u>st_id</u>, <u>d_id</u>, dim, amount, lastdate) \
13б(<u>st_id</u>, <u>doc_id</u>, deal_id, d_id, dim, amount, date) \
09в(<u>deal_id</u>, <u>d_id</u>, dim, start, end, plan_amount, plan_price) 

### RA
```bash
s1000 = [deal_id, d_id][plan_amount > 1000](09в)
bad = [deal_id, d_id][amount <= 50](13б)
temp = s1000 - bad
result = [st_id, d_id](16в * temp)
```

### RIC
find { having.st_id, having.d_id | having ∈ 16в } \
∃ (sup ∈ 09в) sup.d_id = having.d_id ∧ d.plan_amount > 1000 \
∧ ( ∀ (getting ∈ 13б) \
(getting.deal_id = sup.deal_id ∧ getting.d_id = sup.d_id) \
⇒ getting.amount > 50)
### SQL
``` sql
WITH s1000 AS (
    SELECT deal_id, d_id
    FROM 09в
    WHERE plan_amount > 1000
),
bad AS (
    SELECT DISTINCT deal_id, d_id
    FROM 13б
    WHERE amount <= 50
),
temp AS (
    SELECT s1000.deal_id, s1000.d_id
    FROM s1000
    WHERE NOT EXISTS (
        SELECT 1
        FROM bad
        WHERE bad.deal_id = s1000.deal_id AND bad.d_id = s1000.d_id
    )
)
SELECT DISTINCT h.st_id, h.d_id
FROM 16в h
WHERE EXISTS (
    SELECT 1
    FROM temp 
    WHERE temp.d_id = h.d_id
)
```
### ORM
``` python
s1000 = set()
for sup in 09в:
    if sup.plan_amount > 1000:
        s1000.add((sup.deal_id, sup.d_id))

bad = set()
for rec in 13б:
    if rec.amount <= 50:
        bad.add((rec.deal_id, rec.d_id))

temp = s1000 - bad

ids = set(d_id for (deal_id, d_id) in temp)

result = set()
for h in 16в:
    if h.d_id in ids:
        result.add((h.st_id, h.d_id))

return result
```

---
## Вариант 15
## Вариант 16
## Вариант 17