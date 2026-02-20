# AI Editor Configs

Real, production-ready configuration files for AI-powered code editors. Copy these into your projects to get the best results from your AI coding tools.

## What's Included

| Editor | Config File | Description |
|--------|------------|-------------|
| [Cursor](cursor/) | `.cursorrules` | Project rules that guide Cursor's AI behavior |
| [Claude Code](claude-code/) | `.claude/CLAUDE.md` | Project instructions for Claude Code CLI |
| [GitHub Copilot](copilot/) | `.github/copilot-instructions.md` | Custom instructions for Copilot Chat |
| [Windsurf](windsurf/) | `.windsurfrules` | Project rules for Windsurf/Codeium IDE |

## Quick Start

```bash
# Clone this repo
git clone https://github.com/runaicode/ai-editor-configs.git

# Copy the config you need
cp ai-editor-configs/cursor/.cursorrules /path/to/your/project/
# or
cp -r ai-editor-configs/claude-code/.claude /path/to/your/project/
# or
cp ai-editor-configs/copilot/.github/copilot-instructions.md /path/to/your/project/.github/
# or
cp ai-editor-configs/windsurf/.windsurfrules /path/to/your/project/
```

## How to Customize

Each config file is designed as a **starting template**. You should customize it for your project:

1. **Technology stack** — Update languages, frameworks, and libraries
2. **Code style** — Match your team's conventions (naming, formatting, patterns)
3. **Architecture** — Describe your project structure and design patterns
4. **Constraints** — Add rules specific to your domain (security, compliance, performance)

## Tips for Effective AI Configuration

### Be Specific
```
Bad:  "Write clean code"
Good: "Use TypeScript strict mode. Prefer named exports. Functions should be < 30 lines."
```

### Provide Examples
```
Bad:  "Follow our naming convention"
Good: "Components: PascalCase (UserProfile.tsx). Hooks: camelCase with 'use' prefix (useAuth.ts)."
```

### Set Boundaries
```
Bad:  "Be careful with the database"
Good: "Never write raw SQL queries. Always use the ORM. Never use CASCADE DELETE."
```

### Include Context
```
Bad:  "This is a web app"
Good: "Next.js 14 app router, PostgreSQL via Prisma, deployed on Vercel. Auth via NextAuth."
```

## Contributing

Have a config that works well for your team? Submit a PR! Include:

- The config file in the appropriate editor directory
- A brief comment at the top explaining the project type it's designed for
- Any editor-specific notes about how the config is loaded

## License

MIT — use and adapt these configs freely.
