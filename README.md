# Skills

Agent skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and other tools supporting the [Agent Skills](https://agentskills.io) spec.

## Available Skills

| Skill | Description |
|-------|-------------|
| [algolia-search-optimizations](skills/algolia-search-optimizations) | Audit and optimize Algolia DocSearch setups: diagnose indexing gaps, fix crawler selectors, add synonyms, and verify improvements via analytics |
| [dokploy](skills/dokploy) | Manage Dokploy infrastructure (projects, applications, databases, domains, compose, deployments, servers) via REST API |
| [review-changes](skills/review-changes) | Review uncommitted changes or the current branch: picks scope, runs a fresh reviewer subagent, and validates its findings before reporting. Manual only: invoke with `/review-changes` |

## Install

```bash
# Install all skills
npx skills add gilbarbara/skills

# Install a specific skill
npx skills add gilbarbara/skills -s algolia-search-optimizations
npx skills add gilbarbara/skills -s dokploy
npx skills add gilbarbara/skills -s review-changes
```

## License

MIT
