# ForgeQA for Claude Code

Adds the ForgeQA MCP server and the `forge-qa` skill, so Claude can write, run and fix browser tests for
your websites.

```
/plugin marketplace add https://app.forgeqa.io/claude/marketplace.json
/plugin install forge-qa@forge-qa
```

Then run `/mcp`, pick **forge-qa** and sign in. Your browser opens ForgeQA, where you choose the workspace,
the websites Claude may use, and whether it may create and run tests. Remove the connection any time in
ForgeQA under Integrations → AI assistants.

Other assistants (Claude.ai, ChatGPT, Codex, Cursor): see Integrations → AI assistants in ForgeQA.
