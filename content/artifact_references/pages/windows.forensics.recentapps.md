---
title: Windows.Forensics.RecentApps
hidden: true
tags: [Client Artifact]
---

GUI Program execution launched on the Win10 system is tracked in the
RecentApps key


```yaml
name: Windows.Forensics.RecentApps
description: |
  GUI Program execution launched on the Win10 system is tracked in the
  RecentApps key

reference:
  - https://www.sans.org/security-resources/posters/windows-forensics-evidence-of/75/download

precondition: SELECT OS From info() where OS = 'windows'

parameters:
  - name: UserFilter
    default: ""
    description: If specified we filter by this user ID.
    type: regex

  - name: ExecutionTimeAfter
    default: ""
    type: timestamp
    description: If specified only show executions after this time.

  - name: RecentAppsKey
    default: Software\Microsoft\Windows\CurrentVersion\Search\RecentApps\*

  - name: UserHomes
    default: C:\Users\*\NTUSER.DAT

sources:
  - query: |
      LET TMP = SELECT * FROM foreach(
         row={
            SELECT FullPath FROM glob(globs=UserHomes)
         },
         query={
            SELECT AppId, AppPath, LaunchCount,
                   timestamp(winfiletime=LastAccessedTime) AS LastExecution,
                   timestamp(winfiletime=LastAccessedTime).Unix AS LastExecutionTS,
                   parse_string_with_regex(
                      string=Key.FullPath,
                      regex="/Users/(?P<User>[^/]+)/ntuser.dat").User AS User
            FROM read_reg_key(
               globs=RecentAppsKey,
               root=pathspec(
                 DelegateAccessor="ntfs",
                 DelegatePath=FullPath),
               accessor="raw_reg")
         })

      LET A1 = SELECT * FROM if(
          condition=UserFilter,
          then={
            SELECT * FROM TMP WHERE User =~ UserFilter
          }, else={ SELECT * FROM TMP})

      SELECT * FROM if(
          condition=ExecutionTimeAfter,
          then={
            SELECT * FROM A1 WHERE LastExecutionTS > ExecutionTimeAfter
          }, else={ SELECT * FROM A1})

```
