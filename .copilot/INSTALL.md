# Installing ZIO Skills for GitHub Copilot

## Prerequisites

- GitHub Copilot with [Agent Skills](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills) support — recent VS Code + Copilot Chat, or the `copilot` CLI

## Installation

Copilot discovers personal skills in `~/.copilot/skills/<name>/`. Clone this repo and
link the skill directory into place:

```bash
git clone https://github.com/linux-root/scala-zio-skills.git ~/.copilot/scala-zio-skills
mkdir -p ~/.copilot/skills
ln -s ~/.copilot/scala-zio-skills/skills/zio-reference ~/.copilot/skills/zio-reference
```

Link the whole directory, not just `SKILL.md` — the skill points at `references/*.md`
for its deep dives.

Reload VS Code. The skill auto-triggers when you're writing ZIO Scala code.

**Project-scoped instead?** Link into a repo's `.github/skills/` rather than `~/.copilot/skills/`.

## Updating

```bash
cd ~/.copilot/scala-zio-skills && git pull
```

The symlink picks up the change — no re-install.

## Troubleshooting

### Skill not found

1. Check the link resolves: `ls -l ~/.copilot/skills/zio-reference/`
   You should see `SKILL.md` and `references/`.
2. Reload VS Code (`Developer: Reload Window`), then type `/` in Copilot Chat and look for `zio-reference`.
3. Confirm your Copilot build supports Agent Skills — it is a recent addition.

## Getting Help

- Report issues: https://github.com/linux-root/scala-zio-skills/issues
