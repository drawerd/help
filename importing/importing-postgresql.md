# postgresql

## Import Table Structures

export tables.csv:

{% code title="export\_tables.sh" %}
```sql
psql -d your_database_name <<-EOF > tables.csv
COPY (SELECT c.table_schema, 
       c.table_name, 
       c.column_name, 
       c.udt_name as column_type, 
       c.ordinal_position, 
       case when c.is_nullable = 'YES' then 't' else 'f' end as is_nullable, 
       case when kcu.constraint_name is null then 'f' else 't' end as primary_key,
       pg_catalog.col_description(format('%s.%s', c.table_schema, c.table_name)::regclass::oid, c.ordinal_position) as column_comment,
       obj_description(format('%s.%s', c.table_schema, c.table_name)::regclass::oid, 'pg_class') as table_comment 
 FROM  information_schema.columns c 
       LEFT JOIN information_schema.key_column_usage as kcu ON 
         kcu.table_catalog = c.table_catalog and 
         kcu.table_schema = c.table_schema and 
         kcu.table_name = c.table_name and 
         kcu.column_name = c.column_name and 
         kcu.position_in_unique_constraint is NULL
 WHERE c.table_schema not in ('information_schema', 'pg_catalog') 
 ORDER BY 1,2,5
) TO STDOUT DELIMITER ',' CSV HEADER
EOF 
```
{% endcode %}

{% hint style="info" %}
You need to replace _**your\_database\_name**_ & _**password**_ & _**host**_ & _**username**_
{% endhint %}

If there are foreign keys in your database schema, you can export it：

{% code title="export\_relations.sh" %}
```sql
psql -d your_database_name <<-EOF > relations.csv
COPY (
  SELECT rel_kcu.table_schema AS schema,
         rel_kcu.table_name   AS table,
         rel_kcu.column_name  AS column,
         kcu.ordinal_position AS no,
         kcu.table_schema     AS relation_table_schema,
         kcu.table_name       AS relation_table,
         kcu.column_name      AS relation_column,
         kcu.constraint_name
    FROM information_schema.table_constraints tco
         JOIN information_schema.key_column_usage kcu
            ON tco.constraint_schema = kcu.constraint_schema
            AND tco.constraint_name = kcu.constraint_name
         JOIN information_schema.referential_constraints rco
            ON tco.constraint_schema = rco.constraint_schema
            AND tco.constraint_name = rco.constraint_name
         JOIN information_schema.key_column_usage rel_kcu
            ON rco.unique_constraint_schema = rel_kcu.constraint_schema
            AND rco.unique_constraint_name = rel_kcu.constraint_name
            AND kcu.ordinal_position = rel_kcu.ordinal_position
   WHERE tco.constraint_type = 'FOREIGN KEY' AND kcu.table_schema NOT IN ('information_schema', 'pg_catalog')
ORDER BY kcu.table_schema,
         kcu.table_name,
         kcu.ordinal_position
) TO STDOUT DELIMITER ',' CSV HEADER
EOF
```
{% endcode %}

![](../.gitbook/assets/image%20%2826%29.png)

If there are not any foreign keys in your database schema,  you don't need to import relations CSV file.

You can use \`auto draw\`  feature to detect relationship. It has rule:

```text
users.account_id -> accouts.id
```

and check the AUTO DRAW checkbox:

![](../.gitbook/assets/image%20%2825%29.png)

result:

![](../.gitbook/assets/image%20%285%29.png)

