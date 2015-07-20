# EsutoniaGoDesu database SQL and documentation
This readme describes steps necessary to setup EGD database on Linux.

## Setup
Custom collations are required for the string ordering to work. You need to have both Estonian and Japanese locales installed
 to your computer.

### Install Postgresql in Ubuntu 14.04

Create the file /etc/apt/sources.list.d/pgdg.list, and add a line for the repository: 
    
    sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list.d/pgdg.list


Import the repository signing key, and update the package lists: 

    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

Install PostgreSQL RDBMS version 9.4: 

    sudo apt-get update && sudo apt-get install postgresql-9.4 postgresql-contrib-9.4

### Create EsutoniaGoDesu database

You need to login as database super user under postgresql server. The simplest way to connect as the postgres user is
to change to the postgres unix user on the database server using su command as follows:
    
    sudo sh -c  "su - postgres"
    
Create tablespace directory owned by postgres: 
    
    mkdir /var/lib/postgresql/9.4/main/egdspace

Now connect to database server: 

    psql

Add a user called egdrole. Use this user for all work regarding this database: 

    CREATE USER egdrole WITH PASSWORD 'changeit';
    
Create a tablespace:
    
    CREATE TABLESPACE egdspace OWNER egdrole LOCATION '/var/lib/postgresql/9.4/main/egdspace';

Add a database called egd. Just in case lets use template0 to avoid any site-local additions to template1:

    CREATE DATABASE egd WITH OWNER egdrole ENCODING 'UTF8' TEMPLATE template0 TABLESPACE egdspace;

Connect to database:

    \c egd

Now grant all privileges on database:

    GRANT ALL PRIVILEGES ON DATABASE egd to egdrole;
    
Set public schema owner to egdrole, becuse default is postgres:
    
    ALTER SCHEMA public OWNER TO egdrole;               

### Restore from backup
Your database should be up and ready now, so lets restore schema and data. 
Make sure you'll connect to the datbase with egdrole to avoid mixup with objects belonging to other users.   
 
Restore only the schema. The whole dump should be restored as a single transaction, so the restore is either fully completed or fully rolled back.  

    pg_restore --host=localhost --port=5432 --username=egdrole --dbname=egd "egd.schema.backup" --single-transaction --verbose --exit-on-error
    

Restore only the data:

    pg_restore --host=localhost --port=5432 --username=egdrole --dbname=egd "egd.data.backup" --single-transaction --verbose --exit-on-error


Garbage-collect and analyze a PostgreSQL database.
vacuumdb is a utility for cleaning a PostgreSQL database. 
vacuumdb will also generate internal statistics used by the PostgreSQL query optimizer.
    
    vacuumdb --analyze --host=localhost --port=5432 --username=postgres --dbname=egd


Populate materialized views with data

	REFRESH MATERIALIZED VIEW kanjidic2.vm_kanji_data_with_yomi;

## Backup

Dump only the object definitions (schema), not data.

    pg_dump --schema-only --host=localhost --port=5432 --username=egdrole --format=c --verbose --file="egd.schema.backup" --dbname=egd

Dump only the data, not the schema (data definitions). Table data, large objects, and sequence values are dumped.
    
    pg_dump --data-only --host=localhost --port=5432 --username=egdrole --format=c --blobs --verbose --file="egd.data.backup" --dbname=egd


Dump schema and data (full backup)

	pg_dump --host=localhost --port=5432 --username=egdrole --format=c --blobs --verbose --file="egd.backup" --dbname=egd


## Tips & tricks

This process requires you to to connect several times to the PostgreSQL server, asking for a password each time. 
It is convenient to have a ~/.pgpass file for this purpose.

    echo "localhost:5432:*:egdrole:changeit" >> ~/.pgpass
  
    
If you get this error during restore:    
    
    pg_restore: [archiver (db)] Error while PROCESSING TOC:
    pg_restore: [archiver (db)] Error from TOC entry 4484; 0 0 COMMENT EXTENSION plpgsql 
    pg_restore: [archiver (db)] could not execute query: ERROR:  must be owner of extension plpgsql
        Command was: COMMENT ON EXTENSION plpgsql IS 'PL/pgSQL procedural language';
    
try deleting plpgsql comment before pg_dump and then restore again. The issue is raised because egdrole is not 
superuser and only superuser is allowed to alter objects owned by postgres. 

    COMMENT ON EXTENSION plpgsql IS null;
    

Revoke user privileges    
    
    REVOKE ALL PRIVILEGES ON ALL sequences IN SCHEMA jmet FROM jmdictdbv;
    

    
    
    