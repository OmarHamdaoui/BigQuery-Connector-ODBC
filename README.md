# Download and Install BigQuery Connector ODBC
### 1 Requirements:
>unixodbc

 - The installation is included in this tutorial.
 - UnixODBC is **an open source ODBC driver manager**, (which include **odbcinst** & **isql**)

>ODBC Driver
 -  It will be downloaded and installed in this tutorial.

>BigQuery Service account
or
>a user account
---
If you are going to use the service account:
 - You need to create a service account from your project in GCP IAM and attribute sufficient permission to that service account to use BigQuery. Export the service account to a file (Json Key).

If you are going to use your account (which is not the best practice by Google)

 - A configuration will be provided in the tutorial.
---
### The scenario

 1. **odbcinst** will provide the conf to isql (setup and driver).
 2. **isql** will be used to connect to BQ.
 3. **isql** will use a Trusted Certificate and the 'service account'/'user-token' provided to connect to BigQuery.
> The trusted certificate is already present in the driver that we will download.*
---
### Requirements Installation on Ubuntu & Redhat
>**Ubuntu**

	sudo apt update
	sudo apt install unixodbc
>**Redhat**

	sudo yum install unixODBC.x86_64

### 2 Installation of ODBC
#### 2.1 Download the driver
Use the root user `sudo su`

	wget https://storage.googleapis.com/simba-bq-release/odbc/SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz
#### 2.2 Installing the driver
Creating a repository and extracting the setup and the drivers in it

	mkdir /etc/simba && tar zxvf SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz -C /etc/simba --strip-components 1

creating the driver repository and extracting the 64bit driver

	mkdir /etc/simba/driver && tar zxvf /etc/simba/SimbaODBCDriverforGoogleBigQuery64_2.4.5.1014.tar.gz -C /etc/simba/driver --strip-components 1

Moving the conf file to the driver repository

	mv /etc/simba/GoogleBigQueryODBC.did /etc/simba/driver/lib/
#### 2.3 Deleting used tar
deleting the tar files (64bit) and the setup used on **2.1** & **2.2**

	rm -f /etc/simba/SimbaODBCDriverforGoogleBigQuery* SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz

The structure should look like:

	/etc/simba/
	├── driver
	│   ├── ErrorMessages
	│   │   └── en-US
	│   │       ├── DSCURLHTTPClientMessages.xml
	│   │       ├── DSMessages.xml
	│   │       ├── ODBCMessages.xml
	│   │       ├── SimbaBigQueryODBCMessages.xml
	│   │       └── SQLEngineMessages.xml
	│   ├── lib
	│   │   ├── cacerts.pem
	│   │   ├── EULA.txt
	│   │   ├── GoogleBigQueryODBC.did
	│   │   └── libgooglebigqueryodbc_sb64.so
	│   ├── third-party-licenses.txt
	│   └── Tools
	│       └── get_refresh_token.sh
	└── setup
	    ├── bq.json
	    ├── odbc.ini
	    ├── odbcinst.ini
	    └── simba.googlebigqueryodbc.ini

⚠️  **Important:** Adding ownership to a `user1`:
Giving rights to `/etc/simba` repository to be used by another user change `user1` by the proper user.

	chown -R user1:user1 /etc/simba
### 3 Config ODBC
**3.1 Listing ODBC default installed drivers**

	odbcinst -q -d

*the output should look like that*
```diff
[PostgreSQL ANSI]
[MySQL]
```
**3.2 Listing ODBC default drivers configuration**

	odbcinst -j

*the output will be similar to this*
```diff
unixODBC 2.3.1
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
FILE DATA SOURCES..: /etc/ODBCDataSources
USER DATA SOURCES..: /home/user1/.odbc.ini
SQLULEN Size.......: 8
SQLLEN Size........: 8
SQLSETPOSIROW Size.: 8
```

**3.3 Configuring our driver and connectors**
3 files will be changed according to our BigQuery connection.
**3.3.1 changing the file:** `/etc/simba/odbcinst.ini`  
***original file***
```diff
# To use this INI file, replace <INSTALLDIR> with the
# directory the tarball was extracted to.

[ODBC Drivers]
Simba ODBC Driver for Google BigQuery 32-bit=Installed
Simba ODBC Driver for Google BigQuery 64-bit=Installed

[Simba ODBC Driver for Google BigQuery 32-bit]
Description=Simba ODBC Driver for Google BigQuery(32-bit)
Driver=<INSTALLDIR>/lib/libgooglebigqueryodbc_sb32.so

[Simba ODBC Driver for Google BigQuery 64-bit]
Description=Simba ODBC Driver for Google BigQuery(64-bit)
Driver=<INSTALLDIR>/lib/libgooglebigqueryodbc_sb64.so

## The option below is for using unixODBC when compiled with -DSQL_WCHART_CONVERT.
## Execute 'odbc_config --cflags' to determine if you need to uncomment it.
# IconvEncoding=UCS-4LE
```
***Edited file.***
>Since we will work only with 64bit, I have deleted the 32bit settings and changed the name of the driver as the name was so big and not adequate to the driver name in the other conf file.
```diff
[ODBC Drivers]
Google BigQuery 64-bit=Installed

[Google BigQuery 64-bit]
Description=Simba ODBC Driver for Google BigQuery(64-bit)
Driver=/etc/simba/driver/lib/libgooglebigqueryodbc_sb64.so
```
**3.3.2 changing the file:** `/etc/simba/simba.googlebigqueryodbc.ini`
***original file***
```diff
# To use this INI file, replace <INSTALLDIR> with the
# directory the tarball was extracted to.

[Driver]
DriverManagerEncoding=UTF-32
ErrorMessagesPath=<INSTALLDIR>/ErrorMessages
LogLevel=0
LogPath=

## - Note that the path to your ODBC Driver Manager must be specified in LD_LIBRARY_PATH.
```

***edited file***
> I have just replaced the INSTALLDIR by our ODBC simba installation repository
```diff
# To use this INI file, replace <INSTALLDIR> with the
# directory the tarball was extracted to.

[Driver]
DriverManagerEncoding=UTF-32
ErrorMessagesPath=/etc/simba/ErrorMessages
LogLevel=0
LogPath=

## - Note that the path to your ODBC Driver Manager must be specified in LD_LIBRARY_PATH.
```
