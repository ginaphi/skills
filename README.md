# skills

Personal collection of agent skills. Install into your project's agent setup with a single command.

## Install

```bash
npx skills@latest add ginaphi/skills
```

The CLI ([vercel-labs/skills](https://github.com/vercel-labs/skills)) prompts you to pick which skills to install and which coding agent to wire them into. Project-scoped by default (`.claude/skills/`), or global with `-g`.

## Available skills

| Skill | What it does |
| --- | --- |
| **`genui`** | Workflow skill. Triggers on *"sketch a [thing]", "mock up [X]", "let me see a few options"*. Generates throwaway `.openui` files in `docs/prototype/` for the developer to render live in the Genui extension — available on the [VS Code Marketplace](https://github.com/ginaphi/genui) and [Open VSX Registry](https://open-vsx.org/extension/ginaphi/generative-ui). Bundles the OpenUI Lang authoring reference (positional-arg rules, ~50 verified component signatures, parse + schema validator) as an internal dependency. |

## What is OpenUI Lang?

A compact line-oriented DSL for generative UI. Your AI agent writes a `.openui` file; the official renderer turns it into a React-rendered preview. See [openuidev on GitHub](https://github.com/openuidev) for the language docs, the renderer, and the component library.

## License

MIT — see [LICENSE](LICENSE).
