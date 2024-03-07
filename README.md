# Zabbix template for Intel/LSI/Symbios RAID Controllers

[Topic](https://www.zabbix.com/forum/showthread.php?t=41439) on zabbix forum

## LSI_RAID_template
Zabbix template "Template LSI RAID". Should be imported after Zabbix value mappings via "Configuration" -> "Templates" -> "Import".

### Available items:
**Adapter**
- Adapter model (agent, disable by default)
- Firmware version (agent, disable by default)

**Battery Backup Unit**
- BBU State (trapper)(+trigger)
- BBU State of charge (trapper)(+trigger)
- BBU manufacture date (agent, disabled by default)
- BBU design capacity (agent, disabled by default)
- BBU current capacity (agent, disabled by default)

**Physical drives**
- Firmware state (trapper)(+trigger)
- Predictive errors (trapper)(+trigger)
- Media errors (trapper)(+trigger)
- Size (agent, disabled by default)
- Model (agent, disabled by default)

**Logical volumes**
- Volume state (trapper)(+trigger)
- Volume size (agent, disabled by default)

## Scripts
3 scripts are available for unix (perl) servers

### RAID Discovery script
This script is used for Low Level Discovery of RAID configuration. Scripts uses zabbix_sender and agent configuration file to report RAID configuration to zabbix.

### RAID checks script
This script is used by zabbix agent to check "non critical" items, such as adapter model, physical drive size, etc. But this script also supports checking of all items from template (i.e. you can change type for all items from 'Zabbix trapper' to 'Zabbix Agent'). I've created "trapper" version of this script, because zabbix agent very often reports that some item is not supported (I noticed that problem only on Windows servers). Probably there are some locking occurs, when zabbix agent checks a lot of items at the same time.

### RAID "trapper" check script
This scrips reports all values for "critical" items at once. Currently it reports BBU state and state of charge, physical drives state, predictive and media erros, logical volumes states. All other checks are performed by zabbix agent using RAID checks scripts and userparameters.

Discovery and "trapper" scripts are executed by system scheduler.

## Agent userparameters:

### Unix/Linux
   
    UserParameter=hw.raid.physical_disk[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode pdisk -item $4 -adapter $1 -enclosure $2 -pdisk $3
    UserParameter=hw.raid.logical_disk[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode vdisk -item $3 -adapter $1 -vdisk $2
    UserParameter=hw.raid.bbu[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode bbu -item $2 -adapter $1
    UserParameter=hw.raid.adapter[*],/usr/bin/perl -w /etc/zabbix/scripts/raid_check.pl -mode adapter -item $2 -adapter $1

For agents on unix servers RAID tool should be executed via sudo, add this to sudoers file:

    Defaults:zabbix !requiretty
    # path to your tool can be different
    zabbix  ALL=NOPASSWD:/opt/MegaRAID/CmdTool2/CmdTool2

## Scheduled Tasks

The scripts raid_trapper_checks and raid_discovery.pl must be run via cron/scheduled tasks. There is a trigger defined that alerts when the data is older than 10 minutes, so the script should be executed every 3 minutes or so.

### Linux/Unix

    # crontab -e
    * */3 * * * perl /etc/zabbix/scripts/raid_trapper_check.pl> /dev/null 2>&1
    * */3 * * * perl /etc/zabbix/scripts/raid_discovery.pl> /dev/null 2>&1
