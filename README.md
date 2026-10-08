# decipher

**Plain English for dense agent output.** decipher is a Claude Code plugin and Agent Skill that translates an agent-written spec, or any other technical document, into plain English an experienced engineer can follow on the first read. It keeps every identifier, number, and caveat.

[![validate](https://github.com/amk9978/decipher/actions/workflows/validate.yml/badge.svg)](https://github.com/amk9978/decipher/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-D97757.svg)](#install)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/hero-dark.png">
  <img alt="A dense agent reply full of undefined terms on the left, and decipher's plain explanation of it on the right" src="assets/hero-light.png" width="880">
</picture>

## Install

**Claude Code** gets the skill and the output style:

```text
/plugin marketplace add amk9978/claude-plugins
/plugin install decipher@amk9978
```

**Other agents that read `SKILL.md`** get the skill:

```bash
npx skills add amk9978/decipher
```

Tested with Claude Code 2.1.284.

<details>
<summary>Install from a clone for live editing</summary>

```bash
git clone https://github.com/amk9978/decipher ~/decipher
ln -s ~/decipher/skills/decipher ~/.claude/skills/decipher
mkdir -p ~/.claude/output-styles
ln -s ~/decipher/output-styles/decipher.md ~/.claude/output-styles/decipher.md
```

</details>

## The problem

Agent replies drift into shorthand. They use labels the agent invented, such as "Path B", and project terms nobody defined. They list files before saying what changed, and they report "31/31 green" without saying who ran the tests. A long report loses you by the third section, and a follow-up question loses your place.

decipher fixes the reply you already have. Every explanation opens with a Gist, a few word phrases that carry the whole idea, and then the same six sections (Bottom line, Terms, Needs you, How it works, What changes, Held back). It stays under about 400 words however long the source is, so you learn where to look. It defines each term before the text uses it, puts events in the order they happened, and keeps what the source claims apart from what anyone verified.

## Why not just ask for shorter output?

A concision rule in `CLAUDE.md` or a planning plugin changes what an agent writes next. decipher works on a document that already exists, including one a teammate, another tool, or an earlier session wrote. It rewrites that document for a reader who lost the thread. It defines the source's terms from the source, puts events in time order, separates what the document claims from what anyone verified, and keeps your place through a long read. The output gets shorter only because it leaves out what you do not need.

## Use

decipher works best on agent-written specs and design docs. It also handles PRs, reports, incident write-ups, READMEs, and agent replies. Say one of these after a document you could not follow.

| You say | decipher does |
|---|---|
| simplify this, explain that again | Rewrites the latest reply, or a passage you name, in plain English |
| walk me through a PR, report, or spec | Maps it into three to five parts and explains one part per turn |
| continue, back, all at once, stop | Moves through a walkthrough |
| I'm lost | Rebuilds the explanation from the goal with a different example |

Every walkthrough turn ends with a bookmark footer. Side questions leave the bookmark where it was, and your unanswered questions stay under **Open** until they are answered.

```markdown
**Paused at:** part 2 of 4, retry logic
**Open:** why the lock is per-tenant (+1 more)
**Next:** part 3 of 4, the migration
```

### Plain replies on every turn

The plugin and the clone install also ship an output style that applies decipher's writing rules to every reply. Select it in `~/.claude/settings.json`. A plugin install names it `decipher:Decipher`, and a clone install names it `Decipher`.

```json
{ "outputStyle": "decipher:Decipher" }
```

## Example

Input from a real `claude -p` run:

```text
Simplify this, I'm lost:

Root cause: reconciler watch cache went stale after informer resync was bumped to 0, so the lease-holder never saw the CR update and the finalizer blocked GC. Patched resync back to 10m + explicit requeue on Conflict. 31/31 green. Webhook still double-admits on HA (2 replicas, no leader gate), out of scope, tracked in #412.
```

Opening of the reply:

> Deleted custom resources got stuck because the controller stopped re-checking objects after a missed update. The fix turns periodic re-checking back on and adds a retry. A separate webhook bug is still open.

<details>
<summary>Full reply</summary>

Deleted custom resources got stuck because the controller stopped re-checking objects after a missed update. The fix turns periodic re-checking back on and adds a retry. A separate webhook bug is still open.

**What broke, in order**

1. The informer's resync period was set to 0. The informer is the client-go component that keeps a local copy of the custom resources and passes changes to the controller. Resync makes the controller re-check every object on a timer, even when nothing new arrives, and a period of 0 turns it off.
2. The lease-holder missed an update to a custom resource. The lease-holder is the one controller replica that holds the leader lock and does the actual work. With resync off, nothing ever made it look at that object again.
3. The object had a finalizer. A finalizer is a marker that blocks deletion until the controller finishes its cleanup and removes the marker. The controller never ran that cleanup, so the finalizer stayed on.
4. Kubernetes garbage collection could not delete the object while the finalizer was there. I infer it sat in the Terminating state.

**The fix**

- Resync is back to 10 minutes. The controller now re-checks every object at least that often.
- Some writes fail with a Conflict error. The API server returns that error when the object changed after the controller read it. The controller now puts the object back in its queue so it tries again.

The report says all 31 tests pass. I have not run them.

**A caveat on the diagnosis (my reading, not the report's)**

In client-go, resync replays the local copy and does not fetch fresh data from the API server. Resync alone therefore cannot repair a local copy that is out of date. It only retries objects the controller skipped. The Conflict retry is more likely the part that handles a controller acting on an old version. Check the phrase "watch cache went stale" before it goes into a postmortem.

**Still open**

The admission webhook is the service the API server calls to approve each request. It runs as 2 replicas for high availability, and it has no leader election to pick one of them. The report says this makes it "double-admit" but does not say what harm that causes. This fix leaves it alone, and #412 tracks it.

</details>

## How it works

The skill holds every rule. The output style is a short switch that tells Claude to load the skill at the start of each session and again after compaction. While the rules apply, the fixed format's limits replace brevity budgets from other output styles, `CLAUDE.md`, and `AGENTS.md`. Accuracy and punctuation rules from the host still apply.

## Feedback

Found a reply decipher left unclear? [Open an issue](https://github.com/amk9978/decipher/issues/new?template=unclear-reply.yml) with the original text, decipher's reply, and the point where you lost the thread. Real failures are how the rules improve.

## License

[MIT](LICENSE)
