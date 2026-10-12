# decipher

**Plain English for dense agent output.** decipher is a Claude Code plugin and Agent Skill that translates an agent-written spec, or any other technical document, into plain English an experienced engineer can follow on the first read. It keeps every identifier, number, and caveat.

[![validate](https://github.com/amk9978/decipher/actions/workflows/validate.yml/badge.svg)](https://github.com/amk9978/decipher/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-D97757.svg)](#install)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/hero-dark.png">
  <img alt="A 1,323-word agent-written spec on the left, and the opening of decipher's 402-word reply on the right, starting with its Gist line" src="assets/hero-light.png" width="880">
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

decipher fixes the reply you already have. Every explanation opens with a Gist, a few word phrases that carry the whole idea, and then the same fixed sections (Bottom line, Terms, Needs you, Self-check, How it works, What changes, Held back). The fixed layout teaches you where to look, and the reply stays under about 400 words however long the source is. decipher defines each term before the text uses it, puts events in the order they happened, and keeps what the source claims apart from what anyone verified.

The Self-check is one question built from the document's own premise, which you answer to yourself. It asks for your judgment on the matter, so you can't answer it by scrolling back. If you can't answer it, you have stopped following, and that is the moment to step back before you approve anything.

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

A real run on a 1,323-word agent-written spec for link previews in LiCode, a code-review practice app. The reply is 402 words.

```text
/decipher specs/link-previews.md
```

Opening of the reply:

**Gist.** shared links become cards, case question not answer, images drawn ahead and committed, browser tab icon, approval and previews pending

**Bottom line.** The spec makes a shared LiCode link show a card with a picture, a title and one line of text, instead of a bare URL. Nothing is built yet, because it waits for your approval and for you to see image previews. The only design choice that isn't routine is drawing the card images ahead of time.

<details>
<summary>Full reply</summary>

**Gist.** shared links become cards, case question not answer, images drawn ahead and committed, browser tab icon, approval and previews pending

**Bottom line.** The spec makes a shared LiCode link show a card with a picture, a title and one line of text, instead of a bare URL. Nothing is built yet, because it waits for your approval and for you to see image previews. The only design choice that isn't routine is drawing the card images ahead of time.

**Terms.**
- **card**: the box a chat or social app shows for a pasted link
- **crawler**: the program Slack, LinkedIn or X sends to read a pasted page
- **manifest**: a file recording the text each committed image was drawn from

**Needs you.**
- Approve the spec, or tell me what to change.
- Say whether I should render previews of the site card, a case card, a track card and the tab icon first.
- After deploy, Cloudflare's bot protection might block the crawlers without any error showing, so the PR will record checks in each platform's inspector.

**How it works.**
1. A person pastes a LiCode link, and the app's crawler fetches that page without running JavaScript.
2. LiCode's server adds the card details to the page it sends, because React's tags would arrive too late for a crawler.
3. The server picks the details from the address. A case link gets its question, a track link gets its destination, and every other page gets the site card.
4. A card shows only what the page shows before anyone answers, so it never reveals an answer, outcome or sender.
5. The server can't draw images, because production has no browser or image library, so a script draws every image ahead of time.
6. A test fails if a question changes after its image was drawn, which forces a redraw before release.

**What changes.**
- Links unfurl everywhere, and a shared case reads as a challenge, which the share buttons spec depends on.
- Apps cache a card for about a week, so a changed image ships under a new versioned filename.
- Images stay under 300 KB, because WhatsApp drops larger ones.
- The browser tab gets an icon, and /favicon.ico stops returning the page's HTML.

**Held back.** The exact card text for each page kind, the image layout, escaping rules, the no-public-address local behaviour and the file list.

</details>

## How it works

The skill holds every rule. The output style is a short switch that tells Claude to load the skill at the start of each session and again after compaction. While the rules apply, the fixed format's limits replace brevity budgets from other output styles, `CLAUDE.md`, and `AGENTS.md`. Accuracy and punctuation rules from the host still apply.

## Feedback

Found a reply decipher left unclear? [Open an issue](https://github.com/amk9978/decipher/issues/new?template=unclear-reply.yml) with the original text, decipher's reply, and the point where you lost the thread. Real failures are how the rules improve.

## License

[MIT](LICENSE)
