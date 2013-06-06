rds-importer
============

A tool to help import existing MySQL databases into Amazon's RDS service.  Inspired by https://engineering.gosquared.com/migrating-mysql-to-amazon-rds.

Requirements
============
- An EC2 instance
  * running Linux (tested with Amazon Linux)
  * in the same region as your target RDS instance
  * enough space to hold your initial database dump and later incremental dumps
  * provisioned IOPS don't hurt to make sure EBS isn't the bottleneck (it usually isn't)
- An RDS instance
  * running MySQL (obviously)
  * you should turn off backups to turn off binlogging to speed up importing (you can turn this on afterwards)
  * turn off [innodb_flush_log_at_trx_commit](http://dev.mysql.com/doc/refman/5.5/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit) in your RDS parameter group
  * turn off multi-AZ as this requires all writes to be synced to secondary during import (you can turn this on afterwards)
  * set log_bin_trust_function_creators to 1 in your RDS parameter group
  * this should be the largest instance class you can afford, and provisioned IOPS here never hurt either.  You can scale the instance down after the import is done.
- On your current MySQL setup
  * binlogging must be on
  * ideally [max_binlog_size](http://dev.mysql.com/doc/refman/5.5/en/replication-options-binary-log.html#sysvar_max_binlog_size) should be set to its maximum of 1073741824.  This is a dynamic variable and can be set via `SET GLOBAL` - make sure you also make the change in your my.cnf so the change persists.
  * as few foreign keys as possible, and ideally none!
- SSH connectivity between your target MySQL machine and your utility EC2 instance

Additionally, all of your tables should be in InnoDB, as this method makes heavy use of transactions, not to mention that RDS is known to not play nicely with anything else.

