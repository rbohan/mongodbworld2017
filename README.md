# MongoDB World 2017 BI Connector Demo

## Overview

This repo contains the vagrant files for the VMs used in the '[Jumpstart: BI Connector & Tableau](https://explore.mongodb.com/vidyard-all-players/mongodb-world-presentations-crystal-b-ronan-bohan-vaidy-krishnan-6-20-2017)' presentation from MongoDB World 2017.

It contains two separate VMs:

* `worldvm1` sets up a BI Connector instance with *no auth enabled*
* `worldvm2` sets up a BI Connector instance *with auth enabled*

## VM Info

Each VM is configured as follows:

* 1 CPU, 4GB RAM
* Ubuntu/trusty64

The Vagrant setup scripts:

* Download and install the latest MongoDB 3.4 community server (bound to `0.0.0.0`)
* Download and install version 2.1.0 of the MongoDB BI Connector (listening on `0.0.0.0:3307`). Note: The process is not started automatically.

### `worldvm1` VM Info

* Runs on private network `192.168.15.100`
* Sets up and runs a simple MongoDB database containing owner and vehicle data (auth not enabled)
* Creates a script to start the BI Connector `startmongosqld.sh`
* Creates two scripts to create the DRDL output files `genowners.sh` and `gencars.sh`

### `worldvm2` VM Info

* Runs on private network `192.168.16.100`
* Sets up and runs a secured MongoDB database containing airline data and a TOPSECRET database
* Two users are created:
 * `root/root` with the `root` role
 * `viewer/viewer` with a customer `viewer` role (which can only see the airline data)
* Creates two scripts to start the BI Connector:
 * `startmongosqld1.sh` which is auth enabled but does not contain certificate information (this is for demonstration purposes and is expected to fail)
 * `startmongosqld2.sh` also auth enabled but will the full set of command line parameters (this is expected to succeed, assuming the certificates are in place and valid)

 Note: Starting the BI Connector with auth requires some certificate files, speficially PEM key file for the mongod server and (optionally) a certificate authority file.
 
## Creating certificate files

Detailed instructions for creating the certificate files is beyond the scope of this article. However, you can generate some self-signed certificates using the following script:

```
#! /bin/bash

export HOSTNAME=`hostname -f`


openssl genrsa -out private.key 2048

openssl req -x509 -nodes -new -key private.key -days 1000 \
-out ca.crt -subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=myCA" 

openssl req -new -nodes -newkey rsa:2048 \
-keyout mongod.key -out mongod.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=$HOSTNAME"

openssl x509 -CA ca.crt -CAkey private.key -CAcreateserial \
-req -days 1000 -in mongod.csr -out mongod.crt

cat mongod.key mongod.crt > mongod.pem

openssl req -new -nodes -newkey rsa:2048 \
-keyout mongosqld-client.key -out mongosqld-client.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=mongosqld-client"

openssl x509 -CA ca.crt -CAkey private.key -CAserial ca.srl -req -days 1000 -in mongosqld-client.csr -out mongosqld-client.crt

cat mongosqld-client.key mongosqld-client.crt > mongosqld-client.pem

openssl req -new -nodes -newkey rsa:2048 \
-keyout mongosqld-server.key \
-out mongosqld-server.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=$HOSTNAME"

openssl x509 -CA ca.crt -CAkey private.key -CAserial ca.srl -req -days 1000 -in mongosqld-server.csr -out mongosqld-server.crt

cat mongosqld-server.key mongosqld-server.crt > mongosqld-server.pem

openssl req -new -nodes \
-keyout mysql.key \
-out mysql.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=mysql"

openssl x509 -CA ca.crt -CAkey private.key -CAserial ca.srl -req -days 1000 -in mysql.csr -out mysql.crt

openssl rsa -in mysql.key -out mysql.key
cat mysql.key mysql.crt > mysql.pem

openssl verify -CAfile ca.crt mongod.crt
openssl verify -CAfile ca.crt mongosqld-client.crt
openssl verify -CAfile ca.crt mongosqld-server.crt
openssl verify -CAfile ca.crt mysql.crt
```

The output files should be copied into a new `certs` subfolder, i.e., `world2\certs`

## Creating a Tableau Datasource Connection file

To ensure Tableau connects to the BI Connector with SSL create a TDC file similar to the following (update the values for `SSLKEY`, `SSLCERT` and `SSLCA` using the full path to the respective file): 

```
<?xml version='1.0' encoding='utf-8' ?>
<connection-customization class='mongodb' version='7.7' enabled='true'>
    <vendor name='mongodb' />
    <driver name='mongodb' />
    <customizations>
      <customization name='odbc-connect-string-extras' value='SSLKEY={/.../world2/certs/mysql.key};SSLCERT={/.../world2/certs/mysql.crt};SSLCA={/.../world2/certs/ca.crt};ENABLE_CLEARTEXT_PLUGIN=1;SSL_ENFORCE=1' />
      </customizations>
</connection-customization>
```

This file should be saved with a `.tdc` extensions in the following folder (for Tableau Desktop versions):

* macOS: `$user/Documents/My Tableau Repository/Datasources`
* Windows: `Documents\My Tableau Repository\Datasources`

# Running the Demos

## `worldvm1`

Switch to the `worldvm1` directory and run `vagrant up`

Once the VM is running, ssh into the instance `vagrant ssh`

To start the BI Connector, launch the startup script `startmongosqld.sh`

Once the BI Connector script is running you should be able to connect to it from your BI tool of choice, e.g. Tableau, by connecting to Server 192.168.15.100 on port 3307. No user details are required.

Note that only the `people` table is visible the `owners` database. You can run the `genpeople.sh` script to regenerate this table at any time if you modify the schema. You can also run the `gencars.sh` script to generate the `cars` table, as outlined in the presentation. Note: You will need to restart the BI Connector process if you create any new, or update any existing, DRDL schema files.

## `worldvm2`

Switch to the `worldvm2` directory and run `vagrant up`

Once the VM is running, ssh into the instance `vagrant ssh`

To start the BI Connector:

* launch the startup script `startmongosqld1.sh`. This should fail with an error message complaining about the lack of the `--sslPEMKeyFile` parameter.
* launch the startup script `startmongosqld2.sh`. This is expected to succeed. You can verify the process has started with auth enabled by checking for the presence of the `--auth true` arguement in the console output.

Once the BI Connector script is running you should be able to connect to it from your BI tool of choice, e.g. Tableau, by connecting to Server 192.168.16.100 on port 3307.

* If you connect with no (or invalid) user details you will get an error dialog
* If you connect with the `root/root` user you can see the `flights` database, containing two tables: `airlines` and `TOPSECRET`
* If you connect with the `viewer/viewer` user you can only see the `airlines` table in the `flights` database

