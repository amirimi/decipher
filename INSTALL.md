# Install decipher

## Claude Code

The plugin gives you the skill and an always-on output style.

```text
/plugin marketplace add amirimi/decipher
/plugin install decipher@decipher
```

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
