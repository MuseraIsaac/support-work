# Analysis: YARN NodeManager reported 12/12 local directories as unwritable

## Executive summary

A Hadoop worker reported every configured NodeManager local directory as unhealthy. The initial symptom looked like a storage problem because the health report stated that all 12 local directories were "not writable." Disk availability, however, was not the differentiator. The useful clue came from comparing identity and ownership metadata on the affected host with healthy peers.

The affected directories were owned by numeric UID/GID values that resolved to the wrong local identities. Healthy nodes resolved the same service UID to `yarn` and the expected group to `hadoop`; the affected node resolved those numbers differently. The result was a permissions failure even though the directory mode bits appeared superficially reasonable.

The durable fix was to align the service identities and directory ownership with the healthy cluster nodes, then restart/revalidate NodeManager health.

## Context

The NodeManager used local directories similar to:

```text
/disk1/yarn/nm
/disk2/yarn/nm
...
/disk12/yarn/nm
```

The cluster health report showed all 12 paths as unwritable.

## Observations

I began by treating the message as a symptom rather than a diagnosis.

Questions I wanted to answer:

1. Are the filesystems mounted and writable at all?
2. Is the problem capacity, inode exhaustion, mount options, SELinux, or Unix permissions?
3. Does the NodeManager service run as the identity I expect?
4. How does the unhealthy node differ from a healthy peer?

Representative checks:

```bash
findmnt /disk1
findmnt /disk9

df -h /disk1 /disk9
df -i /disk1 /disk9

namei -l /disk9/yarn/nm
stat -c '%u:%g %U:%G %a %n' /disk9/yarn/nm

id yarn
getent passwd yarn
getent group hadoop
```

## Healthy vs affected comparison

On a healthy worker, the NodeManager directories resolved to the expected service identities:

```text
owner: yarn
group: hadoop
```

On the affected worker, one of the same paths showed numeric ownership equivalent to:

```text
uid 982
gid 976
```

but those values did not resolve to the same service identities as on the healthy nodes. In particular, the ownership/group mapping had drifted and the NodeManager process no longer had the effective access expected by the cluster configuration.

This comparison changed the working hypothesis from "twelve disks are independently unwritable" to "one host-level identity/ownership condition is affecting all twelve paths."

That hypothesis better explained the blast radius: twelve independent disk failures at once were less likely than one common permission condition.

## Resolution approach

Before changing anything, I recorded the UID/GID mapping and ownership on healthy peers. The repair was then constrained to making the affected node consistent with the known-good state.

Representative remediation pattern:

```bash
# Verify the intended service identities first.
id yarn
getent group hadoop

# Correct ownership only after confirming the expected UID/GID mapping.
chown -R yarn:hadoop /disk*/yarn/nm
```

Where identity databases themselves differ, the safer order is to correct the service-account UID/GID mapping first and only then correct ownership. Blind recursive `chown` is not a substitute for understanding the identity mismatch.

After correction, I revalidated access as the service account and then checked NodeManager health:

```bash
sudo -u yarn test -w /disk9/yarn/nm && echo writable
sudo -u yarn touch /disk9/yarn/nm/.support-write-test
rm -f /disk9/yarn/nm/.support-write-test
```

Then I restarted or allowed the NodeManager to re-evaluate local directory health according to the environment's operational procedure.

## Why the analysis mattered

The literal error message pointed at storage paths. The root cause lived one layer away in Linux identity and ownership. Comparing a failing node with a healthy peer reduced the search space much faster than repeatedly changing permissions on individual disks.

The broader lesson I carried forward was to ask what single condition can explain the entire blast radius before treating each failed component independently.
