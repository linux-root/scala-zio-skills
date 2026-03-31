# Installing ZIO Skills for Opencode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed

## Installation

Clone this repo into your Opencode skills directory:

```bash
git clone https://github.com/linux-root/scala-zio-skills.git ~/.config/opencode/skills/scala-zio-skills
```

Then add the skill path to your global or project-level `opencode.json`:

```json
{
  "skills": ["~/.config/opencode/skills/scala-zio-skills/skills/zio-reference"]
}
```

Restart Opencode. The skill auto-triggers when you're writing ZIO Scala code.

## Updating

```bash
cd ~/.config/opencode/skills/scala-zio-skills && git pull
```

## Troubleshooting

### Skill not found

1. Verify the path in `opencode.json` is correct
2. Check that the directory exists: `ls ~/.config/opencode/skills/scala-zio-skills/skills/zio-reference/`

## Getting Help

- Report issues: https://github.com/linux-root/scala-zio-skills/issues
