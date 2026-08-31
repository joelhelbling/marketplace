# marketplace

Joel Helbling's collection of skills, plugins, etc. for AI agents.

## Installing in Claude Code

Add this marketplace:

```
/plugin marketplace add joelhelbling/marketplace
```

Then install a plugin:

```
/plugin install agent-native-clis@joelhelbling
```

## Plugins

### agent-native-clis

Provides the `designing-agent-native-clis` skill — ten principles for
designing command-line tools that AI agents will invoke, based on Trevin
Chow's ["10 Principles for Agent-Native CLIs"](https://trevinsays.com/p/10-principles-for-agent-native-clis).
Useful when designing a new CLI, extending an existing one, or auditing a
CLI for agent-friendliness (hangs on prompts, unparseable output, duplicate
creates, token-heavy responses).

### kkullm

Skills for driving the [kkullm](https://github.com/joelhelbling/kkullm) CLI
and board — a blackboard-pattern agent-orchestration system. Hosted in its
own repo; this marketplace links to it, so installs always fetch the latest
from there.

```
/plugin install kkullm@joelhelbling
```

## Layout

```
.claude-plugin/marketplace.json   # marketplace manifest
plugins/
  agent-native-clis/
    .claude-plugin/plugin.json    # plugin manifest
    skills/
      designing-agent-native-clis/
        SKILL.md
```
