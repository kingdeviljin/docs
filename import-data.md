---
title: Import Data
summary: Learn how to import data into a CockroachDB cluster.
toc: false
---

CockroachDB supports importing data from from `.sql` and some `.csv` files.

{{site.data.alerts.callout_info}}We're also working to develop more robust ways to import data, such as from PostgreSQL and from backups stored on cloud hosting providers.{{site.data.alerts.end}}

<div id="toc"></div>

## Import from SQL File

You can execute batches of `INSERT` statements stored in `.sql` files from the command line, letting you import data into your cluster.

~~~ shell
$ cockroach sql --database=[database name] < statements.sql
~~~

{{site.data.alerts.callout_success}}Grouping each <code>INSERT</code> statement to include approximately 500 rows will provide the best performance.{{site.data.alerts.end}}

<!--## Import from PostgreSQL

>>>>>The following section can be uncommented and published once the following issue is resolved: https://github.com/cockroachdb/cockroach/issues/13490

If you're importing data from a PostgreSQL deployment, you can import the `.sql` file generated by the `pg_dump` command to more quickly import data.

{{site.data.alerts.callout_success}}The <code>.sql</code> files generated by <code>pg_dump</code> provide better performance because they use the <code>COPY</code> statement instead of bulk <code>INSERT</code> statements.{{site.data.alerts.end}}

### Create PostgresSQL SQL File

Which `pg_dump` command you want to use depends on whether you want to import your entire database or only specific tables:

- Entire database:

  ~~~ shell
  $ pg_dump [database] > [filename].sql
  ~~~

- Specific tables:

  ~~~ shell
  $ pg_dump -t [table] [table's database] > [filename].sql
  ~~~

For more detail, see PostgreSQL's documentation on [`pg_dump`](https://www.postgresql.org/docs/9.1/static/app-pgdump.html).

### Reformat SQL File

After generating the `.sql` file, you need to perform a few editing steps before importing it:

1. Manually add the table's `PRIMARY KEY` constraint to the `CREATE TABLE` statement. 

   This has to be done manually because PostgreSQL attempts to add the primary key after creating the table, but CockroachDB requires the primary key be defined upon table creation.
2. Review any other [constraints](constraints.html) to ensure they're properly listed on the table.
3. Remove all statements from the file besides the `CREATE TABLE` and `COPY` statements.

### Import Data

After reformatting the file, you can import it through `psql`:

~~~ shell
$ psql -p [port] -h [node host] -d [database] -U [user] < [file name].sql
~~~

For reference, CockroachDB uses these defaults:

- `[port]`: **26257**
- `[user]`: **root**
-->

## Import from CSV

You can import numeric data stored in `.csv` files by executing a bash script that reads values from the files and uses them in `INSERT` statements.

{{site.data.alerts.callout_danger}}To import non-numerical data, convert the <code>.csv</code> file to a <code>.sql</code> file (you can find free conversion software online), and then import the <code>.sql</code> file.{{site.data.alerts.end}}

### Template

This template reads 3 columns of numerical data, and converts them into `INSERT` statements, but you can easily adapt the variables (`a`, `b`, `c`) to any number of columns.

~~~ sql
> \| IFS=","; while read a b c; do echo "INSERT INTO csv VALUES ($a, $b, $c);"; done < test.csv;
~~~

### Example

In this SQL shell example, use `\!` to look at the rows in a CSV file before creating a table and then using `\|` to insert those rows into the table.

~~~ sql
> \! cat test.csv
~~~
~~~
12, 13, 14
10, 20, 30
~~~
~~~ sql
> CREATE TABLE csv (x INT, y INT, z INT);

> \| IFS=","; while read a b c; do echo "INSERT INTO csv VALUES ($a, $b, $c);"; done < test.csv;

> SELECT * FROM csv;
~~~
~~~
+----+----+----+
| x  | y  | z  |
+----+----+----+
| 12 | 13 | 14 |
| 10 | 20 | 30 |
+----+----+----+
~~~

## See Also

- [Use the Built-in SQL Client](use-the-built-in-sql-client.html)
- [Other Cockroach Commands](cockroach-commands.html)
