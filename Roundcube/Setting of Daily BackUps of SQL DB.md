#  **1. Folder Structure of Daily Backups**

It will look like this:

```
/var/backups/roundcube/
├── 2025/
│   ├── 01/
│   │   ├── 2025-01-01.sql.gz
│   │   ├── 2025-01-02.sql.gz
│   │   └── ...
│   ├── 02/
│   │   ├── 2025-02-01.sql.gz
│   │   ├── ...
│   └── ...
└── 2026/
    ├── 01/
    └── ...
```

Each day → one dump.  
Each month → folder.  
Each year → folder.

Perfect and clean.

---

#  **2. Backup Script (Place in /usr/local/bin/sqlite_backup.sh)**

Create the script:

```
sudo vi /usr/local/bin/sqlite_backup.sh
```

Paste this EXACT script:

```bash
#!/bin/bash

# === CONFIG ===
DB_PATH="/var/www/roundcube/db/sqlite.db"
BACKUP_BASE="/var/backups/roundcube"

# === DATE STRUCTURE ===
YEAR=$(date +%Y)
MONTH=$(date +%m)
DAY=$(date +%Y-%m-%d)

# === BUILD DESTINATION DIRECTORY ===
DEST_DIR="$BACKUP_BASE/$YEAR/$MONTH"
mkdir -p "$DEST_DIR"

# === DUMP FILE NAME ===
BACKUP_FILE="$DEST_DIR/${DAY}.sql.gz"

# === PERFORM SQLITE BACKUP ===
sqlite3 "$DB_PATH" ".backup '/tmp/roundcube_$DAY.db'"

# === CONVERT BACKUP TO SQL TEXT (safer for restores) ===
sqlite3 /tmp/roundcube_$DAY.db .dump | gzip > "$BACKUP_FILE"

# === CLEAN TEMP FILE ===
rm -f /tmp/roundcube_$DAY.db

# === OPTIONAL: KEEP ONLY LAST 30 DAYS OF DAILY BACKUPS ===
find "$BACKUP_BASE" -type f -mtime +30 -name "*.gz" -delete
```

Save and exit.

---

#  **3. Make script executable**

```
sudo chmod +x /usr/local/bin/sqlite_backup.sh
```

---

#  **4. Test script manually**

```
sudo /usr/local/bin/sqlite_backup.sh
```

Then check:

```
ls -R /var/backups/roundcube/
```

You should see the year → month → file structure.

---

#  **5. Create a Cron Job**

Edit crontab:

```
sudo crontab -e
```

Add this:

```
0 2 * * * /usr/local/bin/sqlite_backup.sh >/var/log/sqlite_backup.log 2>&1
```

This runs the backup **every day at 2 AM**.

You’ll also have logs in:

```
/var/log/sqlite_backup.log
```

---

#  **6. Verify Cron Job**

Force-run cron:

```
sudo run-parts --test /etc/cron.daily
```

Or wait until next day and check the backup folder.

---

#  ** PERMISSION FIXS**

Make sure Apache & your script can read the DB:

```
sudo chmod 644 /var/www/roundcube/db/sqlite.db
```

And ensure backup directory is owned by root:

```
sudo mkdir -p /var/backups/roundcube
sudo chown -R root:root /var/backups/roundcube
```

---

# **7. Restore Steps (if needed)**

To restore a backup:

```
gunzip -c /var/backups/roundcube/2025/02/2025-02-28.sql.gz | sqlite3 /var/www/roundcube/db/sqlite.db
```

Done.

---
