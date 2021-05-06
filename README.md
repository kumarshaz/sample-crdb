# sample-crdb

CockroachDB grabbed my attention a couple of years ago when I leaned how it introduced distributed aspects in ACID compliant database with SQL interface. Before that you had to choose between one!
Then I started working on MuleSoft and was curious how can MuleSoft connect to a CockroachDB instance as no off the shelf CRDB connector is available.
I came across several paid connectors on internet and was curious if there can be a lot simple and open source solution as CRDB interface is in fact PosgreSQL and MuleSoft provides a generic Database connector.

## Setup CRDB instance
First thing first is you need to understand the basics of CRDB to create a local cluster with a demo schema and this course helped me clear the clouds!

Start a local [single node](https://www.cockroachlabs.com/docs/v20.2/cockroach-start-single-node) or a [multi node](https://www.cockroachlabs.com/docs/v20.2/start-a-local-cluster.html) cluster of CRDB. I decided to stick to a [single node demo instance](https://www.cockroachlabs.com/docs/v20.2/learn-cockroachdb-sql.html#show-tables)  to keep it simple.

## Create a simple Mule project
