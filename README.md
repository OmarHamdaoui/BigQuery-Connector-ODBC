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

	mkdir /etc/simba && tar zxvf SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz -C /etc/simba --strip-components 1

creating the driver repository and extracting the 64bit driver

	mkdir /etc/simba/driver && tar zxvf /etc/simba/SimbaODBCDriverforGoogleBigQuery64_2.4.5.1014.tar.gz -C /etc/simba/driver --strip-components 1

Moving the conf file to the driver repository

	mv /etc/simba/GoogleBigQueryODBC.did /etc/simba/driver/lib/
### 2.3 Deleting used tar
deleting the tar files (64bit) and the setup used on **2.1** & **2.2**

	rm -f /etc/simba/SimbaODBCDriverforGoogleBigQuery* SimbaODBCDriverforGoogleBigQuery_2.4.5.1014-Linux.tar.gz
### 2.4 Upload the service account key
If you are going to use the service account, you need to deposit it in the following repository, it must be here

	/etc/simba/setup/bq-service-account.json
### 2.5 Structure
The structure should look like that:

	/etc/simba/
	├── docs
	    ├── OEM ODBC Driver Installation Instructions.pdf
	    ├── release-notes.txt
	    └── Simba Google BigQuery ODBC Connector Install and Configuration Guide.pdf
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
### 2.6 Ownership usage
⚠️  **Important:** Adding ownership to a `user`.
Giving rights to `/etc/simba` repository to be used by another user change `user1` by the proper user.

	chown -R user1:user1 /etc/simba
---
## 3 Config ODBC
### 3.1 Listing ODBC default installed drivers

	odbcinst -q -s

*the output should look like that*
if you an error on your output like `odbcinst: SQLGetPrivateProfileString failed with Unable to find component name.` don't debug it, we will use our proper configuration in this section.
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
---
#### 3.3.1 changing the file: `/etc/simba/setup/odbcinst.ini`  
***original file:*** [odbcinst.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/original_file/odbcinst.ini)

***Edited file:*** [odbcinst.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/edited_file/odbcinst.ini)
>Since we will work only with 64bit, I have deleted the 32bit settings and changed the name of the driver as the name was so big and not adequate to the driver name in the other conf file.
---
#### 3.3.2 changing the file:`/etc/simba/setup/simba.googlebigqueryodbc.ini`
***original file:*** [simba.googlebigqueryodbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/original_file/simba.googlebigqueryodbc.ini)

***edited file:*** [simba.googlebigqueryodbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/edited_file/simba.googlebigqueryodbc.ini)

> I have replaced the INSTALLDIR by our ODBC simba installation repository
---
#### 3.3.3 changing the file:`/etc/simba/setup/odbc.ini` (For service Account Usage)
> This conf is for service account usage
> This a very important file, I have kept only the 64bit conf in my edited file.

***original file:*** [odbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/original_file/odbc.ini)

***edited file:*** [odbc.ini](https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/edited_file/odbc.ini)

you can use directly this file, and change what I have mentioned in the note section down below.
In this file, you need to focus on editing these parameters:
`Catalog=` : Add the Project ID (Your BigQuery Project ID)

Since you are going to use the service account, ensure or edit these parameters:
    `OAuthMechanism=0` : ensure that it is equal to 0
	`Email=` : Add the email of your service account
	`KeyFilePath=`: the path to your service account if you have changed its location.

#### 3.3.4 changing the file:`/etc/simba/setup/odbc.ini` (For User Account Usage) *- Skip if you will use the Service account.*
> This conf is for user account usage
> The user account usage is not recomanded by google, becose the dev depend on your account.

 1. Open this link: [Google BigQuery Token](https://accounts.google.com/o/oauth2/auth?scope=https://www.googleapis.com/auth/bigquery&response_type=code&redirect_uri=urn:ietf:wg:oauth:2.0:oob&client_id=977385342095.apps.googleusercontent.com&hl=en&from_login=1&as=76356ac9e8ce640b&pli=1&authuser=0)
 2. Select your Google account
 3. Allow BigQuery to access your Google Account
  <img src="https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/tmp/1.png?raw=true" width="300">
  
 4. Scroll and copy the "Authorization code"
 <img src="https://github.com/OmarHamdaoui/BigQuery-Connector-ODBC/blob/main/tmp/2.png?raw=true" width="300">
 5. Execute the script in your machine :
	    sh /etc/simba/Tools/get_refresh_token.sh "token you have copied"
	copy the result of the script without taking `refresh_token :`
 6. open the: `/etc/simba/odbc.ini`
 7. `#RefreshToken=`: uncomment this line and past your refresh_token
	`Email=` : comment this line
	`KeyFilePath=` comment this line 	
	`OAuthMechanism=1` ensure that it is equal to 1

### 3.4 Exporting the driver

Switch the wanted user, and execute this command, that will set the variables to your environment profile.

	export ODBCINI=/etc/simba/setup/odbc.ini
	export ODBCINSTINI=simba/setup/odbcinst.ini
	export SIMBAGOOGLEBIGQUERYODBCINI=/etc/simba/setup/simba.googlebigqueryodbc.ini

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
	DRIVERS............: /etc/simba/setup/odbcinst.ini
	SYSTEM DATA SOURCES: /etc/odbc.ini
	FILE DATA SOURCES..: /etc/ODBCDataSources
	USER DATA SOURCES..: /etc/simba/setup/odbc.ini
	SQLULEN Size.......: 8
	SQLLEN Size........: 8
	SQLSETPOSIROW Size.: 8

We can see that our driver points on the right configuration.
### Connecting to BigQuery
repeat the section 4.1 to get the Data source name, and connect using isql, like the command below:

	isql 'Google BigQuery 64-bit'
If everything went well the previous command will show the following message:

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
