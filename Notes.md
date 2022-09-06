# Notes

## Setup

- X1850 Mixer: `192.168.86.185` (PC port connected to LAN)
- SC6000 Player 1: `192.168.86.201` (Port 2 on mixer)
- SC6000 Player 2: `192.168.86.202` (Port 3 on mixer)
- Music is on a 1TB internal SSD inside Player 1
- Mixer and players get IPs via DHCP but are set with reserved IPs on the router.

---

## Reconnecting to Player After Rebooting

The `StageLinqListener` is listening for messages from players. When we get a
`DISCOVERER_HOWDY_` messages, we will try to connect to it and track the
IP/Port so we don't try to connect to it again when we see the next
`DISCOVERY_HOWDY_` message.

```
Discovery packet from the SC6000 looks like this:
UDP 192.168.86.201 => 192.168.86.255

0000   ff ff ff ff ff ff 00 05 95 03 65 ec 08 00 45 00   ..........e...E.
0010   00 82 65 02 40 00 40 11 bd f6 c0 a8 56 ca ff ff   ..e.@.@.....V...
0020   ff ff c5 28 c8 89 00 6e ae 4b 61 69 72 44 82 8b   ...(...n.KairD..
0030   eb 02 da 1f 4e 68 a6 af b0 b1 67 ea f0 a2 00 00   ....Nh....g.....
0040   00 0c 00 73 00 63 00 36 00 30 00 30 00 30 00 00   ...s.c.6.0.0.0..
0050   00 22 00 44 00 49 00 53 00 43 00 4f 00 56 00 45   .".D.I.S.C.O.V.E
0060   00 52 00 45 00 52 00 5f 00 48 00 4f 00 57 00 44   .R.E.R._.H.O.W.D
0070   00 59 00 5f 00 00 00 08 00 4a 00 50 00 31 00 33   .Y._.....J.P.1.3
0080   00 00 00 0a 00 32 00 2e 00 33 00 2e 00 31 88 e3   .....2...3...1..
```

```ts
// We track the device like this
private deviceId(device: ConnectionInfo) {
  return `${device.address}:${device.port}:` +
    `[${device.source}/${device.software.name}]`;
}

// After successully connecting to the player ...
Logger.info(`Successfully connected to ${this.deviceId(connectionInfo)}`);
this.discoveryStatus.set(this.deviceId(connectionInfo), ConnectionStatus.CONNECTED);

// And when any discover packets come in, we ignore ones we've seen ...
if (this.isConnected(connectionInfo)
  || this.isConnecting(connectionInfo)
  || this.isIgnored(connectionInfo)) return;
```

(Note: We're tracking the IP, port ... along with source and software name.)

It's important to track the IP *and* the port because every time the player
reboots it will pick a random new port. When you reboot the player it will
send another `DISCOVERY_HOWDY_` message with the same IP but new port.

After a player reboot the `StageLinqListener` will see an IP and port we
haven't connected to and connect it as if it were a newly discovered device on
the network. Meaning `StageLinqListener` will automatically reconnect to
the player after you reboot it!
