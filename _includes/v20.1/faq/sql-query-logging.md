There are several ways to log SQL queries. The type of logging you use will depend on your requirements.

- For per-table audit logs, turn on [SQL audit logs](#sql-audit-logs).
- For system troubleshooting and performance optimization, turn on [cluster-wide execution logs](#cluster-wide-execution-logs).
- For local testing, turn on [per-node execution logs](#per-node-execution-logs).

### SQL audit logs

{% include {{ page.version.version }}/misc/experimental-warning.md %}

SQL audit logging is useful if you want to log all queries that are run against specific tables.

- For a tutorial, see [SQL Audit Logging](sql-audit-logging.html).

- For SQL reference documentation, see [`ALTER TABLE ... EXPERIMENTAL_AUDIT`](experimental-audit.html).

### Cluster-wide execution logs

For production clusters, the best way to log all queries is to turn on the [cluster-wide setting](cluster-settings.html) `sql.trace.log_statement_execute`:

~~~ sql
> SET CLUSTER SETTING sql.trace.log_statement_execute = true;
~~~

With this setting on, each node of the cluster writes all SQL queries it executes to a secondary `cockroach-sql-exec` log file. Use the symlink `cockroach-sql-exec.log` to open the most recent log. When you no longer need to log queries, you can turn the setting back off:

~~~ sql
> SET CLUSTER SETTING sql.trace.log_statement_execute = false;
~~~

Another useful cluster setting is `sql.log.slow_query.latency_threshold`, which is used to log only queries whose service latency exceeds a specified threshold value (e.g., 100 milliseconds):

~~~ sql
> SET CLUSTER SETTING sql.log.slow_query.latency_threshold = '100ms';
~~~

Each node that serves as a gateway will then record slow SQL queries to a `cockroach-sql-slow` log file. Use the symlink `cockroach-sql-slow.log` to open the most recent log. For more details on logging slow queries, see [Slow query log](query-behavior-troubleshooting.html#slow-query-log).

Log files are written to CockroachDB's standard [log directory](debug-and-error-logs.html#write-to-file).

### Per-node execution logs

Alternatively, if you are testing CockroachDB locally and want to log queries executed just by a specific node, you can either pass a CLI flag at node startup, or execute a SQL function on a running node.

Using the CLI to start a new node, pass the `--vmodule` flag to the [`cockroach start`](cockroach-start.html) command. For example, to start a single node locally and log all client-generated SQL queries it executes, you'd run:

~~~ shell
$ cockroach start --insecure --listen-addr=localhost --vmodule=exec_log=2 --join=<join addresses>
~~~

{{site.data.alerts.callout_success}}
To log CockroachDB-generated SQL queries as well, use `--vmodule=exec_log=3`.
{{site.data.alerts.end}}

From the SQL prompt on a running node, execute the `crdb_internal.set_vmodule()` [function](functions-and-operators.html):

{% include copy-clipboard.html %}
~~~ sql
> SELECT crdb_internal.set_vmodule('exec_log=2');
~~~

This will result in the following output:

~~~
  crdb_internal.set_vmodule
+---------------------------+
                          0
(1 row)
~~~

Once the logging is enabled, all client-generated SQL queries executed by the node will be written to the primary [CockroachDB log file](debug-and-error-logs.html) as follows:

~~~
I180402 19:12:28.112957 394661 sql/exec_log.go:173  [n1,client=127.0.0.1:50155,user=root] exec "psql" {} "SELECT version()" {} 0.795 1 ""
~~~
