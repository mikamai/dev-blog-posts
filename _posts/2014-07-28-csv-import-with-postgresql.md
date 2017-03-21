---
ID: 220
post_title: CSV import with PostgreSQL
author: Nicola Racco
post_date: 2014-07-28 12:22:18
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/28/csv-import-with-postgresql/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/93106409299/csv-import-with-postgresql
tumblr_mikamayhem_id:
  - "93106409299"
---
Importing data is one of the tasks I like the least. It requires to write messy code and it will inevitably be really slow, maybe too slow to be viable.
<!--more-->

A good approach in these cases is to rely as much as possible on the database engine using a temporary table to store the CSV content and then calling the various _insert into select_ or _update from select_ queries needed to import data from the temp table. They can definitely save your day.

Last day I stumbled upon [this wonderful feature](http://www.postgresql.org/docs/9.2/static/sql-copy.html) of PostgreSQL that could avoid even the need of parsing the CSV. So, let’s say we have the following database:

![Sample ER Model](https://dev.mikamai.com/wp-content/uploads/2014/07/sample_db_er.png) {.aligncenter}

And the following CSV:

```text
Code;Name;Description
SHO32_LEA01_BLA10;Black shoes;Black leather shoes
SHO32_LEA01_RED10;Red shoes;Red leather shoes
SHO32_PLA90_BLA10;Black shoes;Black plastic shoes
SHO32_PLA90_RED10;Red shoes;Red plastic shoes
HAT76_LEA01_BLA10;Black hat;Black leather hat
HAT76_LEA01_RED10;Red hat;Red leather hat
HAT76_PLA90_BLA10;Black hat;Black plastic hat
HAT76_PLA90_RED10;Red hat;Red plastic hat
```

we can write the entire import algorithm in the DB. Pseudocode:

- import the csv inside a temporary table;
- for each row, detect the correct model code, material code and color code splitting the code we have in the first column;
- create all missing colors, materials and models;
- update all existing products (setting the new name and description);
- insert all new products;

To achieve this we can write a [PostgreSQL function](http://www.postgresql.org/docs/9.1/static/sql-createfunction.html). Basically you can see a function like an aggregate of SQL statements. With a function we can store in the DB all the import logic and when we have to effectively import the data, we can simply execute the function.

Okay, let’s see a function that will load our CSV and will import the data (see the comments for info):
```sql
/* a function named import_products that requires an argument an returns nothing */
CREATE OR REPLACE FUNCTION import_products(file_path text)
  RETURNS VOID
  SECURITY DEFINER
  AS $BODY$
    BEGIN
      DROP TABLE IF EXISTS tmp_import_data;
      /*
       * create the temporary table
       * we start inserting only not null columns
       * and then we update the other columns to set references
       */
      CREATE TEMP TABLE tmp_import_data(
        product_id     integer,
        product_code   varchar(255) NOT NULL,
        model_id       integer,
        model_code     varchar(5),
        material_id    integer,
        material_code  varchar(5),
        color_id       integer,
        color_code     varchar(5),
        name           varchar(255) NOT NULL,
        description    text NOT NULL
      );
      
      /* copy the entire csv in tmp_import_data */
      execute format (
        $$copy tmp_import_data (product_code, name, description)
        from %L
        delimiter &#039;;&#039;
        header
        csv$$
      , file_path);
      
      /* 
       * set model code, material code and color code individually
       * splitting the product code
       */
      UPDATE tmp_import_data
        SET
          model_code    = split_part(product_code, &#039;_&#039;, 1),
          material_code = split_part(product_code, &#039;_&#039;, 2),
          color_code    = split_part(product_code, &#039;_&#039;, 3);
      /* insert new colors */
      INSERT INTO models (code)
        SELECT DISTINCT model_code FROM tmp_import_data
        WHERE NOT EXISTS (SELECT id FROM models WHERE code = model_code);
      /* set references */
      UPDATE tmp_import_data
        SET model_id = s.id FROM (SELECT id, code FROM models) AS s
        WHERE s.code = model_code;
      /* insert new materials */
      INSERT INTO materials (code)
        SELECT DISTINCT material_code FROM tmp_import_data
        WHERE NOT EXISTS (SELECT code FROM materials WHERE code = material_code);
      /* set references */
      UPDATE tmp_import_data
        SET material_id = s.id FROM (SELECT id, code FROM materials) AS s
        WHERE s.code = material_code;
      /* insert new colors */
      INSERT INTO colors (code)
        SELECT DISTINCT color_code FROM tmp_import_data
        WHERE NOT EXISTS (SELECT code FROM colors WHERE code = color_code);
      /* set references */
      UPDATE tmp_import_data
        SET color_id = s.id FROM (SELECT id, code FROM colors) AS s
        WHERE s.code = color_code;
      /* update name and description for existing products */
      UPDATE products 
        SET name = s.name, description = s.description
        FROM (SELECT model_id, material_id, color_id, name, description FROM tmp_import_data) AS s
        WHERE
          products.model_id    = s.model_id AND
          products.material_id = s.material_id AND
          products.color_id    = s.color_id;
      /* insert new products */
      INSERT INTO products (model_id, material_id, color_id, name, description)
        SELECT DISTINCT t.model_id, t.material_id, t.color_id, t.name, t.description
          FROM tmp_import_data AS t
        WHERE NOT EXISTS (
          SELECT id FROM products
          WHERE model_id = t.model_id AND material_id = t.material_id AND color_id = t.color_id
        );
    END;
  $BODY$
  LANGUAGE plpgsql;
```

Now we need only to call the function passing a file name, and the function will take care of loading the stuff:

```sql
SELECT import_products(&quot;/home/user/import.csv&quot;);
```