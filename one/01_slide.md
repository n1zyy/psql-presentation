!SLIDE
# Using Postgres and psql #
## For people who kind of know MySQL ##

!SLIDE bullets incremental
# Assumptions #

* You're at least marginally familiar with MySQL
* You find psql really cryptic
* You don't just want to use a GUI

!SLIDE subsection
# Launching It #

!SLIDE commandline
    $ cat /etc/aeolus-conductor/database.yml
    ## snip ##
    production:
      adapter: postgresql
      database: conductor
      username: aeolus
      password: v23zj59an
      host: localhost
    ## snip ##

!SLIDE commandline
# Launching It as a User#

    $ psql conductor aeolus
    psql (9.0.5)
    Type "help" for help.

    conductor=>

!SLIDE commandline
# Launching It (Not) #

    $ sudo psql
    psql: FATAL:  role "root" does not exist

!SLIDE commandline
# Launching It (Insanely) #

    $ sudo su - postgres
    -bash-4.2$ psql conductor
    psql (9.0.5)
    Type "help" for help.

    conductor=#

!SLIDE bullets incremental
# Launching - Lessons Learned #

* Syntax: "psql [opts]... [db_name [username]]"
* Will be prompted for a password if required
* "postgres" system is user is analagous to root


!SLIDE subsection
# Listing Things #
## Yes, it needs a subsection ##
### No, I'm not joking. ###

!SLIDE
# MySQL is Easy #

    @@@ sql
    SHOW DATABASES;
    SHOW TABLES FROM conductor;
    USE conductor;
    SHOW TABLES LIKE "%pool%";

!SLIDE
# Postgres is Not Intuitive #

    @@@ sql
    \l
    \dt
    \c conductor
    \dt *pool*

!SLIDE bullets incremental
# But It's Not Entirely Crazy #
* \l -- list databases
* \d -- describe/list things
* \dx variations
* \+ makes it verbose

!SLIDE commandline
# \l lists databases #

    $ postgres=# \l
                                        List of databases
             Name          |   Owner   | Encoding | Collation | Ctype |   Access privileges
    -----------------------+-----------+----------+-----------+-------+-----------------------
     candlepin             | candlepin | UTF8     | C         | C     |
     conductor             | aeolus    | UTF8     | C         | C     |
     conductor_development | aeolus    | UTF8     | C         | C     |
     conductor_test        | aeolus    | UTF8     | C         | C     |
     postgres              | postgres  | UTF8     | C         | C     |
     template0             | postgres  | UTF8     | C         | C     | =c/postgres          +
                           |           |          |           |       | postgres=CTc/postgres
     template1             | postgres  | UTF8     | C         | C     | =c/postgres          +
                           |           |          |           |       | postgres=CTc/postgres


!SLIDE commandline
# \c changes database #

    $ conductor=# \c conductor
    You are now connected to database "conductor".


!SLIDE commandline
# \d lists everything #

    $ conductor=# \d
    .                       List of relations
     Schema |                Name                |   Type   | Owner
    --------+------------------------------------+----------+--------
     public | base_permission_objects            | table    | aeolus
     public | base_permission_objects_id_seq     | sequence | aeolus
     public | catalog_entries                    | table    | aeolus
     public | catalogs                           | table    | aeolus
     public | catalogs_id_seq                    | sequence | aeolus
     public | cloud_accounts_id_seq              | sequence | aeolus
     public | config_servers                     | table    | aeolus
     public | config_servers_id_seq              | sequence | aeolus


!SLIDE commandline
# \d NAME describes NAME #

    $ conductor=# \d roles
                                            Table "public.roles"
         Column      |            Type             |                     Modifiers
    -----------------+-----------------------------+----------------------------------------------------
     id              | integer                     | not null default nextval('roles_id_seq'::regclass)
     name            | character varying(255)      | not null
     scope           | character varying(255)      | not null
     lock_version    | integer                     | default 0
     created_at      | timestamp without time zone |
     updated_at      | timestamp without time zone |
     assign_to_owner | boolean                     | default false
    Indexes:
        "roles_pkey" PRIMARY KEY, btree (id)

!SLIDE commandline
# + makes it verbose #

    $ conductor=# \d+ roles
                                                        Table "public.roles"
         Column      |            Type             |                     Modifiers                      | Storage  | Description
    -----------------+-----------------------------+----------------------------------------------------+----------+-------------
     id              | integer                     | not null default nextval('roles_id_seq'::regclass) | plain    |
     name            | character varying(255)      | not null                                           | extended |
     scope           | character varying(255)      | not null                                           | extended |
     lock_version    | integer                     | default 0                                          | plain    |
     created_at      | timestamp without time zone |                                                    | plain    |
     updated_at      | timestamp without time zone |                                                    | plain    |
     assign_to_owner | boolean                     | default false                                      | plain    |
    Indexes:
        "roles_pkey" PRIMARY KEY, btree (id)
    Has OIDs: no



!SLIDE bullets
# \dx lists X things #

* \dt - Tables
* \di - Indexes
* \ds - Sequences
* \dv - Views
* \dS - System Views

!SLIDE commandline
# Tables: \dt #


    $ conductor=# \dt
                       List of relations
    Schema |              Name               | Type  | Owner
    --------+---------------------------------+-------+--------
    public | base_permission_objects         | table | aeolus
    public | catalog_entries                 | table | aeolus
    public | catalogs                        | table | aeolus
    public | config_servers                  | table | aeolus
    public | credential_definitions          | table | aeolus
    public | credentials                     | table | aeolus
    public | delayed_jobs                    | table | aeolus
    public | deployments                     | table | aeolus



!SLIDE commandline
# + still makes it verbose #
    $ conductor=# \dt+
    .                                 List of relations
     Schema |              Name               | Type  | Owner  |    Size    | Description
    --------+---------------------------------+-------+--------+------------+-------------
     public | base_permission_objects         | table | aeolus | 8192 bytes |
     public | catalog_entries                 | table | aeolus | 0 bytes    |
     public | catalogs                        | table | aeolus | 0 bytes    |
     public | config_servers                  | table | aeolus | 0 bytes    |
     public | credential_definitions          | table | aeolus | 8192 bytes |
     public | credentials                     | table | aeolus | 8192 bytes |
     public | delayed_jobs                    | table | aeolus | 0 bytes    |
     public | deployments                     | table | aeolus | 8192 bytes |


!SLIDE commandline
# Indexes: \di #
    $ conductor=> \di
                                        List of relations
     Schema |               Name               | Type  | Owner  |            Table
    --------+----------------------------------+-------+--------+-----------------------------
     public | base_permission_objects_pkey     | index | aeolus | base_permission_objects
     public | catalogs_pkey                    | index | aeolus | catalogs
     public | cloud_accounts_pkey              | index | aeolus | provider_accounts
     public | config_servers_pkey              | index | aeolus | config_servers
     public | credential_definitions_pkey      | index | aeolus | credential_definitions
     public | credentials_pkey                 | index | aeolus | credentials
     public | delayed_jobs_pkey                | index | aeolus | delayed_jobs
     public | delayed_jobs_priority            | index | aeolus | delayed_jobs



!SLIDE subsection
# Some Simple Queries #
## Showing the subtle differences from MySQL ##

!SLIDE commandline
# Simplish Select (Wrong) #

    $ conductor=# SELECT * FROM users WHERE login = "admin";
    ERROR:  column "admin" does not exist
    LINE 1: SELECT * FROM users WHERE login = "admin";
                                               ^
    $ conductor=#

!SLIDE bullets
# Double quotes are verboten #
* They're actually not part of the ANSI standard
* Optional double quotes for **field names** and such
* Double quotes roughly equivalent to backticks in MySQL


!SLIDE commandline
# Simple Select (correct!) #
    $ conductor=# SELECT id, login FROM users WHERE login = 'admin';
     id | login
    ----+-------
      1 | admin
    (1 row)


!SLIDE commandline
# With Excessive Double Quotes #

    $ conductor=# SELECT "id", "login" FROM users WHERE "login" = 'admin';
     id | login
    ----+-------
      1 | admin
    (1 row)


!SLIDE commandline
# MySQL is Case Insensitive #
    $ mysql> SELECT user_login FROM wp_users where user_login = 'n1zyy';
    +------------+
    | user_login |
    +------------+
    | n1zyy      |
    +------------+
    1 row in set (0.00 sec)


    $ mysql> SELECT user_login FROM wp_users where user_login = 'N1ZYY';
    +------------+
    | user_login |
    +------------+
    | n1zyy      |
    +------------+
    1 row in set (0.00 sec)


!SLIDE commandline
# Postgres is Case Sensitive #
    $ conductor=# SELECT id, login FROM users WHERE "login" = 'admin';
     id | login
    ----+-------
      1 | admin
    (1 row)

    $ conductor=# SELECT id, login FROM users WHERE "login" = 'ADMIN';
     id | login
    ----+-------
    (0 rows)
