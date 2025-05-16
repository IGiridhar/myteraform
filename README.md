Check PostgreSQL Service Status on both primary and Standby
systemctl status postgresql

Check repmgr Daemon
systemctl status repmgrd

Check cluster status and Replication and Role Recognition
repmgr cluster show



Check for Long-Running Transactions

SELECT pid, usename, state, query_start, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;

Optional filter for active transactions only:

SELECT * FROM pg_stat_activity
WHERE state = 'active' AND now() - query_start > interval '2 minutes';


Check Replication Lag On the current primary:

SELECT 
  client_addr,
  state,
  sync_state,
  write_lag,
  flush_lag,
  replay_lag
FROM pg_stat_replication;

Ensure:
state = 'streaming'
replay_lag is small or 0
sync_state = 'sync'

replay_lag shows how far behind the standby is in replaying WALs.

Values like 00:00:00 mean the standby is fully caught up.


Check Replication Slot Lag (Optional)
SELECT slot_name, restart_lsn, confirmed_flush_lsn
FROM pg_replication_slots;

You can compute the difference between pg_current_wal_lsn() and confirmed_flush_lsn.


Check WAL Sender/Receiver Status
On the standby
SELECT status, receive_start_lsn, latest_end_lsn
FROM pg_stat_wal_receiver;

On the primary:
SELECT application_name, state, sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;


Check for Replication Errors in Logs
journalctl -u postgresql


Check Disk Health
Ensure the data directory  has plenty of free space.   make sure all other folders has enough space

df -h


Check Disk I/O Performance
iostat -xz 1 3
Look at:
%util → should be well below 100%
await → average wait time; lower is better (typically < 10 ms)

Check for Disk Errors
dmesg | grep -i error


Check Network Health

ping <standby-hostname-or-IP> -c 4

Look for low latency, no packet loss.

Check TCP Port Accessibility
Ensure standby can connect to the primary:
nc -zv <primary-IP> 5432
















How to Start PostgreSQL Manually (without systemctl)
✅ Step 1: Locate the PostgreSQL Data Directory
This is where the database stores its files. Common locations:

/var/lib/postgresql/14/main

/usr/local/pgsql/data

Your custom path (check postgresql.conf or ask your admin)

If unsure, try:

bash
Copy
Edit
ps -ef | grep [p]ostgres
Look for a line like:

bash
Copy
Edit
postgres -D /var/lib/postgresql/14/main
✅ Step 2: Start PostgreSQL
bash
Copy
Edit
pg_ctl -D /path/to/data_directory start
Example:

bash
Copy
Edit
pg_ctl -D /var/lib/postgresql/14/main start
You should see:

text
Copy
Edit
server starting
Then verify:

bash
Copy
Edit
ps -ef | grep [p]ostgres
🔴 How to Stop PostgreSQL
bash
Copy
Edit
pg_ctl -D /path/to/data_directory stop
You can use shutdown modes:

-m smart (waits for clients to disconnect – default)

-m fast (recommended – safely terminates sessions)

-m immediate (dangerous – abrupt shutdown)

Example:

bash
Copy
Edit
pg_ctl -D /var/lib/postgresql/14/main stop -m fast
🔄 How to Restart PostgreSQL
bash
Copy
Edit
pg_ctl -D /path/to/data_directory restart -m fast
🧪 Verify PostgreSQL Status
bash
Copy
Edit
pg_ctl -D /path/to/data_directory status
✅ Will tell you whether it's running and show the PID.

📝 Notes
You must be the PostgreSQL OS user (usually postgres) to run these commands.

If pg_ctl is not in your PATH, try:

bash
Copy
Edit
/usr/pgsql-14/bin/pg_ctl -D /your/data/dir start
Or add it:

bash
Copy
Edit
export PATH=$PATH:/usr/pgsql-14/bin
Would you like a custom script that can start/stop PostgreSQL manually for your setup?
















