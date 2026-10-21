# Automation outlook: detect configuration drift

A replacement PLC was connected to the wrong VLAN after maintenance. Automation should compare what the port is meant to do with its actual state, then make a proposed correction reviewable. A successful automation job alone does not prove that the PLC works.

**Optional, 15 minutes after the main exercises.** Reuse Exercise 1's switch-port evidence. No software installation or device changes are needed.

## Define the intended result

For sw-acc1 ether2, write the desired facts separately from your observations:

| Intended fact | Evidence to compare |
| --- | --- |
| The port serves `wago-plc2a-vlan10` | Port description, topology and endpoint identity |
| Untagged endpoint traffic enters VLAN 10 | PVID and actual untagged membership |
| The path carries VLAN 10 to the gateway | Uplink membership and tagged ping capture |
| The application is available | A successful, approved application request |

On **sw-acc1's configured terminal**, collect the first two configuration observations:

```routeros
/interface bridge port print detail where interface=ether2
/interface bridge vlan print detail
```

A matching baseline is a valid outcome: report “no change needed”. On paper, suppose the first output instead showed PVID 20. Describe which fact would differ, which traffic could be affected and why a blanket reset of all switch ports would be a poor correction.

## Write a reviewable change proposal

Record the exact target, observed old value, intended value, reason, owner, expected impact, application check and rollback. Require the automation to refuse the change if the observed old value has changed since review. Separate reading state, previewing a diff, applying an approved change and verifying its outcome.

**Checkpoint:** Your proposal manages only the intended field, preserves unrelated settings and includes application evidence after the change. Repeating an already-correct operation should make no further change; this is **idempotence**.

The handout's automation cycle is collect actual configuration → compare intended configuration → validate → apply. This exercise prepares that specification; it does not deploy an automation tool or claim to have executed an automated correction. Keep credentials and configuration backups in the instructor's designated private storage, outside course notes and Git.

Continue with the [exercise overview](exercises/index.md).
