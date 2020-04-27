# mysql

## Import Table Structures

export tables.csv:

{% code title="export\_tables.sh" %}
```sql
mysql -uroot -p your_password -hlocalhost --batch --raw <<-EOF > tables.csv
select 'public' as table_schema,
       col.table_name,
       col.column_name,
       col.ordinal_position,
       col.data_type as column_type,
       case when (group_concat(constraint_type separator ', '))
                  like '%PRIMARY KEY%'
            then 't' else 'f' end as primary_key,
       case when col.is_nullable = 'YES' then 't' else 'f' end as is_nullable,
       tab.table_comment,
       col.column_comment
from information_schema.columns col
join information_schema.tables tab
     on col.table_schema = tab.table_schema
     and col.table_name = tab.table_name
     and tab.table_type = 'BASE TABLE'
left join information_schema.key_column_usage kcu
     on col.table_schema = kcu.table_schema
     and col.table_name = kcu.table_name
     and col.column_name = kcu.column_name
left join information_schema.table_constraints tco
     on kcu.constraint_schema = tco.constraint_schema
     and kcu.constraint_name = tco.constraint_name
     and kcu.table_name = tco.table_name
where col.table_schema = 'your_database_name' and col.table_schema not in('information_schema', 'sys',
                              'performance_schema', 'mysql')
group by 1,2,3,4,5,7,8,9
order by col.table_schema,
         col.table_name,
         col.ordinal_position
EOF
```
{% endcode %}

{% hint style="info" %}
 You need to replace _**your\_database\_name**_ & _**password**_ & _**host**_ & _**username**_
{% endhint %}

If there are foreign keys in your database schema, you can export it：

{% code title="export\_relations.sh" %}
```sql
mysql -uroot --batch --raw <<-EOF > relations.csv
SELECT 'public' AS 'schema',
       kcu.referenced_table_name AS 'table',
       kcu.referenced_column_name AS 'column',
       kcu.ordinal_position AS no,
       'public' AS relation_table_schema,
       kcu.table_name AS relation_table,
       kcu.column_name AS relation_column
  FROM information_schema.key_column_usage AS kcu
 WHERE referenced_column_name IS NOT NULL 
       AND CONSTRAINT_SCHEMA = 'your_database_name'
EOF
```
{% endcode %}

{% hint style="info" %}
 You need to replace _**your\_database\_name**_ & _**password**_ & _**host**_ & _**username**_
{% endhint %}

![](../.gitbook/assets/image%20%289%29.png)

If there are not any foreign keys in your database schema,  you don't need to import relations csv file.

You can use \`auto draw\`  feature to detect relationship. It has rule:

```text
users.account_id -> accouts.id
```

and check the AUTO DRAW checkbox:

![](../.gitbook/assets/image%20%281%29.png)

result:

![](../.gitbook/assets/image%20%282%29.png)

