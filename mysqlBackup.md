## MySQL Database Backup Guide

### 1. Prerequisites (Passwordless SSH)

*** Generate an SSH key on the Database/App server and copy it to the storage server to enable passwordless transfer:
  ``` 
    ssh-keygen -t rsa
    ssh-copy-id user@storageServ
  ```
### 2. Core mysqldump Command Syntax
To dump a database into an .sql file:

```Bash
mysqldump -h <host_or_ip> -u <username> -p'<password>' <database_name> > <output_file.sql>
⚠️ Note: Do not leave a space between -p and your password if specifying it directly in the command line (e.g., -pasdfgdsd).

```
### 3. Quick Local Command & Manual Transfer
```
  # Take local dump
  mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > db_$(date +%F).sql

  # Copy to storage server
  scp db_$(date +%F).sql natasha@ststor01:/tmp/
```
### 4. Full Backup Script
```
  #!/bin/bash

# 1. Create a local variable for the filename
BACKUP_FILE="db_$(date +%F).sql"

# 2. Take the database dump from stapp01
mysqldump -h stapp01 -u kodekloud_roy -pasdfgdsd kodekloud_db01 > /tmp/$BACKUP_FILE

# 3. Ensure target directory exists on ststor01
ssh natasha@ststor01 "mkdir -p /home/natasha/db_backups"

# 4. Copy the backup file to ststor01
scp /tmp/$BACKUP_FILE natasha@ststor01:/home/natasha/db_backups/

# 5. Clean up temporary local file
rm -f /tmp/$BACKUP_FILE

```

### 5. Implementation Steps for Jenkins Job (database-backup)

1. Add Build Step:
  - In your Jenkins job configuration, add an Execute shell build step.
  - Paste the script from Section 4 into the command box.

2. Configure Schedule (Triggers):
  - Under Build Triggers, check Build periodically.
  - Set the cron schedule to run every 10 minutes:
```bash
*/10 * * * *
```

