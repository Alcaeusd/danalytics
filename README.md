<h3>Data Analytics - Parse health data to postgreSQL database and analyze them using WEKA data mining tool</h3>
<hr>
<h5>Installation on Windows 10</h5>
<li>Install XAMPP from here: https://www.apachefriends.org/download.html
<li>Install PostgreSQL v.10: https://www.postgresql.org/download/windows/ in C:\xampp\pgsql\10
  
<h3>Steps after installation:</h3>
<hr>

**Step A:**

_Create PostgreSQL, Explode Data to CSV, Create Table DataSubject and Import Data to Table from CSV:_

**A)** Rename the columns to an acceptable postgresql format

**B)** Run a query to create the DB

```mysql
CREATE TABLE public."DataSubject"
(
	subject integer,
	age integer,
	sex integer,
	test_time decimal,
	motor_UPDRS decimal,
	total_UPDRS decimal,
	JitterPerCent double precision,
	JitterAbs decimal,
	JitterRAP decimal,
	JitterPPQ5 decimal,
	JitterDDP decimal,
	Shimmer decimal,
	ShimmerDb decimal,
	ShimmerAPQ3 decimal,
	ShimmerAPQ5 decimal,
	ShimmerAPQ11 decimal,
	ShimmerDDA decimal,
	NHR decimal,
	HNR decimal,
	RPDE decimal,
	DFA decimal,
	PPE decimal
)
```

**C)** Explode data to CSV, store it to a known location and run the following query to parse the data into DB

```mysql
COPY "DataSubject"(subject,age,sex,test_time,motor_updrs,total_updrs,Jitterpercent,Jitterabs,Jitterrap,Jitterppq5,Jitterddp,Shimmer,Shimmerdb,Shimmerapq3,Shimmerapq5,Shimmerapq11,Shimmerdda,nhr,hnr,rpde,dfa,ppe) 
FROM 'C:\xampp\pgsql\parkinsons_updrs.csv' DELIMITER ',' CSV HEADER;
```

**Step B: **

_Connect Databse HealthData with WEKA, Check for NULL strings or NULL numerics and change them to "NULL" and "0":_

**A)** Change `DatabaseUtils.props.postgresql` to `DatabaseUtils.props`

Edit the file and Insert 

`jdbcURL=jdbc:postgresql://localhost:5432/HealthData?user=postgres&password=root`

in order to predefine the connection with pgAdmin Server and HealthData Database.

**B)** After connecting with the DB, we need to change empty values to either "NULL" for string and 0 for integers:

`Filter -> unsupervised -> ReplaceMissingWithUserConstant (nominalStringReplacementValue = NULL, numericReplacementValue = 0)`

**C)** If there are any columns like "sex" where the type is boolean, we change it to a numeric one for example:

```mysql
ALTER TABLE "DataSubject" ALTER sex SET DEFAULT null;
ALTER TABLE "DataSubject"
ALTER sex TYPE INTEGER
USING
CASE
WHEN false THEN 0 ELSE 1
END;
```

In our dataset, column "sex" is the only one that needs to be converted to a numeric type.

**D)** Check for negative values within Table DataSubject and after evaluation, remove these rows:

```mysql
SELECT * FROM "DataSubject" WHERE subject < 0 OR age < 0 OR sex < 0 OR test_time < 0 OR motor_UPDRS < 0 OR	total_UPDRS < 0 OR JitterPerCent < 0 OR JitterAbs < 0 OR JitterRAP < 0 OR JitterPPQ5 < 0 OR JitterDDP < 0 OR Shimmer < 0 OR ShimmerDb < 0 OR ShimmerAPQ3 < 0 OR ShimmerAPQ5 < 0 OR ShimmerAPQ11 < 0 OR ShimmerDDA < 0 OR NHR < 0 OR HNR < 0 OR RPDE < 0 OR DFA < 0 OR PPE < 0
```

We notice that these rows have negative values ONLY for column test_time. Therefore, we drop any row that has a negative value on this column:

```mysql
DELETE FROM "DataSubject" WHERE test_time < 0
```
