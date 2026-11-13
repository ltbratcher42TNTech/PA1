# AI Use Disclosure — PA1

_Generative AI is allowed on PA1 as a tutor, reviewer, debugging partner and
helper with tooling. It is not allowed to be the author of your work. See the PA1
brief, "AI, Authorship and You". Fill in every section. "None" is an acceptable
answer only if it is true._

## 1. Did you use generative AI on this assignment?

- [ ] No. I did not use any generative AI tool for any part of PA1.
- [x] Yes, as described below.

## 2. Statement (the syllabus form)

> The author acknowledges the utilization of ChatGPT a language model
> developed by OpenAI, in the preparation of this assignment. ChatGPT
> was employed in the following manner(s) within this assignment: _[for example:
> explaining socket errors; reviewing my framing code; drafting the Makefile;
> suggesting test cases; editing DESIGN.md for clarity]_.
> Explaining errors for the programming assignment
> helping create a temporary mock server for testing,
> 
> Claude Code was also used by my teammate to review DESIGN.md and provide suggestions for technical accuracy
> and drafting team_plan.md
## 3. Log

_One row per substantive interaction. Summaries are fine; you do not need to paste
whole conversations, but keep them in case you are asked._

| # | What I asked (summary) | What I used from the answer | What I changed or rejected | How I checked it was right |
|---|---|---|---|---|
| 1 | asked for explanations of ETP/1 framing, message headers, and recv_exact | Used the explanations to understand the 8-byte header, payload length, and fragmented messages | Applied concepts to my own implementation | Ran the self check tests and reviewed the protocol specification |
| 2 | Asked for help reviewing and debugging my client implementation, including HELLO, LIST, GET, PUT, and BYE | Used debugging ideas for error handling, flow, and the EOF check | Modified the suggestions to work with my existing code and transport interface | Ran the client self-checks, eventually passing all six tests. |

## 4. Where AI-produced material remains

src/client.py, run(args): AI provided a simplified example for the EOF check, including exception handling and behavior. This logic remains in my modified implementation. It also helped with error handling and improved my try/finally blocks.
TEAM_PLAN.md, fully ai generated plan that fairly assigned work to both of us. 


## 5. One thing the AI got wrong — or how you checked it

Chat initially gave too general advice about handling connection closure without knowing the actual transport implementation. After reviewing Tyler's transport_tcp.py, I confirmed that the close() function ALREADY handled graceful shutdown and adjusted my approach to use that instead. 
