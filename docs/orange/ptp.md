# PTP setup

PTP is required for sub-frame synchronization across cameras. Two setups are supported:

- **Host-only**, using `linuxptp` to sync NIC ports on a single host (cameras attached directly to a NIC).
- **Switched**, using a PTP-capable switch (Arista and Mellanox/Onyx examples below).

## Host-only PTP

### 1. Install linuxptp

```bash
sudo apt install linuxptp
```

### 2. Set boundary clock

If you don't have a `ptp4l.conf` file, create `/etc/ptp4l.conf`:

```
[global]
verbose 1
boundary_clock_jbod 1
```

### 3. Synchronize NICs

If you only have one NIC card:

```bash
sudo ptp4l -i enp66s0f0np0 -i enp66s0f1np1 -f /etc/ptp4l.conf
```

This syncs the ports of the NIC. `enp66s0f0np0` and `enp66s0f1np1` are example port names — change them to your port names.

For multiple NICs, sync all ports across all NICs:

```bash
sudo ptp4l -i enp66s0f0np0 -i enp66s0f1np1 -i enp97s0f0np0 -i enp97s0f1np1 -f /etc/ptp4l.conf
```

In another terminal, run:

```bash
sudo phc2sys -a -rr -m
```

This syncs the computer time and treats it as a time source.

## Switched PTP

If you're using a network switch, enable PTP on all the ports connected to either cameras or host NICs. Examples below for Arista and Mellanox (Onyx) switches.

### Arista

Once logged in:

```
en
conf t
show ptp
```

If PTP is disabled:

```
ptp mode boundary
```

Configure each port:

```
show interface
```

Example: enable PTP on port 5

```
interface Ethernet 5
ptp enable
```

Multiple ports:

```
interface Ethernet49/1-Ethernet60/1
ptp enable
```

```
interface Ethernet1-Ethernet48
ptp enable
```

```
show int sta
```

If a port is in `errdisabled`, restart it:

```
interface Ethernet ##
shutdown
no shutdown
```

To disable spanning tree:

```
spanning-tree bpduguard disable
```

### Mellanox (Onyx)

Reference: [MLNX-OS Ethernet Command Reference Guide](https://safe.nrao.edu/wiki/pub/Beamformer/DDLTestingCommModeOne/MLNX-OS_SW_Eth_3.2.0506_Command_Reference_Guide.pdf).

Useful commands:

```
show interface ethernet 1/11
```

```
enable
configure terminal
interface ethernet 1/11
shutdown
mtu 9216
speed 25000
fec rs-fec
no shutdown
```

To configure multiple ports:

```
interface ethernet 1/1-1/16 shutdown
interface ethernet 1/1-1/16 mtu 9216
interface ethernet 1/1-1/16 fec rs-fec
interface ethernet 1/1-1/16 speed 25G
interface ethernet 1/1-1/16 no shutdown
```
