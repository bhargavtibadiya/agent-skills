# agent-skills

A collection of reusable skills for AI coding agents — works with Claude Code, Cursor, GitHub Copilot, Cline, Windsurf, and 20+ other agents via [skills.sh](https://skills.sh).

## Install

Install all skills at once:

```bash
npx skills add bhargavtibadiya/agent-skills
```

Or install a specific skill:

```bash
npx skills add bhargavtibadiya/agent-skills --skills commit
npx skills add bhargavtibadiya/agent-skills --skills coding-guidelines
```

## Skills

| Skill | Description |
|-------|-------------|
| [`commit`](skills/commit/SKILL.md) | Generate and run conventional commits with strict staging rules and AI co-author trailers |
| [`coding-guidelines`](skills/coding-guidelines/SKILL.md) | Enforce 15 mandatory backend TypeScript/Node.js rules — Cloudinary storage, soft deletes, Swagger sync, route ordering, strict typing, error handling, and more |

## Structure

Each skill lives in `skills/<name>/SKILL.md`. Complex skills use a `references/` folder for detailed documentation loaded on demand.

```
skills/
└── <skill-name>/
    ├── SKILL.md          # Entry point: metadata + instructions
    └── references/       # Optional: detailed docs loaded on demand
```

## Credits

Created and maintained by **[Bhargav Tibadiya](https://github.com/bhargavtibadiya)**.

## License

MIT
