# Contributing

This is a **skills-only** Pi package. The skill is vendored **1:1** from
[mcollina/skills](https://github.com/mcollina/skills).

```
skills/
└── node/
    ├── SKILL.md          # Collina's `node` skill
    ├── LICENSE           # MIT (© Matteo Collina)
    ├── tile.json
    └── rules/
        ├── *.md          # 14 best-practice rules
        └── assets/       # graceful-server examples
```

## How to contribute

1. Open an issue first to discuss.
2. Upstream content lives in [mcollina/skills](https://github.com/mcollina/skills) — consider
   contributing there; this repo vendors it 1:1.
3. `npm test` validates skill frontmatter.

## Conventions

- Conventional commits (`feat:`/`fix:`) — semantic-release on push to `main`.

## License

[MIT](LICENSE). Skill content © Matteo Collina (MIT) — see `skills/node/LICENSE`.
