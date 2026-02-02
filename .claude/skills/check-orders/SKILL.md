---
name: check-orders
description: Check Gmail for new TCGPlayer orders and listing notifications, then process them into the vault database.
disable-model-invocation: true
context: fork
agent: tcgplayer-order-processor
argument-hint: "[optional: number of days to override search window]"
---

Check my Gmail inbox for TCGPlayer notification emails since the last check date stored in `Database/.last-check`. If a number of days is provided as an argument, override the search window to the last $ARGUMENTS days instead.

Process all matching emails following the full workflow defined in the agent instructions.
