# AI Use Disclosure — PA1

## 1. Did you use generative AI on this assignment?

- [ ] No. I did not use any generative AI tool for any part of PA1.
- [x] Yes, as described below.

## 2. Statement (the syllabus form)

> The authors acknowledge the utilization of Claude (Claude Code), a language model
> developed by Anthropic, in the preparation of this assignment. Claude was employed
> in the following manner(s) within this assignment: reviewing our server and
> protocol code and pointing out bugs (we wrote all fixes ourselves); running manual
> and raw-protocol tests of our programs on Windows; giving the commands used to
> make the packet capture and the TIME-WAIT evidence; checking the capture file was
> complete (pass/fail only, without reporting its contents); explaining how to use
> Wireshark filters and columns; troubleshooting git push errors; reviewing our
> capture analysis answers against the specification and reformatting them; fixing
> typos in DESIGN.md; filling in factual parts of README.md; drafting TEAM_PLAN.md;
> and drafting this disclosure.
>
> The authors acknowledge the utilization of ChatGPT, a language model developed by
> OpenAI, in the preparation of this assignment. ChatGPT was employed in the
> following manner(s) within this assignment: explaining errors for the programming
> assignment; explaining ETP/1 framing, message headers and recv_exact; reviewing
> and debugging the client implementation; giving examples of EOF and exception
> handling for the client; and helping create a temporary mock server used to test
> the client before our real server existed.

## 3. Log

| # | Who | Tool | What I asked (summary) | What I used from the answer | What I changed or rejected | How I checked it was right |
|---|---|---|---|---|---|---|
| 1 | Tyler | Claude | Review my `server.py` and the `ValueError` change in `etp.py`; list issues | List of issues (error ordering, LIST/GET/PUT edge cases) | Wrote every fix myself; Claude only reviewed the result | Re-review after each fix; `make check` |
| 2 | Tyler | Claude | Review my fixes to `handle_list`, `handle_put`, `handle_get` | Feedback on symlinks/non-regular files, temp-file cleanup, error codes | Rewrote the handlers myself based on the feedback | Re-review; manual tests |
| 3 | Tyler | Claude | What commands do I run on the VM to build, capture and record TIME-WAIT? | tcpdump / ss / client commands; Claude wrote the "How I captured it" section of `ANALYSIS.md` from those commands | Ran every command myself on the VM | Capture opened and checked in Wireshark |
| 4 | Tyler | Claude | Check the capture files are good without telling me what is in them | PASS/FAIL integrity checks only | First capture had extra sessions, second was truncated; I re-captured three times | Final capture passed all checks |
| 5 | Tyler | Claude | Git push rejected (divergent branches); how to get a token | `git pull --rebase`; classic PAT with `repo` scope | — | Push succeeded |
| 6 | Tyler | Claude | Review my etp-facts values (after I filled them in by hand) | Values must be bare integers; flagged facts that were wrong | Re-measured the wrong values myself in Wireshark | Re-checked in Wireshark |
| 7 | Tyler | Claude | How to apply a display filter / add a column in Wireshark | Step-by-step UI instructions | — | Worked in Wireshark |
| 8 | Tyler | Claude | Review and reformat my answers to Q1–Q5 | Markdown tables, code blocks, headings; checked answers against the specification | Answers' content is ours | Compared with the specification and our capture |
| 9 | Tyler | Claude | Review `DESIGN.md`; fix the must-fix items | Typo fixes, template prompts removed, file names | Content gaps left for us to write | Read through the result |
| 10 | Tyler | Claude | Fill in `AI_USE.md` from our session | Draft of §2–§5 | _[edit as needed]_ | Read and corrected by both of us |
| 11 | Tyler | Claude | Draft a team plan that splits the work fairly | `TEAM_PLAN.md` | _[edit as needed]_ | Agreed on by both of us |
| 12 | Lanis | ChatGPT | Asked for explanations of ETP/1 framing, message headers, and recv_exact | Used the explanations to understand the 8-byte header, payload length, and fragmented messages | Applied concepts to my own implementation | Ran the self check tests and reviewed the protocol specification |
| 13 | Lanis | ChatGPT | Asked for help reviewing and debugging my client implementation, including HELLO, LIST, GET, PUT, and BYE | Used debugging ideas for error handling, flow, and the EOF check | Modified the suggestions to work with my existing code and transport interface | Ran the client self-checks, eventually passing all six tests |
| 14 | Lanis | ChatGPT | Generate a mock server to test the client | Temporary mock server (HELLO, LIST, GET, BYE); not submitted | Replaced by Tyler's real server | Client later passed all six client self-checks against the real server |
| 15 | Tyler | Claude Code| In general would ask Claude Code for advice when I ran into road blocks or got confused, which sometimes would and sometimes wouldn't result in any code. | It is sadly, very ethical, and adamantly refused to help beyond a certain point | I would make changes or often review documentation based on its recommendation on where to look | I would verify correctness via documents and testing |
| 16 | Lanis | ChatGPT | Asked for help reviewing DESIGN.md, resolving Git merge conflicts, and completing AI_USE.md | Used suggested wording, technical explanations, and Git instructions | Edited suggestions to match our implementation and combined my changes with Tyler's | Compared documentation against our source code, reviewed merge results, and checked the repository |

## 4. Where AI-produced material remains

- `src/client.py`, `run(args)` (Lanis): AI provided a simplified example for the EOF check, including exception handling and behavior. This logic remains in my modified implementation. It also helped with error handling and improved my try/finally blocks.
- `evidence/ANALYSIS.md`, Q1–Q5: the formatting (tables, code blocks, headings) is Claude's; the answer content is ours.
- `DESIGN.md`: typo and wording fixes by Claude; the content is Lanis's and Tyler's.
- `TEAM_PLAN.md`: fully AI-generated plan (Claude) that fairly assigned work to both of us... which we partially followed
- `AI_USE.md`: drafted by Claude, then reviewed and corrected by us.
- No server, protocol or transport code was written by AI.

## 5. One thing the AI got wrong — or how you checked it

**Lanis:** Chat initially gave too general advice about handling connection closure without knowing the actual transport implementation. After reviewing Tyler's transport_tcp.py, I confirmed that the close() function ALREADY handled graceful shutdown and adjusted my approach to use that instead.

**Tyler:** AI would typically run into issues when it didn't know something, like whether we are using python or C, or what we are trying to achieve. So it would recommend changes outside of scope that weren't necessary that would be ignored or just give entirely wrong advice. The problem with review can be alleviated by having claude code directly in your machine as it has access to everything, but it will still stumble from time to time. To be specific, one time AI was asking me to expand a function to use IPv6, but we only needed IPv4, so I ignored it.
