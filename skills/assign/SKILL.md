---
name: assign
description: Split a task into independent pieces and hand each piece to its own session or subagent, with a model and reasoning effort matched to how hard that piece is. Use when the user types /assign, or says "assign this", "split this up", "farm this out", "delegate this across sessions", "run these in parallel sessions", or describes a multi-part task and wants it divided among separate sessions with different models or effort levels. Use it even if the user does not name models or effort.
---

# Assign

The user wants a task divided among separate sessions, split by difficulty: cheap, fast models take easy pieces; stronger models take hard ones. They already approved starting sessions, so do not ask first.

**You are the manager, not the worker.** The session running `/assign` only reads the task, decides who does what, writes short briefs, and reads short reports. It never does the building, fixing, checking or putting-together itself. Every piece, and the final assembly of the results, goes to another model chosen by step 3.

Goals: use as few expensive tokens as possible, work fast, and keep everything the user reads short and simple.

**Plain language (applies to every level).** The user builds software but is not an expert. Everything they read (summaries, tables, reports from sessions) uses short sentences and everyday words. If a technical word is needed, add a few plain words explaining it, e.g. "branch (a separate copy of the code)". Say what happened and what it means, not how.

This skill applies at every level. A session that gets a piece with several steps splits it again by the same rules.

## 1. Read the task

The task is what follows `/assign`. If empty, use what the conversation is about. If there is none, ask for it in one line.

## 2. Split it

A piece deserves its own session when it is independent and ends in something of its own (a file, a report, a code change).

- If piece B needs A's result, merge them or run B after A.
- Too small for a session (one edit, one lookup)? Give it to a Haiku subagent (low effort). Do not do it yourself.
- Pieces that change the same files must not run together. Use `use_worktree: true` (a separate copy of the code) or merge them.
- One indivisible task? Say so, then send it whole to one session or subagent at the right model. Do not do it yourself.
- Split thinking from checking. Mechanical steps (browser checks, searches, running tests, reading logs) go to a cheaper model.

## 3. Pick model and effort

Judge by how much thinking a piece needs, not by length. Use the cheapest row that works.

| Piece | Model | Effort |
|---|---|---|
| Search, read, summarize, list, rename, simple edits, browser checks | Haiku (`claude-haiku-5-5`) | low |
| Normal building, tests, docs, clear tasks | Sonnet (`claude-sonnet-5-5`) | medium |
| Debugging (finding and fixing a bug) | Sonnet (`claude-sonnet-5-5`) | medium |
| Changes across many files, unclear requirements | Sonnet or Opus (`claude-opus-5-5`) | high |
| Design decisions, security, hard-to-undo work | strongest allowed | high or xhigh |

`start_session` only accepts the same model family as the session starting it, or cheaper, and the same effort or lower. Higher requests are silently ignored. If a piece deserves more, say so in the summary and suggest running `/assign` from a stronger session.

Put only the part that needs Opus on Opus, not the whole task.

## 4. Subagent or session

- **Subagent** (`Agent` tool): small, read-only or mechanical, you need the answer to continue, no file or branch of its own. Set `model` (`haiku`, `sonnet`, `opus`) and `effort` explicitly, or it copies yours. Start independent ones in one message so they run together.
- **Session** (`start_session`): big, long, edits files, or the user may want to open it.
- Unsure: subagent for reading, session for editing.

## 5. Start sessions

Call `mcp__ccd_session__start_session` (if deferred, load with ToolSearch `select:mcp__ccd_session__start_session`). Use `initiation: "user_asked"`, `context: "fresh"`, model and effort from step 3, a short `name` (like `api-tests`), and `wake_when_done: true`.

Keep each brief short: where to work, what "done" looks like, any limits. A fresh session knows nothing else, so give only what it needs, not the whole conversation. In a shared folder, tell it to touch only its own files and commit nothing, unless it has its own worktree.

End every brief with these rules:

> **Save tokens.** Do the hard thinking yourself. Give easy steps to something cheaper: a Haiku subagent (low effort) for small checks and lookups, or a new session on a cheaper model than yours for bigger steps. Set model and effort every time. Never use your own model or a stronger one for easy steps. Sessions you start may use subagents but must not start more sessions.
>
> **Report short, in plain words.** Reply in at most 5 lines: done / partly done / blocked; what you did, in one sentence; what is left; where the result is; anything I must decide. No step lists, no code, no jargon. Pass these rules into any session you start.

Before each call, write a bold title and one sentence on what it will do. Starting several? Call `mcp__ccd_session__mark_chapter` first with a short batch title.

## 6. Report

After starting, show a short table: piece (sessions linked as `[title](#local_<id>)`), session or subagent, model, effort, and a few plain words on why. Then stop. Do not poll.

When sessions finish, check the results before saying it is done. Do not read long output yourself: send a Haiku subagent (low effort) to check the changes and answer in a line or two. Work in worktrees lives in its own branch, not in this folder yet. Say where each piece is and ask before combining them. When the user says to combine, or the pieces must be put together into one result, hand that to a Sonnet session (medium effort) with a short brief. Do not combine them yourself.

Final summary: one line per piece (status, where it is), then anything needing the user's decision. Under 150 words, plain language, no repeating the sessions' reports. Offer more detail only if asked.
