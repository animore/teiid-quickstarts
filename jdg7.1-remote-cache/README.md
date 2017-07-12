---
Level: Intermediate
Technologies: Teiid, Infinispan, Hot Rod, Remote Query
Target Product: DV
Product Versions: DV 6.4+,  JDG 7.1+
Source: https://github.com/teiid/teiid-quickstarts
---

JDG Remote-Cache Quickstart using JDG Hot Rod that supports Google Protocol Buffers for Serialization
================================

# What is it?

This quickstart demonstrates how Teiid can connect to a remote Infinispan cache and perform queries and/or update data in the cache using the Hot Rod client protocol.

# Quick Start requirements

-  If you have not done so, please review the System Requirements [../README.md](../README.md) 
-  For Red Hat product related kits, will need to access Red Hat's Customer Portal at https://access.redhat.com/jbossnetwork/restricted/listSoftware.html
=======

# 1. JDG setup

* Install the JDG server

* Start JDG Server

Assumptions:
- It is assumed that you will be running both servers on the same box, therefore, start the JDG server by using the port offset argument:  -Djboss.socket.binding.port-offset=
- That majority of usecases will be using the cache in a distributed fashion, so the clustered configuration will be used


[source,xml]
.*Example*
----
./standalone.sh -c clustered.xml -Djboss.socket.binding.port-offset=100
----

* The following shows the command line to start the JDG server with the port offset:

        For Linux:   $JDG_HOME/bin/standalone.sh -c clustered.xml -Djboss.socket.binding.port-offset=100
        For Windows: %JDG_HOME%\bin\standalone.bat -c clustered.xml -Djboss.socket.binding.port-offset=100

For the purpose of this quick start, it assumes running both servers on the same machine and is expecting the JDG server to have its ports incremented. The port adjustment has been made in the _Setup JDG Data Source_ step to match the above offset.


* Configure JDG Cache

Note:  The clustered.xml configuration will be used, instead of the standalone.xml, because use cases will generally expect to be using a clustered environment.

You have 2 options for configuring the cache:
 
a) running CLI script to configure cache
b) editing standalone configuration(requires restarting the JDG server)



a) Perform the following steps to configure the cache by running the CLI script against the infinispan server

-  locate the ./src/jdg/setup-infinispan-cache.cli script in the quickstart
-  cd to the ${JDG_HOME}/bin directory
-  execute the setup-jdg-cache.cli script by running the following:

	./cli.sh  --file={path.to.cli.script}/setup-jdg-cache.cli

Since the server is running with the offset, use the -c option:

	 ./cli.sh -c 127.0.0.1:10099 --file=./setup-jdg-cache.cli
	 

Note the name of the cache - _personCache_ , as it needs to match the teiid-ispn Option property in the source model of the vdb..


b) Edit the standalone.xml configuration at ${JDG_HOME}/standalone/configuration

Note:  if the JDG server is running, then it will need to be stopped before editing the configuration.

Copy into the "clustered" section into the configuration  within the infinispan:server subsystem.

[source,xml]
----
        <subsystem xmlns="urn:infinispan:server:core:8.3">
            <cache-container name="clustered" default-cache="default" statistics="true">
                .... 
                <distributed-cache-configuration name="templateTeiidCache" start="EAGER"/>
                <distributed-cache name="personCache" configuration="templateTeiidCache"/>
                ....
            </cache-container>
        </subsystem>
----

-  start the JDG server


# 2. Setup Teiid Server

* Start the server

To start the server, open a command line and navigate to the "bin" directory under the root directory of the Teiid server and run:

[source,xml]
----
./standalone.sh //For Linux
standalone.bat //for Windows
----

If Teiid isn't configured in the default configuration, append the following arguments to the command to specify the configuration `-c {configuration.file}`

[source,xml]
.*Example*
----
./standalone.sh -c standalone-teiid.xml
----

* Install the infinispan-hotrod translator

----
cd $\{JBOSS_HOME}/bin

./jboss-cli.sh --connect --file=../docs/teiid/datasources/infinispan-hotrod-7.1/add-infinispan-hotrod-translator.cli
----


* Setup the JDG Cache to be accessed as a data source

The quickstart has a CLI script that can be used to configure the resource adapter. 

[source]
.*Setup JDG Data Source*
----
cd $\{JDG_HOME}/bin

./jboss-cli.sh --connect --file=${PATH}/teiid-quickstarts/jdg7.1-remote-cache/src/scripts/create-infinispan-hotrod-protobuf-ds.cli
----
 

* Deploy the VDB

Deploy for reading/writing to a remote cache, by copying files jdg-person-cache-vdb.xml and jdg-person-cache-vdb.xml.dodeploy to {JBOSS_HOME}/standalone/deployments


# 3. Query Demonstrations

NOTE: before querying, if not already, will need to add user/pasword.

1. Change your working directory to "${quickstart.install.dir}/simpleclient"
2. Use the simpleclient example to run the following queries:

Example: mvn exec:java -Dvdb="PersonVDB" -Dsql="Insert into Person (id, name, salary) Values (100, 'TestPerson', 25000.00)" -Dusername="teiidUser" -Dpassword="pwd"

or

Use a sql tool, like SQuirreL, to connect and issue following example query:

Queries for reading/writing to a remote cache via VDB People

URL connect: jdbc:teiid:PersonVDB@mm://{host}:31000 

[source,sql]
.*Example Query SQL*
----
select name, salary, id from Person 
Insert into Person (id, name, salary) Values (100, 'TestPerson', 35000.00);
Insert into Person (id, name, salary) Values (200, 'TestPerson2', 44123.12);

select name, salary, id from Person where id = 100
Update Person set name='testPerson 100' where id = 100 then - select name, salary, id from Person 


delete from Person where id = 100
select name, salary, id from Person
----