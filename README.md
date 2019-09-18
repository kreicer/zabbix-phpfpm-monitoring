# Zabbix PHP-fpm Template with Auto-discovery and support for multiple pools

## Main features

- Supports auto discovery of pool (name)
- Easy and flexible configuration
- Low load on Zabbix server: most elements sending info by zabbix_sender
- Bash: no need to install Perl, PHP, Go or other languages
- Safe: no need to allow root for zabbix agent

## Provided Items
We capture useful data from host and fpm status page:

- PHP-fpm main statistic:

	- **Number of running php-fpm** - the number of running php-fpm processes

- For each pool:

    - **Uptime** - the uptime of current pool
	- **Start time** - the moment of launch of current pool
	- **Processes** - the number of processes in total / active / idle state of current pool
	- **Memory used** - the number and % of used memory by current pool
	- **Requests** - the number of reqiests in accepted (per min) / queue / slow state of current pool
	- **Other** - another stats from status page of current pool

History storage period is 7 days, trend storage period is 30 days.
Data is captured every minute.
These timings can be adjusted in template or per host if needed.

## Provided Triggers

- PHP-fpm is not running
- {#POOL}: accepted connections over threshold
- {#POOL}: process manager was changed
- {#POOL}: queue detected
- {#POOL}: slow requests detected
- {#POOL}: used memory over threshold
- {#POOL}: was restarted

## Provided Graphs
#### Processes for each pool
![Zabbix Php-fpm Graph #1](https://github.com/kreicer/zabbix-phpfpm-monitoring/raw/master/img/graph1.png)

Displays the following data:

- Number of Active processes
- Number of IDLE processes
- Total number of processes

#### Requests for each pool
![Zabbix Php-fpm Graph #2](https://github.com/kreicer/zabbix-phpfpm-monitoring/raw/master/img/graph2.png)

Displays the following data:

- Number of Accepted requests
- Number of Slow requests
- Queue
    
## Installation

### 1. On Target server
Perform the following operations on all servers with Zabbix Agent and PHP-fpm from which you want to capture the data.

#### 1.1 Install required packages

```console
apt-get update
apt-get install jq libxml2-utils
```

#### 1.2 Download the latest version of the php-fpm monitoring, install files, restart zabbix-agent

```console
wget https://github.com/kreicer/zabbix-phpfpm-monitoring/archive/master.zip /tmp/zabbix-phpfpm-monitoring.zip
unzip /tmp/zabbix-phpfpm-monitoring.zip
cp /tmp/zabbix-phpfpm-monitoring/fpm-monitoring.conf /etc/zabbix/zabbix_agentd.conf/
cp /tmp/zabbix-phpfpm-monitoring/fpm-monitoring.sh /etc/zabbix/scripts/
chmod +x /etc/zabbix/scripts/fpm-monitoring.sh
systemctl restart zabbix-agent.service
```

If you using non-standart zabbix-agent.conf path change it in fpm-monitoring.sh

```console
zabbixconf="/etc/zabbix/zabbix_agentd.conf"
```

#### 1.3 Clean up
Delete temporary files:

```console
rm /tmp/zabbix-phpfpm-monitoring.zip
rm -r /tmp/zabbix-phpfpm-monitoring/
```

### 2. On Zabbix Server
#### 2.1 Import Zabbix php-fpm template
In Zabbix frontend go to `"Configuration"->"Templates"->"Import"`:

Upload file `/fpm_template.xml` from the [archive](https://github.com/kreicer/zabbix-phpfpm-monitoring/archive/master.zip).

#### 2.2 Add the template to your hosts
Add template "PHP-fpm Template" to the hosts.

Add your status page address in the macros section of the host by adding value:

```
{$FPM_STATUS_URL}=your status page address
```

For few pools on one host:
Add your status pages addresses in the macros section of the host. Example:

```
{$FPM_STATUS_URL}=statuspage1oIostatuspage2...
```

oIo - speacial symbols sequence for dividing status pages addresses (you can rewrite it in fpm-monitoring.sh).

```
div="oIo"
```

Rewrite tresholds (if needed):

```
{$FPM_MEM_WARN}=50 //mem % of total mem for trigger
{$FPM_CONN_WARN}=150 //number of accepted reqs per minute for trigger
```

Setup is finished, just wait 15 minutes till Zabbix discovers your providers and captures the data (or use manual check).

# Compatibility
Tested with:
- Zabbix 4.2.5
- PHP 7.3

Should work:
- Zabbix 4.x
- Zabbix 3.x
- PHP 5.6

Not tested with:
- Zabbix 2.x

If it works, please let me know. 