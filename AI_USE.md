# AI Use Disclosure — PA1

_Generative AI is allowed on PA1 as a tutor, reviewer, debugging partner and
helper with tooling. It is not allowed to be the author of your work. See the PA1
brief, "AI, Authorship and You". Fill in every section. "None" is an acceptable
answer only if it is true._

## 1. Did you use generative AI on this assignment?

- [ ] No. I did not use any generative AI tool for any part of PA1.
- [x] Yes, as described below.

## 2. Statement (the syllabus form)

> The author acknowledges the utilization of _[tool name]_, a language model
> developed by _[provider]_, in the preparation of this assignment. _[Tool name]_
> was employed in the following manner(s) within this assignment: _[for example:
> explaining socket errors; reviewing my framing code; drafting the Makefile;
> suggesting test cases; editing DESIGN.md for clarity]_.

## 3. Log

_One row per substantive interaction. Summaries are fine; you do not need to paste
whole conversations, but keep them in case you are asked._

| # | What I asked (summary) | What I used from the answer | What I changed or rejected | How I checked it was right |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |

## 4. Where AI-produced material remains

src/client.py, run(args): AI provided a simplified example for the EOF check, including exception handling and behavior. This logic remains in my modified implementation.

## 5. One thing the AI got wrong — or how you checked it

Chat initially gave too general advice about handling connection closure without knowing the actual transport implementation. After reviewing Tyler's transport_tcp.py, I confirmed that the close() function ALREADY handled graceful shutdown and adjusted my approach to use that instead. 
