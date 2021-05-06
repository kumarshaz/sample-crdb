# sample-crdb

CockroachDB grabbed my attention a couple of years ago when I leaned how it introduced distributed aspects in ACID compliant database with SQL interface. Before that you had to choose between one!
Then I started working on MuleSoft and was curious how can MuleSoft connect to a CockroachDB instance as no off the shelf CRDB connector is available.
I came across several paid connectors on internet and was curious if there can be a lot simple and open source solution as CRDB interface is in fact PosgreSQL and MuleSoft provides a generic Database connector.

## Setup CRDB instance
First thing first is you need to understand the basics of CRDB to create a local cluster with a demo schema and this course helped me clear the clouds!

Start a local [single node](https://www.cockroachlabs.com/docs/v20.2/cockroach-start-single-node) or a [multi node](https://www.cockroachlabs.com/docs/v20.2/start-a-local-cluster.html) cluster of CRDB. I decided to stick to a [single node demo instance](https://www.cockroachlabs.com/docs/v20.2/learn-cockroachdb-sql.html#show-tables)  to keep it simple. Here is the console output:

```console
C:\Users\kumar>cockroach demo
#
# Welcome to the CockroachDB demo database!
#
# You are connected to a temporary, in-memory CockroachDB cluster of 1 node.
#
# This demo session will attempt to enable enterprise features
# by acquiring a temporary license from Cockroach Labs in the background.
# To disable this behavior, set the environment variable
# COCKROACH_SKIP_ENABLING_DIAGNOSTIC_REPORTING=true.
#
# Beginning initialization of the movr dataset, please wait...
#
# The cluster has been preloaded with the "movr" dataset
# (MovR is a fictional vehicle sharing company).
#
# Reminder: your changes to data stored in the demo session will not be saved!
#
# Connection parameters:
#   (console) http://127.0.0.1:61017
#   (sql/tcp) postgres://root:admin@127.0.0.1:61019?sslmode=require
#
#
# The user "root" with password "admin" has been created. Use it to access the Web UI!
#
# Server version: CockroachDB CCL v20.2.5 (x86_64-w64-mingw32, built 2021/02/16 13:03:41, go1.13.14) (same version as client)
# Cluster ID: 7e14f61a-58b3-4f06-87ad-854eb669bf89
#
# Enter \? for a brief introduction.
#
root@127.0.0.1:61019/movr> show tables;
  schema_name |         table_name         | type  | owner | estimated_row_count
--------------+----------------------------+-------+-------+----------------------
  public      | promo_codes                | table | root  |                   0
  public      | rides                      | table | root  |                   0
  public      | user_promo_codes           | table | root  |                   0
  public      | users                      | table | root  |                   0
  public      | vehicle_location_histories | table | root  |                   0
  public      | vehicles                   | table | root  |                   0
(6 rows)
 
Time: 11ms total (execution 11ms / network 0ms)

```

## Create a simple Mule project

### Create sample mule project with http listener
Lets keep this simple as well, we will expose a sample /test endpoint which will query a row from the database and respond as JSON payload.

### Configure Generic DB connector

This is the trickiest part. MuleSoft has excellent documentation on configuring [generic database connection](https://docs.mulesoft.com/db-connector/1.9/database-connector-connection#configure-a-generic-connection) and install [JDBC drivers](https://docs.mulesoft.com/db-connector/1.9/database-connector-connection#configure-the-jdbc-driver).
We will pretend CRDB is nothing but PostgreSQL for which we can get the [JDBC driver info here](https://jdbc.postgresql.org/).

The driver dependency:


    	<dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.19.jre7</version>
        </dependency>

The plugin:

			<plugin>
				<groupId>org.mule.tools.maven</groupId>
				<artifactId>mule-maven-plugin</artifactId>
				<version>${mule.maven.plugin.version}</version>
				<extensions>true</extensions>
				<configuration>
				<sharedLibraries>
                        <sharedLibrary>
                            <groupId>org.postgresql</groupId>
                            <artifactId>postgresql</artifactId>
                        </sharedLibrary>
                    </sharedLibraries>
                </configuration>
			</plugin>


This gave me some hard time to figure out the correct connection string and other parameters in the DB config. Below is the DB connection info after I figured out everything and my test connection was successfull.

``` xml
	<db:config name="crdb_Database_Config" doc:name="Database Config" doc:id="GUID" >
		<db:generic-connection url="jdbc:postgresql://127.0.0.1:61019/movr" driverClassName="org.postgresql.Driver" user="root" password="admin"/>
	</db:config>
```

### Configure Generic DB connector

```xml

<db:select doc:name="Select" doc:id="b2b2798a-c4f8-4c6f-9e0b-d82f8b7e3b64" config-ref="crdb_Database_Config">
			<db:sql ><![CDATA[select city from vehicles where id='aaaaaaaa-aaaa-4800-8000-00000000000a']]></db:sql>
</db:select>

```