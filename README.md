# MTG Merchant

A Claude Code-powered order management system for TCGPlayer card merchants. Uses an Obsidian vault as a flat-file database, with Claude agents that automatically process sales and listing emails from Gmail.

## Commands

### `/check-orders`

Checks Gmail for new TCGPlayer notification emails and processes them into the vault database. Accepts an optional argument to override the search window:

```
/check-orders        # Search since last check (stored in Database/.last-check)
/check-orders 7      # Search the last 7 days
```

This runs the `tcgplayer-order-processor` agent, which:

1. Fetches TCGPlayer emails from Gmail (listings and sales)
2. Deduplicates against existing records
3. Creates/moves card files through their lifecycle stages
4. Creates sale records for new orders
5. Generates `envelope-labels.html` for unshipped orders (printable on #6 3/4 envelopes)
6. Writes a processing summary to `process-summary.txt`

You can also ask Claude directly to generate reports, add cards, search inventory, do bulk operations (mark shipped, update prices), or query by condition/set/date/status.

## Purpose

Manage a TCGPlayer card-selling operation end-to-end: inventory tracking, listing management, sale processing, shipping labels, and revenue reporting -- all through conversational commands in Claude Code, backed by an Obsidian vault you can browse and edit directly.

## Card Database & Lifecycle

The `Database/` directory is an Obsidian vault where each markdown file is a record with YAML frontmatter.

```
Database/
├── 1.Unlisted Cards/    # Inventory not yet listed
├── 2.Listed Cards/      # Active TCGPlayer listings
├── 3.Sold Cards/        # Completed sales
├── Sale Records/        # One per order (multi-card orders share a record)
└── .last-check          # Timestamp of last email processing run
```

### Lifecycle: Unlisted → Listed → Sold

| Stage | Directory | Filename | Trigger |
|-------|-----------|----------|---------|
| Unlisted | `1.Unlisted Cards/` | `{card-name}-{sequential-id}.md` | Manual entry |
| Listed | `2.Listed Cards/` | `{card-name}-{listing-id}.md` | TCGPlayer listing email |
| Sold | `3.Sold Cards/` | `{card-name}-{sale-record-id}.md` | TCGPlayer sale email |

Each card file carries frontmatter fields through its lifecycle (`card-name`, `card-set`, `card-condition`, `card-rarity`, `listing-id`, `listing-price`, `sale-record`). Sale records track buyer info, price, and shipping status.

### Deduplication

Records are deduplicated by `listing-id` and `sale-record-id` against existing files, not by email read status. The `.last-check` file controls the email search window.

## Claude Code Configuration

```
.claude/
├── mcp.json                              # Gmail MCP server (gmail-mcp with OAuth)
├── settings.local.json                   # Permissions for Gmail, search, scripting
├── output-styles/mtg-merchant.md         # Output style: direct, table-heavy, card-focused
├── agents/tcgplayer-order-processor.md   # Agent: email → vault record pipeline
└── skills/check-orders/SKILL.md          # Skill: /check-orders entrypoint
```

- **CLAUDE.md** defines the vault schema, file naming conventions, record formats, and rules that all tools follow.
- **Output style** (`mtg-merchant`) configures Claude's response format for merchant workflows -- concise, tabular, confirmation before destructive ops.
- **Agent** (`tcgplayer-order-processor`) is a Sonnet-based agent that handles the full email-to-database pipeline: fetch, classify, deduplicate, create/move files, generate envelopes, and summarize.
- **Skill** (`/check-orders`) is the user-facing entrypoint that launches the agent with the appropriate search window.
- **MCP server** (`gmail-mcp`) provides Gmail access for reading TCGPlayer notification emails.

## Setup

1. Install [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
2. Install `gmail-mcp` and configure OAuth credentials at `~/.gmail-mcp/gcp-oauth.keys.json`
3. Open this directory in Claude Code
4. Run `/check-orders` to process your first batch of emails
