# PA1 Team Plan: Tyler and Lanis

**Due: Tuesday 10 November 2026, 11:59 pm Central.** Submit the repo URL and the full 40-character commit SHA in Canvas.

The authority is the PA1 brief and the ETP/1 Specification. This file only covers how we split the work. Section numbers (§) refer to the Specification.

---

## 1. What we are building

ETP/1 is a small file-transfer protocol over TCP. The server stores files, and the client can `list`, `get` and `put` them. `put` returns a SHA-256 digest that proves the file arrived intact.

Every message has an 8-byte header followed by a payload:

```
VERSION (1) | TYPE (1) | STATUS (1) | RESERVED (1) | LENGTH (4, big-endian)
```

Requests are HELLO 0x01, LIST 0x02, GET 0x03, PUT 0x04 and BYE 0x05. A response's TYPE is the request's TYPE | 0x80. ERROR is 0xFF, with the error code in STATUS.

### Repository layout

| Path | What it is | Owner |
|---|---|---|
| `src/transport_tcp.py` | The six socket functions. **The only file allowed to use `socket`.** | Tyler |
| `src/transport.py` | One import line that selects the transport. Leave as is. | — |
| `src/etp.py` | `encode()` and `read_message()`, which handle framing | Lanis |
| `src/server.py` | Accept loop, concurrency, session handling, §8.1 checks, logging | Tyler |
| `src/client.py` | `run()`: HELLO, one command, BYE, exit code | Lanis |
| `etp-server`, `etp-client` | Launchers (given, keep executable) | — |
| `tests/etp_selfcheck.py` | The grader's execution tests. Run with `make check`. | — |
| `tests/` | Any test scripts we write. This folder is exempt from the seam scan. | both |
| `evidence/` | Capture, `time-wait.txt`, `ANALYSIS.md` | Lanis (Tyler reviews) |
| `DESIGN.md`, `README.md`, `AI_USE.md` | Documents | split, see §4 |

### How it is scored (100 points)

| Part | Pts | Notes |
|---|---|---|
| Server tests S01–S18 | 24 | script |
| Client tests C01–C05 | 6 | script |
| Capture complete and 8 etp-facts | 10 | script compares our values with our pcap |
| Q1–Q5 in ANALYSIS.md | 20 | people, 4 each, explaining *why* using our numbers |
| Build, files, commits, seam | 10 | script |
| DESIGN.md | 15 | people |
| README.md | 10 | people, must honestly match `make check` output |
| AI use and authorship | 5 | people |

Half the grade is writing. Do not leave the documents until the last hour.

---

## 2. Task split

The two halves are about the same size. Tyler owns the transport and the server. Lanis owns framing, the client and the evidence.

### Tyler: transport and server

1. **`src/transport_tcp.py`**: `listen` (set `SO_REUSEADDR` before bind), `accept`, `connect`, `send_all`, `recv_exact` (raises `ConnectionClosed(clean=...)`), and `close` (shut down, drain for about 1 s, then close, per §8.2). **Do this first, because Lanis needs it.**
2. **`src/server.py` main loop**: listen, then accept forever with **one thread per session**, so a stalled client never blocks the others (S16).
3. **`handle_session`**:
   - the §8.1 checks in order
   - NOT_READY before HELLO
   - HELLO_OK, LIST (sorted by byte value, valid names only, regular files only), GET, PUT (NAMELEN + name + data, write the file, reply with the SHA-256), BYE (BYE_OK, then the server closes)
   - fatal errors (BAD_VERSION, TOO_LARGE, NOT_READY) close the session
   - non-fatal errors (BAD_TYPE, NOT_FOUND, BAD_NAME, BAD_REQUEST) keep it open
   - one log line per response (§9)
4. **Robustness**: a client that vanishes mid-transfer must end only *its* session (S17), and the server must restart on the same port right away (S18).
5. **Tests owned**: S01–S08 and S11–S18.

### Lanis: framing, client and evidence

1. **`src/etp.py`**:
   - `encode(mtype, payload, status)`. Check it byte-for-byte against the §12 table **before** anything else.
   - `read_message(conn, transport)`: read the 8-byte header with `recv_exact`, check VERSION and LENGTH **before** reading the payload, then read exactly LENGTH bytes. This is what makes S09, S10 and S13 pass for the server too.
2. **`src/client.py` `run()`**:
   - connect, HELLO with `--name`, run the command, send BYE, wait for the server's close
   - `list` prints the LIST_OK payload exactly
   - `get` writes the file
   - `put` prints 64 lowercase hex digits and a newline
   - every diagnostic goes to stderr
   - exit codes: 0 ok, 1 server ERROR, 2 failure or no server, 3 digest mismatch (§10)
3. **Client tests**: C01–C06. These run against a scripted fake server, so they **don't need Tyler's server**. Run them with `python3 tests/etp_selfcheck.py --client-only`.
4. **Evidence** (once both programs work): follow brief §4.1 exactly. That means a 256 KiB random file in `scratch/`, a capture with `-s 200`, a PUT then a GET, `cmp`, and `ss` within 60 s into `evidence/time-wait.txt`. Then fill in the `etp-facts` block from the **GET** session using **raw** sequence numbers.

### Shared

- **Q1–Q5**: Lanis drafts Q1–Q3 and Tyler drafts Q4–Q5. Each reviews the other's. Use only our own capture's numbers.
- **Code review**: each of us reads the other's code before submitting. Either of us may be asked to explain *any* line in the explanation conversation.

---

## 3. Schedule

We have Thu 5 Nov through Tue 10 Nov. The existing starter commit is dated 3 Nov, so **committing on at least two more days gets us to 3 distinct days**, which is worth the full 3 commit-history points. We need at least 5 commits after the starter commit, so commit small and often, every working day.

| When | Tyler | Lanis |
|---|---|---|
| **Thu 5 Nov** | `transport_tcp.py`, all six functions. Commit and push. | `encode` and `read_message`, checked against §12. Commit and push. |
| **Fri 6 Nov** | Server: threaded accept loop, HELLO/LIST/GET/BYE. Aim for S01–S04 and S15. | Client: `list`, `get`, `put`, exit codes. Aim for C01–C05 with `--client-only`. |
| **Sat 7 Nov** | PUT and every §8 error. Aim for S05–S14. | Test the client against Tyler's real server. Start DESIGN §1 and §3. |
| **Sun 8 Nov** | S16–S18 (concurrency, disconnects, restart). Full `make check`, aiming for 30/30. | **Capture** (only once `make check` is 30/30 or close), `time-wait.txt`, etp-facts. |
| **Mon 9 Nov** | DESIGN §2, §4, §5 (server). Draft Q4–Q5. | DESIGN §5 (client), §6. Draft Q1–Q3. |
| **Tue 10 Nov** | Together: DESIGN §7, README "what works", AI_USE.md, review each other's answers and code. **Fresh clone in the VM, `make build && make check`**, then submit the SHA. | (same) |

Tuesday is a buffer day. If we're ahead, finish the documents on Monday and use Tuesday only for the fresh-clone check and submission.

If code changes after the capture, **capture again**, so the evidence matches the code we submit.

---

## 4. Documents

| Document | Section | Owner |
|---|---|---|
| DESIGN.md | §1 Architecture | Lanis |
| | §2 The transport seam | Tyler |
| | §3 Framing | Lanis |
| | §4 Concurrency | Tyler |
| | §5 Errors and disconnects | Tyler (server), Lanis (client exit codes) |
| | §6 Threat model (one attack resisted, one not) | Lanis |
| | §7 Testing (one real bug story each) | both |
| README.md | Build/run, "what works" (paste the `make check` score lines), file list | Tyler |
| | Authorship statement | both sign |
| AI_USE.md | Log rows | each person logs their own AI use |
| ANALYSIS.md | Commands used, etp-facts, Q1–Q3 | Lanis |
| | Q4–Q5 | Tyler |

DESIGN.md should be 1–2 pages, specific, and true of our code. Give *reasons* for the seam, framing and concurrency choices.

---

## 5. AI rules for this team

- **AI does not write our code.** This covers `transport_tcp.py`, `encode`, `read_message`, the server session and concurrency, and the client session. We write it, and AI may *review* it afterwards.
- Small help with code is acceptable *only if it is logged in detail*: what we asked, what we used, what we changed, and how we checked it.
- AI is fine for explaining the spec, sockets, error messages, Wireshark and tshark syntax; for reviewing our code; for suggesting test cases; for make/git/shell; and for editing our prose.
- **AI must not produce Q1–Q5 answers or etp-facts values, or read our capture for us.** That includes writing a script that pulls the values out. Reading the capture is our job.
- **Log as you go**, not at the end. Every substantive interaction gets a row in `AI_USE.md` §3. Fill in §5 ("one thing the AI got wrong, or how you checked") with a real example.
- First log entry: Claude (Anthropic) generated this team plan, moved the Python starter files into the repo root, pointed the Makefile's `SELFCHECK` at `tests/etp_selfcheck.py`, and added `.gitattributes` (LF line endings).

---

## 6. Working together in git

- Pull before you start and before you push: `git pull --rebase`.
- Stick to the file ownership above, so we rarely edit the same file at once.
- Write commit messages that say what changed ("transport: recv_exact handles partial reads").
- Run code in the **Lab 0 VM** (Linux). The self-check does not run on Windows.
- Never commit `server-files/`, `scratch/`, or any test pcap other than `evidence/pa1-capture.pcap`. The `.gitignore` already handles this.

### Gotchas from the brief

- **S09/S10 fail**: we assumed one `recv` returns one message. Always use `recv_exact`.
- **S16 fails**: the server handles one client at a time. Use a thread per session.
- **S02 ordering**: sort by byte value, so uppercase comes before lowercase.
- **"Address already in use"**: `SO_REUSEADDR` must be set before `bind()`.
- **Facts marked wrong**: use the GET session (the 2nd connection) and raw sequence numbers, and count all three server responses.

---

## 7. Submission checklist

- [ ] `make check` run on a **fresh clone** in the VM, with the score lines pasted into README
- [ ] `evidence/pa1-capture.pcap`, `evidence/time-wait.txt`, `evidence/ANALYSIS.md` (no empty facts)
- [ ] DESIGN.md has all 7 sections, each true of the code
- [ ] AI_USE.md is complete for both of us, including §5
- [ ] Authorship statement signed
- [ ] At least 5 commits after the starter, on 3 days
- [ ] The instructor and GTA added as collaborators on GitHub
- [ ] Full 40-character SHA submitted in Canvas
