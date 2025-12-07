# SIMRS_PLSQL_Capstone
PL/SQL Capstone Project - Smart Inventory Monitoring &amp; Reordering System.

# File: PHASE_IV_README.md
# Phase IV: Database Creation - SIMRS Project

## Overview
This phase involves the creation of the Oracle pluggable database (PDB) and its core configuration for the Smart Inventory Monitoring & Reordering System (SIMRS).

## Database Details
- **PDB Name:** `
GRPA_27129_DOROTHY_SIMS_DB`
- **Admin User:** `SIMS_admin` (Password: Dorothy) - Has DBA role within the PDB.
- **Application User:** `SIMS_user` (Password: inventory123) - Used for all table creation and PL/SQL development.

## Configuration Summary
1.  **Tablespaces Created:**
    - `SIMS_DATA`: For table data (200MB, auto-extensible)
CREATE TABLESPACE SIMS_data 
DATAFILE 'SIMS_data01.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE 1G;

    - `SIMRS_IDX`: For indexes (100MB, auto-extensible).
    - CREATE TABLESPACE SIMS_index
 DATAFILE 'SIMS_index01.dbf' SIZE 50M AUTOEXTEND ON NEXT 25M MAXSIZE 500M;

    - `SIMRS_TEMP`: Temporary tablespace for sorting (100MB, auto-extensible).
    CREATE TEMPORARY TABLESPACE SIMS_temp
 TEMPFILE 'SIMS_temp01.dbf' SIZE 50M AUTOEXTEND ON;
  
3.  **Privileges:** The application user (`SIMS_user`) has all necessary object creation privileges.
CREATE USER SIMS_user IDENTIFIED BY Dorothy
DEFAULT TABLESPACE SIMS_data
TEMPORARY TABLESPACE SIMS_temp
QUOTA UNLIMITED ON SIMS_data
QUOTA UNLIMITED ON SIMS_index;

'GRANT CONNECT, RESOURCE TO SIMS_user;
GRANT CREATE SESSION, CREATE VIEW, CREATE SEQUENCE TO SIMS_user;
GRANT UNLIMITED TABLESPACE TO SIMS_user;


## Execution Instructions
1.  **Prerequisites:** Oracle Database 12c/19c/21c with multitenant architecture. Access as a privileged user (SYS, SYSTEM, or a user with `CREATE PLUGGABLE DATABASE` privilege).
<img width="734" height="189" alt="GRANTING PREVILEGE TO THE USER" src="https://github.com/user-attachments/assets/87e01855-e4c8-4e1e-928a-39daa093c030" />

2.  **Step 1:** Run the `mon_27129_Dorothy_SIMRS_Database_Creation.sql` script in SQL*Plus or SQL Developer connected to the root container (CDB).
<img width="832" height="97" alt="shifting to the new pdb" src="https://github.com/user-attachments/assets/3ba2cd08-039c-43a4-8f2a-3b3a3b93bc3a" />

3.  **Step 2:** Verify the output. The script auto-switches to the new PDB and creates the tablespaces and users.
<img width="832" height="97" alt="shifting to the new pdb" src="https://github.com/user-attachments/assets/3ba2cd08-039c-43a4-8f2a-3b3a3b93bc3a" />
<img width="782" height="177" alt="pdb creation" src="https://github.com/user-attachments/assets/035b47a6-f592-49c5-b51b-de16de1dd1ae" />
<img width="686" height="319" alt="SHOWING PDBS AND CURRENT CONTAINER" src="https://github.com/user-attachments/assets/efb0bcaa-e320-4922-bdb1-62cdd0d20ccc" />
<img width="1017" height="205" alt="INDEX TABLESPACE CREATION" src="https://github.com/user-attachments/assets/cf72ee97-b64a-4b02-86c6-2042f99c3314" />
<img width="523" height="178" alt="USER CREATION" src="https://github.com/user-attachments/assets/2f485f2c-9ce2-47d3-b3b2-821e851c8b0e" />




## Files in This Phase
- `GRPA_27129_Dorothy_SIMS_Database_Creation.sql`: Main creation script.
- `PHASE_IV_README.md`: This file.
