# MTG Merchant Vault

MTG card merchant order/sales management system using an Obsidian vault as a flat-file database. Each markdown file is a record with YAML frontmatter for structured data.

## Vault Structure

```
Database/
├── 1.Unlisted Cards/    # Cards in inventory, not yet listed for sale
├── 2.Listed Cards/      # Cards actively listed on TCGPlayer
├── 3.Sold Cards/        # Cards that have been sold
├── Sale Records/        # Individual sale transaction records
└── .last-check          # ISO 8601 timestamp of last email processing run
```

Card lifecycle: **Unlisted → Listed → Sold**

- Unlisted → Listed: move to `2.Listed Cards/`, rename with listing-id, populate listing fields
- Listed → Sold: move to `3.Sold Cards/`, rename with sale-record-id, add `sale-record` wiki-link, create sale record in `Sale Records/`

## File Naming

All files use a single `.md` extension (never `.md.md`).

| Type | Pattern | Example |
|------|---------|---------|
| Unlisted card | `{card-name}-{sequential-id}.md` | `Badgermole Cub-1.md` |
| Listed card | `{card-name}-{listing-id}.md` | `Badgermole Cub-722158.md` |
| Sold card | `{card-name}-{sale-record-id}.md` | `Killer Duck-LZZC85F-9VD867-225BZ3.md` |
| Sale record | `{sale-record-id}.md` | `LZZC85F-9VD867-225BZ3.md` |

Multiple sold cards can share the same sale-record-id (multi-item orders).

## Record Schemas

### Card Record (all lifecycle stages)

```yaml
---
datetime: 2024-02-01T21:56:00Z
card-name: Badgermole Cub
card-set: Avatar: The Last Airbender
card-condition: Lightly Played
card-rarity: Rare
listing-id: 722158
listing-url: https://www.tcgplayer.com/product/722158/...
listing-price: $60.14
sale-record: [[LZZC85F-9VD867-225BZ3.md]]
---
Optional notes below frontmatter
```

- `card-condition` valid values: `Near Mint`, `Lightly Played`, `Moderately Played`, `Heavily Played`, `Damaged`
- `listing-id`, `listing-url`, `listing-price` are empty when unlisted
- `sale-record` is empty until sold; uses Obsidian wiki-link `[[filename]]` syntax

### Sale Record

```yaml
---
sale-datetime: 2024-02-03T01:23:00Z
sale-record-id: 5F9C0610-AA96AE-713C6
sale-record-url: https://store.tcgplayer.com/admin/orders/manageorder/5F9C0610-AA96AE-713C6
sale-price: $4.50
buyer-name: Ted Danson
buyer-address: 666 Overrate Drive, Hollywood Hills, California 45690
shipped-date:
---
```

- `shipped-date` is empty until the order ships

## Key Rules

- **Timestamps**: ISO 8601 format `YYYY-MM-DDTHH:MM:SSZ` (UTC)
- **Prices**: always include `$` prefix (e.g., `$60.14`)
- **Cross-references**: use Obsidian wiki-link syntax `[[filename]]`
- **Empty fields**: leave blank (no null or placeholder values)
- **Deduplication**: `Database/.last-check` stores the last processing timestamp. Dedup checks existing records by listing-id (in `2.Listed Cards/` and `3.Sold Cards/`) and sale-record-id (in `Sale Records/`), not email read status. Default to 30 days ago if `.last-check` doesn't exist.

## Merchant Info

```
return-address: |
  YOUR NAME
  YOUR STREET ADDRESS
  YOUR CITY, STATE ZIP
envelope-size: "#6 3/4 (3.625in x 6.5in)"
```

## Dependencies

- **Gmail MCP Server**: `gmail-mcp` configured in `.claude/mcp.json` with OAuth credentials at `~/.gmail-mcp/gcp-oauth.keys.json`.

## Available Tooling

| Tool | Type | Purpose |
|------|------|---------|
| `mtg-merchant` | Output style | Merchant response formatting |
| `tcgplayer-order-processor` | Agent | Process TCGPlayer emails into vault records |
| `/check-orders` | Skill | Trigger order processing (optional arg: days to search) |
