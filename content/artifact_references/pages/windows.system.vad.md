---
title: Windows.System.VAD
hidden: true
tags: [Client Artifact]
---

Enumerate the memory regions of each running process.


```yaml
name: Windows.System.VAD
description: |
  Enumerate the memory regions of each running process.

parameters:
  - name: processRegex
    description: A regex applied to process names.
    default: .
    type: regex

sources:
  - query: |
      LET processes = SELECT Pid, Name
        FROM pslist()
        WHERE Name =~ processRegex

      SELECT * FROM foreach(
          row=processes,
          query={
            SELECT Pid, Name,
                format(format='%x-%x', args=[Address, Address+Size]) AS Range,
                Protection, MappingName
            FROM vad(pid=Pid)
          })

```
