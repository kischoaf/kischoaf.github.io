---
title: Network Automation Toolkit
layout: page
---

## Overview

A Python-based automation toolkit I built to support Cisco Catalyst 9300 switch migrations in production healthcare environments. It wraps four core workflows — config pulling and port mapping, baseline config generation, pre/post cutover comparison, and live port monitoring — into a single GUI-driven application. I use it daily at work and it has cut post-migration validation time by more than three hours per cutover.

## What I Built

The toolkit launched because manual cutover validation was slow, error-prone, and happened under time pressure in environments where mistakes have direct patient care consequences. I built it to solve that specific problem and kept extending it as new pain points surfaced on the job.

**Config Pulling and Port Mapping**

The first module connects to Catalyst 9300 switches via Netmiko over SSH, either pulling live configs directly or loading previously saved configs from disk. From that data it generates a structured Excel port map in a user-specified directory. The first tab is a switch stack summary covering hostname, serial number, switch version, software code version, and MAC addresses for each member in the stack — exactly the information needed to document a site before work begins.

**Baseline Config Generator**

From the pulled configs, the toolkit generates a site-standard pre-configuration for new switches: domain names, NTP servers, syslog destinations, management IP addresses, and any other parameters that need to match the existing environment. The goal is consistent, repeatable staging output rather than hand-typing config from a template.

**MAC Address and CDP Neighbor Comparison**

This is the core cutover validation module. Before the cutover window, I capture MAC address tables and CDP neighbor relationships from the existing switches. After the new switches go live, I run the same capture and compare the two. The tool surfaces any MACs or neighbors that were present before and have not reappeared, flagging potential missed connections or devices that have not come back up. This is the step that previously required manually paging through `show mac address-table` output across multiple switches in a stack.

**Live Port Monitor**

During the cutover itself, the live port monitor tracks port state in real time — status, VLAN assignment, and whether the MAC addresses that were active before the cutover have returned on each port. This gives on-site engineers a single view during a change window instead of polling individual switch CLIs.

**Interface**

All four modules are accessible through a Python GUI that launches from a single script. This was deliberate — the people running cutovers should not need to know which script to call or what arguments to pass at 2 AM during a hospital change window.

## Technical Depth

**Netmiko** handles the SSH connection layer. It manages device-type abstraction for Cisco IOS-XE, connection timing, and the send-and-wait logic for command output, which is meaningfully more reliable than raw Paramiko for structured CLI interactions. All target devices are physical Catalyst 9300 switches in production — not virtual labs.

**Output parsing** from `show` commands varies in consistency across IOS-XE versions, so the config and MAC table parsing had to handle some variation in field alignment and whitespace without breaking. The Excel output uses `openpyxl`, structured so that the port map and stack summary are usable as-is by other engineers on the team without any cleanup.

**The GUI** consolidates all four workflows into one entry point. This was a deliberate design decision — the tool needed to be runnable by any engineer on the project, not just me.

**Credentials** are handled at runtime through the GUI rather than hard-coded or stored in flat files, which matters in a healthcare environment with real compliance requirements.

The toolkit is stored in a private repository and is actively maintained as the migration engagement continues.

## Problem It Solves

Healthcare network migrations run during overnight change windows with hard rollback deadlines. Before this toolkit, post-cutover validation meant SSH-ing to each switch individually, running a series of `show` commands, and mentally cross-referencing output against a pre-cutover snapshot captured in a spreadsheet — across a stack of potentially eight switches. The process was slow, sequential, and easy to misread under pressure.

The MAC and CDP comparison modules turned that into a programmatic diff. The live port monitor replaced CLI polling. The port map generator replaced manual documentation. The baseline config generator replaced copying from a text template and editing by hand.

The result is fewer steps that require human attention during the highest-stress part of a migration, and a structured record that covers the site before and after the cutover.

## Results

The toolkit is in active daily use across an ongoing healthcare network migration engagement covering 10+ hospital locations and 75+ switch migrations. Post-migration validation time is down by more than three hours per cutover. The port maps and stack summaries it generates have been incorporated into the broader documentation workflow for the engagement.

The project also represents the first time I built a tool that other engineers on a project depend on operationally — which changed how I think about usability, error handling, and what "done" actually means for something that runs in a production change window.

## Photos

<!-- Drop your images in /assets/img/projects/ and uncomment the lines below -->
<!-- ![Description](../assets/img/projects/your-image-1.jpg) -->
<!-- ![Description](../assets/img/projects/your-image-2.jpg) -->

## Documentation

<!-- Link to a PDF or embed a document if you have one -->
<!-- [View Documentation](../assets/docs/your-document.pdf) -->
