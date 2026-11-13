# PA1 — ETP/1 Client and Server

**Students:** Tyler Brinkman (`TBrink22`), Lanis Bratcher (`ltbratcher42TNTech`)
**Course:** CSC 4200 — Fall 2026
**Language:** Python 3.14.4
**Submitted commit:** _the SHA you paste into Canvas_

## Build and run

```
make build
make run-server PORT=5000 ROOT=server-files      # in one terminal
./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME list    # in another
```

Other client commands:

```
./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME get <remote> <local>
./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME put <local> <remote>
```

- **Dependencies:** none beyond the Python standard library. No packages were installed.
- **Python version:** _fill in from `python3 --version` on the test VM_
- **Tested on:** Ubuntu VM (loopback); also manually on Windows during development.
- **Own tests:** `make check` runs the course self-check (`tests/etp_selfcheck.py`).
  We also tested by hand with the client against the server (see `DESIGN.md` §7).

Client exit codes: `0` success, `1` server replied with ERROR, `2` other failure,
`3` PUT digest mismatch.

## What works and what does not

chmod +x etp-server etp-client
python3 -m py_compile src/*.py
chmod +x etp-server etp-client
python3 -m py_compile src/*.py
python3 tests/etp_selfcheck.py --no-build || true
ETP/1 self-check 1.0 (2026-10-20)  --  /home/vboxuser/pa1-check

Server tests
  S01  PASS   2 pt  HELLO handshake and HELLO_OK format
  S02  PASS   2 pt  LIST format, sorting and filtering
  S03  PASS   1 pt  GET a small file
  S04  PASS   1 pt  GET a 2 MiB file
  S05  PASS   2 pt  PUT 300 KB, digest, then GET it back
  S06  PASS   1 pt  PUT and GET empty files
  S07  PASS   1 pt  NOT_FOUND is non-fatal
  S08  PASS   1 pt  BAD_NAME blocks traversal; non-fatal
  S09  PASS   2 pt  requests arriving in fragments
  S10  PASS   2 pt  several requests in one write
  S11  PASS   1 pt  unknown TYPE: BAD_TYPE, payload skipped
  S12  PASS   1 pt  bad VERSION is fatal
  S13  PASS   1 pt  oversized LEN is fatal, nothing allocated
  S14  PASS   1 pt  request before HELLO is fatal
  S15  PASS   1 pt  BYE: BYE_OK, then the server closes
  S16  PASS   2 pt  8 clients served while 2 others stall
  S17  PASS   1 pt  survives clients that vanish mid-transfer
  S18  PASS   1 pt  restart on the same port (SO_REUSEADDR)

Client tests (a scripted peer plays the server)
  C01  PASS   1 pt  list: fragmented response printed exactly; sends BYE
  C02  PASS   2 pt  get: 200 KB in fragments written to disk
  C03  PASS   1 pt  put: payload layout and digest printed
  C04  PASS   1 pt  server ERROR: exit 1 and a stderr message
  C05  PASS   1 pt  digest mismatch: exit 3
  C06  PASS   0 pt  no server: exit 2 (not scored)

Evidence and documents
  - capture: 2 TCP connection(s); the GET session looks complete
  - ANALYSIS.md: every etp-facts field is filled in (values are checked at grading)

Repository (scored by the same rules at grading)
  build 3/3  (assumed: --no-build)
  required files 2/2
  commit history 3/3  (49 commits on 9 day(s); full marks: 5 or more after the starter commit, on 3 or more days)
  transport seam 2/2
  Build and repository: 10 / 10

Execution: 30 / 30 points
This is the Execution part of the rubric only. Evidence, design and documentation
are scored by people, using the bands in the PA1 brief.

Known limitations:

- The server starts one thread per connection with no upper limit, so many
  connections at once could exhaust resources.
- There is no idle timeout; a client that connects and stalls holds its thread
  until it disconnects.

## Files

| File | What it is |
|---|---|
| `src/etp.py` | ETP/1 protocol: constants, message encode, and reading/decoding one message |
| `src/transport.py` | transport seam; the single import that selects the transport |
| `src/transport_tcp.py` | TCP transport: listen, accept, connect, send_all, recv_exact, close |
| `src/server.py` | ETP/1 server: one thread per session, HELLO/LIST/GET/PUT/BYE handling, logging |
| `src/client.py` | ETP/1 client: one session per run (list, get, put) |
| `etp-server`, `etp-client` | shell launchers the self-check and grader run |
| `Makefile` | build, run-server, check, clean |
| `tests/etp_selfcheck.py` | course-provided self-check |
| `TEAM_PLAN.md` | team plan and division of work |
| `DESIGN.md` | design document |
| `AI_USE.md` | AI-use disclosure and log |
| `evidence/pa1-capture.pcap` | loopback capture of our PUT and GET sessions |
| `evidence/time-wait.txt` | `ss` output showing TIME-WAIT after our sessions |
| `evidence/ANALYSIS.md` | our capture analysis |

---

## Authorship statement

I am the author of this submission. Except where this README or `AI_USE.md`
says otherwise, I wrote, or rewrote and understand, every part of it — code,
documents and evidence — and I can explain any line of it to the instructor or
GTA. I am responsible for its correctness, including any part a
generative AI tool helped me produce. My use of AI tools is disclosed in full in
`AI_USE.md`. The capture and measurements in `evidence/` come from my own programs,
run by me.

**Signed:** Tyler Brinkman  **Date:** 11/13/2026

**Signed:** Lanis Bratcher  **Date:** 11/13/2026
