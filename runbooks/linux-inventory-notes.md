# Linux fleet inventory: design notes

## Problem

A mixed Linux fleet needed a repeatable operating-system inventory. Manual collection was slow and, more importantly, tended to make unreachable hosts disappear from the final report. I wanted the output to make failure explicit.

## Design choices

- Use the inventory hostname/IP as the primary identifier so the report can be reconciled directly with the source inventory.
- Avoid depending on Python facts on the remote host for the first probe; `raw` can still retrieve `/etc/os-release` on minimal systems.
- Normalize the output into a small record rather than trying to parse distribution-specific command formatting later.
- Preserve `unreachable` as a report outcome rather than silently skipping a machine.
- Render once on the controller to avoid concurrent writes to one file from many hosts.

## Production considerations

The public playbook is intentionally compact. In a large enterprise run I would also set SSH/connect timeouts, batch concurrency, privilege-escalation behavior, and an exception list for appliances or nonstandard Linux distributions.

The key operational property is that a failed probe is visible and actionable.
