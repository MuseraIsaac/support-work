# Linux / Platform Incident Runbook: Evidence before intervention

This is a generalized runbook for production Linux and platform incidents. It is designed to keep diagnosis reproducible while protecting service availability.

## 1. Establish impact

Record:

- customer-visible symptom
- affected service(s) and environment
- start time / last-known-good time
- scope: one host, one AZ/site, one cluster, or all users
- recent changes
- current mitigation, if any

Avoid starting with a favored root cause. Start with what is observed.

## 2. Capture a minimum evidence set

```bash
date -Is
hostnamectl
uptime
who -b

systemctl --failed
journalctl -p warning..alert --since '-30 min'

df -hT
df -i
free -m

ip -br addr
ip route
ss -s
```

Add service-specific logs and metrics without flooding the incident channel.

## 3. Separate facts, hypotheses, and actions

Use a small table in the incident notes:

| Type | Example |
|---|---|
| Fact | API latency rose at 10:14; CPU steal did not rise |
| Hypothesis | Database connection exhaustion is blocking workers |
| Test | Compare active DB sessions with configured pool limit |
| Result | Rejected - sessions are below 40% of limit |

This prevents a plausible early theory from silently becoming "the cause."

## 4. Compare against known-good state

When possible, compare:

- healthy peer vs failing peer
- before vs after change
- current config vs source-controlled/configured intent
- service identity, package, kernel, route, mount, certificate, or resource state

A healthy peer is often the fastest source of high-value differences.

## 5. Prefer reversible mitigation

A mitigation should have:

- expected effect
- explicit rollback
- person responsible
- validation check

Examples: remove one node from rotation, restore a previous route, restart one replica rather than the whole cluster, or revert one configuration change.

## 6. Validate at three levels

After a change, validate:

1. **component** - process, pod, interface, disk, certificate;
2. **service** - API/transaction/cluster health;
3. **customer** - the original user-visible symptom is gone.

Do not declare recovery because a process is merely `active (running)`.

## 7. Close the loop

Capture:

- root cause and contributing factors
- why monitoring did/did not detect it earlier
- permanent corrective action
- runbook/automation changes
- owner and due date

The incident is not complete until the next responder is less likely to repeat the same investigation.
