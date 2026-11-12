# Notol MCP

An agent that pays a pasted or historical address cannot tell a lookalike from the real one (address poisoning). [`@notol/mcp`](https://www.npmjs.com/package/@notol/mcp) is a local MCP server that checks a payee before the agent pays USDC on Arc. The check is free and needs no key and no funds.

This repository holds the listing: this README, the Claude Code plugin and its marketplace file, and the same check-only server as an [Agent Plugins](https://agent-plugins.org) plugin at the root (`plugin.json`, `mcp.json`). The server ships on npm as a bundled build; its source is not in this repository. Docs: https://www.notol.app/docs#mcp · Agent-readable docs: https://www.notol.app/llms.txt

## Add the check (no key, no funds)

Any MCP client (Claude Desktop `claude_desktop_config.json`, a project `.mcp.json` for Claude Code, Cursor, …):

```json
{
  "mcpServers": {
    "notol": {
      "command": "npx",
      "args": ["-y", "@notol/mcp@0.1.3"],
      "env": {
        "NOTOL_API_URL": "https://api.notol.app",
        "NOTOL_SENDER": "0xYOUR_PAYING_WALLET_ADDRESS"
      }
    }
  }
}
```

`NOTOL_SENDER` is optional: the public address your agent pays from. The tool can also take `sender` per call. On Windows, if the client cannot start `npx`, use `npx.cmd`.

### Claude Code plugin

```
claude plugin marketplace add Notol-app/notol-mcp
claude plugin install notol@notol
```

The plugin starts the same server in check-only mode (`NOTOL_API_URL` only).

## What the check catches, and its limits

`notol_check_payee` answers `direct`, `escrow` or `blocked` for a payment from the sender to an address, with a one-line meaning. It flags:

- a **lookalike** of a payee the sender verified in Notol's on-chain registry (the address-poisoning pattern),
- the sender's **own** address,
- **reserved** or system addresses.

Limits:

- It is a hint. A Notol send re-checks on chain.
- It knows only payees verified in Notol's registry on Arc. It says nothing about payees verified elsewhere or on other chains.
- `escrow` / `new_payee` is not a verdict that an address is safe: it means Notol has no verified history for that pair.

`notol_transfer_status` reads the status of a Notol transfer. Both tools are read-only and move no money.

## Give your agent a wallet (optional)

```
npx @notol/mcp@0.1.3 init --owner 0xYOUR_WALLET
```

This creates the agent's key in a local file (`~/.notol/agent-5042.json`, readable only by you) and prints its address and a config snippet. Notol never sees the key. A new wallet holds 0: it can receive USDC right away and pay once funded. `notol_withdraw` sends funds back to the owner address only. The key file is the only copy, and any process running as you can read it: keep only what you can lose in it. Details: https://www.notol.app/docs#mcp

## Links

- Notol: https://www.notol.app
- MCP docs: https://www.notol.app/docs#mcp
- npm: https://www.npmjs.com/package/@notol/mcp
- Partner API (OpenAPI): https://www.notol.app/openapi.yaml

## Licence

MIT, see [LICENSE](LICENSE).
