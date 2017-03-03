# automatic-db-backup-using-git

# What is it?
This is an instruction how to set up automatic DB backup and deploy it to remote repository.

# Set up repository
Create folder for your repository
#+BEGIN_SRC
$ mkdir -p /home/backups/database && cd /home/backups/database
#+END_SRC

Add shell script file and make it executable
#+BEGIN_SRC
$ touch run_backup.sh
$ chmod +x run_backup.sh
#+END_SRC

Init git and add remote repository, you may ignore some files if you need
#+BEGIN_SRC
$ git init
$ git remote add origin YOUR SSH REPOSITORY.git
$ git add -A
$ git commit -am 'New files'
$ git push -u origin HEAD
#+END_SRC

After executing "git push -u origin HEAD" commad you will get authentication error, unless you alredy have ssh key.
Set up ssh key using original git intruction, remember to set empty passphrase on this step "Enter passphrase (empty for no passphrase):", this will hepl you to avoid authentication issues with crontab.

# run_backup.sh
#+BEGIN_SRC
#!/bin/sh
# Set variables
DB_NAME="your db name"

FULLDATE=$(date +"%Y-%d-%m %H:%M")
NOW=$(date +"%Y-%m-%d-%H-%M")
TEMP_BACKUP="backup.sql"
BACKUP_DIR=$(date +"%Y/%m")

# Check current Git status and update
git status
git pull origin HEAD

# Dump database
mysqldump -u "root" $DB_NAME > "latest_backup.sql" &
wait

# Make backup directory if not exists (format: {year}/{month}/)
if [ ! -d "$BACKUP_DIR" ]; then
  mkdir -p $BACKUP_DIR
fi

# Compress SQL dump
tar -cvzf $BACKUP_DIR/$DB_NAME-$NOW-sql.tar.gz $TEMP_BACKUP

# Remove original SQL dump
rm -f $TEMP_BACKUP

# Add to Git and commit
cd /home/backups/database/
git add -A
git commit -m "Automatic backup - $FULLDATE"
git push origin HEAD
#+END_SRC

# mysqldump credentials
#+BEGIN_SRC
$ cd
$ nano ~/.my.cnf

[mysqldump]
user=USER_NAME
password=USER_PASSWORD
#+END_SRC

# Cron job
#+BEGIN_SRC
$ crontab -e

0 0 * * * cd /home/backups/database; /bin/sh /home/backups/database/run_backup.sh > /home/backups/logs/db.log 2>&1
#+END_SRC

# Logs
You can find logs here /home/backups/logs/db.log
