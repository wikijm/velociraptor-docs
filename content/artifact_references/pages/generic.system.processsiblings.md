---
title: Generic.System.ProcessSiblings
hidden: true
tags: [Client Artifact]
---

This artifact queries the process tracker to display all known
sibling processes of the target process (i.e. all other processes
from the same parent).

This is useful to reveal the complete interaction that included
the process in question (e.g. previous shell commands etc).

Minimum Version: 0.6.6


```yaml
name: Generic.System.ProcessSiblings
description: |
  This artifact queries the process tracker to display all known
  sibling processes of the target process (i.e. all other processes
  from the same parent).

  This is useful to reveal the complete interaction that included
  the process in question (e.g. previous shell commands etc).

  Minimum Version: 0.6.6

parameters:
  - name: CommandlineRegex
    default: .
    description: Target process by this command line
    type: regex

  - name: PidFilter
    description: Filter pids by this regex
    default: .
    type: regex

  - name: IncludePstree
    type: bool

sources:
  - query: |
        LET GetDetails(Records) = SELECT
            Id AS ChildPid,
            Data.CommandLine AS CommandLine,
            Data.Username AS Username,
            StartTime, EndTime
          FROM Records
          ORDER BY StartTime

        SELECT * FROM foreach(row={
          SELECT Pid, Ppid, Name
          FROM process_tracker_pslist()
          WHERE CommandLine =~  CommandlineRegex
            AND Pid =~ PidFilter
        }, query={
          SELECT Pid,Ppid, Name, ChildPid,
                 CommandLine, Username, StartTime, EndTime,
                 if(condition=IncludePstree, then=process_tracker_tree(id=Ppid)) AS ParentTree
          FROM foreach(row=GetDetails(
              Records=process_tracker_children(id=Ppid)))
        })

column_types:
  - name: ParentTree
    type: tree

```
