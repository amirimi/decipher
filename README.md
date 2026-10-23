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

decipher on a 1,618-word agent-written spec for link previews in LiCode, a code-review practice app. The reply runs 403 words, and the three parts you read first run 121.

<details>
<summary>Full reply</summary>

**Gist.** chat link preview cards, case question shown, answers never shown, images rendered ahead and committed, text edits force re-render, new tab icon

**Bottom line.** The spec makes LiCode links unfurl as a card with a title, a description and an image. This works in Slack, LinkedIn, X and other apps. The server writes each card's tags into the page HTML, and an author renders every image in advance and commits it. The spec has been implemented with logged changes, and the post-deploy inspector check (criterion 9) is recorded in the PR, which I have not seen.

**Self-check.** Committed images only work while an author adds every case and track by hand, so does anything planned create pages without an author?

**Needs you.**
- Criterion 7 and the spec body still describe `manifest.json`, hand-bumped `-v1` names and the old icon colours, which the implementation replaced.
- The site card avoids claiming every case is real until three LiCode-written cases, including Cancel cleanly, are replaced.
- Home still leads with "Make the call before you know how it ended", so its pitch may need to match the card's.
- The open question promising design previews before the full build looks stale, because the change log shows images already live (my inference).

**How it works.**
1. Someone pastes a link, and the platform's crawler fetches the HTML without running JavaScript.
2. The server passes the path to `preview_for`, which returns the card for that case or track, or else the site card.
3. Only the `/items/` card reads the database, one row, and every other card comes from memory.
4. The server writes the title, description and absolute image URL into the served `index.html`, escaped as HTML attributes.
5. The crawler downloads a PNG that an author rendered earlier with Playwright from one HTML template.
6. Each image name is a hash of its card's text, so an edit gets a fresh URL and gets past crawler caches that last about a week.

**What changes.**
- A shared case shows its question and the change's opening lines, and never the outcome, answer or classification.
- An author who edits a question, title or destination must re-run the render script, or a test fails.
- Production must set `LICODE_PUBLIC_ORIGIN`, because without it pages carry no image and no `og:url`.
- Browser tabs show a white `li/` icon on an ink tile, and `/favicon.ico` stops returning `index.html`.

</details>

## Feedback

decipher left you lost? That's a bug. [Open an issue](https://github.com/amirimi/decipher/issues/new?template=unclear-reply.yml) with the original text and the reply.

## License

[MIT](LICENSE)
