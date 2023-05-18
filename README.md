# geoblocker_bash
Block incoming traffic from anywhere except subnets in a whitelist. Uses iptables. Collection of bash scripts with easy install.

Intended use case is a server that needs to be publically accessible in your country but does not need to be internationally accessible. For example, a server you run your CRM application on.

The collection includes 6 scripts:
1. geoblocker_bash-install
2. geoblocker_bash-uninstall
3. geoblocker_bash-run
4. geoblocker_bash-fetch
5. geoblocker_bash-apply
6. validate_cron_schedule.sh

The install script:
- Checks for prerequisites
- Copies the scripts (including itself) into /usr/local/bin
- Creates system folder for scripts' data in /var/lib/geoblocker_bash. Data consists of a file storing pre-install iptables policies for the INPUT and FORWARD chains, and fetched subnet lists from RIPE.
- Runs geoblocker_bash-apply to immediately apply new firewall config
- Verifies that crond service is enabled. Enables it if not.
- Validates optionally user-specified cron schedule expression (if not specified then uses default schedule "0 4 * * *" (at 4:00 [am] every day). 
(that's what validate_cron_schedule.sh is for)
- Creates cron periodic cron task based on that and a reboot task. Both cron tasks call the geoblocker_bash-run script with the necessary arguments.

The uninstall script:
- Removes the associated ipset
- Removes associated iptables rules
- Restores pre-install iptables policies for INPUT and FORWARD chains from backup
- Deletes scripts' data folder /var/lib/geoblocker_bash
- Deletes the scripts from /usr/local/bin

The run script simply calls the fetch script, then calls the apply script, passing required arguments.

The fetch script is based on a prior script by mivk, called get-ripe-ips. So it's basically a fork, modified and hopefully improved.
It can be used separately from this collection, as it does its own pre-requisite checks and input validation and accepts arguments.
- Gets country IP addresses from RIPE and compiles them into separate ipv4 and ipv6 plain lists
- Attempts to determine local ipv4 subnet for the main network interface and adds that to the end of the list

The apply script:
- creates or updates a named ipset from a user-specified file (which should contain a plain ipv4 subnets list).
- sets default policy on INPUT and FORWARD iptables chains to DROP
- Then creates iptables rules that allow connection from subnets included in the ipset.
It also can be used separately from this collection, as it does its own pre-requisite checks and input validation and accepts arguments.

The validate_cron_schedule script is used by the install script. It accepts cron schedule expression and attempts to make sure that it complies with format that cron expects.

Pre-requisites:
- linux running systemd (tested on Debian, may or may not work on other distributions)
- root access
- ipset (install it with 'apt install ipset' or similar)
- jq - Json processor (install it with 'apt install jq' or similar)


All scripts accept the -d argument for debug (in case troubleshooting is needed).

The run, fetch and apply scripts write to syslog in case critical errors occur. The run script also writes a syslog line upon success.

Note that the install script creates cron jobs that will be run as root. Make appropriate security arrangements to prevent it from getting modified by unauthorized third parties.