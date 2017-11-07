# **Workshop PostGIS Vector**

### This workshop aims to initialize the user in the usage of PostGIS vector.



----------
## Install PostgreSQL and the extension PostGIS
 
PostGIS is a spatial database extender for the database management system (DBMS) PostgreSQL. In order to install PostGIS first we need to install PostgreSQL. You can install PostgreSQL from the official website: [https://www.postgresql.org/download/](https://www.postgresql.org/download/) . 
After the installation of PostgreSQL we need to install PostGIS extension. In windows you can achieve that using Stack Builder. 
After installing PostGIS one can **activate the spatial extension in a new database** with:

```sql
CREATE EXTENSION POSTGIS;
```

If you explore the database you will notice that the schema public is now populated with new tables, views and functions. 
More information about PostGIS instalation can be found in the official documentation: [https://postgis.net/docs/postgis_installation.html](https://postgis.net/docs/postgis_installation.html)

We also recommend a look at PostGIS FAQ:
[https://postgis.net/docs/PostGIS_FAQ.html](https://postgis.net/docs/PostGIS_FAQ.html)

----------

## Load data into the database

We will start by creating (if not already) a new database in your system and then we will load the following shapefiles: *porto_freguesias* ; *ferrovias* and *lugares*

As an alternative you can restore the *postgis_vectors.backup* database from this repository, this DB already as the vectors loaded.
