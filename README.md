# **Workshop PostGIS Vector**

### This workshop aims to initialize the user in the usage of PostGIS using only vector data.



----------
## Install PostgreSQL and the extension PostGIS

PostGIS is a spatial database extender for the database management system (DBMS) PostgreSQL. In order to install PostGIS first we need to install PostgreSQL. You can install PostgreSQL from the official website: [https://www.postgresql.org/download/](https://www.postgresql.org/download/) .
After the installation of PostgreSQL we need to install PostGIS extension. In windows you can achieve that using Stack Builder.
Once PostGIS is installed, one can **activate the spatial extension in a new database** with:

```sql
CREATE EXTENSION POSTGIS;
```

If you explore the database you will notice that the schema public is now populated with new tables, views and functions.
More information about PostGIS instalation can be found in the official documentation: [https://postgis.net/docs/postgis_installation.html](https://postgis.net/docs/postgis_installation.html)

We also recommend a look at PostGIS FAQ:
[https://postgis.net/docs/PostGIS_FAQ.html](https://postgis.net/docs/PostGIS_FAQ.html)

----------

## Load data into the database

Assuming you are not being given access to an already configured version of the database used in this workshop, you will start by creating a new database in your system after which you will load the following shapefiles: *porto_freguesias* ; *ferrovias* and *lugares*

As an alternative you can restore the *postgis_vectors.backup* database from this repository, this DB already as the vectors loaded.

----------


## Requirements

This workshop assumes a basic knowledge of SQL or *Structured Query Language* which is the language you use to interact with a PostgreSQL database (or any other relational database). If you are new to SQL we recommend you to follow the W3 Schools course on SQL :https://www.w3schools.com/sql/

## Explore the database with QGIS DB Manager

Please use your QGIS DB Manager to explore the vectors inside the schema vectors. It is important to know your data before we start analyzing it. Make sure that first you create a connection to the database as explained here: https://www.youtube.com/watch?v=r4pOyVZ3QVs 

From now on we assume QGIS is your client application.

If you are a non-Portuguese speaker, here is a quick explanation of the attributes and table names you may see as you follow the workshop:

Freguesia = Parish
Concelho = Municipality
Distrito = District/region
Ferrovia = railroad
Lugares = places

Therefore the table ```vectors.porto_freguesias``` has all the parishes that exist in the district of Porto, Portugal.

----------

## First queries - standard SQL functionality

**Example 1 - Select all **

The simplest type of query returns all the records and all the attributes of a given table:

```sql 
SELECT * 
FROM  vectors.porto_freguesias;
```
**Example 2 - Condition**

Most of the time you don't want the whole table, you just want the rows that meet certain criteria. The next query will return all the attributes but only for the parishes that belong to the municipality of Matosinhos:

```sql 
SELECT * 
FROM  vectors.porto_freguesias
WHERE concelho = ''MATOSINHOS''
```

**Example 3 - Ilike**

The following query will return the same result as the previous one, but by using  ```Ilike``` instead of the ```=``` operator the string matching becomes case UNsensitive:

```sql 
SELECT * 
FROM  vectors.porto_freguesias
WHERE concelho ILIKE 'mAtOsInHoS';
```
**Example 3 - like**
Like on the other hand is case sensitive therefore the following query will give zero results
```sql
SELECT * 
FROM  vectors.porto_freguesias
WHERE concelho LIKE 'Matosinhos';
```
**Example 4 - Selecting attributes in a query**

You may want to have your query returning specific attributes instead of all of them  (the ```*``` sign) as we have been doing. In the example below we are still getting all the parishes that belong to the municipality of Matosinhos, but we are only interested in knowing a few facts (attributes) about them:

```sql
SELECT freguesia, area_ha 
FROM  vectors.porto_freguesias
WHERE concelho = 'MATOSINHOS'
```
**Example 5 - using alias**

Alias are a very common way of expressing a query. An alias consists on renaming a table on the fly using like in the example below:

```sql
SELECT a.freguesia, a.area_ha 
FROM  vectors.porto_freguesias AS a
WHERE a.concelho = 'MATOSINHOS'
```
Note that under the ```FROM```clause we add the ```AS a```, which is saying that on what this particular query concerns, the table we are calling will be known as 'a' and that is why you see an ```a.``` prefix whenever the query is referring to rows or attributes that belong to table a. Alias are very useful if your query is calling more than one table as you will soon see. 


## Calling PostGIS spatial functions

**Example 6 - ST_Area**

Apart from the geometry columns, so far we have only been doing plain PostgreSQL. We will now start to explore some of the spatial functions offered by PostGIS. A PostGIS function usually takes the form ```name_of_the function(arguments/inputs)`` To demonstrate this principle we will do a simple area calculation:

```sql
SELECT a.freguesia, a.area_ha, st_area(a.geom)--/10000)::int
FROM  vectors.porto_freguesias AS a
WHERE a.concelho = 'MATOSINHOS';
```
As you can see, the function ```ST_Area``` takes one arguments - the geometry .

**Example 7 - ST_Buffer**

Here is another example. This time we call a function that outputs a geometry.

```'sql
SELECT id, ST_Buffer(geom, 1000)
FROM vectors.ferrovia

```



**Example 6 - ST_intersects**

A common spatial problem in GIS is to know if two features share space. There are some variants to this problem but we can use the following as a starting point:

```sql
SELECT b.geom, b.id
FROM vectors.porto_freguesias as a, vectors.ferrovia as b
WHERE a.concelho ilike 'MATOSINHOS' AND ST_intersects(a.geom,b.geom);
```
If you load the results in QGIS, the result might no be exactly what you were expecting - you will get the railroad that intersects the parish of Muro but it is not clipped to the boundaries of the parish because this query only applies a logical test, it does not construct a new geometry.

**Example 7 - ST_intersection**

To get the actual geometry that represents the space shared by two geometries, we have to use the ```ST_Intersection``` function:

```sql
SELECT b.id, st_intersection(b.geom, a.geom) as geom
FROM vectors.porto_freguesias as a, vectors.ferrovia as b
WHERE a.concelho ilike 'MATOSINHOS'  --AND ST_intersects(a.geom,b.geom); 
```

Run the above query again, but this time uncomment the ``` AND ST_intersects(a.geom,b.geom); ```  by deleting the ```--``` characters.




## Integration challenge

Time to put together what you have learned so far to solve a spatial problem. 

**Find all the places that are distanced less than 300m from a railroad**
*Hint*: you will have to nest the function ST_Intersects and ST_Buffer under the WHERE clause

If you manged to solve it, try it with a small variation:
**Find all the places that are distanced less than 300m from a railroad and are located within the municipality of Matosinhos**




















## Solutions to the integration challenges

SELECT a.name, a.geom
FROM vectors.lugares as a, vectors.ferrovia as b
WHERE st_intersects(a.geom, ST_Buffer(b.geom, 300)) 

SELECT a.name, a.geom
FROM vectors.lugares as a, vectors.ferrovia as b, vectors.porto_freguesias as c
WHERE st_intersects(a.geom, ST_Buffer(b.geom, 300)) and c.concelho ilike 'MATOSINHOS' and ST_Intersects(a.geom, c.geom)
