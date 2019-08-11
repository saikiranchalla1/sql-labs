## Oracle
Pull Oracle's docker image```docker pull store/oracle/database-enterprise:12.2.0.1```
Starting an Oracle Database Server instance
Starting an Oracle database server instance is as simple as executing

    docker run -d -it --name <oracle-db> store/oracle/database-enterprise:12.2.0.1

where <oracle-db> is the name of the container and 12.2.0.1 is the Docker image tag.

The database server is ready to use when the STATUS field shows (healthy) in the output of docker ps.

Connecting to the Database Server Container
The default password to connect to the database with sys user is Oradoc_db1.

Connecting from within the container
The database server can be connected to by executing SQL*Plus,

    docker exec -it <oracle-db> bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
and execute
    conn / as sysdba

Connecting from outside the container
The database server exposes port 1521 for Oracle client connections over SQLNet protocol and port 5500 for Oracle XML DB. SQLPlus or any JDBC client can be used to connect to the database server from outside the container.

To connect from outside the container start the container with -P or -p option as,

    docker run -d -it --name <oracle-db> -P store/oracle/database-enterprise:12.2.0.1

option -P indicates the ports are allocated by Docker. The mapped port can be discovered by executing

    docker port <oracle-db> 1521/tcp -> 0.0.0.0:<mapped>

Using this <mapped> and <ip-address> create tnsnames.ora in the directory pointed to by environment variable TNS_ADMIN.

ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<mapped>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address> of host)(PORT=<mapped>))
    (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))
To connect from outside the container using SQL*Plus,

    sqlplus sys/Oradoc_db1@ORCLCDB as sysdba
