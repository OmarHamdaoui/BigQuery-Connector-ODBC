# BigQuery Connector ODBC
## 1 Requirements:

 1.  **unixodbc (Installation included in the tutorial)**

> UnixODBC is **an open source ODBC driver manager**, (which include
> **odbcinst** & **isql**)

 2. **ODBC Driver (Installation included in the turial)**

> The driver will be downloaded from Google.

 3. **BigQuery Service account  or a user account**

> In this tutorial, I will use two methods to connect to BigQuery, first one is by using the service account that I have exported
> beforehand. The second one is by using the user account.
---
If you are going to use the service account:
 - You need to create a service account from your project in GCP IAM and attribute sufficient permission to that service account to use BigQuery. Export the service account to a file (Json Key).

If you are going to use your account (which is not the best practice by Google)
 - A configuration will be provided in the tutorial.
---
### 1.1 The scenario

 1. **odbcinst** will provide the conf to isql (setup and driver).
 2. **isql** will be used to connect to BigQuery.
 3. **isql** will use a Trusted Certificate and the 'service account'/'user-token' provided to connect to BigQuery.
> The trusted certificate is already present in the driver that we will download.*
---
### 1.2 Requirements Installation on Ubuntu & Redhat
>**Ubuntu**

	sudo apt update
	sudo apt install unixodbc
>**Redhat**

	sudo yum install unixODBC.x86_64

## 2 Installation of ODBC
### 2.1 Download the driver
Use the root user: `sudo su`

	wget https://storage.googleapis.com/simba-bq-release/odbc/SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz
### 2.2 Installing the driver
Creating a repository and extracting the setup and the drivers in it

	mkdir /etc/simba && tar zxvf SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz -C /etc/simba --strip-components 1

creating the driver repository and extracting the 64bit driver

	mkdir /etc/simba/driver && tar zxvf /etc/simba/SimbaODBCDriverforGoogleBigQuery64_2.4.5.1014.tar.gz -C /etc/simba/driver --strip-components 1

Moving the conf file to the driver repository

	mv /etc/simba/GoogleBigQueryODBC.did /etc/simba/driver/lib/
### 2.3 Deleting used tar
deleting the tar files (64bit) and the setup used on **2.1** & **2.2**

	rm -f /etc/simba/SimbaODBCDriverforGoogleBigQuery* SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz
### 2.4 Upload the service account key
If you are going to use the service account, you need to deposit it in the following repository

	/etc/simba/setup/
### 2.5 Structure
The structure should look like that:

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
	    ├── bq-service-account.json
	    ├── odbc.ini
	    ├── odbcinst.ini
	    └── simba.googlebigqueryodbc.ini
### 2.5 Ownership usage
⚠️  **Important:** Adding ownership to a `user1`:
Giving rights to `/etc/simba` repository to be used by another user change `user1` by the proper user.

	chown -R user1:user1 /etc/simba
---
## 3 Config ODBC
### 3.1 Listing ODBC default installed drivers

	odbcinst -q -d

*the output should look like that*
```diff
[PostgreSQL ANSI]
[MySQL]
```
### 3.2 Listing ODBC default drivers configuration

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

### 3.3 Configuring our driver and connectors
List of files to edit according to our BigQuery connection.

    /etc/simba/odbcinst.ini
    /etc/simba/simba.googlebigqueryodbc.ini
    /etc/simba/odbc.ini

#### 3.3.1 changing the file: `/etc/simba/odbcinst.ini`  
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
#### 3.3.2 changing the file:`/etc/simba/simba.googlebigqueryodbc.ini`
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
#### 3.3.3 changing the file:`/etc/simba/simba.googlebigqueryodbc.ini`
***original file***

```diff
# To use this INI file, replace <INSTALLDIR> with the 
# directory the tarball was extracted to.

[ODBC]
Trace=no

[ODBC Data Sources]
Google BigQuery 32-bit=Simba ODBC Driver for Google BigQuery 32-bit
Google BigQuery 64-bit=Simba ODBC Driver for Google BigQuery 64-bit

[Google BigQuery 32-bit]

# Description: DSN Description.
# This key is not necessary and is only to give a description of the data source.
Description=Simba ODBC Driver for Google BigQuery (32-bit) DSN

# Driver: The location where the ODBC driver is installed to.
Driver=<INSTALLDIR>/lib/libgooglebigqueryodbc_sb32.so

# These values can be set here, or on the connection string.
# Catalog: The catalog to connect to. This is a required setting.
#Catalog=

# SQLDialect: The SQL Dialect to use.  There are two SQL dialects:
# 0 = BigQuery Legacy SQL
# 1 = BigQuery Standard SQL (SQL 11)
SQLDialect=1

# OAuth Mechanism: The OAuth mechanism to use.  There are two choices:
# 0 = Service Authentication
# 1 = User Authentication
# 
# This is a required setting.
OAuthMechanism=1

# RefreshToken: The Refresh Token used. This can be generated from the Windows connection dialog.
# It can also be generated by executing the following steps:
# 1. Get an Authentication by logging into Google from the following URL: 
# https://accounts.google.com/o/oauth2/auth?scope=https://www.googleapis.com/auth/bigquery&response_type=code&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_id=977385342095.apps.googleusercontent.com&hl=en&from_login=1&as=76356ac9e8ce640b&pli=1&authuser=0
# 2. Run the get_refresh_token.sh shell script and pass in the Authentication Token received in step 1.
# 3. Copy the Refresh Token (the text on the right-side of the colon, without the trailing or leading spaces) from the output of the script.
# This is a required setting.
#RefreshToken=

# Email: For Service Authentication, this is a required setting. It is your GENERATED service account email (not a typical Gmail account).  
# It is unique and associated with at least one public/private key pair.
# Email=

# KeyFile Path: For Service Authentication, this is a required setting.  This is the path to the stored keyfile (.p12).
# KeyFilePath=

# Used to specify the full path of the PEM formatted file containing trusted SSL CA certificates.
# If an empty string is passed in for the configuration, the driver expects the trusted SSL CA
# certificates can be found in the file named cacerts.pem located in the same directory as the
# driver's shared library.
#TrustedCerts=

# AllowLargeResults: When set to 1, the driver allows for result sets in responses to be larger than 128 MB.
AllowLargeResults=0
 
# LargeResultsDataSetId: DatasetId to store temporary tables created.  This is a required setting if AllowLargeResults is set to 1.
LargeResultsDataSetId=_bqodbc_temp_tables
 
# LargeResultsTempTableExpirationTime: Time in milliseconds before the temporary tables created expire.  This is a required setting if AllowLargeResults is set to 1.
LargeResultsTempTableExpirationTime=3600000

[Google BigQuery 64-bit]

# Description: DSN Description.
# This key is not necessary and is only to give a description of the data source.
Description=Simba ODBC Driver for Google BigQuery (64-bit) DSN

# Driver: The location where the ODBC driver is installed to.
Driver=<INSTALLDIR>/lib/libgooglebigqueryodbc_sb64.so

# These values can be set here, or on the connection string.
# Catalog: The catalog to connect to. This is a required setting.
#Catalog=

# SQLDialect: The SQL Dialect to use.  There are two SQL dialects:
# 0 = BigQuery Legacy SQL
# 1 = BigQuery Standard SQL (SQL 11)
SQLDialect=1

# OAuth Mechanism: The OAuth mechanism to use.  There are two choices:
# 0 = Service Authentication
# 1 = User Authentication
# 
# This is a required setting.
OAuthMechanism=1

# RefreshToken: The Refresh Token used. This can be generated from the Windows connection dialog.
# It can also be generated by executing the following steps:
# 1. Get an Authentication by logging into Google from the following URL: 
# https://accounts.google.com/o/oauth2/auth?scope=https://www.googleapis.com/auth/bigquery&response_type=code&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_id=977385342095.apps.googleusercontent.com&hl=en&from_login=1&as=76356ac9e8ce640b&pli=1&authuser=0
# 2. Run the get_refresh_token.sh shell script and pass in the Authentication Token received in step 1.
# 3. Copy the Refresh Token (the text on the right-side of the colon, without the trailing or leading spaces) from the output of the script.
# This is a required setting.
#RefreshToken=

# Email: For Service Authentication, this is a required setting. It is your GENERATED service account email (not a typical Gmail account).  
# It is unique and associated with at least one public/private key pair.
# Email=

# KeyFile Path: For Service Authentication, this is a required setting.  This is the path to the stored keyfile (.p12).
# KeyFilePath=

# Used to specify the full path of the PEM formatted file containing trusted SSL CA certificates.
# If an empty string is passed in for the configuration, the driver expects the trusted SSL CA
# certificates can be found in the file named cacerts.pem located in the same directory as the
# driver's shared library.
#TrustedCerts=

# AllowLargeResults: When set to 1, the driver allows for result sets in responses to be larger than 128 MB.
AllowLargeResults=0
 
# LargeResultsDataSetId: DatasetId to store temporary tables created.  This is a required setting if AllowLargeResults is set to 1.
LargeResultsDataSetId=_bqodbc_temp_tables
 
# LargeResultsTempTableExpirationTime: Time in milliseconds before the temporary tables created expire.  This is a required setting if AllowLargeResults is set to 1.
LargeResultsTempTableExpirationTime=3600000

# EnableHTAPI: This set to 1 by default.
#EnableHTAPI=1

# HTAPI_MinResultsSize: An integer representing the minimum required number of rows in the results of
#   a query to enable the High-Throughput API. This means that when the results of a query are
#   gathered, they must have at least this many rows or the High-Throughput API will not be used to
#   retrieve them.
#HTAPI_MinResultsSize=1000

# HTAPI_MinActivationRatio: An integer representing the minimum required ratio of rows to block size
#   to enable the High-Throughput API. This means that when the results from a query are gathered,
#   if there are not at least N times as many rows in the results as there are in the first page of
#   results, then the High-Throughput API will not be used to retrieve them, where N is the ratio
#   given. Result pages have at most as many rows as are specified by the RowsFetchedPerBlock
#   connection property.
#HTAPI_MinActivationRatio=3
```
