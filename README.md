```bash
███████ ██  ██████      ███████ ██   ██ ██ ██      ██      ███████ 
   ███  ██ ██    ██     ██      ██  ██  ██ ██      ██      ██      
  ███   ██ ██    ██     ███████ █████   ██ ██      ██      ███████ 
 ███    ██ ██    ██          ██ ██  ██  ██ ██      ██           ██ 
███████ ██  ██████      ███████ ██   ██ ██ ███████ ███████ ███████ 
                                                                   
```

> ZIO coding rules, anti-patterns, and decision guides — as a Claude Code skill plugin.

## Install

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

## What's included

- **Coding rules** — error handling, composition, service pattern, resources, concurrency
- **Anti-patterns** — 8 common mistakes with correct alternatives
- **Decision guide** — which abstraction to use (Ref, Queue, Hub, STM, Semaphore, ZStream…)
- **Key patterns** — service pattern, repository, background worker, retry, HTTP routes

## License

MIT — Watson Dinh
