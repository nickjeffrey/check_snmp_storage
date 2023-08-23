# check_snmp_storage
nagios check for disk/RAM/swap utilization via SNMP 

Forked from http://nagios.manubulon.com/snmp_storage.html

# Requirements
perl, Net::SNMP perl module, SNMP enabled on monitored hosts

# Supported Operating Systems
This check assumes the monitored host used the hrStorage MIB, so will support Windows, Linux, MacOS.  Please note that AIX is specifically not supported, because AIX has its own unique MIB tree.

# Usage Examples

This script is executed on the nagios host, and will query the remote hosts via SNMP.  

Add the following stanzas to the commands.cfg file on the nagios host:

```
# 'check_snmp_storage' command definition
# use the --perfdata parameter to include nagios perfdata in output, useful for graphing via pnp4nagios
# use the --gigabyte parameter to output usage in GB instead of the default of MB
define command{
        command_name    check_snmp_storage
        command_line    $USER1$/check_snmp_storage -H $HOSTADDRESS$ -C $ARG1$ -m $ARG2$ -w $ARG3$ -c $ARG4$ --perfdata --gigabyte
        }

# 'check_snmp_storage_ram' command definition
# This is a customized version of check_snmp_storage
# Note that we hardcode -q Ram and set -m to a wildcard.
# This is because some versions of Linux (ie Centos 5.4) called Ram "Physical Memory",
# while other versions of Linux (ie Centos 6.2) called Ram "Real Memory"
define command{
        command_name    check_snmp_storage_ram
        command_line    $USER1$/check_snmp_storage -H $HOSTADDRESS$ -C $ARG1$ -q Ram -m .* -w $ARG2$  -c $ARG3$
        }

# 'check_snmp_storage_swap' command definition
# This measures usage of the swap space on Linux
# This is a customized version of check_snmp_storage
# Note that we hardcode -q VirtualMemory and set -m to ^Swap (regex that finds all VirtualMemory entries beginning with Swap)
# This is because Linux (ie Centos 5.4) calls this "Swap Space", while Windows calls this "Virtual Memory"
define command{
        command_name    check_snmp_storage_swap
        command_line    $USER1$/check_snmp_storage -H $HOSTADDRESS$ -C $ARG1$ -q VirtualMemory -m ^Swap -w $ARG2$  -c $ARG3$
        }

# 'check_snmp_storage_paging' command definition
# This measures usage of the paging file on Windows, or the swap space on Linux
# This is a customized version of check_snmp_storage
# Note that we hardcode -q VirtualMemory and set -m to a wildcard.
# This is because Linux (ie Centos 5.4) calls this "Swap Space", while Windows calls this "Virtual Memory"
define command{
        command_name    check_snmp_storage_paging
        command_line    $USER1$/check_snmp_storage -H $HOSTADDRESS$ -C $ARG1$ -q VirtualMemory -m .* -w $ARG2$  -c $ARG3$
        }
```


