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


Example stanzas to be added to the services.cfg file on the nagios server.  Adjust the hostgroup_name parameter as apppriate for your environment.

```
# Check Windows physical mamory usage (RAM)
# Define a service to check Physical Memory (aka RAM) util on a over SNMP
# We feed this check 3 parameters: SNMP_community_name,warn_threshold,critical_threshold
define service{
        use                             generic-service
        hostgroup_name                  all_windows
        service_description             RAM
        check_command                   check_snmp_storage_ram!public!95!99
        }

# Check Windows virtual mamory usage
# Define a service to check Virtual Memory (aka paging space) util via SNMP
# NOTE: Windows reports virtual memory as physical memory (RAM) plus paging space, so the usage includes physical memory.
#       In other words, this does NOT just show Windows pagefile utilization.
# We feed this check 3 parameters: SNMP_community_name,warn_threshold,critical_threshold
define service{
        use                             generic-service
        hostgroup_name                  all_windows
        service_description             paging space
        check_command                   check_snmp_storage_paging!public!80!90
        }

# Define a service to check disk space on C: via SNMP
# We feed this check 3 parameters: SNMP_community_name,driveletter,warn_threshold,critical_threshold
define service{
        use                             generic-14x7-service
        hostgroup_name                  all_windows
        service_description             Drive C:
        check_command                   check_snmp_storage!public!"C:"!80!90
        }

# Define a service to check disk space on D: via SNMP
# We feed this check 3 parameters: SNMP_community_name,driveletter,warn_threshold,critical_threshold
define service{
        use                             generic-14x7-service
        host_name                       myhost1.example.com,myhost2.example.com
        service_description             Drive D:
        check_command                   check_snmp_storage!public!"D:"!90!95
        }

# Define a service to check Linux filesystem utilization via SNMP
# We feed this check 3 parameters: SNMP_community_name,driveletter,warn_threshold,critical_threshold
define service{
        use                             generic-14x7-service
        hostgroup_name                  all_linux
        service_description             disk space /var
        check_command                   check_snmp_storage!public!"/var"!80!90
        }

# check Linux swap space
define service {
        use                             generic-service
        hostgroup_name                  all_linux
        service_description             swap
        check_command                   check_snmp_storage_swap!public!50!75
        }

# check pfSense swap space
# For BIOS-based systems, you can find the name of the swap space by looking in /dev/label/ from an SSH login
# For UEFI-based systems, you can find the name of the swap space by clicking Diagnostics, Command Prompt, swapinfo
# Or by clicking Diagnostics, Command prompt, ls -l /dev/label
define service {
        use                             generic-14x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             swap
        #check_command                   check_snmp_storage!public!/dev/label/swap0!50!75
        #check_command                   check_snmp_storage!public!/dev/gptid/56b163f3-2e7a-11ea-8!10!20
        #check_command                   check_snmp_storage!public!/dev/gptid/e1afdc72-dffa-11e8-9!10!20
        check_command                   check_snmp_storage!public!/dev/label/swap0!10!20
        }

# check pfSense / filesystem space
# pfSense only has the root filesystem and swap space, but no other filesystems
# For BIOS-based systems, you can figure out the name of the local disk clicking Diagnostics, Command prompt, ls -l /dev/ufsid
# For UEFI-based systems, you can figure out the name of the local disk clicking Diagnostics, Command prompt, df -h
define service {
        use                             generic-14x7-service
        hostgroup_name                  all_pfsense_routers
        service_description             disk space
        #check_command                   check_snmp_storage!public!/dev/ufsid/587f4636bdb209d5!90!95
        #check_command                   check_snmp_storage!public!/dev/gptid/56afe90d-2e7a-11ea-8da7-001018b700f0!90!95
        #check_command                   check_snmp_storage!public!/dev/gptid/e1ae1095-dffa-11e8-9aae-001b2134ce00!90!95
        check_command                   check_snmp_storage!public!/dev/ufsid/5ebc14c59ad6c8d9!90!95
        action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$
        }


```
