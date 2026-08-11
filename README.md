# Support Engineering & Platform Operations Work Samples

This repository contains sanitized, public reconstructions of technical work I have carried out across enterprise Linux, platform, infrastructure, and support environments. Customer names, hostnames, IP addresses, credentials, and proprietary implementation details have been removed or generalized.

The purpose of this repository is to showcase how I approach real production problems: gather evidence, separate observations from assumptions, compare healthy and unhealthy states, test changes carefully, validate recovery, automate repetitive work, and document outcomes clearly.

## Included samples

1. **YARN NodeManager local-directory failure analysis**
   A cross-host investigation that traced a misleading “directory not writable” health failure to an identity and ownership mismatch.

2. **RHEL repository and proxy failure analysis**
   A layered troubleshooting case that separated an internal repository certificate issue from an external authenticated-proxy failure.

3. **Linux fleet inventory with Ansible**
   A sanitized implementation of an operating-system inventory workflow for mixed Linux environments, including explicit handling of unreachable hosts.

4. **Linux platform incident runbook**
   A reusable, evidence-driven workflow for diagnosing and managing production Linux and platform incidents.

## About these samples

These materials have been rewritten specifically for public sharing. They are based on real incidents, troubleshooting patterns, and operational work I have handled, but they are not copies of employer or customer documentation.

Sensitive details have been removed, and commands, names, addresses, and configuration values have been generalized where necessary.

## What this repository demonstrates

* Root-cause analysis rather than symptom treatment
* Structured and reproducible troubleshooting
* Evidence-based hypothesis testing
* Comparison of healthy and unhealthy system states
* Safe change sequencing and rollback awareness
* Linux and platform operations
* Infrastructure automation with Ansible
* Clear incident and technical documentation
* Customer and stakeholder communication
* Continuous improvement after incident resolution

The broader goal is to demonstrate the way I work as an engineer: **understand the system, reduce uncertainty, solve the immediate problem safely, and improve the environment so the same class of issue becomes easier to prevent or diagnose in the future.**
