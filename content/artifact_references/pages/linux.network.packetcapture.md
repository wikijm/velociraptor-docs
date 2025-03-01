---
title: Linux.Network.PacketCapture
hidden: true
tags: [Client Artifact]
---

description: |
This artifact leverages tcpdump to natively capture packets.

The `Duration` parameter is used to define how long (in seconds) the capture should be.  Specific interfaces can be defined using the `Interface` parameter, otherwise the artifact defaults to an interface assignment of `any`.

A `BPF` can also be supplied to filter the captured traffic as desired.


```yaml
name: Linux.Network.PacketCapture
author: Wes Lambert, @therealwlambert
description: |
  description: |
  This artifact leverages tcpdump to natively capture packets.

  The `Duration` parameter is used to define how long (in seconds) the capture should be.  Specific interfaces can be defined using the `Interface` parameter, otherwise the artifact defaults to an interface assignment of `any`.

  A `BPF` can also be supplied to filter the captured traffic as desired.

required_permissions:
  - EXECVE

parameters:
    - name: Duration
      type: integer
      description: Duration (in seconds) of PCAP to be recorded.
      default: 10

    - name: Interface
      type: string
      default: any

    - name: BPF
      type: string
      default:

sources:
    - query: |
            LET pcap <= tempfile(extension=".pcap")

            LET record_pcap =
                    SELECT * FROM execve(argv=['bash', '-c',  '(tcpdump -nni ' + Interface + ' -w ' + pcap + ' ' + '\'' + BPF + '\') & sleep ' + Duration + '; kill $!'])

            SELECT * FROM chain (
              a={SELECT * FROM record_pcap},
              b={SELECT upload(file=pcap) AS Upload FROM scope()}
            )

```
