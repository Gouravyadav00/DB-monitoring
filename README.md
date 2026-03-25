# Database Monitoring Feasibility Matrix

The agent connects with DB credentials from config.json and runs read-only queries. :contentReference[oaicite:0]{index=0}

---

## Feasibility Matrix

---

### a) Monitoring Category: Query Performance Monitoring

────────────────────────────────────────

#### Slow query tracking
MySQL: performance_schema / slow_query_log  
PostgreSQL: pg_stat_statements  
MSSQL: sys.dm_exec_query_stats  
MongoDB: db.currentOp() + profiler  
Cassandra: system.log parse  
Redis: N/A (no SQL)  
Oracle DB: V$SQL / V$SQLAREA  

────────────────────────────────────────

#### Query timeouts / inefficient joins
MySQL: performance_schema  
PostgreSQL: pg_stat_activity  
MSSQL: dm_exec_requests  
MongoDB: Profiler level 2  
Cassandra: Limited (CQL is simple)  
Redis: N/A  
Oracle DB: V$SQL_PLAN  

────────────────────────────────────────

#### Rank by resource consumption
MySQL: performance_schema  
PostgreSQL: pg_stat_statements  
MSSQL: dm_exec_query_stats  
MongoDB: $currentOp aggregation  
Cassandra: system_traces  
Redis: N/A  
Oracle DB: V$SQL order by CPU/IO  

---

### b) Monitoring Category: Connection & Session Monitoring

────────────────────────────────────────

#### Active sessions / max connections
MySQL: SHOW STATUS / max_connections  
PostgreSQL: pg_stat_activity  
MSSQL: dm_exec_sessions  
MongoDB: db.serverStatus().connections  
Cassandra: nodetool info  
Redis: INFO clients  
Oracle DB: V$SESSION / processes  

────────────────────────────────────────

#### Blocked / deadlocked sessions
MySQL: SHOW ENGINE INNODB STATUS  
PostgreSQL: pg_locks + pg_stat_activity  
MSSQL: dm_exec_requests blocking  
MongoDB: db.currentOp() waitingForLock  
Cassandra: Limited  
Redis: N/A (single-threaded)  
Oracle DB: V$LOCK / DBA_BLOCKERS  

────────────────────────────────────────

#### Connection pool exhaustion
MySQL: Threads_connected vs max_connections  
PostgreSQL: numbackends vs max_connections  
MSSQL: dm_exec_connections count  
MongoDB: connections.current vs available  
Cassandra: native driver metrics  
Redis: connected_clients vs maxclients  
Oracle DB: sessions vs processes param  

---

### c) Monitoring Category: Replication & Clustering Health

────────────────────────────────────────

#### Replication lag
MySQL: SHOW SLAVE STATUS / Seconds_Behind_Master  
PostgreSQL: pg_stat_replication / replay_lag  
MSSQL: dm_hadr_* (Always On)  
MongoDB: rs.status() / replSetGetStatus  
Cassandra: nodetool netstats  
Redis: INFO replication  
Oracle DB: V$DATAGUARD_STATS  

────────────────────────────────────────

#### Replica failures / split-brain
MySQL: Slave_IO_Running / Slave_SQL_Running  
PostgreSQL: pg_stat_replication state  
MSSQL: AG health DMVs  
MongoDB: rs.status().members[].health  
Cassandra: nodetool status UN/DN  
Redis: Sentinel / cluster info  
Oracle DB: V$ARCHIVE_GAP  

────────────────────────────────────────

#### Failover events
MySQL: Binary log + error log parse  
PostgreSQL: pg_stat_replication timeline  
MSSQL: AG failover DMVs  
MongoDB: rs.status() election events  
Cassandra: nodetool describecluster  
Redis: Sentinel failover events  
Oracle DB: Alert log parse  

---

### d) Monitoring Category: Storage & Capacity Monitoring

────────────────────────────────────────

#### DB size growth trends
MySQL: information_schema.TABLES  
PostgreSQL: pg_database_size()  
MSSQL: sp_spaceused / dm_db_file_space_usage  
MongoDB: db.stats()  
Cassandra: nodetool tablestats  
Redis: INFO memory / dbsize  
Oracle DB: DBA_SEGMENTS  

────────────────────────────────────────

#### Table/index bloat
MySQL: Table sizes from TABLES  
PostgreSQL: pgstattuple / bloat queries  
MSSQL: dm_db_index_physical_stats  
MongoDB: db.collection.stats()  
Cassandra: nodetool tablestats / compaction  
Redis: N/A (key-value)  
Oracle DB: DBA_SEGMENTS / ANALYZE  

────────────────────────────────────────

#### Disk space forecast
MySQL: Backend task — trend analysis on collected size data over time, linear regression to predict exhaustion date  
PostgreSQL: Same  
MSSQL: Same  
MongoDB: Same  
Cassandra: Same  
Redis: Same  
Oracle DB: Same  

────────────────────────────────────────

#### Tempdb / PGA / Undo usage
MySQL: SHOW GLOBAL STATUS tmp tables  
PostgreSQL: pg_stat_bgwriter  
MSSQL: tempdb.sys.dm_db_file_space_usage  
MongoDB: N/A  
Cassandra: N/A  
Redis: N/A  
Oracle DB: V$PGASTAT / DBA_UNDO_EXTENTS  

---

### e) Monitoring Category: Backup & Restore Monitoring

────────────────────────────────────────

#### Backup job status
MySQL: Parse mysqlbackup / mysqldump log or performance_schema  
PostgreSQL: pg_stat_archiver + pg_backup_start  
MSSQL: msdb.dbo.backupset  
MongoDB: mongodump log parse  
Cassandra: nodetool snapshot list  
Redis: BGSAVE / BGREWRITEAOF status  
Oracle DB: V$RMAN_BACKUP_JOB_DETAILS  

────────────────────────────────────────

#### Last backup age / restore points
MySQL: File timestamp check + backup log  
PostgreSQL: pg_last_xact_replay_timestamp()  
MSSQL: msdb.dbo.backupset MAX date  
MongoDB: Ops Manager API or file check  
Cassandra: Snapshot timestamp check  
Redis: lastsave command  
Oracle DB: V$BACKUP_SET  

────────────────────────────────────────

#### Backup SLA alerts
MySQL: Backend task — compare last backup time against configured SLA threshold  
PostgreSQL: Same  
MSSQL: Same  
MongoDB: Same  
Cassandra: Same  
Redis: Same  
Oracle DB: Same  

---

### f) Monitoring Category: Security & Compliance Monitoring

────────────────────────────────────────

#### Unauthorized / privileged logins
MySQL: mysql.general_log or audit plugin  
PostgreSQL: pg_stat_activity + log_connections  
MSSQL: sys.dm_exec_sessions login audit  
MongoDB: db.getLog('global') auth entries  
Cassandra: system_auth.role_members  
Redis: ACL LIST / AUTH log  
Oracle DB: DBA_AUDIT_TRAIL  

────────────────────────────────────────

#### Failed login attempts
MySQL: Error log parse / performance_schema  
PostgreSQL: pg_stat_activity + CSV log parse  
MSSQL: sys.fn_get_audit_file / error log  
MongoDB: db.getLog('global') auth failures  
Cassandra: system.log auth errors  
Redis: AUTH command failures in log  
Oracle DB: DBA_AUDIT_TRAIL action=LOGON  

────────────────────────────────────────

#### DDL audit (CREATE/ALTER/DROP)
MySQL: performance_schema.events_statements  
PostgreSQL: event_trigger + pg_stat_activity  
MSSQL: DDL triggers / dm_exec_query_stats  
MongoDB: db.getLog('global') command filter  
Cassandra: system_schema changes poll  
Redis: N/A (no schema)  
Oracle DB: DBA_AUDIT_TRAIL  

────────────────────────────────────────

#### SQL injection pattern detection
MySQL: Backend task — regex pattern matching on collected slow/active queries (e.g. UNION SELECT, OR 1=1, stacked queries)  
PostgreSQL: Same  
MSSQL: Same  
MongoDB: Same (NoSQL injection patterns)  
Cassandra: Same  
Redis: N/A  
Oracle DB: Same  

---

### g) Monitoring Category: Lock & Deadlock Monitoring

────────────────────────────────────────

#### Row/table level locks
MySQL: information_schema.INNODB_LOCKS  
PostgreSQL: pg_locks  
MSSQL: dm_tran_locks  
MongoDB: db.currentOp() locks  
Cassandra: Row-level only (lightweight)  
Redis: N/A (single-threaded)  
Oracle DB: V$LOCK / V$LOCKED_OBJECT  

────────────────────────────────────────

#### Deadlock detection + blocking chains
MySQL: SHOW ENGINE INNODB STATUS deadlock section  
PostgreSQL: pg_locks + pg_stat_activity join  
MSSQL: dm_exec_requests + dm_os_waiting_tasks  
MongoDB: db.currentOp() waitingForLock  
Cassandra: Lightweight txn timeouts  
Redis: N/A  
Oracle DB: DBA_BLOCKERS / DBA_WAITERS  

---

### h) Monitoring Category: Resource Utilization

────────────────────────────────────────

#### CPU/Memory/IOPS per instance
MySQL: SHOW GLOBAL STATUS + OS metrics via psutil  
PostgreSQL: pg_stat_bgwriter + OS metrics  
MSSQL: dm_os_sys_info + dm_os_process_memory  
MongoDB: db.serverStatus()  
Cassandra: nodetool tpstats + OS metrics  
Redis: INFO memory / INFO stats  
Oracle DB: V$OSSTAT / V$SGA  

────────────────────────────────────────

#### Buffer cache hit ratio
MySQL: Innodb_buffer_pool_read_requests vs reads  
PostgreSQL: pg_stat_bgwriter buffers_hit ratio  
MSSQL: dm_os_buffer_descriptors  
MongoDB: wiredTiger.cache hit ratio  
Cassandra: Key cache hit rate  
Redis: keyspace_hits vs misses  
Oracle DB: V$SYSSTAT buffer cache hit  

────────────────────────────────────────

#### Log file I/O waits
MySQL: Innodb_log_waits  
PostgreSQL: pg_stat_bgwriter + WAL stats  
MSSQL: dm_io_virtual_file_stats  
MongoDB: db.serverStatus().wiredTiger.log  
Cassandra: Commitlog stats  
Redis: AOF write stats  
Oracle DB: V$LOG_HISTORY / V$SYSTEM_EVENT  

---

### i) Monitoring Category: Index & Fragmentation

────────────────────────────────────────

#### Index fragmentation
MySQL: ANALYZE TABLE + information_schema.STATISTICS  
PostgreSQL: pgstatindex() / bloat queries  
MSSQL: dm_db_index_physical_stats  
MongoDB: db.collection.stats().wiredTiger  
Cassandra: Compaction stats  
Redis: N/A  
Oracle DB: DBA_IND_STATISTICS  

────────────────────────────────────────

#### Missing / unused indexes
MySQL: performance_schema + no-index query scan  
PostgreSQL: pg_stat_user_indexes idx_scan=0  
MSSQL: dm_db_missing_index_details  
MongoDB: $indexStats aggregate  
Cassandra: N/A (primary key only)  
Redis: N/A  
Oracle DB: V$OBJECT_USAGE  

---

### j) Monitoring Category: Job & Scheduler Monitoring

────────────────────────────────────────

#### Scheduled job tracking
MySQL: mysql.event + information_schema.EVENTS  
PostgreSQL: pg_cron extension / pgAgent  
MSSQL: msdb.dbo.sysjobs + sysjobhistory  
MongoDB: Atlas triggers or cron log  
Cassandra: Repair/compaction schedules  
Redis: N/A  
Oracle DB: DBA_SCHEDULER_JOBS / DBA_SCHEDULER_JOB_RUN_DETAILS  

────────────────────────────────────────

#### Job failure detection
MySQL: Event status check  
PostgreSQL: pg_cron.job_run_details  
MSSQL: sysjobhistory run_status  
MongoDB: Log parse  
Cassandra: nodetool compactionstats  
Redis: N/A  
Oracle DB: DBA_SCHEDULER_JOB_RUN_DETAILS status  

---

### k) Monitoring Category: Database Health Score

────────────────────────────────────────

#### Composite KPI score
MySQL: Backend task — weighted score from: availability (ping), performance (slow queries, cache hit), capacity (% used), security (failed logins), replication lag  
PostgreSQL: Same  
MSSQL: Same  
MongoDB: Same  
Cassandra: Same  
Redis: Same (subset)  
Oracle DB: Same  

---

## Summary

┌─────────────────────────────┬───────────┬──────────────────────────────────────────────────────────────┐
│          Category           │ Feasible? │                            Notes                             │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ a) Query Performance        │ All 7 DBs │ Redis has no SQL, so N/A — but that's correct for Redis      │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ b) Connections & Sessions   │ All 7 DBs │ Redis is single-threaded, so deadlocks N/A                   │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ c) Replication & Clustering │ All 7 DBs │ Each has different replication model                         │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ d) Storage & Capacity       │ All 7 DBs │ Disk forecast = backend-side trend analysis                  │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ e) Backup & Restore         │ All 7 DBs │ Some need log/file parsing, MSSQL/Oracle have rich DMVs      │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ f) Security & Compliance    │ All 7 DBs │ SQL injection detection = backend regex on collected queries │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ g) Locks & Deadlocks        │ 6 of 7    │ Redis N/A (single-threaded), Cassandra lightweight only      │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ h) Resource Utilization     │ All 7 DBs │ Agent also uses psutil for OS-level metrics                  │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ i) Index & Fragmentation    │ 5 of 7    │ Redis/Cassandra N/A (no traditional indexes)                 │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ j) Job & Scheduler          │ 5 of 7    │ Redis/Cassandra N/A (no built-in job scheduler)              │
├─────────────────────────────┼───────────┼──────────────────────────────────────────────────────────────┤
│ k) Health Score             │ All 7 DBs │ Backend computes weighted composite from all KPIs            │
└─────────────────────────────┴───────────┴──────────────────────────────────────────────────────────────┘

---

## Final Note

Nothing is "not possible". Every category is monitorable for the databases where it's applicable.  
Where a feature doesn't apply (Redis deadlocks, Cassandra indexes), it's because the DB architecture doesn't have that concept — not a limitation of our agent.
