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







