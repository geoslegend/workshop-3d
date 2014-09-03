3D Spatial queries
==================

Now that we have the 3D data in shape and that we have done some visualization, we can also do some 3D analysis inside the database.

We use QGIS BD manager to launch queries and dynamically see the results.

3D extrusion
------------

```SQL

-- 3D Extrusion of forests

SELECT 
    gid
    , ST_Translate(
                 ST_Extrude(
                     ST_Buffer(geom, 250)
                 , 0, 0, 100)
           , 0, 0, altitude) AS geom
FROM
    lands 
WHERE 
    type='forest'
```

You can now view the generated data with QGIS / Horao.

Some important points :
* The geometry column must be called geom
* A non-null unique integer (e.g gid) is mandatory with QGIS in the result
* When entering queries in the DB manager, you should not write any ";" at the end !

3D Intersection
----------------

We compute the 3D intersection of two 3D volumes around forest, which could be interpreted for exemple as the place where some endemic species would be able to meet each other.

```SQL
SET postgis.backend = 'sfcgal';

WITH 
 f1 AS (
    SELECT 
        gid
        , ST_Translate(
                 ST_Extrude(
                     ST_Buffer(geom, 250)
                 , 0, 0, 100)
           , 0, 0, altitude) AS geom
    FROM 
        lands 
    WHERE 
        type='forest'
        AND gid = 7
),
 f2 AS (
    SELECT 
        gid
        , ST_Translate(
                 ST_Extrude(
                     ST_Buffer(geom, 250)
                 , 0, 0, 100)
           , 0, 0, altitude) AS geom
    FROM 
        lands 
    WHERE 
        type='forest'
        AND gid = 12
 )
SELECT 
    ST_3DIntersection(f1.geom, f2.geom) AS geom, 
    1 AS gid
FROM 
    f1, f2
```

Bar graphs
----------

We can use the extrusion to generate some bar graphs representing the amount of bikes which are available on each public bike station.

```SQL
SELECT 
    gid
    , ST_Extrude(
               ST_Buffer(geom, 20),
               0, 0, nbbornette::integer * 30
            ) AS geom
FROM
    bike
```

Left to the reader :
* Use the queries from previous chapter to elevate the graph bars to the right elevation thanks to the DEM.

Another 3D intersection
-----------------------

Looking for the areas where the forest species meet some water species...

```SQL
-- 3D Intersects

SET postgis.backend = 'sfcgal';

WITH f AS (
    SELECT
        gid
        , ST_Translate(
             ST_Extrude(
                     ST_Buffer(geom, 800)
                 , 0, 0, 50)
           , 0, 0, altitude) AS geom
    FROM 
        lands 
    WHERE 
        type='forest'
)
SELECT 
    l.geom AS geom
    , l.gid AS gid
FROM 
    f, 
    lands l
WHERE 
    f.geom && l.geom
    AND l.type = 'water'
    AND ST_3DIntersects(f.geom, ST_Extrude(l.geom, 0, 0, 50))
```