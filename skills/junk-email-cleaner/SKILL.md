---
name: junk-email-cleaner
description: Finds the most recent promotional/marketing emails (up to 30) sitting in the user's Gmail inbox, presents a numbered shortlist showing sender, subject, and date, and only trashes them after the user explicitly confirms. Use this whenever the user wants to declutter, clean up, or manage promotional email in their Gmail inbox — phrases like "clean up my inbox", "clear out the promo emails", "get rid of all this marketing junk", or "my inbox is full of sales emails" should all trigger this skill, even if they don't say "promotional" or name Gmail explicitly. Requires a connected Gmail account.
compatibility: Gmail connector tools (search_threads, trash_thread or trash_message)
---

# Junk Email Cleaner

Finds promotional clutter in the user's Gmail inbox, shows exactly what would be removed, and only acts on explicit approval.

## Why the confirmation gate matters

Deleting mail is easy to do and easy to regret — even "obviously promotional" threads can have something worth keeping (a coupon the user meant to use, a receipt buried in a marketing digest). The shortlist-then-confirm flow is the actual safety mechanism here, not a formality. Skipping it, or acting on threads beyond what was shown and approved, defeats the point of the skill.

## Step 1: Find the candidates

Load the Gmail tools via `tool_search` if they aren't already available (`Gmail:search_threads`, `Gmail:trash_thread` / `Gmail:trash_message`).

Search with:
```
category:promotions in:inbox
```
Gmail already classifies "Promotions" mail for the user — that's a far more reliable signal than keyword-guessing.

Pull up to the 30 most recent matching threads, most recent first, using the minimal/lightweight view (sender, subject, date, snippet) — this is all the shortlist needs, so there's no reason to fetch full message content. If fewer than 30 exist, work with what's there and say so. If the search returns nothing, tell the user their Promotions inbox is already clean and stop — there's nothing to shortlist.

## Step 2: Present the shortlist

Show all candidates as a numbered list — sender, subject, date — so the user can reference specific ones ("skip #4 and #17"). Be upfront about the count — "here are the 30 most recent promotional emails in your inbox" or however many were actually found.

This step is read-only. Nothing gets trashed yet.

One thing worth knowing: Gmail's own Promotions category isn't a guarantee of "junk." It can catch things like community alerts, nonprofit appeals, or newsletter-ish mail that happened to get auto-classified there. Don't silently filter those out or flag them differently in the list — just present the shortlist honestly and let the user's own read of each entry (sender + subject) do the filtering, same as any other item on the list.

## Step 3: Get explicit confirmation

Ask what to do with the shortlist. If a tappable-choice tool (like `ask_user_input_v0`) is available, use it with exactly two options:
1. Delete
2. Not Delete

Otherwise ask in plain text with the same two choices.

Two things worth stating plainly at this point:
- "Delete" moves the threads to Gmail's Trash (via `trash_thread`/`trash_message`) — Gmail keeps trashed mail for 30 days before permanent removal, so this is recoverable, not instant and irreversible. Say this so the user isn't surprised later.
- The user can reply with adjustments instead of a flat choice — e.g., "delete all except #3 and #12." Treat that as a valid, specific answer and only act on what wasn't excluded.

## Step 4: Act only on what was approved

- **Not Delete** (or any decline): do nothing further, confirm nothing was touched.
- **Delete**: call `trash_thread` (or `trash_message`) only on the exact threads from the shortlist that weren't excluded. Never re-run the search and act on a fresh batch — only ever touch the specific thread IDs already shown and approved.
- Ambiguous reply (doesn't clearly map to "all", "none", or a specific subset): ask for clarification rather than guessing.

Afterward, report what happened — how many were trashed, and that they're recoverable from Trash for the usual retention window if the user changes their mind.

## Hard rules

- Never skip Step 3, even on a follow-up request that sounds like a standing instruction ("just always delete my promo emails from now on"). Inboxes change; get a fresh look-and-confirm each time this skill runs.
- Never act on a thread that wasn't part of the shortlist the user actually saw and approved.
- Never treat silence, a topic change, or a vague reply as approval to delete.
