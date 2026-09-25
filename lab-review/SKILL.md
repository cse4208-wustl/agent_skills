---
name: "lab-review"
description: "Review lab submissions against a rubric: build, run, review code, and write a review.md per student plus a class summary."
---
 
# Lab Review
 
Use this when the user gives you a rubric and a folder with a solution to a programming assignment. The assignment description is in the README.md file of that folder.
 
## Role
 
You are a teaching assistant. You apply the rubric that is given to you; you do not invent categories or point values. If the rubric has no point values, report outcomes only (for example Meets expectations / Partially meets / Does not meet).
 
## Inputs to locate first
 
1. The rubric, the assignment README, and the programming guidelines. If the session is attached to a Project, read them with the Projects tool (`project_read`) rather than searching the disk. Read all three in full before touching any submission.
2. Which compiler and OS the reviewer is meant to use. Note any mismatch with the course toolchain (for example g++ 11 here vs. the course's 8.x) in the summary.
4. The assignment name (for example "Lab 0"), taken from the README or rubric title. You need it for the review heading.

## Working environment rules
 
- When a shell on the user's computer is available (`device_bash`), do the work there so the files do not have to be staged. Each call is a fresh shell: background jobs die when the call ends, so run builds in the foreground (use `xargs -P 8` to build all submissions in parallel and stay under the call time limit) and keep scratch work under `$HOME/work`, outside the mounted submissions folder.
- Never build inside the student directories. Export each repo's committed HEAD into a scratch directory (`git --no-optional-locks -C <repo> archive HEAD | tar -x -C <scratch>`) and build there. This grades exactly what was pushed, and it avoids two traps seen before: the working copy can differ from the repo (a case-insensitive macOS checkout hides files whose names differ only in case, so a repo can build on Linux but look broken in the folder), and committed binaries can make `make` a no-op.
- Do not run `git status` or other index-writing git commands in the student repos; they can leave a stale `.git/index.lock` in a mounted folder that cannot be deleted without permission. Use `--no-optional-locks` and read-only commands (`log`, `ls-tree`, `ls-files`, `show`). If a lock file is left behind, rename it and tell the user.
- Only write `review.md` (and one class summary) into the submissions folder.

## Procedure
 
1. Build with the solution's own Makefile exactly as the instructions say (`make`), capturing the full output. Count warnings and errors. Record the compiler version.
2. Run the executable with a timeout (10 seconds), capturing stdout, stderr and the exit code.
3. Verify the program's output by measuring, not by reading the constructor. Write a small script that parses the output and counts items and multiplicities. Parsers must handle many output formats; eyeball anything the parser flags.
4. If the build fails or the shipped binary cannot run, do a diagnostic rebuild in scratch (for example remove a stale committed binary, or fix a file-name case) so you can still grade the source. State clearly what was changed for the diagnostic and score the underlying defect once, in the most fitting category.
5. Read every source file and the student's `ANSWERS.md`/notes (drop lines that are copied from the README so you only read the student's own text). List tracked files (`git ls-files`) to spot committed binaries, IDE folders, swap files, and case-colliding names.
6. Rate each rubric category, keeping file and line numbers as evidence for every category that is not fully met.
7. Spot-check any surprising finding with a tiny test program (for example private inheritance: try to convert `Derived*` to `Base*`; an operator that writes to `cout` instead of its stream argument: call it with an `ostringstream`; an increment operator that throws: call it on the sentinel value).

## Scoring conventions
 
- Apply the outcome that best fits the whole category. Every Partially meets or Does not meet needs a one-line justification with file and line evidence; Meets expectations gets no justification (see the review format).
- Score a defect in exactly one category. Example: an unresolved compiler warning goes in the build category or the practices category, not both; private inheritance is counted in each derived-class category but not also in the base-class category; a committed binary is a repository-submission issue, not also a build issue.
- Accept any reasonable design for details the README leaves open, as long as the student documented it.
- Do not penalize sound additions that do not change the specification (for example a virtual destructor or `const` on a virtual method); do penalize extra public members on an interface that the README says has a single method.
- Exact-output requirements mean exact: an operator specified to print `"C"` that prints `"C "` is Partially meets.
- "Tested beyond a single run" (a practices category): Meets needs either two or more documented trials/iterations or an edge-case check; one final run with no edge cases is Partially meets.
- A student who records wrong output as "correct" gets the functional defect in the output categories and the missed verification in the testing/practices category, not in the documentation category unless a required documentation element is actually missing or misdescribed.
- Missing submissions (only starter files) are Does not meet in every category; note that the work may exist elsewhere.
## Per-student `review.md` format
 
Create `review.md` in the solution folder with these parts, in this order:
 
1. Heading `# <Assignment name> Review`, for example `# Lab 0 Review`. Use the assignment's name only. Do not put the student's GitHub username, folder name, or any other student identifier in the heading (the file already lives in that student's folder).
2. `**Summary:**` one or two sentences.
3. `**Tally:**` counts of each outcome.
4. `## Build and run evidence`: the `make` result (compiler, flags, warning/error counts, quoted warnings) and what the executable printed (counts, format, exit code).
5. `## Rubric application`: a table with columns `#`, `Category`, `Outcome`, `Justification`, one row per rubric category. Fill the `Justification` cell only when the outcome is Partially meets or Does not meet; it should state what fell short, with file and line references. For a Meets expectations row, leave the `Justification` cell empty: no text, no "all requirements met" filler.
6. `## Details on how the solution can be improved`: bullet examples where the work was not perfect under the rubric. Each bullet starts with `Rubric Category N` (spelled out, never abbreviated as `Cat`), then file:line, then what is wrong and what would fix it. If every category is met, say so and list only cosmetic notes.
Generate these from structured notes (one line per category: number, outcome letter, and a justification only for non-Meets outcomes) with a small script, so wording and layout stay identical across students and a global edit (such as renaming a heading) is one regeneration instead of dozens of hand edits.
 
## Wrap-up message to the user
 
Keep it short: what was produced and where, the headline findings (worst functional bugs, missing submissions, unusual repo problems), the judgment calls that the instructor may want to overrule, roster names without folders, and any housekeeping you left in their folder.
 
## Checklist before finishing
 
- Solution folder has a `review.md` headed `# <Assignment name> Review` with no student identifier, and every table has all rubric categories.
- Justification cells are empty for Meets expectations and filled for Partially meets and Does not meet.
- Output counts in each review were measured by running the program.
- Findings that could look unfair were re-checked with a test program.
- No scratch files or build products were left in the student folders.
