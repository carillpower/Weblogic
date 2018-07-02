How to setup and install Oracle Weblogic in CentOS 7
====================================================

In this tutorial, I'll guide you on how to setup and install Oracle
Weblogic inside CentOS 7 operation system. For anyone that not already
know, Oracle Weblogic are 1 of middleware tools that widely use by
mostly large company industries to serve application that mainly use
Java EE as programming language. With its cool UI, proven features like
coherence module (for caching purpose) , database clustering (for
handling multiple database connection) , Oracle Weblogic shows quite
impressive advantages compare to other similar tools such as Apache
tomcat, JBOSS and WebSphere. As stated by Oracle itself, provides a
complete set of services for those modules and handles many details of
application behavior automatically, without requiring programming
scripting. Below are an example of where did Oracle Weblogic located for
a high level design :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/high_level.jpg)

1. Preliminary Note
-------------------

For this tutorial, I'll be be using CentOS 7.4 in 64bit version. Please
note that eventhough the configuration are made under CentOS 7, yet the
steps and modification are mainly the same when applying on RedHat or
Oracle Linux flavour. The reason I've mention this was mostly the
production installation for Oracle Weblogic will mainly use Oracle Linux
itself as operating system.

By the end of this tutorial, we will manage to bring up 2 server nodes
that will act as Weblogic Managed Server which created by a Weblogic
Admin Server. Apart from that, we will using Admin Server dashboard to
combine both Managed Server into a cluster group.

2. Installation Phase
---------------------

As Oracle Weblogic purpose was to serve high performance application
code by JAVA programming language, it's quite obvious that the
installation for the middleware server itself would require java runtime
to be build in. Therefore, for the installation prerequisite, we will
need to install JAVA package into our admin server and 2 managed nodes.
The steps are like below :-

[root@weblogic\_mgr opt]\# wget --no-cookies --no-check-certificate
--header "Cookie: gpw\_e24=http%3A%2F%2Fwww.oracle.com%2F;
oraclelicense=accept-securebackup-cookie"
"http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm"
\
 --2018-06-09 12:57:05--
http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
\
 Resolving download.oracle.com (download.oracle.com)... 23.49.16.62 \
 Connecting to download.oracle.com
(download.oracle.com)|23.49.16.62|:80... connected. \
 HTTP request sent, awaiting response... 302 Moved Temporarily \
 Location:
https://edelivery.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
[following] \
 --2018-06-09 12:57:10--
https://edelivery.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
\
 Resolving edelivery.oracle.com (edelivery.oracle.com)...
104.103.48.174, 2600:1417:58:181::2d3e, 2600:1417:58:188::2d3e \
 Connecting to edelivery.oracle.com
(edelivery.oracle.com)|104.103.48.174|:443... connected. \
 HTTP request sent, awaiting response... 302 Moved Temporarily \
 Location:
http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm?AuthParam=1528549151\_b1fd01d854bc0423600a83c36240028e
[following] \
 --2018-06-09 12:57:11--
http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm?AuthParam=1528549151\_b1fd01d854bc0423600a83c36240028e
\
 Connecting to download.oracle.com
(download.oracle.com)|23.49.16.62|:80... connected. \
 HTTP request sent, awaiting response... 200 OK \
 Length: 169983496 (162M) [application/x-redhat-package-manager] \
 Saving to: ‘jdk-8u131-linux-x64.rpm’ \
 \

100%[==============================================================================\>]
169,983,496 2.56MB/s in 64s \
 \
 2018-06-09 12:58:15 (2.54 MB/s) - ‘jdk-8u131-linux-x64.rpm’ saved
[169983496/169983496] \
 \
 [root@weblogic\_mgr opt]\# yum localinstall -y jdk-8u131-linux-x64.rpm
\

Once done, we continue to make amendment on the environment path to
create JAVA\_HOME variable inside each server node. Below are the steps
:-

[root@weblogic\_mgr opt]\# vi /root/.bash\_profile \
 export JAVA\_HOME=/usr/java/jdk1.8.0\_131 \
 PATH=\$JAVA\_HOME/bin:\$PATH:\$HOME/bin \
 export PATH \

[root@weblogic\_mgr opt]\# source /root/.bash\_profile \
 [root@weblogic\_mgr opt]\# java -version \
 java version "1.8.0\_131" \
 Java(TM) SE Runtime Environment (build 1.8.0\_131-b11) \
 Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode) \

For Oracle database installation, it's a compulsory that the
installation must be done using non-root user. That's also applies on
Oracle Weblogic installation. As for that policy, to proceed let's
create additional user to be owner for Oracle Weblogic. Below are the
steps :-

[root@weblogic\_mgr opt]\# useradd -s /bin/bash shahril \

[root@weblogic\_mgr opt]\# passwd shahril \
 Changing password for user shahril. \
 New password: \
 BAD PASSWORD: The password fails the dictionary check - it is too
simplistic/systematic \
 Retype new password: \
 passwd: all authentication tokens updated successfully. \
 \
 [root@weblogic\_mgr opt]\# su - shahril \
 [shahril@weblogic\_mgr \~]\$ pwd \
 /home/shahril \

Before we proceed further, let's configure the environment variables for
weblogic owner user for required variables. Below are the best practice
variables need to be assign :-

1.  ORACLE\_BASE :: Default Oracle installer directory location
2.  ORACLE\_HOME :: Default Oracle database directory location /
    optional if have Oracle client inside
3.  MW\_HOME :: Default Middleware installer directory location
4.  WLS\_HOME :: Default Oracle Weblogic managed server directory
    location
5.  WL\_HOME :: Default Oracle Weblogic admin server directory location
6.  DOMAIN\_BASE :: Default Oracle Weblogic global domain
7.  DOMAIN\_HOME :: Default Oracle Weblogic specific domain

[shahril@weblogic\_mgr wls]\$ vi /home/shahril/.bash\_profile \
 \
 export ORACLE\_BASE=/home/shahril/wls/oracle \
 export ORACLE\_HOME=\$ORACLE\_BASE/product/fmw12 \
 export MW\_HOME=\$ORACLE\_HOME \
 export WLS\_HOME=\$MW\_HOME/wlserver \
 export WL\_HOME=\$WLS\_HOME \
 export DOMAIN\_BASE=\$ORACLE\_BASE/config/domains \
 export DOMAIN\_HOME=\$DOMAIN\_BASE/TEST \
 \
 export JAVA\_HOME=/usr/java/jdk1.8.0\_131 \
 PATH=\$JAVA\_HOME/bin:\$PATH:\$HOME/bin \
 export PATH \
 \
 [shahril@weblogic\_mgr wls]\$ source /home/shahril/.bash\_profile \
 [shahril@weblogic\_mgr wls]\$ mkdir -p \$ORACLE\_BASE \
 [shahril@weblogic\_mgr wls]\$ mkdir -p \$DOMAIN\_BASE \
 [shahril@weblogic\_mgr wls]\$ mkdir -p \$ORACLE\_HOME \
 [shahril@weblogic\_mgr wls]\$ mkdir -p
\$ORACLE\_BASE/config/applications \
 [shahril@weblogic\_mgr wls]\$ mkdir -p /home/shahril/wls/oraInventory \
 \

Once done, let's create a file called oraInst.loc and wls.rsp . For
filename oraInst.loc, it is needed as we need to define an inventory
location during the Oracle Weblogic installation. For filename wls.rsp,
it is an optional as it act as a response file that will be use during
the installation. Yet, as moving along we will make the installation
from command line interface (CLI), the wls.rsp would be a compulsory for
us to have. Now, let's proceed with the steps as per below :-

[shahril@weblogic\_mgr wls]\$ pwd \
 /home/shahril/wls \
 [shahril@weblogic\_mgr wls]\$ vi oraInst.loc \
 \
 inventory\_loc=/home/shahril/wls/oraInventory \
 inst\_group=shahril \
 \
 [shahril@weblogic\_mgr wls]\$ vi wls.rsp \
 \
 [ENGINE] \
 Response File Version=1.0.0.0.0 \
 [GENERIC] \
 ORACLE\_HOME=/home/shahril/wls/oracle/product/fmw12 \
 INSTALL\_TYPE=WebLogic Server \
 DECLINE\_SECURITY\_UPDATES=true \
 SECURITY\_UPDATES\_VIA\_MYORACLESUPPORT=false \
 \

As exerything is on place, let's proceed on downloading the Oracle
Weblogic installer. You can go to website URL
[here](http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-main-097127.html)
and choose your favourite version of Oracle Weblogic.

In our case, we will proceed with downloading Oracle Weblogic version
12.1.3 as by far now it's the most up-to-date and stable version (base
on my current facing experience). Below are the steps :-

[shahril@weblogic\_mgr \~]\$ cd \$ORACLE\_BASE \
 [shahril@weblogic\_mgr oracle]\$ wget
http://download.oracle.com/otn/nt/middleware/12c/wls/1213/fmw\_12.1.3.0.0\_wls.jar?AuthParam=1530174357\_1de6ededa212d8bc86524a0fb78ac0df
\
 --2018-06-28 16:24:15--
http://download.oracle.com/otn/nt/middleware/12c/wls/1213/fmw\_12.1.3.0.0\_wls.jar?AuthParam=1530174357\_1de6ededa212d8bc86524a0fb78ac0df
\
 Resolving download.oracle.com (download.oracle.com)... 23.74.208.198 \
 Connecting to download.oracle.com
(download.oracle.com)|23.74.208.198|:80... connected. \
 HTTP request sent, awaiting response... 200 OK \
 Length: 923179081 (880M) [application/x-jar] \
 Saving to:
‘fmw\_12.1.3.0.0\_wls.jar?AuthParam=1530174357\_1de6ededa212d8bc86524a0fb78ac0df’
\
 \

100%[=================================================================\>]
923,179,081 1.05MB/s in 16m 4s \
 \
 2018-06-28 16:40:24 (935 KB/s) -
‘fmw\_12.1.3.0.0\_wls.jar?AuthParam=1530174357\_1de6ededa212d8bc86524a0fb78ac0df’
saved [923179081/923179081] \
 \
 [shahril@weblogic\_mgr oracle]\$ mv
fmw\_12.1.3.0.0\_wls.jar?AuthParam=1530174357\_1de6ededa212d8bc86524a0fb78ac0df
fmw\_12.1.3.0.0\_wls.jar \

Next, proceed with the installation. Steps are as per shown below :-

[shahril@weblogic\_mgr wls]\$ java -jar
/home/shahril/wls/oracle/fmw\_12.1.3.0.0\_wls.jar -silent -responseFile
/home/shahril/wls/wls.rsp -invPtrLoc /home/shahril/wls/oraInst.loc \
 Launcher log file is
/tmp/OraInstall2018-06-10\_12-44-24PM/launcher2018-06-10\_12-44-24PM.log.
\
 Extracting files....... \
 Starting Oracle Universal Installer \
 \
 Checking if CPU speed is above 300 MHz. Actual 3199.968 MHz Passed \
 Checking swap space: must be greater than 512 MB. Actual 7815164 MB
Passed \
 Checking if this platform requires a 64-bit JVM. Actual 64 Passed
(64-bit not required) \
 Checking temp space: must be greater than 300 MB. Actual 393285 MB
Passed \
 \
 \
 Preparing to launch the Oracle Universal Installer from
/tmp/OraInstall2018-06-10\_12-44-24PM \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=512m; support was removed in 8.0 \
 Log:
/tmp/OraInstall2018-06-10\_12-44-24PM/install2018-06-10\_12-44-24PM.log
\
 Copyright (c) 1996, 2014, Oracle and/or its affiliates. All rights
reserved. \
 Reading response file.. \
 Starting check : CertifiedVersions \
 /bin/cat: /proc/sys/net/core/wmem\_default: No such file or directory \
 Starting check : CheckJDKVersion \
 Expected result: 1.7.0\_15 \
 Actual Result: 1.8.0\_131 \
 Check complete. The overall result of this check is: Passed \
 CheckJDKVersion Check: Success. \
 Validations are enabled for this session. \
 Verifying data...... \
 Copying Files... \
 You can find the log of this install session at: \
 /tmp/OraInstall2018-06-10\_12-44-24PM/install2018-06-10\_12-44-24PM.log
\
 -----------20%----------40%----------60%----------80%--------100% \
 \
 The installation of Oracle Fusion Middleware 12c WebLogic Server and
Coherence 12.1.3.0.0 completed successfully. \
 Logs successfully copied to /home/shahril/wls/oraInventory/logs. \

Excellent! Now we've successfully install Oracle Weblogic into our
CentOS 7 server. Next we will proceed with the configuration phase.

3. Configuration Phase
----------------------

As now we are in configuration part, there will be 2 level of
configuration that need to be made which is :-

1.  Weblogic Configuration
2.  Domain Configuration

For a Weblogic Administration server, we need to make both of the
configuration as the main command of weblogic are under weblogic
configuration. But for every Weblogic managed server that will act act
the instance node, it just need to setup for Weblogic Configuration only
as during the initialization of domain, administration can decide which
instance node will be use for which project domain. Below are the simple
example of how weblogic domain works :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/Weblogic_Domain.jpg)

For every weblogic managed server, you can create as many instance node
you need depends on you server resource allocation because each instance
node will be point to its dedicated project domain. To point which
domain to which instance node can easily done by administration server
dashboard.

As per brief, now let's setup the configuration for weblogic and domain
configuration for admin server part. Just to simplified the tutorial
process, we will only create 1 domain called TEST. Below are the steps
:-

[shahril@weblogic\_mgr wls]\$ cd \$WL\_HOME \
 [shahril@weblogic\_mgr wlserver]\$ cd common/bin/ \
 [shahril@weblogic\_mgr bin]\$ ./commEnv.sh \
 [shahril@weblogic\_mgr bin]\$ ./wlst.sh \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=256m; support was removed in 8.0 \
 Initializing WebLogic Scripting Tool (WLST) ... \
 Jython scans all the jar files it can find at first startup. Depending
on the system, this process may take a few minutes to complete, and WLST
may not return a prompt right away. \
 \
 Welcome to WebLogic Server Administration Scripting Shell \
 \
 Type help() for help on available commands \
 \
 wls:/offline\>
readTemplate('/home/shahril/wls/oracle/product/fmw12/wlserver/common/templates/wls/wls.jar')
\
 wls:/offline/base\_domain\>cd('Servers/AdminServer') \

wls:/offline/base\_domain/Server/AdminServer\>set('ListenAddress','172.17.0.6')
\
 wls:/offline/base\_domain/Server/AdminServer\>set('ListenPort',7001)
\#\# Port that will be assign to each domain \
 \

wls:/offline/base\_domain/Server/AdminServer\>create('AdminServer','SSL')
\
 Proxy for AdminServer: Name=AdminServer, Type=SSL \
 \
 wls:/offline/base\_domain/Server/AdminServer\>cd('SSL/AdminServer') \

wls:/offline/base\_domain/Server/AdminServer/SSL/AdminServer\>set('Enabled','True')
\

wls:/offline/base\_domain/Server/AdminServer/SSL/AdminServer\>set('ListenPort',7002)
\
 \
 wls:/offline/base\_domain/Server/AdminServer/SSL/AdminServer\>cd('/') \
 wls:/offline/base\_domain\>cd('Security/base\_domain/User/weblogic') \

wls:/offline/base\_domain/Security/base\_domain/User/weblogic\>cmo.setPassword('Test1234')
\

wls:/offline/base\_domain/Security/base\_domain/User/weblogic\>setOption('OverwriteDomain','true')
\

wls:/offline/base\_domain/Security/base\_domain/User/weblogic\>writeDomain('/home/shahril/wls/oracle/config/domains/TEST')
\
 \
 wls:/offline/TEST/Security/TEST/User/weblogic\>closeTemplate() \
 wls:/offline\>exit() \
 \
 Exiting WebLogic Scripting Tool. \
 \

Great, now we've done the configuration for both, now let's start the
weblogic and TEST services on admin server. Below are the steps :-

[shahril@weblogic\_mgr bin]\$ cd \$DOMAIN\_HOME \
 [shahril@weblogic\_mgr TEST]\$ cd bin/ \
 [shahril@weblogic\_mgr bin]\$ pwd \
 /home/shahril/wls/oracle/config/domains/TEST/bin \
 \
 [shahril@weblogic\_mgr bin]\$ ./startWebLogic.sh & \
 [1] 19303 \
 [shahril@weblogic\_mgr bin]\$ . \
 . \
 JAVA Memory arguments: -Xms256m -Xmx512m -XX:CompileThreshold=8000
-XX:PermSize=128m -XX:MaxPermSize=256m \
 . \

CLASSPATH=/usr/java/jdk1.8.0\_131/lib/tools.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/weblogic\_sp.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/weblogic.jar:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/net.sf.antcontrib\_1.1.0.0\_1-0b3/lib/ant-contrib.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/modules/features/oracle.wls.common.nodemanager\_2.0.0.0.jar:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/com.oracle.cie.config-wls-online\_8.1.0.0.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/common/derby/lib/derbyclient.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/common/derby/lib/derby.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/xqrl.jar
. \

PATH=/home/shahril/wls/oracle/product/fmw12/wlserver/server/bin:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/org.apache.ant\_1.9.2/bin:/usr/java/jdk1.8.0\_131/jre/bin:/usr/java/jdk1.8.0\_131/bin:/usr/java/jdk1.8.0\_131/bin:/usr/java/jdk1.8.0\_131/bin:/usr/java/jdk1.8.0\_131/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/shahril/bin:/home/shahril/bin:/home/shahril/bin
. \

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\
 \* To start WebLogic Server, use a username and \* \
 \* password assigned to an admin-level user. For \* \
 \* server administration, use the WebLogic Server \* \
 \* console at http://hostname:port/console \* \

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\
 starting weblogic with Java version: \
 java version "1.8.0\_131" \
 Java(TM) SE Runtime Environment (build 1.8.0\_131-b11) \
 Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode) \
 Starting WLS with line: \
 /usr/java/jdk1.8.0\_131/bin/java -server -Xms256m -Xmx512m
-XX:CompileThreshold=8000 -XX:PermSize=128m -XX:MaxPermSize=256m
-Dweblogic.Name=AdminServer
-Djava.security.policy=/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/weblogic.policy
-Xverify:none
-Djava.endorsed.dirs=/usr/java/jdk1.8.0\_131/jre/lib/endorsed:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/endorsed
-da -Dwls.home=/home/shahril/wls/oracle/product/fmw12/wlserver/server
-Dweblogic.home=/home/shahril/wls/oracle/product/fmw12/wlserver/server
-Dweblogic.utils.cmm.lowertier.ServiceDisabled=true weblogic.Server \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
PermSize=128m; support was removed in 8.0 \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=256m; support was removed in 8.0 \
 Jun 10, 2018 1:11:46 PM UTC Info Security BEA-090905 Disabling the
CryptoJ JCE Provider self-integrity check for better startup
performance. To enable this check, specify
-Dweblogic.security.allowCryptoJDefaultJCEVerification=true. \
 Jun 10, 2018 1:11:46 PM UTC Info Security BEA-090906 Changing the
default Random Number Generator in RSA CryptoJ from ECDRBG128 to
FIPS186PRNG. To disable this change, specify
-Dweblogic.security.allowCryptoJDefaultPRNG=true. \
 Jun 10, 2018 1:11:47 PM UTC Info WebLogicServer BEA-000377 Starting
WebLogic Server with Java HotSpot(TM) 64-Bit Server VM Version
25.131-b11 from Oracle Corporation. \
 Jun 10, 2018 1:11:47 PM UTC Info Management BEA-141107 Version:
WebLogic Server 12.1.3.0.0 Wed May 21 18:53:34 PDT 2014 1604337 \
 Jun 10, 2018 1:11:48 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to STARTING. \
 Jun 10, 2018 1:11:48 PM UTC Info WorkManager BEA-002900 Initializing
self-tuning thread pool. \
 Jun 10, 2018 1:11:48 PM UTC Info WorkManager BEA-002942 CMM memory
level becomes 0. Setting standby thread pool size to 256. \
 Jun 10, 2018 1:11:48 PM UTC Notice Log Management BEA-170019 The server
log file
/home/shahril/wls/oracle/config/domains/TEST/servers/AdminServer/logs/AdminServer.log
is opened. All server side log events will be written to this file. \
 Jun 10, 2018 1:11:50 PM UTC Notice Security BEA-090082 Security
initializing using security realm myrealm. \
 Jun 10, 2018 1:11:51 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to STANDBY. \
 Jun 10, 2018 1:11:51 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to STARTING. \
 Jun 10, 2018 1:11:51 PM weblogic.wsee.WseeCoreMessages
logWseeServiceStarting \
 \
 INFO: The Wsee Service is starting \
 \
 Jun 10, 2018 1:11:51 PM UTC Notice Log Management BEA-170027 The server
has successfully established a connection with the Domain level
Diagnostic Service. \
 Jun 10, 2018 1:11:51 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to ADMIN. \
 Jun 10, 2018 1:11:51 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to RESUMING. \
 Jun 10, 2018 1:11:51 PM UTC Notice Security BEA-090171 Loading the
identity certificate and private key stored under the alias DemoIdentity
from the jks keystore file
/home/shahril/wls/oracle/config/domains/TEST/security/DemoIdentity.jks.
\
 Jun 10, 2018 1:11:51 PM UTC Notice Security BEA-090169 Loading trusted
certificates from the jks keystore file
/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/DemoTrust.jks.
\
 Jun 10, 2018 1:11:51 PM UTC Notice Security BEA-090169 Loading trusted
certificates from the jks keystore file
/usr/java/jdk1.8.0\_131/jre/lib/security/cacerts. \
 Jun 10, 2018 1:11:51 PM UTC Notice Server BEA-002613 Channel
"DefaultSecure" is now listening on 172.17.0.6:7002 for protocols iiops,
t3s, ldaps, https. \
 Jun 10, 2018 1:11:51 PM UTC Notice Server BEA-002613 Channel "Default"
is now listening on 172.17.0.6:7001 for protocols iiop, t3, ldap, snmp,
http. \
 Jun 10, 2018 1:11:51 PM UTC Notice WebLogicServer BEA-000331 Started
the WebLogic Server Administration Server "AdminServer" for domain
"TEST" running in development mode. \
 Jun 10, 2018 1:11:51 PM UTC Notice WebLogicServer BEA-000360 The server
started in RUNNING mode. \
 Jun 10, 2018 1:11:52 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to RUNNING. \
 \
 [shahril@weblogic\_mgr bin]\$ netstat -apn|grep -i :70 \
 (Not all processes could be identified, non-owned process info \
 will not be shown, you would have to be root to see it all.) \
 tcp 0 0 172.17.0.6:7001 0.0.0.0:\* LISTEN 19360/java \
 tcp 0 0 172.17.0.6:7002 0.0.0.0:\* LISTEN 19360/java \
 \

Excellent! Now we have made the complete configuration on the admin
server part. Now as the complicated process have done, you can relax,
take a cup of coffee then just make a copy of Weblogic configuration
ONLY and paste it on each managed server node. Below are the steps :-

[shahril@weblogic\_mgr bin]\$ \$WL\_HOME/common/bin/pack.sh
-domain=\$DOMAIN\_HOME
-template=\$WL\_HOME/common/templates/domains/TEST\_template.jar
-template\_name=TEST -managed=true \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=256m; support was removed in 8.0 \
 \<\< read domain from "/home/shahril/wls/oracle/config/domains/TEST" \
 \>\> succeed: read domain from
"/home/shahril/wls/oracle/config/domains/TEST" \
 \<\< set config option Managed to "true" \
 \>\> succeed: set config option Managed to "true" \
 \<\< write template to
"/home/shahril/wls/oracle/product/fmw12/wlserver/common/templates/domains/TEST\_template.jar"
\

..........................................................................................
\
 \>\> succeed: write template to
"/home/shahril/wls/oracle/product/fmw12/wlserver/common/templates/domains/TEST\_template.jar"
\
 \<\< close template \
 \>\> succeed: close template \
 \
 [shahril@weblogic\_mgr \~]\$ ls -lh
\$WL\_HOME/common/templates/domains/TEST\_template.jar \
 -rw-r----- 1 shahril shahril 51K Jun 10 14:11
/home/shahril/wls/oracle/product/fmw12/wlserver/common/templates/domains/TEST\_template.jar
\

Shown above we've make a copy of weblogic configuration into a jar file.
We will bring over only this jar file to each weblogic managed server
and set it up from there.

[shahril@weblogic\_mgr \~]\$ scp -r
/home/shahril/wls/oracle/product/fmw12/wlserver/common/templates/domains/TEST\_template.jar
172.17.0.7:/home/shahril/wls/ \
 shahril@172.17.0.7's password: \
 TEST\_template.jar 100% 50KB 58.2MB/s 00:00 \

Now go to the managed server and extract the copied jar file. No
configuration is needed as it will bring over related domain we've
created. Below are the steps :-

[shahril@weblogic\_node1 \~]\$ cd \$WL\_HOME \
 [shahril@weblogic\_node1 wlserver]\$ pwd \
 /home/shahril/wls/oracle/product/fmw12/wlserver \
 [shahril@weblogic\_node1 wlserver]\$ \$WL\_HOME/common/bin/unpack.sh
-template=/home/shahril/wls/TEST\_template.jar -domain=\$DOMAIN\_HOME \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=256m; support was removed in 8.0 \
 \<\< read template from "/home/shahril/wls/TEST\_template.jar" \
 \>\> succeed: read template from "/home/shahril/wls/TEST\_template.jar"
\
 \<\< set config option DomainName to "TEST" \
 \>\> succeed: set config option DomainName to "TEST" \
 \<\< write Domain to "/home/shahril/wls/oracle/config/domains/TEST" \

....................................................................................................
\
 \>\> succeed: write Domain to
"/home/shahril/wls/oracle/config/domains/TEST" \
 \<\< close template \
 \>\> succeed: close template \

Excellent! We have successfully extract the copied of weblogic
configuration. Next step, let's start the Weblogic service in managed
server. Below are the steps :-

[shahril@weblogic\_node1 wlserver]\$ cd \$DOMAIN\_HOME \
 [shahril@weblogic\_node1 TEST]\$ cd bin/ \
 [shahril@weblogic\_node1 bin]\$ ./stopManagedWebLogic.sh Node\_Server01
t3://172.17.0.6:7001 weblogic Test1234 \
 Stopping Weblogic Server... \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
PermSize=128m; support was removed in 8.0 \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=256m; support was removed in 8.0 \
 Initializing WebLogic Scripting Tool (WLST) ... \
 Jython scans all the jar files it can find at first startup. Depending
on the system, this process may take a few minutes to complete, and WLST
may not return a prompt right away. \
 \
 Welcome to WebLogic Server Administration Scripting Shell \
 \
 Type help() for help on available commands \
 \
 Connecting to t3://172.17.0.6:7001 with userid weblogic ... \
 Successfully connected to Admin Server "AdminServer" that belongs to
domain "TEST". \
 \
 Warning: An insecure protocol was used to connect to the \
 server. To ensure on-the-wire security, the SSL port or \
 Admin port should be used instead. \
 \
 Shutting down the server Node\_Server01 with force=false while
connected to AdminServer ... \
 No stack trace available. \
 Problem invoking WLST - Traceback (innermost last): \
 File
"/home/shahril/wls/oracle/config/domains/TEST/shutdown-Node\_Server01.py",
line 4, in ? \
 File "", line 1199, in shutdown \
 File "", line 552, in raiseWLSTException \
 WLSTException: Error occurred while performing shutdown : No Server
with name "Node\_Server01" configured in the domain \
 \
 Done \
 Stopping Derby Server... \
 \
 \
 [shahril@weblogic\_node1 bin]\$ ./startManagedWebLogic.sh
Node\_Server01 t3://172.17.0.6:7001 & \
 [1] 5378 \
 [shahril@weblogic\_node1 bin]\$ . \
 . \
 JAVA Memory arguments: -Xms256m -Xmx512m -XX:CompileThreshold=8000
-XX:PermSize=128m -XX:MaxPermSize=256m \
 . \

CLASSPATH=/usr/java/jdk1.8.0\_131/lib/tools.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/weblogic\_sp.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/weblogic.jar:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/net.sf.antcontrib\_1.1.0.0\_1-0b3/lib/ant-contrib.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/modules/features/oracle.wls.common.nodemanager\_2.0.0.0.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/common/derby/lib/derbyclient.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/common/derby/lib/derby.jar:/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/xqrl.jar
\
 . \

PATH=/home/shahril/wls/oracle/product/fmw12/wlserver/server/bin:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/org.apache.ant\_1.9.2/bin:/usr/java/jdk1.8.0\_131/jre/bin:/usr/java/jdk1.8.0\_131/bin:/usr/java/jdk1.8.0\_131/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/shahril/bin
\
 . \

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\
 \* To start WebLogic Server, use a username and \* \
 \* password assigned to an admin-level user. For \* \
 \* server administration, use the WebLogic Server \* \
 \* console at http://hostname:port/console \* \

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\
 starting weblogic with Java version: \
 java version "1.8.0\_131" \
 Java(TM) SE Runtime Environment (build 1.8.0\_131-b11) \
 Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode) \
 Starting WLS with line: \
 /usr/java/jdk1.8.0\_131/bin/java -server -Xms256m -Xmx512m
-XX:CompileThreshold=8000 -XX:PermSize=128m -XX:MaxPermSize=256m
-Dweblogic.Name=Node\_Server01
-Djava.security.policy=/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/weblogic.policy
-Dweblogic.security.SSL.trustedCAKeyStore=/home/shahril/wls/oracle/product/fmw12/wlserver/server/lib/cacerts
-Xverify:none
-Djava.endorsed.dirs=/usr/java/jdk1.8.0\_131/jre/lib/endorsed:/home/shahril/wls/oracle/product/fmw12/oracle\_common/modules/endorsed
-da -Dwls.home=/home/shahril/wls/oracle/product/fmw12/wlserver/server
-Dweblogic.home=/home/shahril/wls/oracle/product/fmw12/wlserver/server
-Dweblogic.management.server=t3://172.17.0.6:7001
-Dweblogic.utils.cmm.lowertier.ServiceDisabled=true weblogic.Server \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
PermSize=128m; support was removed in 8.0 \
 Java HotSpot(TM) 64-Bit Server VM warning: ignoring option
MaxPermSize=256m; support was removed in 8.0 \
 Jun 10, 2018 3:29:41 PM UTC Info Security BEA-090905 Disabling the
CryptoJ JCE Provider self-integrity check for better startup
performance. To enable this check, specify
-Dweblogic.security.allowCryptoJDefaultJCEVerification=true. \
 Jun 10, 2018 3:29:41 PM UTC Info Security BEA-090906 Changing the
default Random Number Generator in RSA CryptoJ from ECDRBG128 to
FIPS186PRNG. To disable this change, specify
-Dweblogic.security.allowCryptoJDefaultPRNG=true. \
 Jun 10, 2018 3:29:42 PM UTC Info WebLogicServer BEA-000377 Starting
WebLogic Server with Java HotSpot(TM) 64-Bit Server VM Version
25.131-b11 from Oracle Corporation. \
 Jun 10, 2018 3:29:42 PM UTC Info Management BEA-141107 Version:
WebLogic Server 12.1.3.0.0 Wed May 21 18:53:34 PDT 2014 1604337 \
 Jun 10, 2018 3:29:43 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to STARTING. \
 Jun 10, 2018 3:29:43 PM UTC Info WorkManager BEA-002900 Initializing
self-tuning thread pool. \
 Jun 10, 2018 3:29:43 PM UTC Info WorkManager BEA-002942 CMM memory
level becomes 0. Setting standby thread pool size to 256. \
 Jun 10, 2018 3:29:43 PM UTC Notice Log Management BEA-170019 The server
log file
/home/shahril/wls/oracle/config/domains/TEST/servers/Node\_Server01/logs/Node\_Server01.log
is opened. All server side log events will be written to this file. \
 Jun 10, 2018 3:29:45 PM UTC Notice Security BEA-090082 Security
initializing using security realm myrealm. \
 Jun 10, 2018 3:29:46 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to STANDBY. \
 Jun 10, 2018 3:29:46 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to STARTING. \
 Jun 10, 2018 3:29:46 PM weblogic.wsee.WseeCoreMessages
logWseeServiceStarting \
 INFO: The Wsee Service is starting\
 \
 Jun 10, 2018 3:29:48 PM UTC Notice Log Management BEA-170027 The server
has successfully established a connection with the Domain level
Diagnostic Service. \
 Jun 10, 2018 3:29:48 PM UTC Notice Cluster BEA-000197 Listening for
announcements from cluster using unicast cluster messaging \
 Jun 10, 2018 3:29:48 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to ADMIN. \
 Jun 10, 2018 3:29:48 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to RESUMING. \
 Jun 10, 2018 3:29:48 PM UTC Notice Cluster BEA-000162 Starting "async"
replication service with remote cluster address "null" \
 Jun 10, 2018 3:29:48 PM UTC Notice Server BEA-002613 Channel "Default"
is now listening on 172.17.0.7:8001 for protocols iiop, t3,
CLUSTER-BROADCAST, ldap, snmp, http. \
 Jun 10, 2018 3:29:48 PM UTC Notice WebLogicServer BEA-000332 Started
the WebLogic Server Managed Server "Node\_Server01" for domain "TEST"
running in development mode. \
 Jun 10, 2018 3:29:48 PM UTC Notice WebLogicServer BEA-000360 The server
started in RUNNING mode. \
 Jun 10, 2018 3:29:48 PM UTC Notice WebLogicServer BEA-000365 Server
state changed to RUNNING. \

Great! Now we have successfully configure the weblogic services in
managed server. You can do the process to other managed server and later
we will define the cluster grouping from administration server
dashboard.

4. Testing Phase
----------------

As to ensure our Weblogic architecture are work as expected, we'll just
apply a simple test configuration on our server. For this test, we will
open the Weblogic admin server dashboard and from the dashboard console
itself we will add our both 2 managed server into the environment and
define it as a cluster.

Now, let's open up our admin dashboard via
http://172.17.0.6:7001/console . As mention before, for this test we've
just creating only 1 DOMAIN which is TEST therefore the default port
7001 are dedicated for this domain. For multiple domain configuration
can be seperated by its own dedicated port. Once you have launch the URL
on the browser, you should see the console like below, enter the
username and password that we've define during configuration above, for
this test it's weblogic/Test1234 :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.png)

Once you have login successfully, you will see a complete dashboard like
shown below :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.2_.png)

To proceed with our test, from the dashboard click on Environment -\>
Servers tab . You will see the results as shown like below which
automatically the Weblogic Admin server already included inside TEST
domain :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.3_.png)

Next, click on button
![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/button.png)
, it will forward you to page like below . Fill in the Weblogic Managed
Server information like it's IP addresses and the weblogic port as per
shown then click next.

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.4_.add_new_instance_.1_.png)

After that, define the new cluster name you want to the click next.
Remain others as per default like example below :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.4_.add_new_instance_.2_.png)

Great, now you've include an instance node inside your newly created
weblogic cluster. Below are the example snapshot :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.4_.add_new_instance_.3_.png)

Now, let's bring up the instance node. For this case, we'll go back to
CLI shell and start up the weblogic managed server like command we use
before like below :-

[shahril@weblogic\_node1 wlserver]\$ cd \$DOMAIN\_HOME \
 [shahril@weblogic\_node1 TEST]\$ cd bin/ \
 [shahril@weblogic\_node1 bin]\$ ./startManagedWebLogic.sh
Node\_Server01 t3://172.17.0.6:7001 & \

Once done, go back to admin URL and refresh the dashboard. You will see
that now the instance node you've established are up and running. Below
are the example snapshot :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/adminserver.4_.add_new_instance_.4_.png)

Next, using the same process add up another Weblogic Managed Server into
the define cluster. As the final result, you will see all instance node
you've added are up and running and under load balancing mode. Below are
the example result :-

![](https://www.howtoforge.com/images/how_to_setup_and_install_oracle_weblogic_in_centos_6/results.PNG)

Congratulations! Now you've successfully create a new weblogic cluster
architecture.

