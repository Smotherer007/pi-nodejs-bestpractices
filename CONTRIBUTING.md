# Contributing

This is a **skills-only** Pi package (no code, no build step). The whole package is markdown:

```
skills/
└── node-26-best-practices/
    ├── SKILL.md                                  # trigger + checklist + pitfalls
    └── references/
        ├── node-26-whats-new.md                  # Node 26 features & breaking changes
        └── evergreen-best-practices.md           # 2026 best-practice catalog
```

## How to contribute

1. Open an issue to discuss the change (or just send a PR for small fixes).
2. Keep the skill accurate — cite the Node.js release blog / API docs / changelog where possible.
3. Prefer erasable facts over opinion where it matters (e.g. stability levels, exact deprecation codes).
4. Keep `SKILL.md` concise (progressive disclosure); put detail in `references/`.

## Verification

- `name` matches `^[a-z0-9-]{1,64}$`, description ≤ 1024 chars (Pi validates skills against the Agent Skills spec).
- The `pi` manifest in `package.json` points at `./skills`.

## License

Contributions are licensed under [MIT](LICENSE).
