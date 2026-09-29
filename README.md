# decipher

A Claude Code skill that turns dense agent output into plain, precise English. It explains one reply at a time, or walks through a long PR, report, or spec part by part with a bookmark that survives your questions.

## Install

Clone the repository and link both files into Claude Code.

```bash
git clone https://github.com/amk9978/decipher ~/decipher
ln -s ~/decipher/skills/decipher ~/.claude/skills/decipher
mkdir -p ~/.claude/output-styles
ln -s ~/decipher/output-styles/decipher.md ~/.claude/output-styles/decipher.md
```

Running `git pull` in `~/decipher` then updates both. To install only the skill, `npx skills add amk9978/decipher -g` also works. Both routes need read access to the repository while it is private.

## Use

Say one of these after a reply you could not follow.

| You say | Decipher does |
|---|---|
| simplify this, explain that again | Rewrites the latest reply, or a passage you name, in plain English |
| walk me through a PR, report, or spec | Maps it into three to five parts and explains one part per turn |
| continue, back, all at once, stop | Moves through a walkthrough |
| I'm lost | Rebuilds the explanation from the goal with a different example |

Every walkthrough turn ends with a footer that records where you paused, which of your questions are still open, and what comes next.

For plain replies on every turn, select the output style in `~/.claude/settings.json`:

```json
"outputStyle": "Decipher"
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

The skill holds every rule. The output style is a one-line switch that tells Claude to read the skill at the start of each session and again after compaction. While the rules apply, they override brevity budgets from other output styles and CLAUDE.md files. Accuracy and punctuation rules from the host still apply.

## License

[MIT](LICENSE)
