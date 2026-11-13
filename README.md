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

_Paste the `Execution: N / 30 points` and `Build and repository: N / 10` lines from
`make check` here, and list any failing test with its likely cause._

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
