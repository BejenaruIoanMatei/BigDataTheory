# MySQL — CRUD & Comenzi Esențiale

---

## 1. Database

```sql
-- Creare
CREATE DATABASE name;

-- Listare
SHOW DATABASES;

-- Selectare
USE name;

-- Stergere
DROP DATABASE name;
```

---

## 2. Create Table

```sql
CREATE TABLE users (
    id      INT PRIMARY KEY AUTO_INCREMENT,
    email   VARCHAR(255) NOT NULL UNIQUE,
    bio     TEXT,
    country VARCHAR(2)
);
```

### Constraints importante de știut
```sql
NOT NULL       -- câmpul nu poate fi NULL
UNIQUE         -- valori unice în coloană
PRIMARY KEY    -- NOT NULL + UNIQUE automat
AUTO_INCREMENT -- generează ID automat (MySQL specific)
DEFAULT 'RO'   -- valoare implicită
CHECK (age > 0)-- validare la nivel de DB
```

---

## 3. Foreign Key

```sql
CREATE TABLE rooms (
    id       INT AUTO_INCREMENT,
    street   VARCHAR(255),
    owner_id INT NOT NULL,
    PRIMARY KEY (id),
    FOREIGN KEY (owner_id) REFERENCES users(id)
        ON DELETE CASCADE   -- dacă userul e șters, șterge și rooms lui
        ON UPDATE CASCADE   -- dacă userul id se schimbă, actualizează automat
);
```

> **De știut la interviu:** `ON DELETE CASCADE` vs `ON DELETE SET NULL` vs `ON DELETE RESTRICT`  
> - `CASCADE` → șterge și înregistrările copil  
> - `SET NULL` → pune NULL în loc de foreign key  
> - `RESTRICT` → blochează ștergerea dacă există înregistrări copil  

---

## 4. INSERT

### Insert simplu
```sql
INSERT INTO users (email, bio, country)
VALUES ('da@gmail.com', 'wasup', 'US');
```

### Insert multiple rânduri
```sql
INSERT INTO users (email, bio, country)
VALUES
    ('ana@gmail.com', 'hello', 'RO'),
    ('ion@gmail.com', 'world', 'RO'),
    ('john@gmail.com', 'hey', 'US');
```

### Insert from SELECT
```sql
-- Copiezi date dintr-un tabel în altul
INSERT INTO users_backup (email, bio, country)
SELECT email, bio, country
FROM users
WHERE country = 'RO';
```

### INSERT ON DUPLICATE KEY UPDATE ⚠️ important pentru interviu
```sql
-- Dacă emailul există deja (UNIQUE), face UPDATE în loc să arunce eroare
INSERT INTO users (email, bio, country)
VALUES ('da@gmail.com', 'new bio', 'US')
ON DUPLICATE KEY UPDATE
    bio = VALUES(bio),
    country = VALUES(country);
```

### INSERT IGNORE — ignoră eroarea dacă există duplicate
```sql
-- Nu face nimic dacă emailul există deja, nu aruncă eroare
INSERT IGNORE INTO users (email, bio, country)
VALUES ('da@gmail.com', 'wasup', 'US');
```

### PostgreSQL echivalent (de știut diferența)
```sql
-- PostgreSQL folosește ON CONFLICT în loc de ON DUPLICATE KEY
INSERT INTO users (email, bio, country)
VALUES ('da@gmail.com', 'new bio', 'US')
ON CONFLICT (email) DO UPDATE
    SET bio = EXCLUDED.bio,
        country = EXCLUDED.country;

-- PostgreSQL ignore (echivalent INSERT IGNORE)
INSERT INTO users (email, bio, country)
VALUES ('da@gmail.com', 'wasup', 'US')
ON CONFLICT (email) DO NOTHING;
```

---

## 5. SELECT

### Select de bază
```sql
SELECT * FROM users;
SELECT email, country FROM users;
```

### Filtrare
```sql
SELECT * FROM users WHERE country = 'RO';
SELECT * FROM users WHERE country = 'RO' AND bio IS NOT NULL;
SELECT * FROM users WHERE country IN ('RO', 'US', 'DE');
SELECT * FROM users WHERE email LIKE '%@gmail.com';
SELECT * FROM users WHERE id BETWEEN 10 AND 20;
```

### Ordonare + Limitare
```sql
SELECT * FROM users ORDER BY email ASC;
SELECT * FROM users ORDER BY id DESC LIMIT 10;
SELECT * FROM users ORDER BY id DESC LIMIT 10 OFFSET 20; -- paginare
```

### Alias
```sql
SELECT email AS user_email, country AS tara
FROM users AS u;
```

### Distinct
```sql
SELECT DISTINCT country FROM users;
```

---

## 6. UPDATE ⚠️

```sql
-- Mereu pune WHERE! Fără WHERE updatezi tot tabelul
UPDATE users
SET bio = 'updated bio', country = 'DE'
WHERE email = 'da@gmail.com';
```

### Update cu expresii
```sql
-- Incrementare
UPDATE products SET stock = stock - 1 WHERE id = 5;

-- Update bazat pe altă coloană
UPDATE users SET bio = CONCAT('Bio: ', email) WHERE bio IS NULL;
```

### Update cu subquery
```sql
-- Updatează toți userii din țări cu mai mult de 100 de useri
UPDATE users
SET bio = 'popular country'
WHERE country IN (
    SELECT country FROM users
    GROUP BY country
    HAVING COUNT(*) > 100
);
```

> ⚠️ **Greșeală clasică:** `UPDATE users SET country = 'RO'` fără WHERE → modifică TOATE rândurile!

---

## 7. DELETE ⚠️

```sql
-- Șterge rânduri specifice
DELETE FROM users WHERE id = 5;
DELETE FROM users WHERE country = 'RO';
```

### Șterge toate datele — 3 variante, diferențe importante

```sql
-- 1. DELETE fără WHERE — șterge rând cu rând, lent, poate fi rollback
DELETE FROM users;

-- 2. TRUNCATE — șterge tot instant, resetează AUTO_INCREMENT, nu poate fi rollback
TRUNCATE TABLE users;

-- 3. DROP TABLE — șterge și structura tabelului, nu doar datele
DROP TABLE users;
```

| Comandă | Șterge date | Șterge structura | Resetează AI | Rollback |
|---|---|---|---|---|
| `DELETE` | ✅ | ❌ | ❌ | ✅ |
| `TRUNCATE` | ✅ | ❌ | ✅ | ❌ |
| `DROP` | ✅ | ✅ | - | ❌ |

> **Întrebare clasică la interviu:** *"Care e diferența dintre DELETE și TRUNCATE?"*  
> **Răspuns:** DELETE șterge rând cu rând și poate fi dat înapoi cu ROLLBACK. TRUNCATE șterge tot dintr-o dată, e mai rapid, resetează AUTO_INCREMENT dar nu poate fi anulat.

---

## 8. ALTER TABLE — Modificare structură

```sql
-- Adaugă o coloană nouă
ALTER TABLE users ADD COLUMN age INT DEFAULT 0;

-- Șterge o coloană
ALTER TABLE users DROP COLUMN bio;

-- Modifică tipul unei coloane
ALTER TABLE users MODIFY COLUMN country VARCHAR(10);

-- Redenumește o coloană
ALTER TABLE users RENAME COLUMN country TO tara;

-- Adaugă un index după creare
ALTER TABLE users ADD INDEX idx_country (country);

-- Adaugă un foreign key după creare
ALTER TABLE rooms ADD CONSTRAINT fk_owner
    FOREIGN KEY (owner_id) REFERENCES users(id);
```

---

## 9. Indexing

```sql
-- Index simplu — accelerează WHERE și ORDER BY pe acea coloană
CREATE INDEX email_index ON users(email);

-- Index compus — util când filtrezi după mai multe coloane împreună
CREATE INDEX idx_country_email ON users(country, email);

-- Index unic — garantează unicitate + performanță
CREATE UNIQUE INDEX idx_email ON users(email);

-- Vizualizare indexuri existente
SHOW INDEX FROM users;

-- Stergere index
DROP INDEX email_index ON users;
```

> **De știut la interviu:** Un index accelerează SELECT dar încetinește INSERT/UPDATE/DELETE  
> deoarece indexul trebuie actualizat la fiecare modificare.

---

## 10. JOINs — Sintaxă completă

```sql
-- INNER JOIN — doar rândurile cu match în ambele tabele
SELECT u.email, r.street
FROM users u
INNER JOIN rooms r ON u.id = r.owner_id;

-- LEFT JOIN — toți userii, chiar dacă nu au rooms
SELECT u.email, r.street
FROM users u
LEFT JOIN rooms r ON u.id = r.owner_id;

-- RIGHT JOIN — toate rooms, chiar dacă userul a fost șters
SELECT u.email, r.street
FROM users u
RIGHT JOIN rooms r ON u.id = r.owner_id;

-- FULL OUTER JOIN — tot din ambele tabele
-- MySQL NU are FULL OUTER JOIN nativ, se simulează cu UNION:
SELECT u.email, r.street FROM users u LEFT JOIN rooms r ON u.id = r.owner_id
UNION
SELECT u.email, r.street FROM users u RIGHT JOIN rooms r ON u.id = r.owner_id;

-- CROSS JOIN — produs cartezian (fiecare rând din A cu fiecare din B)
SELECT u.email, r.street FROM users u CROSS JOIN rooms r;

-- SELF JOIN — un tabel joinat cu el însuși (ex: ierarhie manager-angajat)
SELECT e.email AS employee, m.email AS manager
FROM users e
LEFT JOIN users m ON e.id = m.id;
```

---

## 11. Funcții utile de știut

### String functions
```sql
SUBSTRING(email, 1, 5)        -- extrage substring
CONCAT(first_name, ' ', last_name) -- concatenare
UPPER(email)                  -- majuscule
LOWER(email)                  -- minuscule
LENGTH(bio)                   -- lungimea stringului
TRIM(email)                   -- elimină spații de la capete
REPLACE(bio, 'old', 'new')    -- înlocuiește text
```

### Numeric functions
```sql
ROUND(salary, 2)              -- rotunjire
FLOOR(salary)                 -- rotunjire în jos
CEIL(salary)                  -- rotunjire în sus
ABS(difference)               -- valoare absolută
MOD(id, 2)                    -- modulo (rest împărțire)
```

### Date functions
```sql
NOW()                         -- data și ora curentă
CURDATE()                     -- doar data curentă
YEAR(created_at)              -- extrage anul
MONTH(created_at)             -- extrage luna
DATEDIFF(NOW(), created_at)   -- diferența în zile între două date
DATE_FORMAT(created_at, '%Y-%m') -- formatare dată
```

### Aggregate functions
```sql
COUNT(*)                      -- numărul de rânduri
COUNT(DISTINCT country)       -- valori unice
SUM(salary)                   -- sumă
AVG(salary)                   -- medie
MAX(salary)                   -- maxim
MIN(salary)                   -- minim
```

---

## 12. GROUP BY + HAVING

```sql
-- Câți useri pe fiecare țară
SELECT country, COUNT(*) AS nr_useri
FROM users
GROUP BY country;

-- Doar țările cu mai mult de 10 useri (HAVING filtrează după GROUP BY)
SELECT country, COUNT(*) AS nr_useri
FROM users
GROUP BY country
HAVING COUNT(*) > 10
ORDER BY nr_useri DESC;
```

> **Diferența WHERE vs HAVING:**  
> `WHERE` filtrează **rânduri individuale** înainte de GROUP BY  
> `HAVING` filtrează **grupuri** după GROUP BY

```sql
-- WHERE + HAVING împreună
SELECT country, AVG(salary) as avg_sal
FROM employees
WHERE department = 'IT'        -- filtrează rânduri înainte de grupare
GROUP BY country
HAVING AVG(salary) > 5000;    -- filtrează grupurile după agregare
```

---

## 13. Subqueries

```sql
-- Subquery în WHERE
SELECT email FROM users
WHERE id IN (SELECT owner_id FROM rooms WHERE street LIKE '%Main%');

-- Subquery cu EXISTS
SELECT email FROM users u
WHERE EXISTS (
    SELECT 1 FROM rooms r WHERE r.owner_id = u.id
);

-- Subquery în FROM (derived table)
SELECT dept, avg_salary
FROM (
    SELECT department AS dept, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_salary > 5000;
```

---

## 14. Transactions — ACID

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

-- Dacă totul e ok
COMMIT;

-- Dacă ceva a mers greșit
ROLLBACK;
```

> **De știut la interviu:** ACID = Atomicity, Consistency, Isolation, Durability  
> Tranzacțiile garantează că fie **ambele** operații reușesc, fie **niciuna** — crucial în sisteme financiare.

---

## 15. Quick Reference — Comenzi de Interviu

| Task | Comandă |
|---|---|
| Inserare dacă nu există | `INSERT IGNORE` sau `ON DUPLICATE KEY UPDATE` |
| Ștergere toate datele (cu reset AI) | `TRUNCATE TABLE` |
| Ștergere structură + date | `DROP TABLE` |
| Adăugare coloană la tabel existent | `ALTER TABLE ... ADD COLUMN` |
| Găsire duplicate | `GROUP BY ... HAVING COUNT(*) > 1` |
| Full Outer Join în MySQL | `LEFT JOIN UNION RIGHT JOIN` |
| Paginare rezultate | `LIMIT 10 OFFSET 20` |
| Inserare date din alt tabel | `INSERT INTO ... SELECT ... FROM` |