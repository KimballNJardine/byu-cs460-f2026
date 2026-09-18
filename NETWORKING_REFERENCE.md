# CS 460 Networking Reference

This is a consolidated, editable reference for the packet structures and implementation helps used throughout the course. The diagrams describe the bytes handled by the labs, not every field that appears on a physical network.

## 1. Encapsulation at a Glance

When application data travels between hosts, the layers wrap it from the inside out:

```text
Ethernet frame
  +-- optional 802.1Q VLAN tag
  +-- IPv4 header
        +-- UDP header + application data
        +-- TCP header + application data
```

At the receiving side, the headers are removed in the reverse order. A router normally removes and examines the Ethernet frame, processes the IPv4 header, then creates a new Ethernet frame for the next interface. The IP payload remains the transport segment unless the packet is delivered locally.

| Layer | Unit | Main addressing | Payload |
| --- | --- | --- | --- |
| Application | data/message | application-defined | user data |
| Transport | UDP datagram or TCP segment | source/destination ports | application data |
| Network | IPv4 datagram | source/destination IP addresses | UDP/TCP/other protocol |
| Link | Ethernet frame | source/destination MAC addresses | IPv4, ARP, or another EtherType |

## 2. Bytes and Network Order

The labs exchange raw Python `bytes` values. A `bytes` value is a sequence of byte values, not a text string.

```python
frame = destination_mac + source_mac + ethertype + payload
first_two = frame[:2]
field = frame[12:14]
```

- Slicing returns a new `bytes` value.
- Byte strings can be concatenated with `+`.
- Multi-byte protocol fields use network byte order, which is big-endian.
- Python's `struct` format prefix for network order is `!`; for example, `struct.pack('!H', port)` packs a 16-bit unsigned integer.
- Convert presentation strings and wire bytes with the provided address helpers rather than treating an IP or MAC string as its wire representation.

## 3. Ethernet and VLAN Frames

### Ethernet frame

The raw Ethernet frame received by these labs is:

| Field | Size | Description |
| --- | ---: | --- |
| Destination MAC address | 6 bytes | Intended receiver; `ff:ff:ff:ff:ff:ff` is broadcast |
| Source MAC address | 6 bytes | Sender on the local link |
| EtherType | 2 bytes | Identifies the payload protocol |
| Payload | variable | Usually an IPv4 datagram or ARP packet |

Common EtherTypes:

| EtherType | Meaning |
| --- | --- |
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x8100` | 802.1Q VLAN tag indicator |

The physical Ethernet frame also has a preamble and CRC. Raw sockets used in the lab provide neither of those fields to the application.

### 802.1Q VLAN tagging

When a VLAN tag is present, the order is:

| Destination MAC | Source MAC | 802.1Q header | EtherType | Payload |
| --- | --- | --- | --- | --- |
| 6 bytes | 6 bytes | 4 bytes | 2 bytes | variable |

The 32-bit 802.1Q header used in the lab is simplified:

- Most significant 16 bits: `0x8100`.
- Least significant 12 bits: VLAN ID.
- Four bits between them: zero in the lab.

Switch behavior to remember:

- Learn the source MAC address on the incoming interface and VLAN.
- Broadcast frames go to other eligible interfaces in the same VLAN.
- Known unicast frames go only to the learned destination interface when that entry is valid.
- An access interface commonly receives or sends untagged frames for one VLAN.
- A trunk interface carries tagged frames and may add or remove the tag at the boundary.

The following diagram uses 16-bit columns to show the bit widths of the Ethernet fields. The first row is the ordinary frame; the second row shows the tagged form.

<table border="1">
<tr><th>00</th><th>16</th><th>32</th><th>48</th><th>64</th><th>80</th><th>96</th><th>112</th><th>128</th></tr>
<tr><td colspan="3">Destination MAC address</td><td colspan="3">Source MAC address</td><td>EtherType</td></tr>
<tr><td colspan="3">Destination MAC address</td><td colspan="3">Source MAC address</td><td colspan="2">802.1Q header</td><td>EtherType</td></tr>
</table>

The ordinary frame has 112 bits before its payload. The VLAN-tagged frame has 144 bits before its payload.

#### Worked VLAN frame example

Example values: destination MAC `02:00:00:00:00:02`, source MAC `02:00:00:00:00:01`, VLAN ID `25`, and IPv4 EtherType `0x0800`.

Bits for each field:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><th colspan="32">Destination MAC (bits 00-31)</th><th colspan="16">Destination MAC (bits 32-47)</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
<tr><th colspan="32">Source MAC (bits 00-31)</th><th colspan="16">Source MAC (bits 32-47)</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="32">802.1Q header</th></tr>
<tr><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="16">EtherType</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
</table>

Bytes for each field (network byte order is big-endian):

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="4">Destination MAC address (bytes 0-3)</th></tr>
<tr><td>02</td><td>00</td><td>00</td><td>00</td></tr>
<tr><th colspan="2">Destination MAC address (bytes 4-5)</th><th colspan="2">Source MAC address (bytes 0-1)</th></tr>
<tr><td>00</td><td>02</td><td>02</td><td>00</td></tr>
<tr><th colspan="4">Source MAC address (bytes 2-5)</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>01</td></tr>
<tr><th colspan="4">802.1Q header</th></tr>
<tr><td>81</td><td>00</td><td>00</td><td>19</td></tr>
<tr><th colspan="2">EtherType</th></tr>
<tr><td>08</td><td>00</td></tr>
</table>

Bytes, in network byte order (big-endian):

```text
02 00 00 00 00 02 02 00 00 00 00 01 81 00 00 19 08 00
```

Expansion:

```text
02 00 00 00 00 02 -> destination MAC = 02:00:00:00:00:02
02 00 00 00 00 01 -> source MAC      = 02:00:00:00:00:01
81 00 00 19       -> tag type 0x8100, VLAN ID = 25
08 00             -> EtherType = 0x0800 = IPv4
```

## 4. ARP Packets

ARP maps an IPv4 address to a MAC address on the local link. Its layout is:

| Field | Size in this course | Description |
| --- | ---: | --- |
| Hardware type | 2 bytes | Ethernet: `ARPHRD_ETHER = 1` |
| Protocol type | 2 bytes | IPv4: `ETH_P_IP = 0x0800` |
| Hardware address length | 1 byte | Ethernet MAC length: 6 |
| Protocol address length | 1 byte | IPv4 address length: 4 |
| Opcode | 2 bytes | Request `1`, reply `2` |
| Sender hardware address | 6 bytes | Sender MAC |
| Sender protocol address | 4 bytes | Sender IPv4 address |
| Target hardware address | 6 bytes | Target MAC; may be zero in a request |
| Target protocol address | 4 bytes | Target IPv4 address |
| Data | variable | Not needed for the basic lab |

Typical exchange:

1. The sender broadcasts an ARP request asking who owns the target IP.
2. The target sends an ARP reply containing its MAC address.
3. The sender caches the IP-to-MAC mapping and uses it to build the Ethernet frame.

Bit-level ARP layout, using the same 32-bit rows as the lab README:

<table border="1">
<tr>
<th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th>
<th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th>
<th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th>
<th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="16">Hardware type</td><td colspan="16">Protocol type</td></tr>
<tr><td colspan="8">Hardware address length</td><td colspan="8">Protocol address length</td><td colspan="16">Opcode</td></tr>
<tr><td colspan="32">Sender hardware address</td></tr>
<tr><td colspan="32">Sender protocol address</td></tr>
<tr><td colspan="32">Target hardware address</td></tr>
<tr><td colspan="32">Target protocol address</td></tr>
<tr><td colspan="32">Data</td></tr>
</table>

#### Worked ARP request example

Example values: Ethernet/IPv4 request, sender MAC `02:00:00:00:00:01`, sender IP `192.0.2.1`, target MAC all zeroes, and target IP `192.0.2.2`.

Bits for each field:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><th colspan="16">Hardware type</th><th colspan="16">Protocol type</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="8">Hardware address length</th><th colspan="8">Protocol address length</th><th colspan="16">Opcode</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="48">Sender hardware address</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="32">Sender protocol address</th></tr>
<tr><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="48">Target hardware address</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="32">Target protocol address</th></tr>
<tr><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
</table>

Bytes for each field (all multi-byte fields are in network byte order, big-endian):

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="2">Hardware type</th><th colspan="2">Protocol type</th></tr>
<tr><td>00</td><td>01</td><td>08</td><td>00</td></tr>
<tr><th colspan="1">Hardware address length</th><th colspan="1">Protocol address length</th><th colspan="2">Opcode</th></tr>
<tr><td>06</td><td>04</td><td>00</td><td>01</td></tr>
<tr><th colspan="4">Sender hardware address (bytes 0-3)</th></tr>
<tr><td>02</td><td>00</td><td>00</td><td>00</td></tr>
<tr><th colspan="2">Sender hardware address (bytes 4-5)</th><th colspan="2">Sender protocol address (bytes 0-1)</th></tr>
<tr><td>00</td><td>01</td><td>c0</td><td>00</td></tr>
<tr><th colspan="2">Sender protocol address (bytes 2-3)</th><th colspan="2">Target hardware address (bytes 0-1)</th></tr>
<tr><td>02</td><td>01</td><td>00</td><td>00</td></tr>
<tr><th colspan="4">Target hardware address (bytes 2-5)</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>00</td></tr>
<tr><th colspan="4">Target protocol address</th></tr>
<tr><td>c0</td><td>00</td><td>02</td><td>02</td></tr>
</table>

Bytes, in network byte order (big-endian):

```text
00 01 08 00 06 04 00 01 02 00 00 00 00 01 c0 00
02 01 00 00 00 00 00 00 00 00 00 00 c0 00 02 02
```

Expansion:

```text
00 01       -> hardware type = 1 = Ethernet
08 00       -> protocol type = 0x0800 = IPv4
06          -> hardware address length = 6 bytes
04          -> protocol address length = 4 bytes
00 01       -> opcode = 1 = request
02 00 00 00 00 01 -> sender MAC = 02:00:00:00:00:01
c0 00 02 01       -> sender IP  = 192:000:002:001
00 00 00 00 00 00 -> target MAC = 00:00:00:00:00:00 (unknown)
c0 00 02 02       -> target IP  = 192:000:002:002
```

## 5. IPv4 Datagrams

An IPv4 datagram consists of an IPv4 header followed by its payload. The minimum IPv4 header is 20 bytes.

| Field | Size | Notes |
| --- | ---: | --- |
| Version | 4 bits | IPv4 value is 4 |
| IHL | 4 bits | Header length in 32-bit words; 5 means 20 bytes |
| DSCP/ECN | 1 byte | Usually not used in these labs |
| Total length | 2 bytes | IPv4 header plus payload |
| Identification | 2 bytes | Fragmentation support |
| Flags | 3 bits | Fragmentation control |
| Fragment offset | 13 bits | Fragmentation support |
| TTL | 1 byte | Decremented by each router |
| Protocol | 1 byte | Identifies the next payload protocol |
| Header checksum | 2 bytes | Covers the IPv4 header |
| Source address | 4 bytes | Sender IPv4 address |
| Destination address | 4 bytes | Receiver IPv4 address |
| Options and padding | variable | Included only when IHL is greater than 5 |
| Payload | variable | UDP, TCP, ICMP, or another protocol |

Bit-level IPv4 header layout for the minimum 20-byte header:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="4">Version</td><td colspan="4">IHL</td><td colspan="8">DSCP/ECN</td><td colspan="16">Total length</td></tr>
<tr><td colspan="16">Identification</td><td colspan="3">Flags</td><td colspan="13">Fragment offset</td></tr>
<tr><td colspan="8">TTL</td><td colspan="8">Protocol</td><td colspan="16">Header checksum</td></tr>
<tr><td colspan="32">Source address</td></tr>
<tr><td colspan="32">Destination address</td></tr>
<tr><td colspan="32">Options and padding :::</td></tr>
</table>

#### Worked IPv4 datagram example

Example values: no options, total length `33` bytes, identification `0x1234`, do-not-fragment flag set, TTL `64`, UDP protocol `17`, source `192.0.2.1`, and destination `192.0.2.2`. The checksum below is shown as zero to keep the example focused on layout.

Bits for each field:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><th colspan="4">Version</th><th colspan="4">IHL</th><th colspan="8">DSCP/ECN</th><th colspan="16">Total length</th></tr>
<tr><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="16">Identification</th><th colspan="3">Flags</th><th colspan="13">Fragment offset</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="8">TTL</th><th colspan="8">Protocol</th><th colspan="16">Header checksum</th></tr>
<tr><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="32">Source address</th><th colspan="32">Destination address</th></tr>
<tr><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
</table>

Bytes for each field (multi-byte fields are in network byte order, big-endian):

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="1">Version/IHL</th><th colspan="1">DSCP/ECN</th><th colspan="2">Total length</th></tr>
<tr><td>45</td><td>00</td><td>00</td><td>21</td></tr>
<tr><th colspan="2">Identification</th><th colspan="2">Flags/fragment</th></tr>
<tr><td>12</td><td>34</td><td>40</td><td>00</td></tr>
<tr><th colspan="1">TTL</th><th colspan="1">Protocol</th><th colspan="2">Header checksum</th></tr>
<tr><td>40</td><td>11</td><td>00</td><td>00</td></tr>
<tr><th colspan="4">Source address</th></tr>
<tr><td>c0</td><td>00</td><td>02</td><td>01</td></tr>
<tr><th colspan="4">Destination address</th></tr>
<tr><td>c0</td><td>00</td><td>02</td><td>02</td></tr>
</table>

Bytes, in network byte order (big-endian):

```text
45 00 00 21 12 34 40 00 40 11 00 00 c0 00 02 01 c0 00 02 02
```

Expansion:

```text
45       -> version = 4, IHL = 5, header length = 20 bytes
00       -> DSCP/ECN = 0
00 21    -> total length = 33 bytes
12 34    -> identification = 0x1234
40 00    -> flags = do not fragment, fragment offset = 0
40       -> TTL = 64
11       -> protocol = 17 = UDP
00 00    -> header checksum shown as 0 in this example
c0 00 02 01 -> source IP      = 192:000:002:001
c0 00 02 02 -> destination IP = 192:000:002:002
```

## 7. UDP Datagrams

The UDP header is 8 bytes and is followed by application data.

| Field | Size | Description |
| --- | ---: | --- |
| Source port | 2 bytes | Sending application port |
| Destination port | 2 bytes | Receiving application port |
| Length | 2 bytes | UDP header plus UDP payload |
| Checksum | 2 bytes | Set to zero in the transport lab |
| Data | variable | Application payload |

Bit-level UDP header layout:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="16">Source port</td><td colspan="16">Destination port</td></tr>
<tr><td colspan="16">Length</td><td colspan="16">Checksum</td></tr>
</table>

#### Worked UDP datagram example

Example values: source port `4000`, destination port `1234`, payload `hello` (5 bytes), length `13`, and checksum `0`.

Bits for each field:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><th colspan="16">Source port</th><th colspan="16">Destination port</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
<tr><th colspan="16">Length</th><th colspan="16">Checksum</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="40">Data</th></tr>
<tr><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td></tr>
</table>

Bytes for each field (multi-byte fields are in network byte order, big-endian):

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="2">Source port</th><th colspan="2">Destination port</th></tr>
<tr><td>0f</td><td>a0</td><td>04</td><td>d2</td></tr>
<tr><th colspan="2">Length</th><th colspan="2">Checksum</th></tr>
<tr><td>00</td><td>0d</td><td>00</td><td>00</td></tr>
<tr><th colspan="4">Data (bytes 0-3)</th></tr>
<tr><td>68</td><td>65</td><td>6c</td><td>6c</td></tr>
<tr><th colspan="1">Data (byte 4)</th></tr>
<tr><td>6f</td></tr>
</table>

Bytes, in network byte order (big-endian):

```text
0f a0 04 d2 00 0d 00 00 68 65 6c 6c 6f
```

Expansion:

```text
0f a0 -> source port = 4000
04 d2 -> destination port = 1234
00 0d -> UDP length = 13 bytes = 8-byte header + 5-byte data
00 00 -> checksum = 0 in this lab
68 65 6c 6c 6f -> data = h e l l o
```

## 8. TCP Segments

The TCP header is at least 20 bytes and is followed by application data. TCP fields are shown in 32-bit rows in the lab documentation.

| Field | Size | Description |
| --- | ---: | --- |
| Source port | 2 bytes | Sending application port |
| Destination port | 2 bytes | Receiving application port |
| Sequence number | 4 bytes | Position of segment data in the byte stream |
| Acknowledgment number | 4 bytes | Next byte expected by the sender of the ACK |
| Data offset | 4 bits | Header length in 4-byte words; 5 means 20 bytes |
| Reserved | 3 bits | Zero in the lab |
| ECN | 3 bits | Not used in the lab |
| Control bits | 6 bits | `URG`, `ACK`, `PSH`, `RST`, `SYN`, `FIN` |
| Window | 2 bytes | Advertised receive window; 64 is used as a reasonable lab value |
| Checksum | 2 bytes | Set to zero in the transport lab |
| Urgent pointer | 2 bytes | Not used in the lab |
| Options and padding | variable | Makes the header a multiple of 4 bytes |
| Data | variable | Application payload |

Bit-level TCP header layout for the minimum 20-byte header:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="16">Source port</td><td colspan="16">Destination port</td></tr>
<tr><td colspan="32">Sequence number</td></tr>
<tr><td colspan="32">Acknowledgment number</td></tr>
<tr><td colspan="4">Data offset</td><td colspan="3">Reserved</td><td colspan="3">ECN</td><td colspan="6">Control bits</td><td colspan="16">Window</td></tr>
<tr><td colspan="16">Checksum</td><td colspan="16">Urgent pointer</td></tr>
<tr><td colspan="32">Options and padding :::</td></tr>
</table>

#### Worked TCP segment example

Example values: source port `4000`, destination port `1234`, sequence `1`, acknowledgment `1`, data offset `5`, flags `ACK+SYN`, window `64`, checksum `0`, urgent pointer `0`, and data `hello`.

Bits for each field:

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><th colspan="16">Source port</th><th colspan="16">Destination port</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
<tr><th colspan="32">Sequence number</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="32">Acknowledgment number</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td></tr>
<tr><th colspan="4">Data offset</th><th colspan="3">Reserved</th><th colspan="3">ECN</th><th colspan="6">Control bits</th><th colspan="16">Window</th></tr>
<tr><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="16">Checksum</th><th colspan="16">Urgent pointer</th></tr>
<tr><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td></tr>
<tr><th colspan="40">Data</th></tr>
<tr><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td></tr>
</table>

Bytes for each field (multi-byte fields are in network byte order, big-endian):

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="2">Source port</th><th colspan="2">Destination port</th></tr>
<tr><td>0f</td><td>a0</td><td>04</td><td>d2</td></tr>
<tr><th colspan="4">Sequence number</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>01</td></tr>
<tr><th colspan="4">Acknowledgment number</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>01</td></tr>
<tr><th colspan="4">Data offset / Reserved / ECN / Control bits / Window</th></tr>
<tr><td>50</td><td>92</td><td>00</td><td>40</td></tr>
<tr><th colspan="2">Checksum</th><th colspan="2">Urgent pointer</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>00</td></tr>
<tr><th colspan="4">Data (bytes 0-3)</th></tr>
<tr><td>68</td><td>65</td><td>6c</td><td>6c</td></tr>
<tr><th colspan="1">Data (byte 4)</th></tr>
<tr><td>6f</td></tr>
</table>

Header bytes, in network byte order (big-endian), followed by the data bytes:

```text
0f a0 04 d2 00 00 00 01 00 00 00 01 50 92 00 40
00 00 00 00 00 00 68 65 6c 6c 6f
```

Expansion:

```text
0f a0 -> source port = 4000
04 d2 -> destination port = 1234
00 00 00 01 -> sequence number = 1
00 00 00 01 -> acknowledgment number = 1
50 -> data offset = 5 words = 20 bytes; reserved/ECN = 0
92 -> control bits = ACK + SYN
00 40 -> window = 64 bytes
00 00 -> checksum = 0 in this lab
00 00 -> urgent pointer = 0
68 65 6c 6c 6f -> data = h e l l o
```

## 11. Full Example

Suppose host `10.0.0.1` sends `b'hello'` from UDP port `4000` to port `1234` at `10.0.0.2`. Use source MAC `02:00:00:00:00:01` and destination MAC `02:00:00:00:00:02` for this example.

### Step 1: Application data

The application starts with five data bytes. No network header exists yet.

```text
+-----------------------+
| Application data      |
| b'hello'              |
+-----------------------+
```

The data bytes are:

```text
68 65 6c 6c 6f -> h  e  l  l  o
```

### Step 2: Add the UDP header

The UDP layer adds source and destination ports, the datagram length, and the checksum field around the application data.

```text
+----------------+------------------+--------+----------+----------------+
| Source port    | Destination port | Length | Checksum | Application    |
| 4000           | 1234             | 13     | 0        | b'hello'       |
+----------------+------------------+--------+----------+----------------+
|       2 bytes  |          2 bytes | 2 bytes| 2 bytes  | 5 bytes        |
```

Field bytes, in network byte order (big-endian):

```text
Source port       0f a0
Destination port  04 d2
Length            00 0d
Checksum          00 00
Data              68 65 6c 6c 6f
```

The resulting UDP datagram is 8-byte header plus 5-byte data, or 13 bytes total.

### Step 3: Add the IPv4 header

The IP layer adds addresses and delivery information in front of the complete UDP datagram. The IPv4 protocol field is `17`, identifying UDP.

```text
+----------------------+------------------------------------------+
| IPv4 header          | IPv4 payload                             |
| 20 bytes             | UDP header + b'hello'                    |
+----------------------+------------------------------------------+
| Version/IHL: 4/5    | Source port: 4000                       |
| Total length: 33    | Destination port: 1234                  |
| TTL: 64             | Length: 13, Checksum: 0, Data: b'hello'|
| Protocol: 17 (UDP)  |                                          |
| Source: 10.0.0.1    |                                          |
| Destination: 10.0.0.2|                                         |
+----------------------+------------------------------------------+
```

IPv4 header bytes, in network byte order (big-endian; checksum is simplified to zero here):

```text
45 00 00 21 00 00 40 00 40 11 00 00 0a 00 00 01 0a 00 00 02
```

The IPv4 datagram is 20-byte header plus 13-byte UDP datagram, or 33 bytes total.

### Step 4: Add the Ethernet frame

Before transmission, the link layer selects the next-hop MAC address. If the mapping is not cached, ARP obtains it. The Ethernet frame then wraps the entire IPv4 datagram.

```text
+----------------------+----------------------+----------+----------------------+
| Destination MAC      | Source MAC           | EtherType| IPv4 payload         |
| 02:00:00:00:00:02   | 02:00:00:00:00:01   | 0x0800  | 33-byte IPv4 datagram|
+----------------------+----------------------+----------+----------------------+
| 6 bytes              | 6 bytes              | 2 bytes | variable             |
```

The frame's header bytes are:

```text
02 00 00 00 00 02 02 00 00 00 00 01 08 00
```

The complete payload nesting is:

```text
Ethernet frame
  +-- Destination MAC | Source MAC | EtherType = 0x0800
  +-- IPv4 datagram
    +-- IPv4 header: src=10.0.0.1, dst=10.0.0.2, protocol=17
    +-- UDP datagram
      +-- UDP header: sport=4000, dport=1234, length=13
      +-- Application data: b'hello'
```

The complete transmitted bytes, omitting the physical Ethernet preamble and CRC used by the wire, are:

```text
02 00 00 00 00 02 02 00 00 00 00 01 08 00
45 00 00 21 00 00 40 00 40 11 00 00 0a 00 00 01 0a 00 00 02
0f a0 04 d2 00 0d 00 00 68 65 6c 6c 6f
```

### Step 5: Receive and unwrap

The receiver reverses the construction sequence:

```text
1. Ethernet checks the destination MAC and EtherType, then removes the frame header.
2. IPv4 checks the destination IP and protocol value, then removes the IPv4 header.
3. UDP checks the destination port and length, then removes the UDP header.
4. The application receives b'hello'.
```

A router performs the Ethernet receive and IPv4 forwarding steps, but does not remove the UDP header. It chooses the next interface, resolves the next-hop MAC with ARP, and creates a new Ethernet frame around the same IPv4 datagram.

## 12. Constants and Conversions

| Item | Value |
| --- | --- |
| Ethernet MAC address | 6 bytes / 48 bits |
| IPv4 address | 4 bytes / 32 bits |
| IPv4 header minimum | 20 bytes |
| UDP header | 8 bytes |
| TCP header minimum | 20 bytes |
| IPv4 EtherType | `0x0800` |
| ARP EtherType | `0x0806` |
| VLAN tag indicator | `0x8100` |
| TCP protocol number | `6` |
| UDP protocol number | `17` |
| ARP request | `1` |
| ARP reply | `2` |

Use binary/network representations on the wire and presentation strings in configuration, logs, and user-facing output.

## 13. Common Pitfalls

- Do not include Ethernet preamble or CRC when parsing the raw-socket frame used by the labs.
- Do not confuse the 4-byte 802.1Q header with the 2-byte EtherType that follows it.
- The UDP length includes both the UDP header and its data; IPv4 total length includes the IPv4 header and its payload.
- The IPv4 header checksum covers the IPv4 header, while the transport lab uses zero for TCP and UDP checksums.
- TCP sequence numbers count bytes, not packets.
- TCP `Data Offset` counts 4-byte words, not bytes.
- Forwarding requires longest-prefix match, not simply the first matching table entry.
- A directly connected route has no explicit next-hop IP; ARP resolves the final destination on that interface.
- The starter files contain `pass` and `FIXME` sections by design. Confirm intended behavior in the corresponding README and tests before relying on an implementation stub.
