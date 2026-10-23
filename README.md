<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/banner-dark.png">
    <img alt="decipher. Your agent writes novels. You read the gist." src="assets/banner-light.png" width="100%">
  </picture>
</p>

**Gist.** agent-written specs decoded, gist first, plain English after, one unskimmable question, every caveat kept

**Bottom line.** Your agent writes a 1,600-word spec, and you approve it after a skim. decipher rewrites any agent-written document so its first three parts carry the whole idea in plain English, in about 120 words. The rest waits below as reference, and anything it skips is named.

**Self-check.** You approved your agent's last spec. Could you name its riskiest assumption without opening it?

[![validate](https://github.com/amirimi/decipher/actions/workflows/validate.yml/badge.svg)](https://github.com/amirimi/decipher/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-D97757.svg)](#install)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/hero-dark.png">
  <img alt="A 1,618-word agent-written spec on the left, and the first three parts of decipher's reply on the right, its Gist, Bottom line, and Self-check" src="assets/hero-light.png" width="880">
</picture>

## Install

decipher is listed in [Claude's plugin directory](https://claude.ai/customize/plugins/id/cba64762-9a2d-42c8-914b-859aaa270617%40anthropic-plugin-directory) and on [cursor.directory](https://cursor.directory/plugins/decipher).

**Claude Code** gets the skill and an always-on output style.

```text
/plugin marketplace add amirimi/decipher
/plugin install decipher@decipher
```

**Codex, OpenCode, and Cursor** each have their own install in [INSTALL.md](INSTALL.md). So does any other agent that reads `SKILL.md`.

Tested with Claude Code 2.1.302, Codex 0.149.1, OpenCode 1.19.3, and the Cursor CLI 2026.10.19.

## Use

Point it at a document, or say "decipher" after a reply that lost you.

```text
/decipher:decipher specs/link-previews.md
```

A clone install shortens the command to `/decipher`, and Codex uses `$decipher`.

| Say | Get |
|---|---|
| decipher, simplify this | The latest reply or a named document, rewritten |
| decipher, again on the same document | A new angle with none of the earlier sentences |
| walk me through a spec, PR, or report | Three to five parts, one per turn, with a bookmark |
| continue, back, all at once, stop | Moves through the walkthrough |
| I'm lost | The goal again, from a different angle |

Specs and design docs are home turf. PRs, reports, incident write-ups, READMEs, and agent replies work too.

## Every answer, same shape

```text
Gist.                a few words that carry the whole idea
Bottom line.         what it does, where it stands, what backs it
Self-check.          one question you answer to yourself
-------------------  most readers stop here
Terms.               only when the jargon would lose you
Needs you.           decisions, blockers, risks
How it works.        the steps, in order
What changes.        who and what feels it
Make sure you know:  a foundation you may be missing, only when your questions show it
```

The Self-check is the point. It comes from the document's own premise and asks for your judgment, so scrolling back won't answer it. Can't answer it? You've stopped steering. Take the wheel back before you approve anything.

When a follow-up shows you are missing a concept the document never explains, the last line names it. Learn that first, and the document reads differently.

## "Can't I just tell Claude to be concise?"

Go ahead. A concision rule shapes the next thing your agent writes. The 1,600-word spec already in your repo stays 1,600 words, and so does the PR a teammate's agent opened. decipher works on what already exists. It defines the jargon from the source, keeps what the document claims apart from what anyone verified, and checks that you still follow.

## Always on

Always-on mode puts every reply in plain English, and decipher's format still waits until you ask for an explanation.

**Claude Code** turns it on with the plugin's output style. Select it in `~/.claude/settings.json`. A plugin install names it `decipher:decipher`, and a clone install names it `decipher`.

```json
{ "outputStyle": "decipher:decipher" }
```

**Every other agent** turns it on with this block in the instruction file it reads every session.

```markdown
## decipher
Use decipher's always-on mode. Before your first reply in a session, load the decipher skill and follow its rules for always-on mode. Explain a source in decipher's format only when I ask. Load the skill again whenever its rules leave your context.
```

| Agent | Instruction file |
|---|---|
| Codex | `~/.codex/AGENTS.md` |
| OpenCode | `~/.config/opencode/AGENTS.md` |
| Cursor | `.cursor/rules/decipher.mdc` in each repository, with `alwaysApply: true` in its front matter |

While decipher's rules apply, its own limits replace brevity budgets from other output styles, `CLAUDE.md`, and `AGENTS.md`. Your accuracy and punctuation rules still win.

## A real run

decipher on a 1,618-word agent-written spec for link previews in LiCode, a code-review practice app. The reply runs 424 words, and the three parts you read first run 113.

<details>
<summary>Full reply</summary>

**Gist.** links unfurl as cards, answers never shown, images prerendered and committed, text edits force rerender, tab icon replaces globe

**Bottom line.** The spec makes every shared LiCode link unfurl into a preview card and gives the site a browser-tab icon. Its implementation notes describe the work as built, with images now named by a digest of their card's text. Support comes from crawler checks against the live site, a LinkedIn render, and a largest image of 162 KB against WhatsApp's 300 KB limit.

**Self-check.** An author renders and commits every card image before deploy, assuming every item page already exists by then. Can items reach your production database without a code change?

**Needs you.**
- Have the agent reconcile the spec body with its notes, because the body still describes a manifest that the digest naming replaced. That stale text appears in Scope, the stale-images constraint, and criterion 7.
- Check whether you approved the image designs, because the spec promised previews first, but the notes show every card already rendered.
- Check that the PR records the post-deploy Facebook, LinkedIn, and Slack results (criterion 9), because the notes mention only a pre-deploy `curl` check.
- Decide whether the site card's careful wording stays, because three authored cases stop it from claiming every change is real.
- Decide whether Home adopts the card's positioning, because Home still leads with "Make the call before you know how it ended."

**How it works.**
1. Someone pastes a LiCode link into Slack, LinkedIn, X, WhatsApp, iMessage, or Discord.
2. That platform's crawler fetches the HTML without running JavaScript and reads the Open Graph tags in its head.
3. The server passes the path to `preview_for`, which returns a case card, a track card, or the site card.
4. The server writes that card's escaped tags into `index.html`, adding image and page URLs only when `LICODE_PUBLIC_ORIGIN` is set.
5. The crawler downloads the named PNG, which an author rendered earlier with Playwright and committed.
6. Each PNG's name changes whenever its card's text changes, so crawler caches never serve an old image.

**What changes.**
- A shared case shows its question but never its outcome, classification, motif, answer, or result.
- The tab shows a white `li/` mark on an ink tile, and `/favicon.ico` stops returning `index.html`.
- Every page's `description` tag changes from "Real code changes. Your call before hindsight." to its card's description.
- Authors must rerun the render script after editing a question, title, or destination, because a test fails until the image exists.

</details>

## Feedback

decipher left you lost? That's a bug. [Open an issue](https://github.com/amirimi/decipher/issues/new?template=unclear-reply.yml) with the original text and the reply.

## License

[MIT](LICENSE)
