# 🐘 Oracle Daily Tasks Cheatsheet

## 👤 User & Session Management

- **Show current user**
  ```sql
  SELECT USER FROM DUAL;
  ```

- **Show running sessions**
  ```sql
  SELECT SID, SERIAL#, USERNAME, STATUS FROM V$SESSION;
  ```

- **Kill a session**
  ```sql
  ALTER SYSTEM KILL SESSION 'sid,serial#' IMMEDIATE;
  ```

## 📋 Schema & Table Management

- **Show tables in current schema**
  ```sql
  SELECT TABLE_NAME FROM USER_TABLES;
  ```

- **Find long-running queries**
  ```sql
  SELECT * FROM V$SESSION_LONGOPS;
  ```

## 💾 Tablespace & Storage

- **Tablespace usage**
  ```sql
  SELECT TABLESPACE_NAME, BYTES/1024/1024 AS MB_USED
  FROM DBA_DATA_FILES;
  ```

- **Free space in tablespaces**
  ```sql
  SELECT TABLESPACE_NAME, SUM(BYTES)/1024/1024 AS MB_FREE
  FROM DBA_FREE_SPACE
  GROUP BY TABLESPACE_NAME;
  ```

- **Create a new tablespace**
  ```sql
  CREATE TABLESPACE my_tablespace
    DATAFILE 'my_tablespace01.dbf' SIZE 100M
    AUTOEXTEND ON NEXT 10M MAXSIZE 500M;
  ```

- **Add datafile to tablespace**
  ```sql
  ALTER TABLESPACE my_tablespace
    ADD DATAFILE 'my_tablespace02.dbf' SIZE 50M;
  ```

## 👥 User Management

- **Create a new user**
  ```sql
  CREATE USER new_user IDENTIFIED BY password;
  ```

- **Grant privileges to a user**
  ```sql
  GRANT CONNECT, RESOURCE TO new_user;
  ```

- **Grant DBA privileges**
  ```sql
  GRANT DBA TO new_user;
  ```

- **Change user password**
  ```sql
  ALTER USER new_user IDENTIFIED BY new_password;
  ```

- **Drop a user (and all objects)**
  ```sql
  DROP USER new_user CASCADE;
  ```

- **List all users**
  ```sql
  SELECT USERNAME FROM DBA_USERS;
  ```

- **Show user roles**
  ```sql
  SELECT * FROM DBA_ROLE_PRIVS WHERE GRANTEE = 'NEW_USER';
  ```

- **Show user system privileges**
  ```sql
  SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE = 'NEW_USER';
  ```

- **Show user object privileges**
  ```sql
  SELECT * FROM DBA_TAB_PRIVS WHERE GRANTEE = 'NEW_USER';
  ```

## 🗄️ Backup & Restore

- **Backup database (RMAN)**
  ```sql
  -- Start RMAN and run:
  BACKUP DATABASE;
  ```

- **Backup a tablespace (RMAN)**
  ```sql
  BACKUP TABLESPACE my_tablespace;
  ```

- **Export a schema (Data Pump)**
  ```bash
  expdp system/password@db schemas=MY_SCHEMA directory=DATA_PUMP_DIR dumpfile=my_schema.dmp logfile=expdp_my_schema.log
  ```

- **Import a schema (Data Pump)**
  ```bash
  impdp system/password@db schemas=MY_SCHEMA directory=DATA_PUMP_DIR dumpfile=my_schema.dmp logfile=impdp_my_schema.log
  ```

## 🩺 Monitoring & Status

- **Check database status**
  ```sql
  SELECT STATUS FROM V$INSTANCE;
  ```