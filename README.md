# MongoDB World 2017 BI Connector Demo

## Overview

This repo contains the vagrant files for the VMs used in the '[Jumpstart: BI Connector & Tableau](https://explore.mongodb.com/vidyard-all-players/mongodb-world-presentations-crystal-b-ronan-bohan-vaidy-krishnan-6-20-2017)' presentation from MongoDB World 2017, updated to use the *MongoDB BI Connector v2.2*.

Note: You can find the instructions and setup files for a version using the *MongoDB BI Connector v2.1*, which was used in the original presentation, in [this branch](https://github.com/rbohan/mongodbworld2017/tree/bi-connector-2.1).

The repo contains two separate VMs:

* `worldvm1` sets up a MongoDB instance and a BI Connector instance with *no auth enabled*.
* `worldvm2` sets up a MongoDB instance and a BI Connector instance *with auth enabled*.

## VM Info

Each VM is configured as follows:

* 1 CPU, 4GB RAM
* Ubuntu/trusty64

The Vagrant setup scripts will:

* Download and install the latest MongoDB 3.4 community server (bound to `0.0.0.0`)
* Download and install version 2.2.0 of the MongoDB BI Connector (listening on `0.0.0.0:3307`). Note: The BI Connector process is not started automatically but scripts are provided to do so.

### `worldvm1` VM Info

* Is assigned IP Address `192.168.15.100`
* Sets up and runs a simple MongoDB database containing owner and vehicle data (auth not enabled)
* Creates a script to start the BI Connector `startmongosqld.sh`
* Creates two scripts to (re)create the DRDL output files `genowners.sh` and `gencars.sh`

### `worldvm2` VM Info

* Is assigned IP Address `192.168.16.100`
* Sets up and runs an auth-enabled MongoDB database containing airline data and a TOPSECRET database
* Two users are created:
 * `root/root` with the `root` role
 * `viewer/viewer` with a custom `viewer` role (which can only see the airline data)
* Creates a script to start the BI Connector `startmongosqld.sh`

 Note: Version 2.2 of the BI Connector uses a MySQL authentication plugin (`mongosql_auth`) to allow clients to connect without the need for additional certificates, greatly simplifying the installation and configuration process when compared to the previous version of the BI Connector (v 2.1).

## Creating a Tableau Datasource Connection file

To ensure Tableau connects to the BI Connector with SSL create a TDC file containing the following:

```
<?xml version='1.0' encoding='utf-8' ?>
<connection-customization class='mongodb' version='7.7' enabled='true'>
  <vendor name='mongodb' />
  <driver name='mongodb' />
  <customizations>
    <customization
      name='odbc-connect-string-extras'
      value='USE_MYCNF=1' />
  </customizations>
</connection-customization>
```

This file should be saved with a `.tdc` extensions in the following folder (for Tableau Desktop):

* macOS: `~/Documents/My Tableau Repository/Datasources`
* Windows: `Documents\My Tableau Repository\Datasources`

Tableau will need to be restarted if you create a new or modify an existing `.tdc` file.

Additional information regarding how to install the BI Connector for use with Tableau can be found in the [online documentation](https://docs.mongodb.com/bi-connector/master/connect/tableau/).

## Running the Demos

### `worldvm1`

Switch to the `worldvm1` directory and run `vagrant up`

Once the VM is running, ssh into the instance `vagrant ssh`

To start the BI Connector, launch the startup script `startmongosqld.sh`

Once the BI Connector script is running you should be able to connect to it from your BI tool of choice, e.g. Tableau, by connecting to Server `192.168.15.100` on port `3307`. No user details are required.

Note that only the `people` table is visible the `owners` database. You can run the `genpeople.sh` script to regenerate this table at any time if you modify the schema. You can also run the `gencars.sh` script to generate the `cars` table, as outlined in the presentation. Note: You will need to restart the BI Connector process if you create any new or update any existing DRDL schema files.

### `worldvm2`

Switch to the `worldvm2` directory and run `vagrant up`

Once the VM is running, ssh into the instance `vagrant ssh`

To start the BI Connector, launch the startup script `startmongosqld.sh`

You can verify the process has started with auth enabled by checking for `security: {enabled: true}}` in the console output.

Once the BI Connector script is running you should be able to connect to it from your BI tool of choice, e.g. Tableau, by connecting to Server `192.168.16.100` on port `3307`.

* If you connect with no (or invalid) user details you will get an error dialog.
* If you connect with the `root/root` user you can see the `flights` database, containing two tables: `airlines` and `TOPSECRET`.
* If you connect with the `viewer/viewer` user you can only see the `airlines` table in the `flights` database.

Note: If you provide invalid details in the Tableau connection dialog you will need to close the dialog and reopen it before you can successfully authenticate. This appears to be an issue with Tableau.
