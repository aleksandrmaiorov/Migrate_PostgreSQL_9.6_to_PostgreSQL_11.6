# Upgrade PostgreSQL from 9.6 to 11.6

## Install Postgresql v11 repo

```bash
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y
```


Connect to current postgresql server and list databases on server

```bash
\l
```

## Install postgresql server and client v11

```bash
yum install postgresql11 postgresql11-server -y
```

To initiate database creation for v11 as root run

```bash
/usr/pgsql-11/bin/postgresql11-setup initdb
```
Once you are done with creation of database change your account to postgres

```bash
su - postgres
```

## IMPORTANT to do next step: 
**Please make sure you are in home dictory of postgres user**

```bash
cd ~
```

**Run to check the ability to upgrade your data from version 9.6 to 11:
**

```bash
/usr/pgsql-11/bin/pg_upgrade --old-bindir=/usr/pgsql-9.6/bin/ --new-bindir=/usr/pgsql-11/bin/ --old-datadir=/var/lib/pgsql/9.6/data/ --new-datadir=/var/lib/pgsql/11/data/ --check
```

Result should be something like this

    Performing Consistency Checks on Old Live Server
    ------------------------------------------------
    Checking cluster versions                                   ok
    Checking database user is the install user                  ok
    Checking database connection settings                       ok
    Checking for prepared transactions                          ok
    Checking for reg* data types in user tables                 ok
    Checking for contrib/isn with bigint-passing mismatch       ok
    Checking for invalid "unknown" user columns                 ok
    Checking for hash indexes                                   ok
    Checking for presence of required libraries                 ok
    Checking database user is the install user                  ok
    Checking for prepared transactions                          ok
    
    *Clusters are compatible*
    

**change account to root in order to stop postgresql-9.6
**
```bash
systemctl stop postgresql-9.6.service
```

Login as postgres once again  and run
```bash
su - postgres
cd ~
```

Run following command to start data migraiton

```bash
/usr/pgsql-11/bin/pg_upgrade --old-bindir=/usr/pgsql-9.6/bin/ --new-bindir=/usr/pgsql-11/bin/ --old-datadir=/var/lib/pgsql/9.6/data/ --new-datadir=/var/lib/pgsql/11/data/ 
```
result should be something like this

    Performing Consistency Checks
    -----------------------------
    Checking cluster versions                                   ok
    Checking database user is the install user                  ok
    Checking database connection settings                       ok
    Checking for prepared transactions                          ok
    Checking for reg* data types in user tables                 ok
    Checking for contrib/isn with bigint-passing mismatch       ok
    Checking for invalid "unknown" user columns                 ok
    Creating dump of global objects                             ok
    Creating dump of database schemas
                                                                ok
    Checking for presence of required libraries                 ok
    Checking database user is the install user                  ok
    Checking for prepared transactions                          ok
    
    If pg_upgrade fails after this point, you must re-initdb the
    new cluster before continuing.
    
    Performing Upgrade
    ------------------
    Analyzing all rows in the new cluster                       ok
    Freezing all rows in the new cluster                        ok
    Deleting files from new pg_xact                             ok
    Copying old pg_clog to new server                           ok
    Setting next transaction ID and epoch for new cluster       ok
    Deleting files from new pg_multixact/offsets                ok
    Copying old pg_multixact/offsets to new server              ok
    Deleting files from new pg_multixact/members                ok
    Copying old pg_multixact/members to new server              ok
    Setting next multixact ID and offset for new cluster        ok
    Resetting WAL archives                                      ok
    Setting frozenxid and minmxid counters in new cluster       ok
    Restoring global objects in the new cluster                 ok
    Restoring database schemas in the new cluster
                                                                ok
    Copying user relation files
                                                                ok
    Setting next OID for new cluster                            ok
    Sync data directory to disk                                 ok
    Creating script to analyze new cluster                      ok
    Creating script to delete old cluster                       ok
    Checking for hash indexes                                   ok
    
    Upgrade Complete
    ----------------
    Optimizer statistics are not transferred by pg_upgrade so,
    once you start the new server, consider running:
        ./analyze_new_cluster.sh
    
    Running this script will delete the old cluster's data files:
        ./delete_old_cluster.sh

After migration complete run as root 
```bash
systemctl start postgresql-11.service && systemctl enable postgresql-11.service
```
The pg_upgrade script creates two new scriptions  in its working directory, analyze_new_cluster.sh  and delete_old_cluster.sh.
Depending on the size of your database probably a good idea to run analyze_new_cluster.sh:
```bash
su - postgres
```
```bash
cd ~
./analyze_new_cluster.sh
```
You should see similar results
    This script will generate minimal optimizer statistics rapidly
    so your system is usable, and then gather statistics twice more
    with increasing accuracy.  When it is done, your system will
    have the default level of optimizer statistics.
    
    If you have used ALTER TABLE to modify the statistics target for
    any tables, you might want to remove them and restore them after
    running this script because they will delay fast statistics generation.
    
    If you would like default statistics as quickly as possible, cancel
    this script and run:
        "/usr/pgsql-11/bin/vacuumdb" --all --analyze-only
    
    vacuumdb: processing database "postgres": Generating medium optimizer statistics (10 targets)
    vacuumdb: processing database "postgres": Generating default (full) optimizer statistics
    
    Done

If everything is good you can remove old cluster using
```bash
cd ~
./delete_old_cluster.sh
```

Uninstall postgresql-9.6 packages
