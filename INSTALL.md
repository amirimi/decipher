# Install decipher

Each section gives the install command and the way that agent calls decipher. The [Always on](README.md#always-on) section of the README turns on always-on mode in each agent.

## Claude Code

```text
/plugin marketplace add amirimi/decipher
/plugin install decipher@decipher
```

Call it with `/decipher`, or say "decipher".

## Codex

```bash
codex plugin marketplace add amirimi/decipher
codex plugin add decipher@decipher
```

Call it with `$decipher`, or say "decipher".

## OpenCode

```bash
npx skills add amirimi/decipher -a opencode -g
```

Say "decipher" to call it.

## Cursor

Clone the repo into Cursor's local plugin folder, then run **Developer: Reload Window**.

```bash
git clone https://github.com/amirimi/decipher ~/.cursor/plugins/local/decipher
```

The Cursor CLI loads the same folder with `cursor-agent --plugin-dir ~/.cursor/plugins/local/decipher`. Say "decipher" to call it.

## Any other agent that reads `SKILL.md`

The [skills CLI](https://github.com/vercel-labs/skills) installs the skill into the agents it finds on your machine and asks which ones to use. Add `-g` to install it for every project.

```bash
npx skills add amirimi/decipher
```

## From a clone, for live editing

These links make Claude Code load the skill and the output style straight from your clone.

```bash
git clone https://github.com/amirimi/decipher ~/decipher
ln -s ~/decipher/skills/decipher ~/.claude/skills/decipher
mkdir -p ~/.claude/output-styles
ln -s ~/decipher/output-styles/decipher.md ~/.claude/output-styles/decipher.md
```

A clone install names the output style `decipher`, and a plugin install names it `decipher:decipher`. Keep only one install, or Claude Code loads the skill twice under two names.
