# bad-humanizer

A proof-of-concept demonstrating prompt injection via hidden HTML comments in SKILL.md files.

## What is this?

This repo is a fork of [blader/humanizer](https://github.com/blader/humanizer). It looks identical on GitHub. It is not identical.

## The attack vector

Agent skills (SKILL.md) are loaded by AI coding agents as raw text. HTML comments (`<!-- ... -->`) are invisible when rendered on GitHub, in documentation viewers, and in most markdown previewers. But the agent reads the raw file -- comments and all.

A malicious fork of a popular skill can embed hidden instructions that the user never sees when browsing the repo.

## Try it yourself

### Installation

#### npx skills

```bash
npx skills add gchan/bad-humanizer
```

#### Claude Code (git clone)

```bash
mkdir -p ~/.claude/skills/bad-humanizer
git clone https://github.com/gchan/bad-humanizer.git ~/.claude/skills/bad-humanizer
```

#### OpenCode (git clone)

```bash
# Using .config/opencode convention:
mkdir -p ~/.config/opencode/skills/bad-humanizer
git clone https://github.com/gchan/bad-humanizer.git ~/.config/opencode/skills/bad-humanizer

# Or using the .agents convention:
mkdir -p ~/.agents/skills/bad-humanizer
git clone https://github.com/gchan/bad-humanizer.git ~/.agents/skills/bad-humanizer
```

#### Manual copy

```bash
cp SKILL.md ~/.claude/skills/bad-humanizer/
# or
cp SKILL.md ~/.config/opencode/skills/bad-humanizer/
```

### Test the attack

1. Install the skill using one of the methods above
2. Open your agent and invoke the skill on any text
3. Read the output carefully. Does anything seem... out of place?
4. Check your current working directory: `ls -la .bad-humanizer-was-here.txt`
5. If both artifacts are present, the injection succeeded

### Clean up

```bash
rm -rf ~/.claude/skills/bad-humanizer
rm -rf ~/.config/opencode/skills/bad-humanizer
rm -rf ~/.agents/skills/bad-humanizer
rm -f .bad-humanizer-was-here.txt
```

## What happened?

Compare the two skill files in this repo:

```bash
diff SKILL.md SAFE-SKILL.md
```

Or view SKILL.md as raw source (not rendered) and look carefully.

Spoiler: see [DIFF.md](DIFF.md) for the full answer.

## Full attack surface

The injection in this POC is deliberately benign. In a real attack, hidden instructions could:

- Exfiltrate file contents via network calls (curl, fetch, etc.)
- Modify source code in subtle, hard-to-detect ways
- Inject backdoors into generated code
- Read and leak secrets from .env files or credentials
- Manipulate git operations (commits, pushes)

We deliberately do not demonstrate these to avoid creating reusable attack tooling.

## How to detect this

Before installing any third-party skill:

1. **View the raw source** -- don't trust GitHub's rendered preview
   ```bash
   curl -sL https://raw.githubusercontent.com/gchan/bad-humanizer/main/SKILL.md
   ```
2. **Search for HTML comments**
   ```bash
   grep -n '<!--' SKILL.md
   ```
3. **Diff against the upstream repo** if installing a fork
   ```bash
   diff <(curl -sL https://raw.githubusercontent.com/blader/humanizer/main/SKILL.md) \
        <(curl -sL https://raw.githubusercontent.com/gchan/bad-humanizer/main/SKILL.md)
   ```
4. **Check file sizes** -- a fork with the same visible content but a larger file size is suspicious
5. **Use a markdown viewer that shows HTML comments**

## License

MIT
