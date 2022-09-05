# Announcement

In order to talk to StageLinq every device seems to need to announce themselves
on the StageLinq network by broadcastig a UDP announce packet to the network.

| Bytes | Description
| ----  | ------
| 4     | Discovery message marker ("airD"): `61 69 72 44`
| 16    | Client token: `52 fd fc 07 21 82 65 4f 16 3f 5f 0f 9a 62 1d 72`
| 4     | `00 00 00 14`
| *n*   | Source string: `nowplaying`
| 4     | `00 00 00 22`
| *n*   | Action (`DISCOVERER_HOWDY_`)
| 4     | `00 00 00 16`
| *n*   | Software name `Now Playing`
| 4     | `00 00 00 0a`
| *n*   | Software version `2.1.3`
| 2     | `00 00`


```
Full UDP Announcement Packet

0000   ff ff ff ff ff ff 3c cd 36 65 e8 5b 08 00 45 00   ......<.6e.[..E.
0010   00 98 86 57 00 00 40 11 c5 8c c0 a8 56 21 c0 a8   ...W..@.....V!..
0020   56 ff f7 a5 c8 89 00 84 2a a6 61 69 72 44 52 fd   V.......*.airDR.
0030   fc 07 21 82 65 4f 16 3f 5f 0f 9a 62 1d 72 00 00   ..!.eO.?_..b.r..
0040   00 14 00 6e 00 6f 00 77 00 70 00 6c 00 61 00 79   ...n.o.w.p.l.a.y
0050   00 69 00 6e 00 67 00 00 00 22 00 44 00 49 00 53   .i.n.g...".D.I.S
0060   00 43 00 4f 00 56 00 45 00 52 00 45 00 52 00 5f   .C.O.V.E.R.E.R._
0070   00 48 00 4f 00 57 00 44 00 59 00 5f 00 00 00 16   .H.O.W.D.Y._....
0080   00 4e 00 6f 00 77 00 20 00 50 00 6c 00 61 00 79   .N.o.w. .P.l.a.y
0090   00 69 00 6e 00 67 00 00 00 0a 00 32 00 2e 00 31   .i.n.g.....2...1
00a0   00 2e 00 33 00 00                                 ...3..

```

```ts
function writeDiscoveryMessage(p_ctx: WriteContext, p_message: DiscoveryMessage): number {
	let written = 0;
	written += p_ctx.writeFixedSizedString(DISCOVERY_MESSAGE_MARKER);
	written += p_ctx.write(p_message.token);
	written += p_ctx.writeNetworkStringUTF16(p_message.source);
	written += p_ctx.writeNetworkStringUTF16(p_message.action);
	written += p_ctx.writeNetworkStringUTF16(p_message.software.name);
	written += p_ctx.writeNetworkStringUTF16(p_message.software.version);
	written += p_ctx.writeUInt16(p_message.port);
	return written;
}
```