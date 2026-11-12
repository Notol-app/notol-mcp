# Notol plugin for Claude Code

Adds the Notol MCP server in check-only mode: a free payee check an agent can run before it pays a USDC address on Arc. It flags a lookalike of a payee the sender verified in Notol's on-chain registry (address poisoning), the sender's own address, and reserved or system addresses.

## What it runs

`.mcp.json` starts one local stdio server: `npx -y @notol/mcp@0.1.3`, pinned to that exact version. The package ships an `npm-shrinkwrap.json`, so its whole dependency tree is pinned too.

It runs with one setting, `NOTOL_API_URL=https://api.notol.app`. There is no key, no credential and no wallet: in this mode the server registers only two read-only tools, `notol_check_payee` and `notol_transfer_status`, and cannot move money.

## What it sends and fetches

- **npm registry** (`registry.npmjs.org`): `npx` downloads `@notol/mcp@0.1.3` and its pinned dependencies the first time it starts.
- **Notol api** (`https://api.notol.app`): each `notol_check_payee` call sends the sender address and the recipient address (public chain addresses) plus `client=mcp`; each `notol_transfer_status` call sends a transfer id. The api keeps only a daily count of checks per client kind, never the addresses.

Nothing else is contacted. The server logs to stderr only, and never logs a key or a link.

## Limits

The check is a hint: a Notol send re-checks on chain. It knows only payees verified in Notol's registry on Arc, and says nothing about payees verified elsewhere or on other chains.

Docs: https://www.notol.app/docs#mcp · Licence: MIT
