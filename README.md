# check_tcp_connections
nagios check for established TCP connections

# Requirements
working SNMP

This check should work on any host that supports the TCP-MIB.  

The objective here is to alert when the number of established TCP connections reaches an upper threshold that is known to be a performance issue.  For example, this check might be used to alert when a web server has more than 999 active sessions, or a load balancer has more than 500 active sessions.

Add the following section to the commands.cfg file.  This assumes that the monitored host has a working SNMP daemon, and the check_snmp check already exists on the nagios host. 
```
define command {
   # Look at the number of TCP connections for which the current state is either ESTABLISHED or CLOSE-WAIT. 
   # .iso.identified-organization.dod.internet.mgmt.mib-2.tcp.tcpCurrEstab.0
   # .1.3.6.1.2.1.6.9.0 
   command_name      check_tcp_connections
   command_line      $USER1$/check_snmp -H $HOSTADDRESS$ -o .1.3.6.1.2.1.6.9.0 -C $ARG1$ -w $ARG2$ -c $ARG3$ 
}
```

Add the following section to the services.cfg file.  It is recognized that different types of hosts may have different upper limits for active TCP connections.  For example, a web server that serves up static HTML pages might be able to handle more connections than a database server.  For this reason, it is expected that you may have multiple stanzas similar to the following, with different thresholds for different types of servers.
```
# check established TCP connections
# These servers are known to start crashing at 350 active connections, so warn at 200, critical at 300.
define service {
       # Look at the number of TCP connections for which the current state is either ESTABLISHED or CLOSE-WAIT. 
       # .iso.identified-organization.dod.internet.mgmt.mib-2.tcp.tcpCurrEstab.0
       # .1.3.6.1.2.1.6.9.0 
       # Syntax is  check_tcp_connections!COMMUNITY!WARN!CRIT! 
       use                             generic-24x7-service
       hostgroup_name                  all_webservers
       service_description             TCP connections
       max_check_attempts              30                  ; Don't alert for brief traffic spikes
       check_command                   check_tcp_connections!public!200!300
}
```

# Output
Based on the thresholds you set in the above stanza, you will get ouput similar to the following:
<br><img src=images/check_tcp_connections.png>
