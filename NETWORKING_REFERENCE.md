# CS 460 Networking Reference

This is a consolidated, editable reference for the packet structures and implementation helps used throughout the course. The diagrams describe the bytes handled by the labs, not every field that appears on a physical network.

### Standalone Bit Layouts

Each protocol layout is shown separately below. The columns are bit positions within a 32-bit row; fields are labeled above the space they occupy.

#### Ethernet frame

<table border="1">
<tr><th>00</th><th>16</th><th>32</th><th>48</th><th>64</th><th>80</th><th>96</th></tr>
<tr><td colspan="3">Destination MAC</td><td colspan="3">Source MAC</td><td colspan="1">EtherType</td></tr>
</table>


#### 802.1Q VLAN Ethernet frame

<table border="1">
<tr><th>00</th><th>16</th><th>32</th><th>48</th><th>64</th><th>80</th><th>96</th><th>112</th><th>128</th></tr>
<tr><td colspan="3">Destination MAC</td><td colspan="3">Source MAC</td><td colspan="2">802.1Q Header</td><td colspan="1">EtherType</td></tr>
</table>


#### ARP packet

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="16">Hardware type</td><td colspan="16">Protocol type</td></tr>
<tr><td colspan="8">Hardware address length</td><td colspan="8">Protocol address length</td><td colspan="16">Opcode</td></tr>
<tr><td colspan="32">Sender hardware address</td></tr>
<tr><td colspan="32">Sender protocol address</td></tr>
<tr><td colspan="32">Target hardware address</td></tr>
<tr><td colspan="32">Target protocol address</td></tr>
<tr><td colspan="32">Data</td></tr>
</table>

#### IPv4 header

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="4">Version</td><td colspan="4">IHL</td><td colspan="8">DSCP/ECN</td><td colspan="16">Total length</td></tr>
<tr><td colspan="16">Identification</td><td colspan="3">Flags</td><td colspan="13">Fragment offset</td></tr>
<tr><td colspan="8">TTL</td><td colspan="8">Protocol</td><td colspan="16">Header checksum</td></tr>
<tr><td colspan="32">Source address</td></tr>
<tr><td colspan="32">Destination address</td></tr>
</table>

#### UDP header

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="16">Source port</td><td colspan="16">Destination port</td></tr>
<tr><td colspan="16">Length</td><td colspan="16">Checksum</td></tr>
</table>

#### TCP header

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th><th>04</th><th>05</th><th>06</th><th>07</th><th>08</th><th>09</th><th>10</th><th>11</th><th>12</th><th>13</th><th>14</th><th>15</th><th>16</th><th>17</th><th>18</th><th>19</th><th>20</th><th>21</th><th>22</th><th>23</th><th>24</th><th>25</th><th>26</th><th>27</th><th>28</th><th>29</th><th>30</th><th>31</th></tr>
<tr><td colspan="16">Source port</td><td colspan="16">Destination port</td></tr>
<tr><td colspan="32">Sequence number</td></tr>
<tr><td colspan="32">Acknowledgment number</td></tr>
<tr><td colspan="4">Data offset</td><td colspan="3">Reserved</td><td colspan="3">ECN</td><td colspan="6">Control bits</td><td colspan="16">Window</td></tr>
<tr><td colspan="16">Checksum</td><td colspan="16">Urgent pointer</td></tr>
<tr><td colspan="32">Options and padding</td></tr>
</table>

## 3. Ethernet and VLAN Frames

### Ethernet frame

The raw Ethernet frame received by these labs is:

| Field | Size | Description |
| --- | ---: | --- |
| Destination MAC address | 6 bytes | Intended receiver; `ff:ff:ff:ff:ff:ff` is broadcast |
| Source MAC address | 6 bytes | Sender on the local link |
| 802.1Q header | 4 bytes | An optional tag to support VLANs |
| EtherType | 2 bytes | Identifies the payload protocol |
| Payload | variable | Usually an IPv4 datagram or ARP packet |

An Ethernet frame also has a preamble and CRC. However, for this class you do not have to deal with these as the physical layer is out of scope of the labs. Further information on the preamble and CRC can be found here if desired: [Ethernet packet - physical layer](https://en.wikipedia.org/wiki/Ethernet_frame#Ethernet_packet_%E2%80%93_physical_layer)

The following diagram uses 16-bit columns to show the bit widths of the Ethernet fields. The first row is the ordinary frame; the second row shows the tagged form.

#### Ethernet frame

<table border="1">
<tr><th>00</th><th>16</th><th>32</th><th>48</th><th>64</th><th>80</th><th>96</th></tr>
<tr><td colspan="3">Destination MAC</td><td colspan="3">Source MAC</td><td colspan="1">EtherType</td></tr>
</table>


#### 802.1Q VLAN Ethernet frame

<table border="1">
<tr><th>00</th><th>16</th><th>32</th><th>48</th><th>64</th><th>80</th><th>96</th><th>112</th><th>128</th></tr>
<tr><td colspan="3">Destination MAC</td><td colspan="3">Source MAC</td><td colspan="2">802.1Q Header</td><td colspan="1">EtherType</td></tr>
</table>

#### Worked VLAN frame example

Example values: destination MAC `02:00:00:00:00:02`, source MAC `02:00:00:00:00:01`, VLAN ID `25`, and IPv4 EtherType `0x0800`.

**Bytes for each field (network byte order is big-endian):**

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

#### Worked non-VLAN Ethernet frame example

Example values: destination MAC `02:00:00:00:00:02`, source MAC `02:00:00:00:00:01`, and IPv4 EtherType `0x0800`. This frame has no 802.1Q header.

**Bytes for each field (network byte order is big-endian):**

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="4">Destination MAC address (bytes 0-3)</th></tr>
<tr><td>02</td><td>00</td><td>00</td><td>00</td></tr>
<tr><th colspan="2">Destination MAC address (bytes 4-5)</th><th colspan="2">Source MAC address (bytes 0-1)</th></tr>
<tr><td>00</td><td>02</td><td>02</td><td>00</td></tr>
<tr><th colspan="4">Source MAC address (bytes 2-5)</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>01</td></tr>
<tr><th colspan="2">EtherType</th></tr>
<tr><td>08</td><td>00</td></tr>
</table>

Bytes, in network byte order (big-endian):

```text
02 00 00 00 00 02 02 00 00 00 00 01 08 00
```

Expansion:

```text
02 00 00 00 00 02 -> destination MAC = 02:00:00:00:00:02
02 00 00 00 00 01 -> source MAC      = 02:00:00:00:00:01
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

**Bytes for each field (all multi-byte fields are in network byte order, big-endian):**

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

**Shared byte note:** ARP bytes ``06 04 00 01`` contain hardware address length ``06``, protocol address length ``04``, and opcode ``00 01``.

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
| DSCP | 6 bits | In this class, set to 0 |
| ECN | 2 bits | In this class, set to 0 |
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
<tr><td colspan="4">Version</td><td colspan="4">IHL</td><td colspan="6">DSCP</td><td colspan="2">ECN</td><td colspan="16">Total length</td></tr>
<tr><td colspan="16">Identification</td><td colspan="3">Flags</td><td colspan="13">Fragment offset</td></tr>
<tr><td colspan="8">TTL</td><td colspan="8">Protocol</td><td colspan="16">Header checksum</td></tr>
<tr><td colspan="32">Source address</td></tr>
<tr><td colspan="32">Destination address</td></tr>
<tr><td colspan="32">Options and padding :::</td></tr>
</table>

#### Worked IPv4 datagram example

Example values: no options, total length `33` bytes, identification `0x1234`, do-not-fragment flag set, TTL `64`, UDP protocol `17`, source `192.0.2.1`, and destination `192.0.2.2`. The checksum below is shown as zero to keep the example focused on layout.

**Bytes for each field (multi-byte fields are in network byte order, big-endian):**

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

**Shared byte note:** IPv4 byte ``45`` contains Version ``4`` and IHL ``5``. Byte ``00`` for DSCP/ECN is just `0` for DSCP and ``0`` for ECN. Bytes ``40 00`` are ``0100000000000000``: the first 3 bits, ``010``, are the Flags (Reserved ``0``, Don't Fragment/DF ``1``, More Fragments/MF ``0``), and the remaining 13 bits are the Fragment Offset, ``0``.

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

**Bytes for each field (multi-byte fields are in network byte order, big-endian):**

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

**Bytes for each field (multi-byte fields are in network byte order, big-endian):**

<table border="1">
<tr><th>00</th><th>01</th><th>02</th><th>03</th></tr>
<tr><th colspan="2">Source port</th><th colspan="2">Destination port</th></tr>
<tr><td>0f</td><td>a0</td><td>04</td><td>d2</td></tr>
<tr><th colspan="4">Sequence number</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>01</td></tr>
<tr><th colspan="4">Acknowledgment number</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>01</td></tr>
<tr><th colspan="2">Data offset / Reserved / ECN / Control bits</th><th colspan="2">Window</th></tr>
<tr><td>50</td><td>92</td><td>00</td><td>40</td></tr>
<tr><th colspan="2">Checksum</th><th colspan="2">Urgent pointer</th></tr>
<tr><td>00</td><td>00</td><td>00</td><td>00</td></tr>
<tr><th colspan="4">Data (bytes 0-3)</th></tr>
<tr><td>68</td><td>65</td><td>6c</td><td>6c</td></tr>
<tr><th colspan="1">Data (byte 4)</th></tr>
<tr><td>6f</td></tr>
</table>

**Shared byte note:** TCP bytes ``50 92`` contain the packed fields: Data Offset ``0101`` = 5 words (20 bytes), Reserved ``000`` = 0, ECN ``010``, and Control Bits ``010010`` = ACK + SYN (using the order URG, ACK, PSH, RST, SYN, FIN). The packed fields concatenate as ``0101 000 010 010010`` or ``0101000010010010``.

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
| IPv4 header          |
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
| Destination MAC      | Source MAC           | EtherType|
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

## 13. Various Notes

### Endianness

In this class, **network byte order** will be used, which is **big-endian**. You likely will not have to worry about this at all in this class, but this needs to be noted.

Examples below use the same numeric value in both byte orders. The bit order within each byte does not change; only the order of complete bytes changes.

| Value size | Numeric value | Big-endian bytes | Little-endian bytes |
| --- | --- | --- | --- |
| 1 byte | `0x12` | `12` | `12` |
| 2 bytes | `0x1234` | `12 34` | `34 12` |
| 4 bytes | `0x12345678` | `12 34 56 78` | `78 56 34 12` |

For example, the 16-bit value `0x1234` is transmitted as `12 34` in network byte order. Optional further reading: [Endianness - Wikipedia](https://en.wikipedia.org/wiki/Endianness).

### Layering Options

These are the main protocol combinations used in the labs:

```text
Ethernet Frame -> IPv4 -> UDP
Ethernet Frame -> IPv4 -> TCP
Ethernet Frame -> ARP
```

Note that you may not construct these full combinations. For instance, in the first lab, link-layer, you will only construct the Ethernet Frame - no payload inside it. However, it is important to remember how the OSI model behind this uses abstraction layers.