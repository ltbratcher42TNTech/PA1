# PA1 Capture Analysis

_Your capture, `evidence/pa1-capture.pcap`, holds two ETP sessions from your own
programs: a PUT, then a GET of the same file. This analysis is about the **GET
session** unless a question says otherwise. Use Wireshark (or tshark) to read the
capture. The numbers must come from your capture: they are checked against it._

## How I captured it

Captured on the Lab 0 Linux VM, on the loopback interface, from the repository
root (`~/csc4200-work/PA1`). The transferred file was 262,144 bytes (256 KiB) of random data.

```
# Terminal 1 -- server
mkdir -p server-files scratch
make run-server PORT=5000 ROOT=server-files

# Terminal 2 -- test file and capture (200-byte snap length)
head -c 262144 /dev/urandom > scratch/test.bin
sudo tcpdump -i lo -s 200 -Z $USER -w evidence/pa1-capture.pcap 'tcp port 5000'

# Terminal 3 -- client: PUT, then GET of the same file, then ss within 60 s
./etp-client --host 127.0.0.1 --port 5000 --name tyler put scratch/test.bin test.bin
./etp-client --host 127.0.0.1 --port 5000 --name tyler get test.bin scratch/got.bin
ss -tan | grep -E 'State|:5000' > evidence/time-wait.txt
cmp scratch/test.bin scratch/got.bin && echo SAME

# Then Ctrl+C in Terminal 2 (stop tcpdump) and Terminal 1 (stop the server)
```

## Facts

_Fill in every value from **your** capture, for the **GET** session. Integers in
decimal. Sequence numbers are the **absolute** (raw) values — in Wireshark, turn
off "relative sequence numbers" (Edit > Preferences > Protocols > TCP) or read
the "Sequence Number (raw)" field. With tshark, add
`-o tcp.relative_sequence_numbers:FALSE`. Do not change the key names or the
`etp-facts` label: a script reads this block._

```etp-facts
server_port: 5000
client_port: 52012
client_isn: 3657328345                  # raw sequence number in the client's SYN
server_isn: 4114998914                  # raw sequence number in the server's SYN/ACK
client_mss: 65495                       # MSS option value in the client's SYN, in bytes
server_to_client_payload_bytes: 262190  # all TCP payload bytes the server sent in this session (HELLO_OK + GET_OK + BYE_OK)
server_to_client_data_segments: 9       # server-to-client frames with payload, across all three responses, counted as captured
first_fin_from: server                  # server or client
```

## Q1. The handshake

_Give the frame number, flags, and raw sequence and acknowledgment numbers of the
three handshake segments of the GET session. Explain why the acknowledgment number
in the third segment is what it is. What MSS did the client advertise, and what on
your machine determines that value?_

The three-way TCP handshake of the GET session consists of frames 25, 26 and 27.

| Frame | Flags | Raw sequence number | Raw acknowledgment number |
|---|---|---|---|
| 25 | SYN | 3657328345 | 0 |
| 26 | SYN, ACK | 4114998914 | 3657328346 |
| 27 | ACK | 3657328346 | 4114998915 |

**Why does the third segment have that acknowledgment number?**

TCP acknowledgment numbers indicate the next sequence number expected.

- The client sends its SYN using sequence number 3657328345.
- A SYN consumes one sequence number.
- The server therefore acknowledges 3657328346.
- The server sends its SYN using sequence number 4114998914.
- The client acknowledges 4114998915 in frame 27.

**Client MSS: 65495 bytes**

The client advertises an MSS of 65495 bytes in frame 25.

MSS is determined by the network interface's MTU and the IP/TCP header sizes. For IPv4:

```
65535 - 20 - 20 = 65495
```

This reflects the maximum IPv4 packet size minus the standard 20-byte IP and 20-byte TCP headers. The loopback interface's MTU and TCP's IPv4 size limit determine the advertised value.

## Q2. Your protocol on the wire

_Find the segment that carries your client's GET request. Give its frame number,
paste the first 8 payload bytes in hex, and decode each header field (VERSION,
TYPE, STATUS, RESERVED, LENGTH). Do the same for the header of the server's
GET_OK. Does GET_OK's LENGTH equal the size of your file? Show the arithmetic that
connects `server_to_client_payload_bytes` to the three responses the server sent in
this session._

### Client's GET request: frame 32

The first eight payload bytes are:

```
01 03 00 00 00 00 00 08
```

| Field | Hex | Meaning |
|---|---|---|
| VERSION | `01` | Protocol version 1 |
| TYPE | `03` | GET request |
| STATUS | `00` | No error |
| RESERVED | `00` | Reserved field |
| LENGTH | `00 00 00 08` | 8 bytes |

The remaining eight bytes contain the filename:

```
74 65 73 74 2e 62 69 6e   ->   "test.bin" (ASCII)
```

### Server's GET_OK response: frame 33

The first eight payload bytes are:

```
01 83 00 00 00 04 00 00
```

| Field | Hex | Meaning |
|---|---|---|
| VERSION | `01` | Protocol version 1 |
| TYPE | `83` | GET_OK response |
| STATUS | `00` | Success |
| RESERVED | `00` | Reserved field |
| LENGTH | `00 04 00 00` | 262,144 bytes |

The LENGTH field equals the size of our file:

```
0x00040000 = 262,144 bytes = 256 KiB
```

### Server-to-client payload arithmetic

The server sends three responses:

| Response | Payload bytes |
|---|---|
| HELLO_OK | 30 |
| GET_OK header | 8 |
| GET_OK file contents | 262,144 |
| BYE_OK | 8 |
| **Total** | **262,190** |

```
30 + (8 + 262,144) + 8 = 262,190
```

This matches `server_to_client_payload_bytes` (262190) in our facts.

## Q3. A byte stream, not messages

_How many segments carried the **GET_OK response** (not counting the segments
that carried HELLO_OK and BYE_OK), and what were their payload sizes? Were they
all the same size? Were they all the MSS? Did any one segment carry the
end of one ETP message and the start of the next, or did one ETP message span
several segments? Explain what this means for code that reads an ETP message, and
point to the place in your own code that deals with it._

The GET_OK response is carried in **7 TCP data segments**: server-to-client frames 33, 34, 36, 38, 39, 41 and 42.

Using the TCP sequence-number progression, their payload sizes are:

| Frame | TCP payload size |
|---|---|
| 33 | 32,768 bytes |
| 34 | 32,768 bytes |
| 36 | 61,440 bytes |
| 38 | 47,360 bytes |
| 39 | 22,272 bytes |
| 41 | 47,360 bytes |
| 42 | 18,184 bytes |
| **Total** | **262,152 bytes** |

Check:

```
32,768 + 32,768 + 61,440 + 47,360 + 22,272 + 47,360 + 18,184 = 262,152
                                                             = 8-byte header + 262,144-byte file
```

**Were they all the same size?**
No. Their payload sizes vary.

**Were they all the MSS?**
No. The client's advertised MSS was 65,495 bytes, and none of these seven payloads is that size.

**Did a segment contain the end of one ETP message and the beginning of another?**
No. In this capture, the GET_OK response occupies its own seven segments. The BYE_OK response is in a separate segment (frame 45).

**Did one ETP message span several segments?**
Yes. The single GET_OK message is spread across all seven segments above.

**What this means for our code**

TCP provides a byte stream, not individual application messages. An application cannot assume that one `recv()` call will return one complete ETP message. A message can be split across multiple TCP segments, and multiple messages can potentially arrive together.

Our code handles this using:

- `src/etp.py:51` reads the 8 byte header, while `src/etp.py:65` reads the LENGTH of payloads.
- `src/transport_tcp.py:74-78` to obtain the exact number of bytes requested.

That is why our implementation needs to handle partial reads instead of assuming one `recv()` equals one message.

## Q4. Teardown and TIME-WAIT

_Which side sent the first FIN in the GET session, and why — cite the rule in the
specification. Paste the relevant line(s) from `evidence/time-wait.txt`. Which end
of the connection (address and port) is in TIME-WAIT, why that end, and roughly how
long Linux keeps it there? Connect this to the SO_REUSEADDR requirement._

## Q5. What an observer learns

_Suppose someone else obtained this capture file. List specifically what they
learn about your session (names, sizes, timing, anything else). You captured with a
200-byte snap length: what does that hide, and what does it not hide? Which kind of
protection would stop a real on-path observer from learning these things, and what
would it still not hide?_
