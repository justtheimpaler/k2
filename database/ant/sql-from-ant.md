# How to run SQL in background from Ant

## Prerequites

- Java installed.
- Jakarta Ant installed.
- JDBC driver on a known location.
- Database details: driver class, JDBC url, username, and password.

## Create the following files

1. build.xml:

```xml
<?xml version="1.0"?>

<project name="SQL-batch-runner" basedir="." default="run-sql">

  <loadproperties srcFile="db.properties" />

  <target name="run-sql">

  <tstamp>
    <format property="start.time" pattern="d-MMM-yyyy, HH:mm:ss" />
  </tstamp>
  <echo message="[ Starting at ${start.time} ]" />
  <echo message="db.driverclass=${db.driverclass}" />
  <echo message="db.url=${db.url}" />
  <echo message="db.username=${db.username}" />
  <echo message="" />

  <sql driver="${db.driverclass}"
       url="${db.url}"
       userid="${db.username}"
       password="${db.password}"
       onerror="abort"
       print="true"
       showheaders="true"
       showtrailers="true"
       showWarnings="true"
       >
    <transaction src="command.sql" />
    <classpath>
      <pathelement location="${db.driverlibrary}" />
    </classpath>
  </sql>

  <tstamp>
    <format property="end.time" pattern="d-MMM-yyyy, HH:mm:ss" />
  </tstamp>
  <echo message="[ Ending at ${end.time} ]" />

</target>

</project>
```

2. db.properties

```properties
db.driverlibrary=/my/path/to/jbdc/driver.jar
db.driverclass=mydriverclass
db.url=url
db.username=username
db.password=password
```

3. run-in-background.sh:

```bash
#!/bin/bash
LOGS_DIR=./all-logs
LOG_LINK=./log.txt
 
if [ ! -d "$LOGS_DIR" ]; then
  mkdir "$LOGS_DIR"
fi
TIME_ID=`date +%Y%m%d-%H%M%S`
 
LOG_FILE=${LOGS_DIR}/log-${TIME_ID}.txt
touch "$LOG_FILE" 
 
if [ -f "$LOG_LINK" ]; then
  rm "$LOG_LINK"
fi
ln -s "$LOG_FILE" "$LOG_LINK"
nohup ant < /dev/null > $LOG_FILE 2>&1 &
 
echo "[ Tailing log file $LOG_FILE ]"
tail -f "$LOG_FILE"
```

Make it executable:

```bash
chmod u+x run-in-background.sh
```

4. command.sql

```
select * from dual;
```

- Add here all the SQL commands you want to execute.
- Only SQL commands allowed, not SQL*Plus or any non-valid commands.
- Individual lines within the statements can be commented using either --, // or REM at the start of the line.

## Run the command

To run the SQL commands in background just type:

```
./run-in-background.sh
```

The script runs the command in background and tails the log file (type Ctrl-C to stop tailing the log file). When the execution ends it show a message like:

```
[ Ending at 2-Jan-2014, 17:05:23 ]
```

If the script runs for a long time, you can check the execution later looking at the log file log.txt.


