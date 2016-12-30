title: notes-on-cpl-dpl-and-rpl-terms
date: 2016-12-30 11:04:33
tags:
- x86
- os
---
*https://iambvk.wordpress.com/2007/10/10/notes-on-cpl-dpl-and-rpl-terms/*

x86 Segmentation architecture uses there privilege concepts.

### CPL – Current Privilege Level

This is the privilege of the currently executing code. Last two bits of CS register are considered as CPL.

Inter-segment calls, jumps, external interrupts, exceptions, task switching etc. operations can change the CS register contents thus, the privilege of currently executing code.

### DPL – Descriptor Privilege Level

All 8 byte descriptors that define code, data, stack etc., segments have two bits reserved for specifying a privilege-level for that segment. This is known as DPL.

DPL bits specify the minimum (or sometimes maximum) privilege required for using (i.e, executing/reading/writing) that segment contents.

### RPL – Requested Privilege Level

These are the last two bits of DS, ES, SS, FS, GS registers. RPL field is used to harden the CPL, when higher-privileged code is servicing lower-privileged processes requests.

Assume a higher-privileged device-driver that supports a mechanism where, it can copy data from disks directly into lower-privileged processes’ data-segments. Lower-privileged processes must pass their data-segment details (selector, address and size of data to copy) to the device-driver so that device-driver can copy data into appropriate location.

Since a device-driver is higher-privileged, a lower-privileged process can trick the driver to copy data into high-privileged data-segments, simply by passing wrong selector value. This kind of exploit is called, Privilege Escalation.

How RPL helps to solve Privilege Escalation problem?

Continuing the above example, whenever device-driver loads the destination segment, it modifies the destination segment’s RPL to match the requestor (lower-privileged) process. Since protection rules for data-segments check for both CPL <= DPL and RPL <= DPL conditions, higher-privileged process gets a protection-fault on RPL <= DPL check.

The point to note is, higher-privileged code, when it is providing services to lower-privileged processes should reduce its privilege temporarily to the requestors’ privilege-level.

