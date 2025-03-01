---
title: Windows.Detection.Thumbdrives.List
hidden: true
tags: [Client Event Artifact]
---

Users inserting Thumb drives or other Removable drive pose a
constant security risk. The external drive may contain malware or
other undesirable content. Additionally thumb drives are an easy way
for users to exfiltrate documents.

This artifact watches for any removable drives and provides a
complete file listing to the server for any new drive inserted. It
also provides information about any addition to the thumb drive
(e.g. a new file copied onto the drive).

We exclude very large removable drives since they might have too
many files.


```yaml
name: Windows.Detection.Thumbdrives.List
description: |
  Users inserting Thumb drives or other Removable drive pose a
  constant security risk. The external drive may contain malware or
  other undesirable content. Additionally thumb drives are an easy way
  for users to exfiltrate documents.

  This artifact watches for any removable drives and provides a
  complete file listing to the server for any new drive inserted. It
  also provides information about any addition to the thumb drive
  (e.g. a new file copied onto the drive).

  We exclude very large removable drives since they might have too
  many files.

type: CLIENT_EVENT

parameters:
  - name: maxDriveSize
    type: int
    description: We ignore removable drives larger than this size in bytes.
    default: "32000000000"


sources:
  - query: |
        LET removable_disks = SELECT Name AS Drive,
            atoi(string=Data.Size) AS Size
        FROM glob(globs="/*", accessor="file")
        WHERE Data.Description =~ "Removable" AND Size < atoi(string=maxDriveSize)

        LET file_listing = SELECT FullPath,
            Mtime As Modified,
            Size
        FROM glob(globs=Drive+"\\**", accessor="file")
        LIMIT 1000

        SELECT * FROM diff(
          query={
             SELECT * FROM foreach(
                 row=removable_disks,
                 query=file_listing)
          },
          key="FullPath",
          period=10)
          WHERE Diff = "added"

```
