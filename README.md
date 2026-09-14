# Codex Skills

Reusable skills for ChatGPT work and Codex.

## Orchestrate

[`orchestrate`](skills/orchestrate) helps Codex delegate bounded work, choose workers suited to its difficulty, and integrate their results. It uses completion notifications and permits ending a pending turn only when the runtime supports waking the coordinator. Small, tightly coupled tasks stay with the coordinator.

The pattern comes from [Practical multi-agent orchestration in Codex](https://x.com/pvncher/status/2080707291603407077).

### Install

Copy the skill into your personal Codex skills directory:

```sh
mkdir -p ~/.codex/skills
cp -R skills/orchestrate ~/.codex/skills/orchestrate
```

Then invoke it explicitly with `$orchestrate`, or let Codex select it when a task calls for multi-agent coordination.

## License

MIT

## Releases

The complete Orchestrate Pack is published as an immutable GitHub Release tagged
`pack-v<version>`. Its root `pack.json` declares the exact reviewed closure.
