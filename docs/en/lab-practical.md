# Prepare the VLAN practical

Exercise 7 changes one endpoint, two switches and the gateway in each student's own lab. Prepare a checkpoint of that lab's flat baseline for recovery. No additional appliance is required for this exercise.

## Before students begin

1. Use the student's existing `ot-sec-flat` installation and its assigned access URL. Keep its credentials and private checkpoint with that appliance.
2. Open the lab's Exercises reader and select its live network. Verify independent terminals to `hirschmann-edge1-vlan50`, `wago-plc2a-vlan10`, `sw-acc1`, `sw-dist` and `gw-firewall`. `hirschmann-edge1-vlan50` must have management on `eth0` and `10.50.0.12/8` on `eth1`.
3. Verify the starting addresses, VLAN membership and rule anchors in Exercise 7 Task 1. VLAN 70 and `172.30.70.0/24` must be unused. From `hirschmann-edge1-vlan50`, `wago-plc2a-vlan10` TCP 22 and 8080 must be open and `/webvisu.htm` must return HTTP 200. From `wago-plc2a-vlan10`, `hirschmann-edge1-vlan50` TCP 22 must be open. Resolve a failed baseline check before proceeding.
4. Close captures and terminal tabs. On the **student lab's appliance console**, create a private checkpoint with the installed lifecycle command:

    ```bash
    sudo nwos-ot-student checkpoint ex7-flat
    sudo nwos-ot-student start
    sudo nwos-ot-student status
    ```

    Checkpoint stops the owned lab; `start` brings it back. Use a fresh checkpoint name if `ex7-flat` already exists, record it, and replace `ex7-flat` with that same name in the restore command below. Wait until the platform is healthy and the node terminals and baseline probes work again.
5. Record the checkpoint name for that student's lab. Keep instructor console access available for recovery.

## Finish and reset

Students complete the lesson's manual cleanup after the two optional extras and repeat both directions of the baseline tests. The lesson's Linux address changes are temporary; RouterOS changes persist. A restart during the lesson can therefore leave a mixed state. Restore the whole checkpoint if a step fails or a device restarts.

On the **same student lab's appliance console**:

```bash
sudo nwos-ot-student restore ex7-flat
sudo nwos-ot-student status
```

Restore stops and replaces the owned installation state and starts the lab. Wait for healthy services, reopen the reader and repeat Task 1. Confirm `hirschmann-edge1-vlan50` is back at `10.50.0.12/8`, VLAN 70 and `ex7-` rules are absent, HTTP returns 200 and both baseline SSH probes are open. RAM logs are not rolled back; match test evidence by time rather than assuming old log entries disappear.

Keep checkpoints root-owned, mode `0600`, outside Git and distribution artifacts: they include private installation data. The lifecycle command needs the matching installed release and its locked images. A failed restore needs instructor console diagnostics before more student changes.

The reader provides terminals, capture and VLAN views. It does not create or reset an appliance for the student. Toolbox saves a design; it does not apply this VLAN configuration.
