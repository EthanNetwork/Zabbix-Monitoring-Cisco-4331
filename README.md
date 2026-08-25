# Cisco IOS-XE SNMPv3 + Zabbix Monitoring

Homelab project: configured SNMPv3 on a Cisco IOS-XE router and set it up as a monitored host in a Zabbix appliance (KVM/qcow2).

## Environment

- **Router:** Cisco IOS-XE 17.3.4a
- **Monitoring:** Zabbix 7.4.11 (official appliance image, running as a KVM guest)
- **Host OS:** Nobara Linux

## Router-side configuration

Built out SNMPv3 with a dedicated view, group, and user:

```
snmp-server view VIEW1 iso included
snmp-server group MYGROUP v3 priv read VIEW1 write VIEW1 access SNMP-access
snmp-server user ethangrishin MYGROUP v3 auth sha <authpass> priv aes 128 <privpass>
```

Restricted access with a named standard ACL:

```
ip access-list standard SNMP-access
 10 permit 192.168.0.0 0.0.255.255
 20 deny any
```

Verified with:

```
show snmp user
show running-config | section snmp
show running-config | section access-list
```

## Zabbix-side configuration

- Created host with an SNMP interface (SNMPv3, security level `authPriv`, auth protocol SHA1, privacy protocol AES128)
- Applied the built-in **Cisco IOS by SNMP** template
- Enabled `EnableGlobalScripts=1` in `zabbix_server.conf` (disabled by default on the appliance image) to allow manual script checks like "Check now" / ping from the web UI

## Troubleshooting along the way

- `systemctl restart zabbix_server` fails — the actual unit name uses a hyphen: `zabbix-server`
- Manual scripts ("Check now", ping) failed with `Cannot execute script. Global script execution on Zabbix server is disabled by server configuration.` — fixed by setting `EnableGlobalScripts=1` in `/etc/zabbix/zabbix_server.conf` and restarting the service
- Confirmed the fix by running a manual ping check from the Zabbix host config

## Result

- SNMPv3 credentials, group, and view configured and verified directly on the router (`show snmp user`)
- Host added to Zabbix with the SNMPv3 interface and Cisco IOS template
- ICMP/ping monitoring confirmed working end-to-end from Zabbix to the router
- Zabbix monitoring is configured but SNMP polling isn't fully functional end-to-end because my test environment sits behind a restrictive firewall; I left firewall changes out of scope for this iteration and documented the setup steps above for when that's opened up.
