# **Connection pooling in PostgreSQL :  PgBouncer**

| **Author**    | **Created on** | **Version** | **Last updated on** | **Reviewed by** |
|---------------|----------------|-------------|----------------------|------------------------|
| Shubhanshi    | 11.06.2025     | v1          | 11.06.2025         | Deepak Nishad       |

# Table of content :
-   [Overview](#overview)
-   [Purpose of PgBouncer](#purpose-of-pgbouncer)
-   [System Requirements](#ystem-Requirements)
-   [How does PgBouncer work?](#how-does-pgbouncer-work?)
-   [When should we use PgBouncer?](when-should-we-use-pgbouncer?)
-   [Dataflow Diagram](#Dataflow-Diagram)
-   [Step-by-step installation & configuration of pgbouncer](#Step-by-step-installation-&-configuration-of-pgbouncer)
-   [Troubleshooting](#Troubleshooting)
-   [Conclusion](#conclusion)
-   [Contact Information](#contact-information)
-   [References](#references)

# Overview:
Connection pooling means that connections are reused rather than created each time when the connection is required.To facilitate connection reuse , a memory cache or DB connections, called connection pool.



# Purpose of PgBouncer:
PgBouncer is an open source , single binary ,lightweight PostgreSQL connection pooler designed to reduce connection overhead and improve performance for PostgreSQL databases. It manages incoming client connections and efficiently allocates them to PostgreSQL backend connections.
PgBouncer is used as a connection pool mechanism for PostgreSQL.


# How does PgBouncer works?
- Acts as an intermediary layer that pools and manages database connections.

- Instead of each client opening a direct connection to the database (which is expensive), PgBouncer reuses a limited number of persistent connections.

- PgBouncer supports all the authentication mechanism that the PostgreSQLserver supports, after this PgBouncer checks for a cached connection with the same username+DB combination.
If a cached connection is found , it returns the connection to the client, if not it creates a new connection.

# When should we use PgBouncer?
- It reduces PostgreSQL resource consumption (memory , backends , forks),...
Thus, if you have a lot of client connections to the DB , PgBouncer can reduce the no. of PostgreSQL backend processes. The response time between database and a client is also decreases , because no need to fork a new backend process.


# Dataflow Diagram:





![Screenshot 2025-06-12 152001](https://github.com/user-attachments/assets/d21dab8f-1514-49c1-a2ef-421ba7834fc6)




# Step-by-step installation & configuration of pgbouncer:
Before you start, you must have:

* A Scaleway account(aws ,azure etc) logged into the console
* An SSH key
* An Instance running on Ubuntu or Debian
* sudo privileges or access to the root user

## *Steps to Install PgBouncer on Ubuntu/Debian:*
* Update package lists:
```
sudo apt update
```
* Install PgBouncer:
```
sudo apt install pgbouncer
```
* Check PgBouncer version:
```
pgbouncer --version
```
**Next step is to configure the pgbouncer👇**
## *Configuration*:
Path of config file : /etc/pgbouncer/pgbouncer.ini

**Step 1:** 
```
sudo vi/nano /etc/pgbouncer/pgbouncer.ini
```
In Database section:

Add `dbname➡Targetted/Destination dbname , Default=same as client dbname`






![Screenshot 2025-06-13 161531](https://github.com/user-attachments/assets/6b6c2360-6aca-48f4-b1c9-612e9cf1fa11)







For ex: employee=host=127.0.0.1 port=5432 dbname=employee

**Step 2:** In pooler personality question section add:
```
pool_mode=transaction
``` 
**Reason**: `pool_mode = transaction` instead of session to allow better connection reuse by releasing the backend connection after each transaction, improving resource efficiency for applications with short-lived queries.

and under connection limit block `set max_client_connection=1000` or any value this basically controls how many clients can connect to PgBouncer simultaneously.

**Step 3:** Under authentication setting
```
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
```

**Step 4:** Under users allowed into database 'pgbouncer'
```
admin_users = postgres or any user that has sudo privileges
```
Now save and close this file and move to next step 

**Step 5:** Open /etc/pgbouncer/userlist.txt file in any editor (if you're using PostgreSQLversion starting form 14 then the default pswd encryption method will be scram-sha-256)

and paste the pswd in double quotes format like shown below
```
 "shubhanshi" "SCRAM-SHA-256$4096:xd55Qkz5WNarPsBWG5wTIw==$ZsgbdrzalbuawI2RJmWEITyNQvBnoGRz/IRgd+7Ktdc=:ru+QjXpBUYf0WAaAR/b/q2g7P8eYSE1sBG0RYps28u4="
```
To check the hash pswd you can use the following method:
```
SELECT usename , paaswd from pg_shadow;
```
**Step 6:**
Now connecting to the PostgreSQL without using pgbouncer. The following command will start a test with 1000 concurrent clients for 60 seconds, connecting directly to the PostgreSQL.
```
pgbench -c <no.of connectionS> -T <time> <my_db> -h <ip> -p <port> -U <user_name>
```

but firstly you need to initialize pgbench by using 
```
pgbench-i <my_db>

```
**Step 7:**  and then go to 
```
/etc/postgresql/<version>/main/pg_hba.conf 
```
and add md5 under 
#IPv4 local connections :
```
host             all               all                 127.0.0.1/32           md5

```
**Step 8:** Restart postgresql and pgbouncer
```
sudo systemctl restart pgbouncer
sudo systemctl restart postresql
```

**Step 10:** Run with port 6432 (default port for pgbouncer)
```
pgbench -c <no.of connectionS> -T <time> <my_db> -h <ip> -p <port> -U <user_name>
```
**Step 11:** Now we will test and compare the no of transactions per seconds that the DB performs when the application connects to the db without pgbouncer and when it uses pgbouncer.

create a file `vi test.sql` and I am adding a simple query `SELECT 1;`


**Step 10:** without pgbouncer
```
pgbench -c <no.of connectionS> -T <time> <my_db> -h <ip> -p <5432> -U <user_name> -f<file_name>
```

**Step 11:** With pgbouncer

```
pgbench -c <no.of connectionS> -T <time> <my_db> -h <ip> -p <6432> -U <user_name> -f <file_name>
```




![Screenshot 2025-06-16 125202](https://github.com/user-attachments/assets/da410a4c-b364-4e27-b8f8-c8ba5640a35f)

## Conclusion

Using PgBouncer significantly improved overall performance in high-concurrency scenarios by reducing latency, increasing transaction throughput, and optimizing connection management.



## Contact Information

| Name       | Email address     |
|------------|-------------------|
| Shubhanshi | shubhanshi.suryal@mygurukulam.co |

