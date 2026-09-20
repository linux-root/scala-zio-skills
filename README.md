```bash
███████ ██  ██████      ███████ ██   ██ ██ ██      ██      ███████ 
   ███  ██ ██    ██     ██      ██  ██  ██ ██      ██      ██      
  ███   ██ ██    ██     ███████ █████   ██ ██      ██      ███████ 
 ███    ██ ██    ██          ██ ██  ██  ██ ██      ██           ██ 
███████ ██  ██████      ███████ ██   ██ ██ ███████ ███████ ███████ 
                                                                   
```

> ZIO coding rules, anti-patterns, and decision guides — as a Claude Code skill plugin.

## Install

### Any agent — one command (recommended)
```bash
npx skills add linux-root/scala-zio-skills
```
Uses the [skills CLI](https://github.com/vercel-labs/skills); auto-detects your agent. Add `-g` to install globally instead of into the current project.

No Node/npx? Use the per-agent instructions below.

### Claude Code
**Step 1** — Add the marketplace source:
```
/plugin marketplace add linux-root/scala-zio-skills
```
**Step 2** — Install the plugin:
```
/plugin install zio-reference@scala-zio-skills
```

### Opencode
Open Opencode and send it this prompt:

```
Follow the instructions at https://raw.githubusercontent.com/linux-root/scala-zio-skills/refs/heads/main/.opencode/INSTALL.md and install the skill.
```

### GitHub Copilot
Open Copilot and send it this prompt:

```
Follow the instructions at https://raw.githubusercontent.com/linux-root/scala-zio-skills/refs/heads/main/.copilot/INSTALL.md and install the skill.
```

Once installed, the skill auto-triggers whenever you're writing ZIO Scala code.

## Quick test

Open an existing ZIO project, load the skill with `/zio-reference`, then ask:

```
Given the ZIO knowledge from zio-reference, scan this project
and find areas for easy-win improvements.
```

You should get concrete hits — `Task` used for business logic, `ensuring` where
`acquireRelease` belongs, blocking calls wrapped in `ZIO.attempt`, and so on.

## What's included

- **Coding rules** — error handling, composition, service pattern, resources, concurrency
- **Anti-patterns** — 8 common mistakes with correct alternatives
- **Decision guide** — which abstraction to use (Ref, Queue, Hub, STM, Semaphore, ZStream…)
- **Key patterns** — service pattern, repository, background worker, retry, HTTP routes

## License

MIT — Watson Dinh
