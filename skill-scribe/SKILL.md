---
name: skill-scribe
description: Set up a scribe subagent that watches the user do a process live and writes down every step, decision and nuance, then, when the user says stop, turns those notes (checked against the session's traces) into a reusable skill for that process. Use when the user says "set up a scribe", "watch me and write down everything I do", "record how I do this so we can repeat it", "make a skill out of what I'm about to do", or is about to perform by hand a process that will need repeating (another service, another migration, another release). Optionally ends with a state machine that repeats the process across several instances.
---

# Skill scribe

The user is about to do something they know how to do but have never written down. Writing it down beforehand loses the details that only show up while doing it. So a second agent, the scribe, writes it down while it happens, and the user doesn't have to stop and describe their own work.

You are the orchestrator. The user steers the work through you. The scribe is a separate subagent with one job: record. Keep the two jobs apart. The agent doing the work is trying to finish it. The scribe is only asking "what about this would carry over to the next one?"

## 1. Start the scribe

When the user asks for a scribe, confirm in one line which process is being recorded and where the notes will go, then start the scribe in the background. A good default location is `scribe/<process-slug>/` in the working directory.

Brief the scribe with:

- **The process** being recorded, in the user's words.
- **The notes file**: `scribe/<process-slug>/notes.md`. Append only; never rewrite earlier entries.
- **Whose actions to record:** the user's, and those of the orchestrator and every other subagent doing the work. The user's choices are the point, but the other agents' actions are how those choices got carried out.
- **What to record for each step:**
  - what the user asked for, in their words where possible
  - what was done (commands, files created or changed, tools used)
  - the decision made and the reason given, including options rejected
  - anything that went wrong and how it was fixed
  - whether the step looks **general** (it would happen on every instance) or **specific** (only true of this instance). Mark each one.
- **What not to do:** no editing of project files, no suggestions to the user, no summarising away detail. Precise and boring is the goal.
- **Where the traces are**, so it can check itself later: the session transcript, git history, shell history, build or test logs. Name the actual paths if you know them.

## 2. Keep the scribe fed

A subagent does not see the conversation after it was started. It only knows what you send it. This is the main way the scribe fails, so be disciplined:

- After each meaningful step (a decision, a file written, a command run, a correction from the user), send the scribe a short milestone message: what was asked, what was done, why.
- Send the user's exact words when they explain a reason or correct you. Those corrections are the nuance the skill is for.
- Do not wait to batch several steps. Batched summaries drop the order and the reasons.
- If the scribe has stopped (a session ended, a timeout), resume it or start a new one pointed at the same notes file, and tell it where it left off.

## 3. Stop, reconcile, draft

When the user says stop (or "that's it", "done, write it up"):

1. Tell the scribe to stop recording.
2. Have it **reconcile its notes against the traces**: the transcript, git log and diffs, logs. Notes alone will have gaps; traces alone have no reasons. Use both. Add missing steps, and mark anything it could not confirm. Save the result as `scribe/<process-slug>/sequence.md`: the ordered sequence of steps plus a short summary of the trace. This is useful on its own, even if no skill is made.
3. Have it draft `scribe/<process-slug>/SKILL.md` for the process:
   - when to use it
   - the ordered steps, each with its check (how you know the step is done)
   - the **parameters** that change from one instance to the next (names, ports, domain terms, data shapes), pulled from the steps marked specific
   - the decisions and their reasons, so a later agent doesn't undo them
   - the failures seen and their fixes
4. Show the user the draft. Do not install it into a skills directory or overwrite an existing skill until they approve it. A wrong step in a skill gets repeated every time it's used.

## 4. Optional: repeat it across many instances

When the process has to be done again several times (the same kind of service built for several more domains, say), turn the skill's steps into a simple state machine:

- **States**: one per step, in order, each with its entry condition and its check.
- **Per-instance context**: for each instance, fill in the parameters from step 3 plus what makes this instance different (its purpose, its data, its integrations).
- **Orchestration**: the user talks to the orchestrator about the stages; the orchestrator writes the instructions for each instance's state from the template and that instance's context, and moves each instance forward only when its check passes.

The template keeps every instance on the same sequence. The per-instance context is what stops them coming out identical.

## Pitfalls

- **The scribe only sees what you send it.** If the skill comes out thin, it's usually because milestone messages were skipped. The traces at stop time are the backstop.
- **Overfitting to the first instance.** Anything marked specific must become a parameter, not a fixed step.
- **Recording the agent instead of the user.** The point is the user's approach: what they chose and why. Routine agent mechanics only matter where they changed the outcome.
- **Not finishing.** Notes that never become a reviewed skill don't help next time. Close the loop at stop time while the user still remembers.
