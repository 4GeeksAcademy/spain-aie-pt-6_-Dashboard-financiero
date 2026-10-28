# react-testing-best-practices

> Part of [`rutpshah/skills`](https://github.com/rutpshah/skills) — a collection of [skills.sh](https://skills.sh) skills for React development.

Teaches your AI agent React testing best practices using **React Testing Library**, **Vitest**, **Jest**, and **MSW**. No more hallucinated test patterns or outdated enzyme-style code.

## What It Covers

- ✅ RTL query priority (role → label → text → testId)
- ✅ Component, form, and async testing patterns
- ✅ Custom hook testing with `renderHook`
- ✅ Context & Redux provider test utilities
- ✅ API mocking with MSW (Mock Service Worker)
- ✅ Accessibility testing with `jest-axe`
- ✅ Vitest + Jest setup instructions
- ✅ Common mistakes to avoid
- ✅ File/folder organization conventions

Advanced patterns (in `references/testing-patterns.md`):
- Snapshot testing guidance
- React Router navigation testing
- TanStack Query / React Query testing
- Drag-and-drop testing
- Visual regression with Playwright
- CI/GitHub Actions setup

## Security

> ✅ **This skill contains documentation only.** No shell scripts, network requests, or file system operations are executed. It is safe to inspect — the entire skill is human-readable Markdown.

## Installation

```bash
# Via skills.sh CLI
npx skills add rutpshah/skills@react-testing-best-practices

# Or manually — copy this folder into your agent's skills directory
# Claude Code: ~/.claude/skills/
# Cursor:      .cursor/skills/
```

## Usage

Once installed, simply ask your agent:

- "Write tests for this component"
- "Add unit tests to this hook"
- "Mock this API call in my test"
- "Set up Vitest for my React project"
- "Improve test coverage for this form"

The skill will automatically guide the agent to follow RTL best practices.

## Structure

```
skills/                                          ← GitHub repo (rutpshah/skills)
└── react-testing-best-practices/
    ├── SKILL.md                    # Core skill instructions (loaded by agent)
    ├── references/
    │   └── testing-patterns.md    # Advanced patterns (loaded on demand)
    └── README.md                  # This file
```

## Contributing

1. Fork [`rutpshah/skills`](https://github.com/rutpshah/skills)
2. Edit `SKILL.md` or add files to `references/`
3. Keep `SKILL.md` under 500 lines — move verbose content to `references/`
4. Test your changes by installing locally and running against real components
5. Open a PR with a description of what pattern you added/fixed

### Ideas for contributions
- Storybook interaction testing patterns
- Zustand store testing
- React Server Component testing (Next.js App Router)
- Playwright component testing setup
- Testing with `react-hook-form` + Zod

## License

MIT
