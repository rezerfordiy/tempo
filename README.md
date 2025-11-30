
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
$
\begin{aligned}
&\mathrm{find} \ \{c.c\_id \mid c \in \mathrm{15}\} \\
&\quad \exists \ (d \in \mathrm{02}) \ d.plan > 100 \\
&\quad \& \ (\forall \ (s \in \mathrm{14}) \\
&\qquad (s.d\_id = d.d\_id \ \& \ c.c\_id = s.c\_id) \\
&\qquad \Rightarrow s.st\_id = 5)
\end{aligned}
$
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
$
\begin{aligned}
&\mathrm{find} \ \{ p.prof\_id \mid  p \in \mathrm{07a}\} \\
&\quad \exists \ (s \in \mathrm{07в}) \ s.ceh\_id = 7 \\
&\quad \& \ (\forall \ (w \in \mathrm{10}) \\
&\qquad (w.prof\_id = p.prof\_id \ \& \ w.ceh\_id = s.ceh\_id \ \& \ w.space\_id = s.space\_id) \\
&\qquad \Rightarrow w.power > 4)
\end{aligned}
$
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
16в(<u>st_id</u>, <u>material_id</u>, dim, amount, lastdate) \
14(<u>st_id</u>, <u>doc_id</u>, c_id, d_id, dim, amount, date) 
### RA
```bash
i100 = [st_id][amount > 100](16в)
details = [d_id](02)
bad = [d_id, st_id][amount <= 200](14)
result = [d_id]((details • i100) - bad)
```

### RIC
$
\begin{aligned}
&\mathrm{find} \ \{ d.d\_id \mid d \in \mathrm{02}\} \\
&\quad \exists \ (i \in \mathrm{16в}) \ i.amount > 100 \\
&\quad \& \ (\forall \ (o \in \mathrm{14}) \\
&\qquad (o.st\_id = i.st\_id \ \& \ o.d\_id = d.d\_id) \\
&\qquad \Rightarrow o.amount > 200)
\end{aligned}
$
### SQL
``` sql

WITH i100 AS (
    SELECT DISTINCT st_id
    FROM 16в
    WHERE amount > 100
),
bad AS (
    SELECT DISTINCT d_id, st_id 
    FROM 14
    WHERE amount <= 200
)
SELECT DISTINCT d.d_id
FROM 02 d
CROSS JOIN i100
WHERE NOT EXISTS(
        SELECT 1 
        FROM bad
        WHERE d.d_id = bad.d_id AND
            i100.st_id = bad.st_id    )
```
### ORM
``` python
i100 = [i.st_id for i in 16в if i.amount > 100]
bad = set()
for o in 14:
    if o.amount <= 200:
        bad.add((o.d_id, o.st_id))

res = set()
for d in 02:
    for i in i100:
        if (d.d_id, i.st_id) not in bad:
            res.add(d.d_id)
return res
```

---

## Вариант 10
## Вариант 11
## Вариант 15
## Вариант 16
## Вариант 17