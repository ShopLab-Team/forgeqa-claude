# ForgeQA for Claude

The ForgeQA plugin: the ForgeQA MCP server and the `forge-qa` skill, so Claude can write, run and fix
real-browser tests for your websites on [ForgeQA](https://forgeqa.io).

## Claude chat (claude.ai and the desktop app)

1. Open **Customize → Plugins → Add → Add marketplace**.
2. Paste `https://github.com/ShopLab-Team/forgeqa-claude` and add it, then add the **ForgeQA** plugin.
3. Open the plugin's **Connectors** tab, choose **Connect** and sign in to ForgeQA.

## Claude Code

```
/plugin marketplace add https://app.forgeqa.io/claude/marketplace.json
/plugin install forge-qa@forge-qa
```

Then run `/mcp`, pick **forge-qa** and sign in.

When you sign in, ForgeQA asks which workspace, which websites and whether Claude may create and run tests.
Remove the connection any time in ForgeQA under **Integrations → AI assistants**.

This repository is generated from the ForgeQA app; changes made here are overwritten.
