---
title: "Building the Network Automation Toolkit: From CLI Tedium to Production Tool"
date: 2025-06-01
tags: [Python, Netmiko, Automation, Cisco]
layout: post
excerpt: >-
  I built a Python toolkit to automate Cisco Catalyst 9300 migration validation
  in live healthcare environments. Here's what the problem looked like, how I solved it,
  and what I learned building tools other engineers depend on.
---

The change window was at 2 AM. We had four hours to cut over a hospital switch stack,
validate that every device had reconnected, and be ready to roll back if anything looked
wrong. The validation step alone — running `show mac address-table` across eight switches,
then cross-referencing against a pre-cutover snapshot in a spreadsheet — was taking 45
minutes by itself. Under time pressure, in a hospital, on infrastructure where mistakes
have consequences.

That was the problem. This is the tool I built to solve it.

## The Core Workflow

Before any migration, the toolkit connects to every switch in scope using Netmiko and captures a baseline snapshot: MAC address tables, interface states, VLAN assignments, CDP neighbors. It dumps everything into structured JSON, timestamped and named by device.

Post-cutover, it runs the same capture again and diffs the two snapshots. Any MAC that was present before and missing after gets flagged. Any interface that changed state gets flagged. Any VLAN that disappeared gets flagged. The engineer gets a clean report instead of a spreadsheet.

```python
from netmiko import ConnectHandler
import json
from datetime import datetime

def capture_snapshot(device_list, output_dir):
    snapshots = {}
    for device in device_list:
        conn = ConnectHandler(**device)
        hostname = conn.find_prompt().strip('#>')
        snapshots[hostname] = {
            'mac_table': conn.send_command('show mac address-table', use_textfsm=True),
            'interfaces': conn.send_command('show interfaces status', use_textfsm=True),
            'timestamp': datetime.now().isoformat()
        }
        conn.disconnect()
    return snapshots
```

## What I Learned Building This

**TextFSM is worth the learning curve.** Netmiko's `use_textfsm=True` parameter parses CLI output into structured data automatically, using templates from the ntc-templates library. Instead of writing regex to parse `show mac address-table`, you get a list of dictionaries. The first time it worked I felt like I'd been doing it wrong for years.

**Error handling in network automation is brutal.** Devices go unreachable mid-script. SSH timeouts during long operations. A switch reboots unexpectedly. I added retry logic with exponential backoff and made sure every failed device produced a clear error entry in the output rather than crashing the whole run. In healthcare environments, a partial result is better than no result.

**Build for the next engineer.** The tool needed to work for other team members who weren't Python-fluent. That's why it's a GUI that launches from a single script: nobody running a cutover at 2 AM should have to remember which script to call or what arguments to pass. Credentials are entered at runtime in the GUI instead of living in files, and the Excel port maps and stack summaries it produces are usable by other engineers without any cleanup.

## The Result

Validation time dropped from 45 minutes to under 5. The tool ran its own checks, and the engineer reviewed a diff instead of a spreadsheet.

Validation turned out to be just the first workflow. The toolkit has since grown to cover config pulling and port mapping, baseline config generation, and live port monitoring during the change window. Together they save 3+ hours per cutover, and the toolkit has been part of all 75+ switch migrations on the engagement.

I'm still building it out, with OSPF neighbor verification and BGP session state checks next.
