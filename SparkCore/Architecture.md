# Spark Architecture — Note Complete pentru Interviu

---

## 1. Ce este Spark

Apache Spark este un **unified computing engine** pentru procesare paralelă pe clustere.  
**NU este un storage system** — citește date, le transformă, le trimite mai departe.

> **De știut la interviu:** Spark e in-memory by default, de aceea e de ~100x mai rapid decât Hadoop MapReduce care scrie pe disk între fiecare step.

---

## 2. Driver vs Executor

### Driver
- Planifică joburile, coordonează executorii
- Menține **DAG-ul** (Directed Acyclic Graph) al operațiilor
- Aproape niciodată nu atinge datele direct
- Dacă Driver-ul pică → **jobul întreg eșuează** (single point of failure)

### Executor
- Execută tasks și raportează statusul către Driver
- Fiecare executor are **slots = nr. CPU cores**
- Pe Databricks: 1 executor per worker node (1:1)
- Dacă un Executor pică → Driver-ul **reasignează task-urile** la alt executor (fault tolerance)

### Cluster Manager — intermediarul între Driver și resurse
- Spark Standalone, YARN, Kubernetes, Mesos
- Pe Databricks: managerul e gestionat automat

---

## 3. DAG — Directed Acyclic Graph

Acesta e unul dintre cele mai frecvente subiecte la interviu.

- Când scrii transformări în Spark, **nu se execută imediat** (lazy evaluation)
- Spark construiește un DAG al tuturor operațiilor
- DAG-ul e împărțit în **Stages**, iar fiecare Stage în **Tasks**
- **Stage boundary** = apare la fiecare Wide Transformation (shuffle)
- La o Action, Driver-ul trimite DAG-ul către Cluster Manager care îl execută

```
DAG
 └── Stage 1 (narrow transformations)
      └── Task 1 | Task 2 | Task 3 ...  (unul per partition)
 └── Stage 2 (după un shuffle)
      └── Task 1 | Task 2 | Task 3 ...
```

> **Întrebare tipică:** *"De ce Spark folosește lazy evaluation?"*  
> **Răspuns:** Pentru că permite Catalyst Optimizer să vadă întregul plan și să optimizeze înainte de execuție — de exemplu, poate elimina coloane inutile sau reordona filtrele.

---

## 4. RDD vs DataFrame vs Dataset

| | RDD | DataFrame | Dataset |
|---|---|---|---|
| API | Low-level | High-level | High-level |
| Type safety | Da | Nu (la compile time) | Da |
| Optimizer | Nu | Catalyst ✅ | Catalyst ✅ |
| Limbaj | Scala, Java, Python | Toate | Scala, Java |
| Folosit azi | Rar | **Cel mai comun** | Mai ales Scala |

> **De reținut:** În Python (PySpark) nu există Datasets — doar RDD și DataFrame. Dataset există doar în Scala/Java.

---

## 5. Transformations vs Actions

### Narrow Transformations — fără shuffle, datele rămân în aceeași partiție
- `filter()`, `select()`, `withColumn()`, `map()`, `flatMap()`, `union()`

### Wide Transformations — cauzează shuffle, date redistribuite între partiții
- `groupBy()`, `join()`, `sort()`, `orderBy()`, `distinct()`, `repartition()`

### Actions — declanșează execuția DAG-ului
- `collect()`, `count()`, `show()`, `take(n)`, `save()`, `first()`

---

## 6. Shuffling

Shuffling = redistribuirea datelor între executori prin rețea.

### De ce e scump
1. Scrie date pe disk
2. Trimite date prin rețea
3. Citește date de pe disk pe nodul destinație

### Cum eviți sau reduci shuffling-ul
- Folosești **Broadcast Join** în loc de shuffle join (când o tabelă e mică)
- Eviți `orderBy()` global când nu e necesar
- Folosești `coalesce()` în loc de `repartition()` când reduci partiții
- **Bucketing** — pre-partiționezi datele la scriere, evitând shuffle la join-uri viitoare

---

## 7. Partitions — Repartition vs Coalesce

**Partition** = unitatea de bază a paralelismului. 1 partition → 1 task → 1 slot pe executor.

**Regula generală:** 2-4 partiții per CPU core disponibil în cluster.

| | `repartition(n)` | `coalesce(n)` |
|---|---|---|
| Shuffle | **DA** (full shuffle) | **NU** (evită shuffle) |
| Crește partiții | ✅ | ❌ |
| Scade partiții | ✅ | ✅ (mai eficient) |
| Când folosești | Când mărești sau distribui uniform | Când reduci partiții înainte de save |

> **Regula de aur:** Vrei să mergi de la 200 → 10 partiții? Folosești `coalesce`.  
> Vrei să mergi de la 10 → 200? Folosești `repartition`.

---

## 8. Broadcast Join

Un subiect aproape garantat la interviu de Big Data.

- **Problema:** Join-urile normale cauzează shuffle masiv (Wide Transformation)
- **Soluția:** Dacă **una dintre tabele e mică** (default < 10MB în Spark), o trimiți (broadcast) pe fiecare executor în memorie
- Astfel fiecare executor face join-ul local, **fără nicio comunicare prin rețea**

```python
from pyspark.sql.functions import broadcast

result = large_df.join(broadcast(small_df), "id")
```

> **Întrebare tipică:** *"Când folosești Broadcast Join?"*  
> **Răspuns:** Când una dintre tabele încape în memoria unui executor (câteva sute de MB maxim). Evită complet shuffle-ul pentru acel join.

---

## 9. Optimizările Spark

### Catalyst Optimizer — optimizare logică și fizică a planului de query
- **Logical Plan** → Analizează și simplifică (predicate pushdown, column pruning)
- **Physical Plan** → Alege cel mai eficient plan de execuție
- **Code Generation** → Generează bytecode Java optimizat

### Tungsten — optimizare la nivel de memorie și CPU
- Gestionează memoria direct (off-heap) pentru a evita garbage collection din JVM
- Generează cod binar optimizat

### AQE (Adaptive Query Execution) — Spark 3.0+
- Ajustează planul **în timpul execuției**, bazat pe statistici reale
- Poate fuziona automat partiții mici (**coalesce small partitions**)
- Poate schimba un Shuffle Join într-un Broadcast Join dacă vede că o tabelă e mică

### Cache / Persist

```python
df.cache()          # shortcut pentru MEMORY_AND_DISK
df.persist()        # configurabil cu StorageLevel
df.unpersist()      # eliberezi memoria manual
```

- `cache()` = salvează DataFrame în memorie (MEMORY_ONLY)
- `persist(StorageLevel.DISK_ONLY)` = salvează pe disk
- Folosit când refolosești același DataFrame de mai multe ori în pipeline

### Alte optimizări (lista completă)
- Folosește **Parquet sau Delta** în loc de CSV
- Evită operații scumpe ca `sort()` global
- Minimizează volumul de date cât mai devreme (filter early)
- Repartition / Coalesce înainte de scriere
- Evită **UDF-uri** (User Defined Functions) — nu beneficiază de Catalyst
- Bucketing pentru join-uri repetate
- Optimizează configurația clusterului (memorie, cores)

---

## 10. Data Skewness

**Problema:** Dacă datele nu sunt distribuite uniform între partiții, unele task-uri durează mult mai mult (skewed partitions), iar restul executoarelor stau degeaba.

**Exemplu:** Faci un `groupBy("country")` și 80% din date sunt din "US" → o partiție e uriașă.

### Soluții
- **Salting** — adaugi o coloană artificială random pentru a distribui datele
- **Broadcast Join** — dacă una din tabele e mică
- **AQE** în Spark 3.0+ detectează automat skew și îl mitigează

---

## 11. Formate de date

| Format | Avantaje | Dezavantaje | Când folosești |
|---|---|---|---|
| **CSV** | Simplu, human-readable | Lent, fără schema, mare | Niciodată în producție |
| **JSON** | Flexibil, semi-structurat | Lent, mare | API responses, ingestie |
| **Parquet** | Columnar, comprimat, rapid | Binary, nu human-readable | **Analytics, Data Lake** |
| **Delta Lake** | Parquet + ACID + time travel | Necesită Databricks/Delta | **Producție, Lakehouse** |

> **Regula:** CSV pentru development/ingestie. Parquet/Delta pentru orice altceva.

---

## 12. Top 10 Întrebări de Interviu — Răspunsuri Scurte

| # | Întrebare | Răspuns scurt |
|---|---|---|
| 1 | Ce e lazy evaluation? | Transformările nu se execută până la o Action — permite Catalyst să optimizeze întregul plan. |
| 2 | Diferența RDD vs DataFrame? | RDD e low-level, fără optimizer. DataFrame are Catalyst Optimizer și e mult mai rapid în practică. |
| 3 | Ce e un DAG? | Grafic orientat aciclic al tuturor operațiilor — împărțit în Stages la fiecare shuffle. |
| 4 | De ce e shuffling scump? | Scrie pe disk, trimite prin rețea, citește de pe disk — implică I/O masiv. |
| 5 | repartition vs coalesce? | repartition face shuffle (poate crește/scădea). coalesce nu face shuffle (doar scade). |
| 6 | Când folosești Broadcast Join? | Când una dintre tabele e mică — evită complet shuffle-ul. |
| 7 | Ce face AQE? | Optimizează planul de execuție în timp real bazat pe statistici colectate în runtime. |
| 8 | Ce e o partiție? | Unitatea de bază a paralelismului — un task procesează o partiție pe un slot de executor. |
| 9 | Ce se întâmplă dacă un Executor pică? | Driver-ul reasignează task-urile la alți executori (fault tolerance prin data lineage). |
| 10 | Narrow vs Wide transformation? | Narrow = fără shuffle (filter, select). Wide = cu shuffle (groupBy, join, sort). |

---
