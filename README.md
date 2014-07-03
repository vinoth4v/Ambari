# 1. Install Ambary on management system

## 1.1. Add Hortonworks Ambari repostiry with wget

```bash
 wget http://public-repo-1.hortonworks.com/ambari/suse11/1.x/updates/1.6.0/ambari.repo  && cp ambari.repo /etc/zypp/repos.d
 # check repositories
 zypper repos | egrep -i "HDP|Ambari"
```

## 1.2. Patch ambari 1.6 configs for SLES 11sp3
### 1.2.1 Ganglia
*1. line  94: <name>apache2-mod_php5</name> to <name>apache2-mod_php5</name>*

```bash
 vim /var/lib/ambari-server/resources/stacks/HDP/1.3.2/services/GANGLIA/metainfo.xml
 vim /var/lib/ambari-server/resources/stacks/HDP/2.0.6/services/GANGLIA/metainfo.xml
```

### 1.2.2. Nagios
*1. line  110: <name>php5-json</name> to <name>php53-json</name>*

*2. line  114: <name>apache2-mod_php5</name> to <name>apache2-mod_php53</name>*

```bash
 vim /var/lib/ambari-server/resources/stacks/HDP/1.3.2/services/NAGIOS/metainfo.xml
 vim /var/lib/ambari-server/resources/stacks/HDP/2.0.6/services/NAGIOS/metainfo.xml
```


# 2. Install knox on knox gateway

## 2.1. Install knox
```bash
 zypper install knox
```

## 2.2. Set knox home
```bash
 # export knox home
 export gateway_home=/usr/lib/knox
```

## 2.3. Setup knox
```bash 
 su -l knox -c "$gateway_home/bin/gateway.sh setup"
```

### 2.3.1. Change knox configuration for testing
Knox default config file is **sandbox.xml** move it to your cluster name,
your need to know *master* service hosts of your cluster
in this example: **hwtestc01**

Change all services hostnames from your hortonworks cluster!

```bash
 cd /usr/lib/knox/conf/topologies
 export cluster_name=hwtestc01
 mv sandbox.xml "$cluster_name".xml
``` 

### 2.3.2. Test one of services from your hortonworks cluster

```bash
 # testing webhdfs
 telnet hwnode1.divbox.net 50070
```

## 2.4. Start knox
```bash
 su -l knox -c "$gateway_home/bin/gateway.sh setup"
```

### 2.4.1. Testing knox with guest account (inbuild LDAP)
```bash
 su -l knox -c "$gateway_home/bin/ldap.sh start"
```

## 2.6. Check knox service running
```bash
 su -l knox -c "$gateway_home/bin/gateway.sh status"
 >> Gateway is running with PID 23387.
```

### 2.6.1. Check testing ldap service

```bash
 su -l knox -c "$gateway_home/bin/ldap.sh status"
 >> LDAP is running with PID 23566.
```

### 2.7 Testing knox gateway host
* **cluster_name** is filename of your knox configuration file, s. 2.3.1.

```bash
 # gateway_host system with installed knox
 export knox_gateway_host=hwnode1.divbox.net

 # test connection with guest user from test ldap
 curl -k -ssl3 -u guest:guest-password -X GET "https://$knox_gateway_host:8443/gateway/$cluster_name/webhdfs/v1/?op=LISTSTATUS"
```
