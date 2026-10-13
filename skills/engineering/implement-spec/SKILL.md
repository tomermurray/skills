---
name: implement-spec
description: "Implement the result of /to-spec and /to-tickets in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

The goal is every `ready-for-agent` ticket implemented on a single **integration branch** and resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. Read the spec and tickets to understand the task graph, then gate on triage. The gate is done when every ticket carries `ready-for-agent`, `ready-for-human` or `wontfix`:
   - Triage each `needs-triage` or `needs-info` ticket with the user, then re-read the task graph.
   - When a ticket can only be triaged once an earlier ticket is built, the spec holds more than one block of work. Propose a split to the user: the triaged tickets stay in this spec, the rest move to a follow-up spec.

   Agents build only `ready-for-agent` tickets. A `ready-for-human` ticket stays open and the **frontier** stops at it: tickets it blocks wait for a human. `wontfix` tickets leave the graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 5 (a branch with no commits ahead of main can't open one), marked as closing the `ready-for-agent` tickets, and the spec too when no `ready-for-human` ticket remains.

4. Use **implementer subagents** to implement each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

5. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

7. Once all tickets are complete, call the Skill tool with `code-review` on the integration branch. Fix all issues raised by the code review in a single **implementer subagent**.

8. If a draft PR exists, mark it ready for review. Otherwise, resolve each `ready-for-agent` ticket the way the issue tracker closes work. Report the integration branch, plus each `ready-for-human` ticket left open and the tickets it blocks.

9. Clean up all **implementer subagent** worktrees.
