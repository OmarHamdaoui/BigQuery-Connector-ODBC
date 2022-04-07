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
> The trusted certificate is already present in the driver that we will download.
---
### 1.2 Requirements Installation on Ubuntu & Redhat
>**Ubuntu**

	sudo apt update
	sudo apt install unixodbc
>**Redhat**

	sudo yum install unixODBC.x86_64

## 2 Installation of ODBC
### 2.1 Download the driver
Switch to root user: `sudo su`

	wget https://storage.googleapis.com/simba-bq-release/odbc/SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz
### 2.2 Installing the driver
Creating a repository and extracting the setup and the drivers in it

	mkdir /opt/simba && tar zxvf SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz -C /opt/simba --strip-components 1

creating the driver repository and extracting the 64bit driver

	mkdir /opt/simba/driver && tar zxvf /opt/simba/SimbaODBCDriverforGoogleBigQuery64_2.4.5.1014.tar.gz -C /opt/simba/driver --strip-components 1

Moving the conf file to the driver repository

	mv /opt/simba/GoogleBigQueryODBC.did /opt/simba/driver/lib/
### 2.3 Deleting used tar
deleting the tar files (64bit) and the setup used on **2.1** & **2.2**

	rm -f /opt/simba/SimbaODBCDriverforGoogleBigQuery* SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz
### 2.4 Creating Log repo

    mkdir /opt/simba/logs
    
### 2.5 Moving Error Message Repo

    mv /opt/simba/driver/ErrorMessages /opt/simba/

### 2.6 Upload the service account key
If you are going to use the service account, you need to deposit it in the following repository

	/opt/simba/setup/bq-service-account.json
### 2.7 Structure
The structure should look like that:

    simba/
    ├── docs
    │   ├── OEM ODBC Driver Installation Instructions.pdf
    │   ├── release-notes.txt
    │   └── Simba Google BigQuery ODBC Connector Install and Configuration Guide.pdf
    ├── driver
    │   ├── lib
    │   │   ├── cacerts.pem
    │   │   ├── EULA.txt
    │   │   ├── GoogleBigQueryODBC.did
    │   │   └── libgooglebigqueryodbc_sb64.so
    │   ├── third-party-licenses.txt
    │   └── Tools
    │       └── get_refresh_token.sh
    ├── ErrorMessages
    │   └── en-US
    │       ├── DSCURLHTTPClientMessages.xml
    │       ├── DSMessages.xml
    │       ├── ODBCMessages.xml
    │       ├── SimbaBigQueryODBCMessages.xml
    │       └── SQLEngineMessages.xml
    ├── logs
    └── setup
        ├── bq-service-account.json
        ├── odbc.ini
        ├── odbcinst.ini
        └── simba.googlebigqueryodbc.ini


### 2.8 Ownership usage
⚠️  **Important:** Adding ownership to a `user`.
Giving rights to `/opt/simba` repository to be used by another user change `user1` by the proper user.

	chown -R user1:user1 /opt/simba
---
## 3 Config ODBC
### 3.1 Listing ODBC default installed drivers

	odbcinst -q -s

*The output should look like that*
```diff
[PostgreSQL ANSI]
[MySQL]
```
If you have an error on your output like `odbcinst: SQLGetPrivateProfileString failed with Unable to find component name.` don't debug it, we will use our proper configuration in this section.
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

    /opt/simba/odbcinst.ini
    /opt/simba/simba.googlebigqueryodbc.ini
    /opt/simba/odbc.ini
---
#### 3.3.1 changing the file: `/opt/simba/setup/odbcinst.ini`  
***original file:*** [odbcinst.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/original_file/odbcinst.ini)

***Edited file:*** [odbcinst.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/edited_file/odbcinst.ini)
>Since we will work only with 64bit, I have deleted the 32bit settings.
---
#### 3.3.2 changing the file:`/opt/simba/setup/simba.googlebigqueryodbc.ini`
***original file:*** [simba.googlebigqueryodbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/original_file/simba.googlebigqueryodbc.ini)

***edited file:*** [simba.googlebigqueryodbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/edited_file/simba.googlebigqueryodbc.ini)

> I have replaced the INSTALLDIR by our ODBC simba installation repository
---
#### 3.3.3 changing the file:`/opt/simba/setup/odbc.ini` (For service Account Usage)
> This conf is for service account usage
> This is a very important file, I have kept only the 64bit conf in my edited file.

***original file:*** [odbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/original_file/odbc.ini)

***edited file:*** [odbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/edited_file/odbc.ini)

you can use directly this edited file, and change what I have mentioned in the note section down below.
In this file, you need to focus on editing these parameters:

 - `Catalog=` : Add the Project ID (Your BigQuery Project ID)
 - `OAuthMechanism=0` : ensure that it is equal to 0
 - `Email=` : Add the email of your service account
 - `KeyFilePath=`: the path to your service account if you have changed its location.
 - `TrustedCerts=`: Path to your Trusted cert which is present in (/opt/simba/driver/lib/cacerts.pem)

#### 3.3.4 changing the file:`/opt/simba/setup/odbc.ini` (For User Account Usage) *- Skip if you have used the Service account in 3.3.3*
> This conf is for user account usage
> The user account usage is not recomanded by google, becose the dev depend on your account.

 1. Open this link: [Google BigQuery Token](https://accounts.google.com/o/oauth2/auth?scope=https://www.googleapis.com/auth/bigquery&response_type=code&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_id=977385342095.apps.googleusercontent.com&hl=en&from_login=1&as=76356ac9e8ce640b&pli=1&authuser=0)
 2. Select your Google account 4. Allow BigQuery to access your Google Account
   <img src="https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/tmp/1.png?raw=true" width="300">
  
 3. Scroll and copy the "Authorization code"
 <img src="https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/tmp/2.png?raw=true" width="300">
 
 4. Execute the script in your machine:
 After executing the command below, copy the result of the script without taking `refresh_token :`
 
    sh /opt/simba/driver/Tools/get_refresh_token.sh "token you have copied"
    
 5. Open the file: `/opt/simba/odbc.ini` 
    Since you are going to use a user account, ensure or edit these parameters:
    
	 - `#RefreshToken=`: uncomment this line and past your refresh_token
	 - `Email=` : comment this line
	 - `KeyFilePath=` comment this line 	
	 - `OAuthMechanism=1` ensure that it is equal to 1
	 - `TrustedCerts=`: Path to your Trusted cert which is present in (/opt/simba/driver/lib/cacerts.pem)

### 3.4 Exporting the driver

Switch to the wanted user

    su - user1
 
Execute these commands, that will set the variables to your environment profile.

    export ODBCSYSINI=/opt/simba/setup
    export ODBCINI=/opt/simba/setup/odbc.ini
    export SIMBAGOOGLEBIGQUERYODBCINI=/opt/simba/setup/simba.googlebigqueryodbc.ini

## 4 Testing the driver
### 4.1 DSN Check
Checking if the DSN (Data Source Name) has been installed successfully:

	odbcinst -q -s
the result should be like that

	[ODBC]
	[Google BigQuery 64-bit]
	
### 4.2 Driver Check

	odbcinst -j

the result should be like that

    unixODBC 2.3.6
    DRIVERS............: /opt/simba/setup/odbcinst.ini
    SYSTEM DATA SOURCES: /opt/simba/setup/odbc.ini
    FILE DATA SOURCES..: /opt/simba/setup/ODBCDataSources
    USER DATA SOURCES..: /opt/simba/setup/odbc.ini
    SQLULEN Size.......: 8
    SQLLEN Size........: 8
    SQLSETPOSIROW Size.: 8

We can see that our driver points on the right configuration.
### Connecting to BigQuery
repeat the section 4.1 to get the Data source name, and connect using isql, like the command below:

	isql 'Google BigQuery 64-bit'
If everything went well, the previous command will show the following message:

	+---------------------------------------+
	| Connected!                            |
	|                                       |
	| sql-statement                         |
	| help [tablename]                      |
	| quit                                  |
	|                                       |
	+---------------------------------------+
	SQL>
Now we can test a query:

	SQL> select name from element.fruits limit 402;

Testing several Queries in script SQL

> put all your SQL queries in a file (query.sql), and execute this batch

    cat query.sql | isql "Google BigQuery 64-bit" -b -v
