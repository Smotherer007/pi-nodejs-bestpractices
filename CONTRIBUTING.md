# Contributing

This is a **skills-only** Pi package (no runtime code). The package is markdown plus a
semantic-release setup.

```
skills/
└── nodejs-best-practices/
    ├── SKILL.md
    └── references/
        ├── evergreen-best-practices.md    # complete catalog (curated from nodebestpractices)
        ├── node-26-whats-new.md           # Node 26 features & breaking changes
        ├── security-deep-dive.md          # concrete security reference
        └── framework-and-testing.md       # framework + testing patterns
```

## How to contribute

1. Open an issue to discuss the change (or send a PR for small fixes).
2. Keep facts accurate — cite the Node.js release blog / API docs / changelog where possible.
3. Prefer erasable facts over opinion where it matters (stability levels, exact deprecation codes).
4. Keep `SKILL.md` concise (progressive disclosure); put detail in `references/`.
5. For best-practice content, attribute to `nodebestpractices` (MIT).

## Conventions

- **Conventional commits** (`feat:`/`fix:`) — releases are automated by semantic-release on push to `main`.
- `npm test` validates skill frontmatter (`node --test`).

## Verification

- `npm test` is green.
- `name` matches `^[a-z0-9-]{1,64}$`, `description` ≤ 1024 chars (Agent Skills spec).

## License

Contributions are licensed under [MIT](LICENSE). Best-practice content is derived from
`nodebestpractices` (MIT) — see the README attribution.
