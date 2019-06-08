---
title: "As a Porter Among Tables in Postgresql"
date: 2016-03-25T16:00:00+08:00
tags: [PostgreSQL, Database, SQL]
categories: [engineering]
---

Imagine we have two tables, **creations** and **creation_metadata**. **id** of **creations** table is foreign key of **creation_metadata** table. They should be one-to-one relation but actually, records in **creations** table might not have a corresponding row in **creation_metadata** table.

We want to fix this problem and also move some data from **creations** table to **creation_metadata** table.

Before continuing, let’s make the target clear. What we want is

1. Avoid inserting with existing **creation_id** in target table.
2. Update value for those existing ones.

---

### [UPSERT](https://wiki.postgresql.org/wiki/UPSERT) command lands on PostgreSQL!

What a wonderful thing when we need to deal with the problem above. Command looks like below.

![Hooray](https://giphy.com/gifs/celebrate-DKnMqdm9i980E?utm_source=iframe&utm_medium=embed&utm_campaign=Embeds&utm_term=https%3A%2F%2Fcdn.embedly.com%2Fwidgets%2Fmedia.html%3Fsrc%3Dhttps%3A%2F%2Fgiphy.com%2Fembed%2FDKnMqdm9i980E%2Ftwitter%2Fiframe&%3Bsrc_secure=1&%3Burl=http%3A%2F%2Fgiphy.com%2Fgifs%2Fcelebrate-DKnMqdm9i980E&%3Bimage=https%3A%2F%2Fmedia.giphy.com%2Fmedia%2FDKnMqdm9i980E%2Fgiphy.gif&%3Bkey=d04bfffea46d4aeda930ec88cc64b87c&%3Btype=text%2Fhtml&%3Bschema=giphy)

```sql
INSERT INTO creation_metadata (creation_id, category)
     SELECT id, type
       FROM creations
  ON CONFLICT DO UPDATE SET category = EXCLUDED.type;
```
You can check the syntax [here](http://www.postgresql.org/docs/9.5/static/sql-insert.html).

As you see, [UPSERT](https://wiki.postgresql.org/wiki/UPSERT) in PostgreSQL is an extended clause for [INSERT](http://www.postgresql.org/docs/9.5/static/sql-insert.html). Since “**creation_id**“ is **unique** in target table, insertion would fail for existing records and fall into **ON CONFLICT** clause. In the sample above, we do update for conflictions.

---

## Unfortunately...

I’m afraid you cannot use [UPSERT](https://wiki.postgresql.org/wiki/UPSERT) if your PostgreSQL is 9.4 or earlier. Good news is that there are approaches.

## Let’s list the steps.

1. Get those rows needs move.
2. Update for existing ones.
3. Lastly, insert new rows.

---

### Use WITH query (Common Table Expression)

**WITH** query can help to get result that can be referenced in the main query. So we can still have a command to make the goal.

```sql
WITH creations_need_move AS (
  SELECT id, type
    FROM creations
   WHERE type IS NOT NULL
),
update_existing_metadata AS (
  UPDATE creation_metadata (creation_id, category)
     SET category = source.type
    FROM creations_need_move source
   WHERE creation_id = source.id
   RETURNING creation_metadata.*
)
INSERT INTO creation_metadata (creation_id, category)
     SELECT id, type
       FROM creations_need_move
      WHERE NOT EXISTING ( SELECT 1
                             FROM update_existing_metadata up
                            WHERE up.creation_id = creations_need_move.id )
```

### Use PL/PGSQL

Another way is to define new function in PostgreSQL which gives a more straightforward point of view when reading the code.

```sql
CREATE OR REPLACE FUNCTION upsert(target_id INT, type_value TEXT) RETURNS VOID AS $$
  BEGIN
    -- Try update first
    UPDATE creation_metadata SET category = type_value
     WHERE creation_id = target_id;
    -- Return if UPDATE command runs successfully
    IF FOUND THEN
      RETURN;
    END IF;
    -- Since there's no record in creation_metada
    -- We then add new row with INSERT command
    INSERT INTO creation_metadata (creation_id, category)
         VALUES (target_id, type_value);
  END;co
$$ LANGUAGE 'plpgsql';

SELECT upsert(id, type)
  FROM creations
 WHERE type IS NOT NULL
```

---

Relational database performs better when queries are straightforward. We also care about atomic operation and race condition during the process. Two approaches above do loops and this may make the performance worse. Further, they might overwrite the rows comes with new application code.

### INSERT with LEFT JOIN

```sql
INSERT INTO creation_metadata (creation_id, category)
     SELECT creations.id, creations.type
       FROM creations
       LEFT JOIN creation_metadata
              ON creations.id = creation_metadata.creation_id
      WHERE creations.type IS NOT NULL
        AND creation_metadata.category IS NULL
```

The idea here is we actually want the result to be one-to-one relation among two tables and **LEFT JOIN** helps us to get the full results for it. Moreover, we can know which record has no relation in another table. The best part is this is a single query and only take effect **ONCE**!