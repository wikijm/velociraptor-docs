---
title: Windows.Sys.FirewallRules
hidden: true
tags: [Client Artifact]
---

List windows firewall rules.

```yaml
name: Windows.Sys.FirewallRules
description: List windows firewall rules.
reference:
  - https://social.technet.microsoft.com/Forums/azure/en-US/aaed9c6a-fb8b-4d43-8b69-9f4e0f619a8c/how-to-check-the-windows-firewall-settings-from-netsh-command?forum=winserverGP

parameters:
  - name: regKey
    default: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\**\FirewallRules\*

sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'
    query: |
        LET rules = SELECT Name as Value,
               parse_string_with_regex(string=Data,
                 regex=["Action=(?P<Action>[^|]+)",
                        "Active=(?P<Active>[^|]+)",
                        "Dir=(?P<Dir>[^|]+)",
                        "Protocol=(?P<Protocol>[^|]+)",
                        "LPort=(?P<LPort>[^|]+)",
                        "Name=(?P<Name>[^|]+)",
                        "Desc=(?P<Desc>[^|]+)",
                        "App=(?P<App>[^|]+)"]) as Record,
               Data,
               FullPath
        FROM glob(globs=regKey, accessor="registry")

        SELECT Value,
               Record.Action as Action,
               Record.Name as Name,
               get(item=Record, field="Desc") as Description,
               Record.App as App,
               Record.Action as Action,
               Record.Dir as Dir,
               if(condition=Record.Protocol = "6",
                  then="TCP",
                  else=if(condition=Record.Protocol = "17",
                          then="UDP",
                          else=Record.Protocol)) as Protocol,
               if(condition=Record.LPort = NULL,
                  then="Any",
                  else=Record.LPort) as LPort,
               Record.Name as Name
        FROM rules

```
