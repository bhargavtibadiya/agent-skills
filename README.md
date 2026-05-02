# agent-skills

A collection of reusable skills for AI coding agents — works with Claude Code, Cursor, GitHub Copilot, Cline, Windsurf, and 20+ other agents via [skills.sh](https://skills.sh).

## Install

Install all skills at once:

```bash
npx skills add bhargavtibadiya/agent-skills
```

Or install a specific skill:

```bash
npx skills add bhargavtibadiya/agent-skills/commit
```

## Skills

| Skill | Description |
|-------|-------------|
| [`commit`](skills/commit/SKILL.md) | Generate and run conventional commits with strict staging rules and AI co-author trailers |

## Structure

Each skill lives in `skills/<name>/SKILL.md` and contains agent instructions.

```
skills/
└── <skill-name>/
    └── SKILL.md
```

## Credits

Created and maintained by **[Bhargav Tibadiya](https://github.com/bhargavtibadiya)**.

## License

MIT
